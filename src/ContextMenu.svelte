<script lang="ts">
    import { useEdges, useNodes } from '@xyflow/svelte';
  
    export let onClick: () => void;
    export let id: string;
    export let top: number | undefined;
    export let left: number | undefined;
    export let right: number | undefined;
    export let bottom: number | undefined;
  
    const nodes = useNodes();
    const edges = useEdges();
  
    function deleteNode() {
      $nodes = $nodes.filter((node) => node.id !== id);
      $edges = $edges.filter((edge) => edge.source !== id && edge.target !== id);
    }
</script>
  
<div
    style="top: {top}px; left: {left}px; right: {right}px; bottom: {bottom}px;"
    class="context-menu"
    on:click={onClick}
>
    <button on:click={deleteNode}>Delete</button>
</div>
  
<style>
    .context-menu {
        background: #ffffff;
        border: 1px solid #cccccc;
        border-radius: 8px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        position: absolute;
        z-index: 10;
        width: 150px;
        overflow: hidden;
        padding: 5px;
    }
  
    .context-menu button {
        background: #eeeeee;
        border: none;
        display: block;
        padding: 10px;
        text-align: left;
        width: 100%;
        cursor: pointer;
        font-size: 14px;
        color: #333333;
        transition: background 0.3s;
    }
  
    .context-menu button:hover {
        background: #cccccc;
    }
  
    .context-menu button:focus {
        outline: none;
        background: #bbbbbb;
    }
</style>
