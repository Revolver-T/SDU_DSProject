<template>
    <div id="main" style='height: 900px' class="main"></div>
</template>

<script>
import * as echarts from 'echarts/core';
import { TitleComponent, TooltipComponent } from 'echarts/components';
import { GraphChart } from 'echarts/charts';
import { CanvasRenderer } from 'echarts/renderers';

echarts.use([TitleComponent, TooltipComponent, GraphChart, CanvasRenderer]);
import { onMounted } from 'vue'
export default {
    setup() {


        onMounted(() => {            //echarts的配置项
            var chartDom = document.getElementById('main');
            var myChart = echarts.init(chartDom);
            var option;

            option = {
                title: {
                    text: '基于Prim生成最小生成树的图算法'
                },
                tooltip: {},
                animationDurationUpdate: 1500,
                animationEasingUpdate: 'quinticInOut',
                series: [
                    {
                        type: 'graph',
                        layout: 'force',
                        force: {
                            repulsion: 2000,
                            edgeLength: 500,
                        },
                        symbolSize: 35,
                        roam: true,
                        label: {
                            show: true
                        },
                        edgeSymbol: ['circle'],
                        edgeSymbolSize: [4, 10],
                        edgeLabel: {
                            fontSize: 20
                        },
                        data: [
                        ],
                        links: [
                            {
                                source: 0,
                                target: 1,
                                symbolSize: [5, 20],

                                label: {
                                    show: true,
                                    formatter: function () {
                                        return '100';
                                    }
                                },
                                lineStyle: {

                                    curveness: 0.2
                                }
                            },
                        ],
                        lineStyle: {
                            opacity: 0.9,
                            width: 2,
                            curveness: 0
                        }
                    }
                ]
            };

            const sleep = (milliseconds) => {
                return new Promise(resolve => setTimeout(resolve, milliseconds))
            }

            class Graph {

                constructor(theNumOfVexs, theNumOfEdges) {
                    this.vertices = []; // 用来存放图中的顶点

                    this.adjList = new Array(theNumOfVexs + 1);
                    //初始化邻接矩阵
                    for (let i = 0; i < theNumOfVexs; i++) {
                        this.adjList[i] = new Array(theNumOfVexs).fill(65535);//邻接矩阵点与点之间无连线，距离为正无穷，此处用65535代替
                    }

                    this.numOfVertexes = theNumOfVexs;
                    this.numOfEdges = theNumOfEdges;
                }

                // 向图中添加一个新顶点
                addVertex(v) {
                    if (!this.vertices.includes(v)) {
                        this.vertices.push(v);
                    }
                }

                // 向图中添加a和b两个顶点之间的边
                addEdge(a, b, weight) {
                    // 如果图中没有顶点a，先添加顶点a
                    if (!this.adjList.includes(a)) {
                        this.addVertex(a);
                    }
                    // 如果图中没有顶点b，先添加顶点b
                    if (!this.adjList.includes(b)) {
                        this.addVertex(b);
                    }
                    //无向带权图的加边(对称矩阵)
                    this.adjList[a][b] = weight
                    this.adjList[b][a] = weight
                }

                // 获取图的vertices
                getVertices() {
                    return this.vertices;
                }

                // 获取图中的adjList
                getAdjList() {
                    return this.adjList;
                }

                adjustAdjList() {
                    //对角处置零
                    for (let i = 0; i < this.numOfVertexes; i++) {
                        for (let j = 0; j < this.numOfVertexes; j++) {
                            if (i == j)
                                this.adjList[i][j] = 0;
                        }
                    }
                }

                adjustVertices() {
                    this.vertices.sort();
                }

                toString() {
                    let s = '';
                    this.vertices.forEach((v) => {
                        s += `${v} -> `;
                        this.adjList[v].forEach((n) => {
                            s += `${n} `;
                        });
                        s += '\n';
                    });
                    return s;
                }

                MiniSpanTree_Prim(start) {
                    let selected = new Array(this.numOfVertexes).fill(0);//被选中的顶点
                    let minDist = new Array(this.numOfVertexes).fill(65535);//与被选顶点的距离最小值
                    let parent = new Array(this.numOfVertexes).fill(-1);///距离最小值的顶点父亲记录为被选顶点，达到记录路径作用
                    //更新起始点信息
                    let numOfChoose = 1;
                    let chosenVex = start;
                    while (numOfChoose != this.numOfVertexes) {
                        //修改选中点的颜色
                        const sleep = (timeout = 1000) => new Promise((resolve, reject) => {
                            setTimeout(resolve, timeout);
                        });
                        // 可以使用 bluebird模块中的 bluebird.delay() 替换 sleep()
                        // const bluebird = ruquire('bluebird');
                        let timer = async (timeout) => {
                            await sleep(1000);
                            console.log('233');
                            let tmdata = option.series[0].data.filter((item) => (item.name == chosenVex));
                            console.log(tmdata)
                            tmdata[0].itemStyle.normal.color = '#28e698';
                            option && myChart.setOption(option);
                            console.log('done')
                        }
                        timer(10);




                        selected[chosenVex] = 1;
                        minDist[chosenVex] = 65535;
                        //更新列表信息
                        for (let i = 0; i < this.numOfVertexes; i++) {
                            if (this.adjList[chosenVex][i] != 65535 && selected[i] != 1) {                    //该顶点与被选顶点邻接且不是已被选过的顶点，更新信息
                                if (this.adjList[chosenVex][i] < minDist[i]) {
                                    minDist[i] = this.adjList[chosenVex][i];
                                    parent[i] = chosenVex;
                                }

                            }
                        }
                        //扫描列表
                        let nextVex = 0;
                        let min = 65535;
                        for (let i = 0; i < this.numOfVertexes; i++) {

                            if (minDist[i] <= min) {
                                min = minDist[i];
                                nextVex = i;
                            }
                        }
                        //将符合条件顶点加入被选顶点集合中
                        chosenVex = nextVex;
                        numOfChoose++;
                    }


                    for (let i = 0; i < this.numOfVertexes; i++) {
                        console.log(parent[i])
                    }

                }

            }


            let theNumOfVexs = 9;
            let theNumOfEdges = 10;
            let graph = new Graph(theNumOfVexs, theNumOfEdges);

            graph.addEdge(0, 1, 4);
            graph.addEdge(0, 7, 8);
            graph.addEdge(1, 7, 11);
            graph.addEdge(1, 2, 8);
            graph.addEdge(2, 3, 7);
            graph.addEdge(2, 8, 2);
            graph.addEdge(2, 5, 4);
            graph.addEdge(3, 5, 14);
            graph.addEdge(3, 4, 9);
            graph.addEdge(4, 5, 10);
            graph.addEdge(5, 6, 2);
            graph.addEdge(6, 7, 1);
            graph.addEdge(6, 8, 6);
            graph.addEdge(7, 8, 7);

            graph.adjustAdjList();
            graph.adjustVertices();
            //echarts图的初始化
            console.log(option)
            console.log(option.series[0].data)
            option.series[0].data = [];
            for (let i = 0; i < theNumOfVexs; i++) {
                option.series[0].data.push({ name: graph.vertices[i], draggable: true, itemStyle: { normal: { color: '	#5b5b5b' } } })
                option.series[0].links
            }
            console.log(option.series[0].links)
            option.series[0].links = [];
            for (let i = 0; i < theNumOfVexs; i++) {
                for (let j = 0; j < i; j++) {
                    if (graph.adjList[i][j] != 65535) {
                        option.series[0].links.push({
                            source: i, target: j,
                            label: { show: true, formatter: function () { return graph.adjList[i][j] } },
                            lineStyle: {
                                curveness: 0,
                                color: '#CC0000'
                            }
                        })
                    }
                }
            }
            let temp = option.series[0].links.filter((item) => (item.source == 1 && item.target == 0))
            console.log(temp)

            graph.MiniSpanTree_Prim(0);
            console.log(graph.toString());
            option && myChart.setOption(option);






        })

    }
}
</script>
<style>
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}

.main {}
</style>
