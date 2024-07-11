<script lang="ts">
    import { writable } from 'svelte/store';
    import DownloadButton from './DownloadButton.svelte';
    import { 
        SvelteFlow, 
        SvelteFlowProvider,
        Background, 
        BackgroundVariant, 
        Controls, 
        MiniMap, 
        useSvelteFlow, 
        type Edge, 
        type Node, 
        type NodeTypes 
    } from '@xyflow/svelte';

    // test
    import LampuNode from './LampuNode.svelte';
    import SaklarNode from './SaklarNode.svelte';
    import AndNode from './AndNode.svelte';
    import OrNode from './OrNode.svelte';
    import NotNode from './NotNode.svelte';
    import NandNode from './NandNode.svelte';
    import NorNode from './NorNode.svelte';
    import XorNode from './XorNode.svelte';
    import XnorNode from './XnorNode.svelte';

    // real
    import '@xyflow/svelte/dist/style.css';

    const nodeTypes: NodeTypes = {
        // nama node: nama impor
        saklar_node: SaklarNode,
        lampu_node: LampuNode,
        and_node: AndNode,
        or_node: OrNode,
        not_node: NotNode,
        nand_node: NandNode,
        nor_node: NorNode,
        xor_node: XorNode,
        xnor_node: XnorNode
    };

    
    const nodes = writable<Node[]>([
       {
        id: '1',
        type: 'lampu_node',
        data: {},
        position: { x: -100, y: -50 }
       },
       {
        id: '2',
        type: 'saklar_node',
        data: {text: "Off"},
        position: { x: -400, y: -50 }
       },
    ]);
    
    const edges = writable<Edge[]>([]);

    // drag and drop
    const { screenToFlowPosition } = useSvelteFlow();
    const onDragOver = (event: DragEvent) => {
        event.preventDefault();

        if (event.dataTransfer) {
            event.dataTransfer.dropEffect = 'move';
        }
    };

    let saklarCounter = 0;
    let lampuCounter = 0; 

    const onDrop = (event: DragEvent) => {
        event.preventDefault();

        if (!event.dataTransfer) {
            return null;
        }

        const type = event.dataTransfer.getData('application/svelteflow');
        const position = screenToFlowPosition({
            x: event.clientX,
            y: event.clientY
        });

        const newNode = {
            id: `${Math.random()}`,
            type,
            position,
            data: {
                label: `${type} node`,
                ...(type === 'saklar_node' && { nama_saklar: String.fromCharCode(65 + saklarCounter++) }),
                ...(type === 'lampu_node' && { nama_lampu: `Lampu_${++lampuCounter}` })
            },
            origin: [0.5, 0.0]
        } satisfies Node;

        $nodes.push(newNode);
        $nodes = $nodes;
    };

</script>

<div style="height: 98vh;">
    <SvelteFlow {nodeTypes} {nodes} {edges} fitView on:dragover={onDragOver} on:drop={onDrop}>
        <Background variant={BackgroundVariant.Lines} />
        <Background patternColor="#aaa" gap={16} />
        <Controls position="bottom-center" orientation="horizontal" />
        <MiniMap zoomable pannable height={120} />
        <DownloadButton />
    </SvelteFlow>
</div>
