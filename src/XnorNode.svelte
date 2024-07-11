<script lang="ts">
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
  
    $: isConnectable = $connections.length <= 3;
    $: nodesData = useNodesData($connections.map((connection) => connection.source));
  
    function AndProcess(value: any){
    if(value != undefined){
      // hitung jumlah input "on"
      let count = 0;
      for (const nodeData of $nodesData){
        if(nodeData.data.text === "On"){
          count++;
        }
      }
      if(count === 1 || count === 3){
        updateNodeData(id, { text: "Off" });
      }else{
        updateNodeData(id, { text: "On"})
      }
    }else{
      updateNodeData(id, { text: "Off"})
    }
  }
  
    $: AndUpdate = AndProcess($nodesData);
    $$restProps;
  </script>
  
  <div class="custom">
    <div class="label" style="margin-top: auto; margin-bottom: auto;">XNOR</div>
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
      background-color: #eee;
      padding: 10px;
      border-radius: 10px;
      font-size: 12px;
    }
  
    .label {
      margin-bottom: 5px;
    }
  </style>
  