<script lang="ts">
    import { writable } from 'svelte/store';
    import { Panel, getNodesBounds, getViewportForBounds, useEdges, useNodes, useConnection, useSvelteFlow } from '@xyflow/svelte';

    const nodes = useNodes();
    const edges = useEdges();
    const connection = useConnection();
    const { toObject } = useSvelteFlow();

    let projectData = toObject();
    const nodesArray = projectData.nodes;
    const EdgesArray = projectData.edges;

</script>