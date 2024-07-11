<script lang="ts">
  import { slide } from 'svelte/transition';
  import DragArea from "../drag-area.svelte";
  import { SvelteFlowProvider } from '@xyflow/svelte';

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
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'and_node')} draggable={true}>AND</div>
        </li>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'or_node')} draggable={true}>OR</div>
        </li>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'not_node')} draggable={true}>NOT</div>
        </li>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'nand_node')} draggable={true}>NAND</div>
        </li>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'nor_node')} draggable={true}>NOR</div>
        </li>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'xor_node')} draggable={true}>XOR</div>
        </li>
        <li>
          <div class="input-node node" on:dragstart={(event) => onDragStart(event, 'xnor_node')} draggable={true}>XNOR</div>
        </li>
      </ul>
      {/if}
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
    /* border-right: 1px solid #bdc3c7; */
    /* box-shadow: 2px 0 5px rgba(0, 0, 0, 0.1); */
  }
  .empty {
    height: 5vh;
    line-height: 5vh;
    text-align: center;
    font-size: large;
    font-weight: bold;
    /* border-bottom: 1px solid black; */
  }
  .real-menu {
    padding: 1rem;
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
  .node:hover {
    background-color: #3b5998;
  }
</style>
