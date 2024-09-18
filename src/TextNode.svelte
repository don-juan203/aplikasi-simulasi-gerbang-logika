<script lang="ts">
    import {
      Handle,
      Position,
      useSvelteFlow,
      NodeResizer,
      type NodeProps
    } from '@xyflow/svelte';
    
  
    type $$Props = NodeProps;
    const { updateNodeData } = useSvelteFlow();
  
    export let id: $$Props['id'];
    export let data: $$Props['data'];
    export let selected: $$Props['selected'] = undefined;
    const defaultText = "Masukkan Teks..";
    data.text = data.text || defaultText;
  
    let isEditing = false;
  
    function handleTextClick() {
      isEditing = true;
    }
  
    function handleInputBlur() {
      isEditing = false;
      if (!data.text.trim()) {
        data.text = defaultText;
      }
      updateNodeData(id, { text: data.text });
    }
  </script>
  
  <div class="custom">
    <NodeResizer minWidth={100} minHeight={30} isVisible={selected} />
    {#if isEditing}
      <input
        bind:value={data.text}
        on:blur={handleInputBlur}
        on:keydown={(e) => e.key === 'Enter' && handleInputBlur()}
        autofocus
      />
    {:else}
      <span on:click={handleTextClick}>{data.text}</span>
    {/if}
  </div>
  
  <style>
    .custom {
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 10px;
      border-radius: 10px;
      font-size: 12px;
      width: fit-content;
      height: fit-content;
      cursor: pointer;
    }
  
    input {
      font-size: 12px;
      padding: 5px;
      border: none;
      border-bottom: 1px solid black;
    }
  
    span {
        /* border: 1px solid black; */
        padding: 5px;
      font-size: 12px;
    }
  </style>
  