<script lang="ts">
    import { onMount } from 'svelte';
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
    import TextNode from './TextNode.svelte';
    import ContextMenu from './ContextMenu.svelte';

    // real
    import '@xyflow/svelte/dist/style.css';

    const nodeTypes: NodeTypes = {
        saklar_node: SaklarNode,
        lampu_node: LampuNode,
        and_node: AndNode,
        or_node: OrNode,
        not_node: NotNode,
        nand_node: NandNode,
        nor_node: NorNode,
        xor_node: XorNode,
        xnor_node: XnorNode,
        teks_node: TextNode
    };

    const nodes = writable<Node[]>([]); 
    const edges = writable<Edge[]>([]);

    // drag and drop
    const { screenToFlowPosition } = useSvelteFlow();
    const onDragOver = (event: DragEvent) => {
        event.preventDefault();

        if (event.dataTransfer) {
            event.dataTransfer.dropEffect = 'move';
        }
    };

    let saklarCounter = 1;
    let lampuCounter = 1; 

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
                ...(type === 'saklar_node' && { nama_saklar: saklarCounter++ }),
                ...(type === 'lampu_node' && { nama_lampu: lampuCounter++ })
            },
            origin: [0.5, 0.0]
        } satisfies Node;

        $nodes.push(newNode);
        $nodes = $nodes;
    };

    let menu: { id: string; top?: number; left?: number; right?: number; bottom?: number } | null;
    let container;

    onMount(() => {
        container = document.querySelector('.svelte-flow-container');
    });

    function handleContextMenu({ detail: { event, node } }) {
        // Prevent native context menu from showing
        event.preventDefault();

        const rect = container.getBoundingClientRect();

        // Calculate position of the context menu. We want to make sure it
        // doesn't get positioned off-screen.
        const menuHeight = 200; // approximate height of your context menu
        const menuWidth = 200;  // approximate width of your context menu

        menu = {
            id: node.id,
            top: event.clientY < (rect.height - menuHeight) ? event.clientY - rect.top : undefined,
            left: event.clientX < (rect.width - menuWidth) ? event.clientX - rect.left : undefined,
            right: event.clientX >= (rect.width - menuWidth) ? rect.width - (event.clientX - rect.left) : undefined,
            bottom: event.clientY >= (rect.height - menuHeight) ? rect.height - (event.clientY - rect.top) : undefined
        };
    }

    // Close the context menu if it's open whenever the window is clicked.
    function handlePaneClick() {
        menu = null;
    }
</script>

<div class="svelte-flow-container" style="height: 98vh;">
    <SvelteFlow {nodeTypes} {nodes} {edges} fitView on:dragover={onDragOver} on:drop={onDrop}
        on:nodecontextmenu={handleContextMenu}
        on:paneclick={handlePaneClick}>
        <Background variant={BackgroundVariant.Lines} />
        <Background patternColor="#aaa" gap={16} />
        <Controls position="bottom-center" orientation="horizontal" />
        <MiniMap zoomable pannable height={120} />
        <DownloadButton />
        {#if menu}
            <ContextMenu
                onClick={handlePaneClick}
                id={menu.id}
                top={menu.top}
                left={menu.left}
                right={menu.right}
                bottom={menu.bottom}
            />
        {/if}
    </SvelteFlow>
</div>
