<script lang="ts">
  import img from '$lib/assets/and.png';
  import {
    Handle,
    Position,
    useHandleConnections,
    useNodesData,
    useSvelteFlow,
    type NodeProps
  } from '@xyflow/svelte';

  type $$Props = NodeProps;

  export let isConnectable: $$Props['isConnectable'];
  const { updateNodeData } = useSvelteFlow();

  // export id(?)
  export let id: $$Props['id'];

  // style handle
  const DEFAULT_HANDLE_STYLE = 'width: 5px; height: 5px; bottom: -5px;';

  // koneksi menyimpan id ketika koneksi dengan handle berubah -> intinya simpen id node dan id edge  
  const connections = useHandleConnections({
    nodeId: id,
    type: 'target'
  });

  // limit koneksi hanlde max 1
  $: isConnectable = $connections.length <= 3;

  // menerima data dari id node yang disebutkan (yg disimpan dalam connections) -> simpen data (kayak pas buat node manual)
  // $nodesData[0]?.data -> data yang ada pada array ke-0 (setiap node yang tersambung disimpan dalam array)
  $: nodesData = useNodesData($connections.map((connection) => connection.source));

  function AndProcess(value: any){
    if(value != undefined){
      for(const nodeData of value){
        if(nodeData.data.text === "Off"){
          updateNodeData(id, { text: "Off"})
          break;
        }else{
          updateNodeData(id, { text: "On"})
        }
      }
    }else{
      updateNodeData(id, { text: "Off"})
    }
  }

  $: AndUpdate = AndProcess($nodesData);
  $$restProps;
</script>

<div class="custom">
  <!-- <div class="label" style="margin-top: auto; margin-bottom: auto;">AND</div> -->
  <img class="node-img" src="{img}" alt="AND">
  <Handle
    type="target"
    position={Position.Left}
    style="{DEFAULT_HANDLE_STYLE}; left: 0%;"
    {isConnectable}
  />
  <Handle
    type="source"
    position={Position.Right}
    style="{DEFAULT_HANDLE_STYLE}; right: 0%;"
  />
</div>

<style>
  .custom {
    display: flex;
    align-items: center;
    justify-content: center;
    /* background-color: #eee; */
    padding: 10px;
    border-radius: 10px;
    font-size: 12px;
    width: 40px;
    height: 20px;
  }

  .node-img {
    width: 100%;
    max-width: 40px;
    height: auto;
    display: block;
    /* border: 2px solid black; */
  }
</style>
