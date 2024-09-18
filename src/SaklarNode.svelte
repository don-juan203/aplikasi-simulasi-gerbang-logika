<script lang="ts">
  import { text } from '@sveltejs/kit';
import { Handle, Position, type NodeProps, useSvelteFlow } from '@xyflow/svelte';

  type $$Props = NodeProps;

  export let id: $$Props['id'];
  export let data: $$Props['data'];

  const { updateNodeData } = useSvelteFlow();
  data.text = "Off";
  $$restProps;
</script>

<div class="custom">
  <div class="label">Saklar {data.nama_saklar}: { data.text }</div>
  <div>
    {#if data.text === "Off"}
      <button on:click={() => updateNodeData(id, { text: "On"})}>Switch</button>
    {:else if data.text === "On"}
      <button on:click={() => updateNodeData(id, { text: "Off"})}>Switch</button>
    {/if}
  </div>
  <Handle type="source" position={Position.Right} />
</div>

<style>
  .custom {
    background-color: #eee;
    padding: 10px;
    border-radius: 10px;
    font-size: 11px;
  }

  .label {
    font-size: 12px;
    margin-bottom: 5px;
  }

  button {
    font-size: 11px;
  }
</style>