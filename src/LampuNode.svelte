<script lang="ts">
    import {
      Handle,
      Position,
      useHandleConnections,
      useNodesData,
      useInternalNode,
      type NodeProps
    } from '@xyflow/svelte';
  
    type $$Props = NodeProps;
  
    export let id: $$Props['id'];
    export let data: $$Props['data'];
  
    const connections = useHandleConnections({
      nodeId: id,
      type: 'target'
    });
    $: isConnectable = $connections.length === 0;
  
    $: nodesData = useNodesData($connections.map((connection) => connection.source));

    // function tampil(){
    // for(const nodeData of $nodesData){
    //   console.log(nodeData.data.text);
    // }
    // }
    $$restProps;
  </script>
  
  <div class="custom">
    <Handle type="target" position={Position.Left} {isConnectable}/>
    <div class="label">Lampu {data.nama_lampu}: </div>
    <!-- <button on:click={tampil}></button> -->
    {#if $nodesData == undefined}
      <div>Off</div>
    {:else}
      <div>{$nodesData[0].data.text}</div>
    {/if}
  </div>
  
  <style>
    .custom {
      background-color: #eee;
      padding: 10px;
      border-radius: 10px;
      font-size: 12px;
    }
  
    .label {
      margin-bottom: 5px;
    }
  </style>
  