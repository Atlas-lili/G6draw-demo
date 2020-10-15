<template>
    <div id="app">
        <g6 :value="value" calss="container"></g6>
    </div>
</template>
<script>
import G6 from '@/components/G6.vue';

export default {
    data() {
        return {
            value: {
                nodes: [
                    {
                        id: 'node1',
                        type: 'circle',
                        size: 40,
                        x: 100,
                        y: 200
                    },
                    {
                        id: 'node2',
                        type: 'rect',
                        size: [50, 20],
                        x: 300,
                        y: 200
                    }
                ],
                edges: [
                    {
                        source: 'node1',
                        target: 'node2'
                    }
                ]
            }
        };
    },
    components: {
        G6
    }
};
</script>
<style lang="less">
.container {
    height: 500px;
}
</style>
