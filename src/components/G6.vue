<template>
  <div 
    class="container"
    ref="container"></div>
</template>

<script>
import G6 from '@antv/g6';

const drawSizePoint = () => {
    const rectPoint = id => ({
        id,
        type: 'rect',
        size: [5, 5],
        x: 0,
        y: 0,
        style: {
            stroke: '#000',
            fill: '#fff',
            cursor: id
        }
    });
    return {
        'e-resize': rectPoint('e-resize'),
        'w-resize': rectPoint('w-resize'),
        'n-resize': rectPoint('n-resize'),
        's-resize': rectPoint('s-resize'),
        'ne-resize': rectPoint('ne-resize'),
        'nw-resize': rectPoint('nw-resize'),
        'se-resize': rectPoint('se-resize'),
        'sw-resize': rectPoint('sw-resize')
    };
};

const SizeControl = {
    'e-resize': [1, 0.5],
    'w-resize': [0, 0.5],
    'n-resize': [0.5, 0],
    's-resize': [0.5, 1],
    'ne-resize': [1, 0],
    'nw-resize': [0, 0],
    'se-resize': [1, 1],
    'sw-resize': [0, 1]
};
export default {
    name: 'G6',
    props: {
        value: {
            type: Object,
            required: true,
            default: () => ({})
        }
    },
    data() {
        return {
            selectItem: null,
            sizePoint: null
        };
    },
    mounted() {
        this.init();
        this.update();
    },
    watch: {
        value: {
            handler() {
                this.update();
            },
            deep: true
        },
        selectItem(node) {
            if (node) {
                if (!this.sizePoint) {
                    // 生成边界点
                    this.sizePoint = drawSizePoint();
                }
                // 调整边界点位置
                this.positionSizePoint(node.getBBox());
            }
            else if (this.sizePoint) {
                this.sizePoint = null;
            }
            this.update();
        }
    },
    methods: {
        positionSizePoint({x, y, width, height}) {
            for (let key in this.sizePoint) {
                const point = this.sizePoint[key];
                point.x = x + SizeControl[key][0] * width;
                point.y = y + SizeControl[key][1] * height;
            }
        },
        init() {
            if (!this.graph) {
                const mountResizeEvents = this.resize();
                let {offsetWidth, offsetHeight} = this.$refs.container;
                this.graph = new G6.Graph({
                    container: this.$refs.container,
                    width: offsetWidth,
                    height: offsetHeight,
                    modes: this.modes
                });
                mountResizeEvents(this.graph);
            }
            return this.graph;
        },
        update() {
            this.graph.data({
                nodes: [
                    ...this.value.nodes,
                    ...Object.values(this.sizePoint || {})
                ],
                edges: this.value.edges
            });
            this.graph.render();
        },
        resize() {
            this.modes = {
                default: [{
                    type: 'click-select',
                    multiple: false
                }]
            };
            return graph => {
                graph.on('nodeselectchange', ({target}) => {
                    if (target && !/-resize$/.test(target.getModel().id)) {
                        this.selectItem = target;
                    }
                    else if (!target) {
                        this.selectItem = null;
                    }
                });
                graph.on('node:drag', ({item, x, y}) => {
                    const id = item.getModel().id;
                    if (!/-resize$/.test(id)) {
                        return;
                    }
                    const offsetX = SizeControl[id][0] - 0.5;
                    const offsetY = SizeControl[id][1] - 0.5;
                    this.positionSizePoint(this.resizeBox([offsetX, offsetY], [x, y]));
                    for (let key in this.sizePoint) {
                        this.graph.findById(key).updatePosition(this.sizePoint[key]);
                    }
                });
            };

        },
        resizeBox([offsetX, offsetY], [pointX, pointY]) {
            const {x, y, width, height} = this.selectItem.getBBox();
            const newOffset = {
                    x: offsetX >= 0 ? x : pointX,
                    y: offsetY >= 0 ? y : pointY,
                    width: offsetX === 0 ? width
                        : offsetX > 0 ? pointX - x : x + width - pointX,
                    height: offsetY === 0 ? height
                        : offsetY > 0 ? pointY - y : y + height - pointY
                };
            // this.selectItem.update({
            //     ...this.selectItem.getModel(),
            //     x: newOffset.x + newOffset.width / 2,
            //     y: newOffset.y + newOffset.height / 2,
            //     size: [newOffset.width, newOffset.height]
            // });
            return newOffset;
        }
    }
};
</script>

<style>

</style>