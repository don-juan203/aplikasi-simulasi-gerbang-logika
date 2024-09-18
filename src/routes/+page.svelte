<script lang="ts">
  import { slide } from 'svelte/transition';
  import DragArea from "../drag-area.svelte";
  import { SvelteFlowProvider } from '@xyflow/svelte';

  import and from '$lib/assets/and_inv.png';
  import or from '$lib/assets/or_inv.png';
  import not from '$lib/assets/not_inv.png';
  import nand from '$lib/assets/nand_inv.png';
  import nor from '$lib/assets/nor_inv.png';
  import xor from '$lib/assets/xor_inv.png';
  import xnor from '$lib/assets/xnor_inv.png';

  // drag and drop
  const onDragStart = (event: DragEvent, nodeType: string) => {
    if (!event.dataTransfer) {
      return null;
    }

    event.dataTransfer.setData('application/svelteflow', nodeType);
    event.dataTransfer.effectAllowed = 'move';
  };

  let Input_isExpanded = true;
  let Output_isExpanded = true;
  let Gerbang_isExpanded = true;
  let Teks_isExpanded = true;
  let Ic_isExpanded = true;

  function InputMenuclickHandler() {
    Input_isExpanded = !Input_isExpanded;
  }
  function OutputMenuclickHandler() {
    Output_isExpanded = !Output_isExpanded;
  }
  function GerbangMenuclickHandler() {
    Gerbang_isExpanded = !Gerbang_isExpanded;
  }
  function TeksMenuclickHandler() {
    Teks_isExpanded = !Teks_isExpanded;
  }
  function IcMenuclickHandler() {
    Ic_isExpanded = !Ic_isExpanded;
  }
</script>

<main>
  <div class="menu">
    <div class="empty" style="">Simulator Gerbang Logika</div>
    <div class="real-menu">
      <button class="menu-section-button" on:click={InputMenuclickHandler}>Input</button>
      {#if Input_isExpanded}
      <ul transition:slide>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'saklar_node')} draggable={true}>Saklar</div>
        </li>
      </ul>
      {/if}
      <button class="menu-section-button" on:click={OutputMenuclickHandler}>Output</button>
      {#if Output_isExpanded}
      <ul transition:slide>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'lampu_node')} draggable={true}>Lampu</div>
        </li>
      </ul>
      {/if}
      <button class="menu-section-button" on:click={GerbangMenuclickHandler}>Gerbang</button>
      {#if Gerbang_isExpanded}
      <ul transition:slide>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'and_node')} draggable={true}>
            <img src="{and}" alt="and">
          </div>
        </li>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'or_node')} draggable={true}>
            <img src="{or}" alt="or">
          </div>
        </li>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'not_node')} draggable={true}>
            <img src="{not}" alt="not">
          </div>
        </li>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'nand_node')} draggable={true}>
            <img src="{nand}" alt="nand">
          </div>
        </li>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'nor_node')} draggable={true}>
            <img src="{nor}" alt="nor">
          </div>
        </li>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'xor_node')} draggable={true}>
            <img src="{xor}" alt="xor">
          </div>
        </li>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'xnor_node')} draggable={true}>
            <img src="{xnor}" alt="xnor">
          </div>
        </li>
      </ul>
      {/if}
      <button class="menu-section-button" on:click={TeksMenuclickHandler}>Teks</button>
      {#if Teks_isExpanded}
      <ul transition:slide>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'teks_node')} draggable={true}>Teks</div>
        </li>
      </ul>
      {/if}
      <div class="empty" style=""></div>
    </div>

  </div>
  <SvelteFlowProvider>
    <DragArea />
  </SvelteFlowProvider>
</main>

<style>
  main {
    font-family: Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
    line-height: 1.5;
    font-weight: 400;
    margin-left: 15vw;
  }
  img {
    margin: auto;
    height: 3vw;
  }
  ul {
    list-style-type: none;
    padding: 0;
  }
  .menu {
    height: 100vh;
    width: 15vw;
    position: fixed;
    top: 0;
    left: 0;
    z-index: 9;
    background-color: #2c3e50;
    color: #ecf0f1;
  }
  .empty {
    height: 5vh;
    line-height: 5vh;
    text-align: center;
    font-size: large;
    font-weight: bold;
  }
  .real-menu {
    padding: 1rem;
    height: 100vh;
    overflow: scroll;
  }
  .menu-section-button {
    width: 100%;
    padding: 0.5rem 1rem;
    font-size: medium;
    font-weight: bold;
    text-align: left;
    color: #ecf0f1;
    background: none;
    border: none;
    border-bottom: 1px solid #bdc3c7;
    cursor: pointer;
    transition: background-color 0.3s;
  }
  .menu-section-button:hover {
    background-color: #34495e;
  }
  .node {
    padding: 0.5rem 1rem;
    cursor: grab;
    background-color: #34495e;
    margin-bottom: 0.5rem;
    border-radius: 4px;
    transition: background-color 0.3s;
  }

  .input-node {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .node:hover {
    background-color: #3b5998;
  }
</style>
