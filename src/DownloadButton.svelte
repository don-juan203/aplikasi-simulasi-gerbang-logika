<script lang="ts">
  import { toPng } from 'html-to-image';
  import Carousel from 'svelte-carousel';
  import img1 from '$lib/assets/1.gif';
  import img2 from '$lib/assets/2.gif';
  import img3 from '$lib/assets/3.gif';
  import img4 from '$lib/assets/4.gif';
  import img5 from '$lib/assets/5.gif';
  import img6 from '$lib/assets/6.gif';
  import img7 from '$lib/assets/7.gif'
  import { writable } from 'svelte/store';
  import { Panel, getNodesBounds, getViewportForBounds, useEdges, useNodes, useConnection, useSvelteFlow } from '@xyflow/svelte';

  interface NodeData {
    id: string;
    type: string;
    data: {
      text?: string;
    };
  }

  const nodes = useNodes();
  const edges = useEdges();
  const connection = useConnection();
  const { toObject } = useSvelteFlow();

  const imageWidth = 1024;
  const imageHeight = 768;

  function downloadImage() {
    const nodesBounds = getNodesBounds($nodes);
    const viewport = getViewportForBounds(nodesBounds, imageWidth, imageHeight, 0.5, 2.0, 0.2);
    const viewportDomNode = document.querySelector<HTMLElement>('.svelte-flow__viewport')!;

    if (viewport) {
      toPng(viewportDomNode, {
        backgroundColor: '#ffffff',
        width: imageWidth,
        height: imageHeight,
        style: {
          width: `${imageWidth}px`,
          height: `${imageHeight}px`,
          transform: `translate(${viewport.x}px, ${viewport.y}px) scale(${viewport.zoom})`
        }
      }).then((dataUrl) => {
        const link = document.createElement('a');
        link.download = 'svelte-flow.png';
        link.href = dataUrl;
        link.click();
      });
    }
  }

  let showModal = writable(false);
  let filename = writable('');
  let showTruthTableModal = writable(false);
  let truthTable = writable([]);
  let showHelpModal = writable(false);


  function download_file(name: string) {
    let projectData = toObject();
    const jsonString = JSON.stringify(projectData);
    const blob = new Blob([jsonString], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = `${name}.lgproj`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
    console.log(projectData.nodes);
  }

  function openModal() {
    showModal.set(true);
  }

  function closeModal() {
    showModal.set(false);
  }

  function handleSave() {
    download_file($filename);
    closeModal();
  }

  function uploadFile(event: Event) {
    const input = event.target as HTMLInputElement;
    const file = input.files?.[0];

    if (file) {
      const reader = new FileReader();
      reader.onload = (e: ProgressEvent<FileReader>) => {
        const fileContents = e.target?.result as string;
        try {
          const parsedData: any = JSON.parse(fileContents);
          const { nodes: parsedNodes, edges: parsedEdges } = parsedData;
          nodes.set(parsedNodes || []);
          edges.set(parsedEdges || []);
        } catch (error) {
          console.error('Error parsing JSON:', error);
        }
      };

      reader.readAsText(file);
    }
  }

  let showConfirmModal = false;

  function confirmNewFile() {
    showConfirmModal = true;
  }

  function createNewFile() {
    nodes.set([]);
    edges.set([]);
    showConfirmModal = false;
  }

  function closeConfirmModal() {
    showConfirmModal = false;
  }

  function toggleTruthTableModal() {
    showTruthTableModal.update(n => !n);
  }

  function generateTruthTable() {
    const jsonData = toObject();
    const nodes = jsonData.nodes;
    const edges = jsonData.edges;

    function getNodeInputs(nodeId: string): NodeData[] {
      return edges
        .filter(edge => edge.target === nodeId)
        .map(edge => nodes.find(node => node.id === edge.source)!);
    }

    function evaluateNode(node: NodeData): number {
      if (node.type === 'saklar_node') {
        return node.data.text === 'On' ? 1 : 0;
      } else if (node.type === 'and_node') {
        const inputs = getNodeInputs(node.id).map(inputNode => evaluateNode(inputNode));
        return inputs.every(input => input === 1) ? 1 : 0;
      } else if (node.type === 'or_node') {
        const inputs = getNodeInputs(node.id).map(inputNode => evaluateNode(inputNode));
        return inputs.some(input => input === 1) ? 1 : 0;
      } else if (node.type === 'not_node') {
        const inputs = getNodeInputs(node.id).map(inputNode => evaluateNode(inputNode));
        return inputs[0] === 1 ? 0 : 1;
      } else if (node.type === 'nand_node') {
        const inputs = getNodeInputs(node.id).map(inputNode => evaluateNode(inputNode));
        return inputs.every(input => input === 1) ? 0 : 1;
      } else if (node.type === 'nor_node') {
        const inputs = getNodeInputs(node.id).map(inputNode => evaluateNode(inputNode));
        return inputs.some(input => input === 1) ? 0 : 1;
      } else if (node.type === 'xor_node') {
        const inputs = getNodeInputs(node.id).map(inputNode => evaluateNode(inputNode));
        return inputs.reduce((acc, input) => acc ^ input, 0);
      } else if (node.type === 'xnor_node') {
        const inputs = getNodeInputs(node.id).map(inputNode => evaluateNode(inputNode));
        return inputs.reduce((acc, input) => acc ^ input, 0) === 1 ? 0 : 1;
      } else if (node.type === 'lampu_node') {
        const inputs = getNodeInputs(node.id).map(inputNode => evaluateNode(inputNode));
        return inputs[0];
      }
      return 0;
    }

    const truthTable = [];
    const inputNodes = nodes.filter(node => node.type === 'saklar_node');
    const outputNodes = nodes.filter(node => node.type === 'lampu_node');

    const numInputs = inputNodes.length;
    const numCombinations = 1 << numInputs;

    for (let i = 0; i < numCombinations; i++) {
      const inputs = [];
      for (let j = 0; j < numInputs; j++) {
        inputs.push((i >> j) & 1);
      }

      inputNodes.forEach((node, index) => {
        node.data.text = inputs[index] ? 'On' : 'Off';
      });

      const outputs = outputNodes.map(lampNode => evaluateNode(lampNode));

      truthTable.push({
        inputs,
        outputs
      });
    }

    return truthTable;
  }

  function showTruthTable() {
    const table = generateTruthTable();
    truthTable.set(table);
    toggleTruthTableModal();
  }

  function downloadTruthTableImage() {
    const truthTableElement = document.querySelector('.truth-table');
    if (truthTableElement) {
      toPng(truthTableElement, { backgroundColor: '#ffffff' }).then((dataUrl) => {
        const link = document.createElement('a');
        link.download = 'truth-table.png';
        link.href = dataUrl;
        link.click();
      });
    }
  }

  function openHelpModal() {
    showHelpModal.set(true);
  }

  function closeHelpModal() {
    showHelpModal.set(false);
  }
</script>

<!-- Panel section -->
<Panel position="top-right">
  <div class="navbar">
    <div class="real-navbar">
      <ul class="navbar">
        <div class="empty-space">Judul File</div>
        <li><button class="nav-button" on:click={confirmNewFile}>Baru</button></li>
        <li>
          <button class="nav-button" on:click={() => document.getElementById('fileInput')?.click()}>Pilih</button>
          <input type="file" id="fileInput" accept=".lgproj" on:change={uploadFile} style="display: none;" />
        </li>
        <li><button class="nav-button" on:click={openModal}>Simpan</button></li>
        <li><button class="nav-button" on:click={showTruthTable}>Tabel Kebenaran</button></li>
        <li><button class="nav-button" on:click={downloadImage}>Cetak Gambar</button></li>
        <li><button class="nav-button" on:click={openHelpModal}>Bantuan</button></li>
      </ul>
    </div>
  </div>
</Panel>

<!-- Modals -->
{#if $showModal}
  <div class="modal">
    <div class="modal-content small-modal">
      <span class="close" on:click={closeModal}>&times;</span>
      <h2>Simpan File</h2>
      <input type="text" bind:value={$filename} placeholder="Masukkan nama file" />
      <button class="modal-button" on:click={handleSave}>Simpan</button>
    </div>
  </div>
{/if}

<div class="modal" style:display={showConfirmModal ? 'flex' : 'none'}>
  <div class="modal-content small-modal">
    <p>Apakah Anda yakin ingin membuat file baru?</p>
    <div class="button-group">
      <button class="modal-button" on:click={createNewFile}>Ya</button>
      <button class="modal-button" on:click={closeConfirmModal}>Tidak</button>
    </div>
  </div>
</div>

{#if $showTruthTableModal}
  <div class="modal">
    <div class="modal-content large-modal">
      <span class="close" on:click={toggleTruthTableModal}>&times;</span>
      <h2>Tabel Kebenaran</h2>
      <table class="truth-table">
        <thead>
          <tr>
            {#each Array($truthTable[0]?.inputs.length || 0) as _, i}
              <th>Saklar {i + 1}</th>
            {/each}
            {#each Array($truthTable[0]?.outputs.length || 0) as _, i}
              <th>Lampu {i + 1}</th>
            {/each}
          </tr>
        </thead>
        <tbody>
          {#each $truthTable as row}
            <tr>
              {#each row.inputs as input}
                <td>{input}</td>
              {/each}
              {#each row.outputs as output}
                <td>{output}</td>
              {/each}
            </tr>
          {/each}
        </tbody>
      </table>
      <button class="modal-button" on:click={downloadTruthTableImage}>Unduh sebagai Gambar</button>
    </div>
  </div>
{/if}

{#if $showHelpModal}
  <div class="modal">
    <div class="modal-content extra-large-modal">
      <span class="close" on:click={closeHelpModal}>&times;</span>
      <h2>Bantuan</h2>
      <Carousel
		>
    <div class="carousel-content">
      <img src={img1} alt="1">
      <p>Tekan dan tahan komponen untuk menambahkannya ke dalam rangkaian. Tekan komponen yang sudah ada dan tekan "Delete" pada keyboard untuk menghapus</p>
    </div>
    <div class="carousel-content">
      <img src={img2} alt="2">
      <p>Tekan dan tahan handle pada komponen untuk menyambungkannya dengan komponen lain. Tekan sambungan yang sudah ada dan tekan "Delete" pada keyboard untuk menghapus</p>
    </div>
    <div class="carousel-content">
      <img src={img3} alt="3">
      <p>Tekan tombol "Cetak Gambar" untuk mencetak rangkaian sebagai gambar</p>
    </div>
    <div class="carousel-content">
      <img src={img4} alt="4">
      <p>Tekan tombol "Tabel Kebenaran" untuk menampilkan tabel kebenaran dari rangkaian yang ada. Tabel dapat disimpan sebagai gambar dengan menekan "Simpan sebagai Gambar"</p>
    </div>
    <div class="carousel-content">
      <img src={img5} alt="5">
      <p>Simpan proyek dalam bentuk file dengan menekan tombol "Simpan". File akan disimpan dengan format .lgproj</p>
    </div>
    <div class="carousel-content">
      <img src={img6} alt="6">
      <p>Pilih proyek yang sudah ada dengan menekan tombol "Pilih".</p>
    </div>
    <div class="carousel-content">
      <img src={img7} alt="7">
      <p>Mulai proyek baru dengan menekan tombol "Baru"</p>
    </div>
    </Carousel>
      <button class="modal-button" on:click={closeHelpModal}>Tutup</button>
    </div>
  </div>
{/if}

<!-- Style section -->
<style>
  .navbar {
    height: 5vh;
    width: 100vw;
    position: fixed;
    top: 0;
    left: 0;
    z-index: 10;
    background-color: #2c3e50;
    border-bottom: 1px solid #bdc3c7;
    display: flex;
    align-items: center;
    padding: 0 1rem;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    font-family: 'Inter', system-ui, Avenir, Helvetica, Arial, sans-serif;
  }

  .empty-space {
    height: 100%;
    width: 15vw;
    line-height: 5vh;
    font-weight: bold;
    font-size: large;
    color: #ecf0f1;
    display: flex;
    align-items: center;
  }

  .real-navbar {
    height: 100%;
  }

  .real-navbar .navbar {
    list-style-type: none;
    margin: 0;
    padding: 0;
    display: flex;
    align-items: center;
    gap: 1rem;
  }

  .real-navbar .navbar li {
    display: flex;
    align-items: center;
  }

  .real-navbar .navbar li button {
    color: #ecf0f1;
    font-size: medium;
    font-weight: bold;
    text-align: center;
    text-decoration: none;
    background: none;
    border: none;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    transition: background-color 0.3s;
  }

  .real-navbar .navbar li button:hover {
    background-color: #34495e;
  }

  .panel-style {
    display: flex;
    gap: 10px;
    background-color: #eeeeee;
    padding: 10px;
    border-radius: 5px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  }

  .panel-style button {
    background-color: #1a365d;
    color: white;
    border: none;
    border-radius: 5px;
    padding: 10px 20px;
    cursor: pointer;
  }

  .modal {
    display: flex;
    position: fixed;
    z-index: 1000;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: auto;
    background-color: rgba(0, 0, 0, 0.4);
    justify-content: center;
    align-items: center;
  }

  .modal-content {
    background-color: #fefefe;
    padding: 20px;
    border: 1px solid #888;
    border-radius: 10px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
    max-width: 100%;
  }

  .carousel-content {
    text-align: center;
  }

  .carousel-content img {
    width: 100%;
  }

  .small-modal {
    width: 300px;
  }

  .large-modal {
    width: 600px;
  }

  .extra-large-modal {
    width: 800px;
  }

  .close {
    color: #aaa;
    float: right;
    font-size: 28px;
    font-weight: bold;
  }

  .close:hover,
  .close:focus {
    color: black;
    text-decoration: none;
    cursor: pointer;
  }

  .modal-button {
    background-color: #1a365d;
    color: #ecf0f1;
    border: none;
    border-radius: 5px;
    padding: 10px 20px;
    font-size: medium;
    font-weight: bold;
    cursor: pointer;
    transition: background-color 0.3s;
    margin-top: 10px;
  }

  .modal-button:hover {
    background-color: #34495e;
  }

  .button-group {
    display: flex;
    justify-content: space-between;
  }

  .truth-table {
    width: 100%;
    border-collapse: collapse;
    background-color: #ffffff; /* Set background color to white */
  }

  th,
  td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: center;
  }

  th {
    background-color: #f2f2f2;
  }
</style>
