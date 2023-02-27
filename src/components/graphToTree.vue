<template >
    <div class="function">
        <input placeholder="请输入起点" v-model="startvex" style="margin-right: 5px;">
        <button @click="th" style="margin-right: 5px;">点击生成最小生成树</button>
        <input placeholder="输入起点" v-model="a" style="margin-right: 5px;">
        <input placeholder="输入终点" v-model="b" style="margin-right: 5px;">
        <input placeholder="输入权值" v-model="weight" style="margin-right: 5px;">
        <button @click="addedge" style="margin-right: 5px;">点击加边</button>
        <button @click="makeExa">点击生成例子</button>
    </div>

    <div id="main" style="height:900px" class="main">
    </div>
</template>

<script >
import * as echarts from 'echarts/core';
import { TitleComponent, TooltipComponent } from 'echarts/components';
import { GraphChart } from 'echarts/charts';
import { CanvasRenderer } from 'echarts/renderers';

echarts.use([TitleComponent, TooltipComponent, GraphChart, CanvasRenderer]);
import { onMounted, ref } from 'vue'

export default {
    setup() {


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
        var myChart2;
        class Graph {
            constructor(theNumOfVexs, theNumOfEdges) {
                this.vertices = []; // 用来存放图中的顶点

                this.adjList = new Array(100 + 1);
                //初始化邻接矩阵
                for (let i = 0; i < 100; i++) {
                    this.adjList[i] = new Array(100).fill(65535);//邻接矩阵点与点之间无连线，距离为正无穷，此处用65535代替
                }

                this.numOfVertexes = theNumOfVexs;
                this.numOfEdges = theNumOfEdges;
            }

            // 向图中添加一个新顶点
            addVertex(v) {
                if (!this.vertices.includes(v)) {
                    this.vertices.push(v);
                    this.numOfVertexes++;
                }
            }

            // 向图中添加a和b两个顶点之间的边
            addEdge(a, b, weight) {
                // 如果图中没有顶点a，先添加顶点a
                    this.addVertex(a);
                // 如果图中没有顶点b，先添加顶点b
                    this.addVertex(b);
                
                //无向带权图的加边(对称矩阵)
                this.adjList[a][b] = weight
                this.adjList[b][a] = weight
                this.numOfEdges++;
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
                for (let i = 0; i < this.vertices[this.vertices.length-1]; i++) {
                    for (let j = 0; j <this.vertices[this.vertices.length-1]; j++) {
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
                let chosenVex = start;
                const sleep = (timeout = 1000) => new Promise((resolve, reject) => {
                    setTimeout(resolve, timeout);
                });

                let timer = async () => {
                    for (let numOfChoose = 1; numOfChoose <= this.numOfVertexes; numOfChoose++) {
                        await sleep(1000);
                        //渲染被选中点
                        let tmdata = option.series[0].data.filter((item) => (item.name == chosenVex));
                        tmdata[0].itemStyle.normal.color = '#28e698';
                        option && myChart2.setOption(option);
                        console.log('done')

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
                        //渲染符合条件的边
                        if (numOfChoose != 1) {
                            console.log(chosenVex);
                            console.log(parent[chosenVex]);
                            console.log(option.series);
                            let tmlink = option.series[0].links.filter((item) => ((item.source == parent[chosenVex] && item.target == chosenVex) || (item.source == chosenVex && item.target == parent[chosenVex])));
                            console.log(tmlink)
                            tmlink[0].lineStyle.color = '#fed71a';
                            option && myChart2.setOption(option);
                        }
                        //将符合条件顶点加入被选顶点集合中
                        chosenVex = nextVex;

                    }
                }

                timer()
            }

            initGraph(theNumOfVexs) {
                console.log('233');        //echarts图的初始化
                console.log(theNumOfVexs);
                option.series[0].data = [];
                for (let i = 0; i < theNumOfVexs; i++) {
                    option.series[0].data.push({ name: graph.vertices[i], draggable: true, itemStyle: { normal: { color: '	#5b5b5b' } } })

                }
                console.log(option.series[0].data)

                option.series[0].links = [];
                console.log(graph.vertices[theNumOfVexs - 1])
                for (let i = 0; i <= graph.vertices[theNumOfVexs - 1]; i++) {
                    for (let j = 0; j < i; j++) {
                        // console.log(graph.adjList[i][j]);

                        if (graph.adjList[i][j] != 65535) {
                            // console.log("push");
                            option.series[0].links.push({
                                source: i, target: j,
                                label: { show: true, formatter: function () { return graph.adjList[i][j] } },
                                lineStyle: {
                                    curveness: 0,
                                    color: '#5b5b5b'
                                }
                            })
                        }
                    }
                }
                console.log(graph.vertices);
                console.log(option.series[0].links)

                // console.log(graph.toString());
            }

            example() {
                this.addEdge(0, 1, 4);
                this.addEdge(0, 7, 8);
                this.addEdge(1, 7, 11);
                this.addEdge(1, 2, 8);
                this.addEdge(2, 3, 7);
                this.addEdge(2, 8, 2);
                this.addEdge(2, 5, 4);
                this.addEdge(3, 5, 14);
                this.addEdge(3, 4, 9);
                this.addEdge(4, 5, 10);
                this.addEdge(5, 6, 2);
                this.addEdge(6, 7, 1);
                this.addEdge(6, 8, 6);
                this.addEdge(7, 8, 7);
                graph.adjustVertices();
                graph.adjustAdjList();

                console.log(graph.numOfVertexes)
                graph.initGraph(graph.numOfVertexes);
            }

        }


        let vexsnum = 0;
        let edgenum = 0;
        let graph = new Graph(vexsnum, edgenum);

        const makeExa = () => {
            graph.example();
            option && myChart2.setOption(option);
        }

        // graph.adjustAdjList();
        // graph.adjustVertices();
        // console.log(graph.numOfVertexes)
        // graph.initGraph(graph.numOfVertexes);
        //输入最小生成树的起点并执行
        let startvex = ref();
        const th = () => {
            if (!graph.vertices.includes(Number(startvex.value))) {
                alert("请输入正确的起点")
                return;
            }
            graph.MiniSpanTree_Prim(startvex.value);
            option && myChart2.setOption(option);
        }
        let a = ref(), b = ref(), weight = ref();
        const addedge = () => {
            if(isNaN(Number(a.value)) ||isNaN(Number(b.value)) || isNaN(Number(weight.value)) ){
                alert("请合法输入");
                return;
            }
            graph.addEdge(Number(a.value), Number(b.value), Number(weight.value));
            graph.adjustVertices();
            graph.adjustAdjList();

            graph.initGraph(graph.numOfVertexes);
            option && myChart2.setOption(option);
        }

        const editedge = () => {

        }

        onMounted(() => {            //echarts的配置项
            var chartDom = document.getElementById('main');
            var myChart = echarts.init(chartDom);
            myChart2 = myChart;
            option && myChart.setOption(option);
        })
        return { th, startvex, a, b, weight, addedge, editedge, makeExa }
    },

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

.main {
    background-image: radial-gradient(73% 147%, #EADFDF 59%, #ECE2DF 100%),
        radial-gradient(91% 146%, rgba(255, 255, 255, 0.50) 47%, rgba(0, 0, 0, 0.50) 100%);
    background-blend-mode: screen;
}

input {
    outline-style: none;
    border: 1px solid #ccc;
    border-radius: 3px;
    padding: 6px;
    width: 150px;
    height: 30px;
    font-size: 14px;
    font-family: "Microsoft soft";
}

input:focus {
    border-color: #66afe9;
    outline: 0;
    -webkit-box-shadow: inset 0 1px 1px rgba(0, 0, 0, .075), 0 0 8px rgba(102, 175, 233, .6);
    box-shadow: inset 0 1px 1px rgba(0, 0, 0, .075), 0 0 8px rgba(102, 175, 233, .6)
}

button {
    /* 按钮美化 */
    width: 200px;
    /* 宽度 */
    height: 50px;
    /* 高度 */
    border-width: 0px;
    /* 边框宽度 */
    border-radius: 3px;
    /* 边框半径 */
    background: #51c4d3;
    /* 背景颜色 */
    cursor: pointer;
    /* 鼠标移入按钮范围时出现手势 */
    outline: none;
    /* 不显示轮廓线 */
    font-family: Microsoft YaHei;
    /* 设置字体 */
    color: white;
    /* 字体颜色 */
    font-size: 10px;
    /* 字体大小 */
}

.login-button:hover {
    /* 鼠标移入按钮范围时改变颜色 */
    background: #5599FF;
}

.flamingo {
    background-image: radial-gradient(73% 147%, #EADFDF 59%, #ECE2DF 100%),
        radial-gradient(91% 146%, rgba(255, 255, 255, 0.50) 47%, rgba(0, 0, 0, 0.50) 100%);
    background-blend-mode: screen;
}

.function {
    margin-top: -55px;
    background-image: radial-gradient(73% 147%, #EADFDF 59%, #ECE2DF 100%),
        radial-gradient(91% 146%, rgba(255, 255, 255, 0.50) 47%, rgba(0, 0, 0, 0.50) 100%);
    background-blend-mode: screen;
}
</style>
