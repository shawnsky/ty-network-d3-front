<style src="vue-d3-network/dist/vue-d3-network.css"></style>
<template>
  <div id="app">
    <d3-network :net-nodes="nodes" :net-links="links" :options="options" />
    <button @click="start()">Start</button>
    <button @click="pause()">Pause</button>

    <!-- <my-line-chart id="line" :dataset="data1"></my-line-chart>

    <button @click="append()">Append</button> -->
  </div>
</template>

<script>
import D3Network from "vue-d3-network";
// import MyLineChart from "./components/MyLineChart2";
// import LineChart from "./components/LineChart";
const nidIndexMap = new Map();
const colorMap = new Map([]);

export default {
  name: "App",
  components: {
    D3Network,
    // MyLineChart,
  },
  data() {
    return {
      data1: [
        {
          name: "哈尔滨",
          value: [
            { key: "2015-1-1", value: 10 },
            { key: "2015-1-2", value: 12 },
            { key: "2015-1-3", value: 13 },
            { key: "2015-1-17", value: 17 },
          ],
        },
        {
          name: "海南",
          value: [
            { key: "2015-1-1", value: 9 },
            { key: "2015-1-2", value: 48 },
            { key: "2015-1-3", value: 5 },
            { key: "2015-1-17", value: 49 },
          ],
        },
        {
          name: "天津",
          value: [
            { key: "2015-1-2", value: 30 },
            { key: "2015-1-3", value: 1 },
            { key: "2015-1-4", value: 32 },
            { key: "2015-1-5", value: 10 },
          ],
        },
      ],
      hasStruct: false,
      nidIndexMap: nidIndexMap,
      nodes: [
        // { id: 1, name: "my awesome node 1" },
        // { id: 2, name: "my node 2" },
        // { id: 3, name: "orange node", _color: "orange" },
        // { id: 4, _color: "#4466ff" },
        // { id: 5 },
        // { id: 6 },
        // { id: 7 },
        // { id: 8 },
        // { id: 9 },
      ],
      links: [
        // { sid: 1, tid: 2 },
        // { sid: 2, tid: 8 },
        // {
        //   sid: 3,
        //   tid: 4,
        //   _svgAttrs: { "stroke-width": 8, opacity: 1 },
        //   name: "custom link",
        // },
        // { sid: 4, tid: 5 },
        // { sid: 5, tid: 6 },
        // { sid: 7, tid: 8 },
        // { sid: 5, tid: 8 },
        // { sid: 3, tid: 8 },
        // { sid: 7, tid: 9 },
      ],
      nodeSize: 25,
      canvas: false,
      i: 0,
    };
  },
  created() {
    this.initWebSocket();
  },
  destroyed() {
    this.websock.close(); //离开路由之后断开websocket连接
  },
  computed: {
    options() {
      return {
        force: 1000,
        size: { w: 1200, h: 1200 },
        nodeSize: this.nodeSize,
        nodeLabels: true,
        linkLabels: true,
        canvas: this.canvas,
      };
    },
  },
  methods: {
    rainbow(min, max, val) {
      var minHue = 240,
        maxHue = 0;
      var curPercent = (val - min) / (max - min);
      var colString =
        "hsl(" + (curPercent * (maxHue - minHue) + minHue) + ",100%,50%)";
      return colString;
    },
    append() {
      this.data1.push({
          name: "beijing",
          value: [
            { key: "2015-1-1", value: 11 },
            { key: "2015-1-2", value: 1 },
            { key: "2015-1-3", value: 20 },
            { key: "2015-1-17", value: 16 },
          ],
        },)
    },
    getColor(val) {
      var c = Math.round(Math.abs(val - 0.5) * 510);
      return `rgb(${c},${c},${c})`;
      // if (val > 0 && val < 0.5) {
      //   return `rgb(${c},${c},${c})`;
      // } else if (val > 0.5 && val < 1) {
      //   return `rgb(${Math.round(Math.abs(val-0.5)*510)},${0},${0}})`;
      // } else {
      //   return 'rgb(255,255,255)';
      // }
    },
    initWebSocket() {
      //初始化weosocket
      const wsuri = "ws://127.0.0.1:7341/ws";
      this.websock = new WebSocket(wsuri);
      this.websock.onmessage = this.websocketonmessage;
      this.websock.onopen = this.websocketonopen;
      this.websock.onerror = this.websocketonerror;
      this.websock.onclose = this.websocketclose;
    },
    start() {
      let actions = { BrowserMessage: "start" };
      this.websocketsend(JSON.stringify(actions));
    },
    pause() {
      let actions = { BrowserMessage: "pause" };
      this.websocketsend(JSON.stringify(actions));
      console.log("pause");
    },
    websocketonopen() {
      //连接建立之后执行send方法发送数据
      let actions = { BrowserMessage: "register" };
      this.websocketsend(JSON.stringify(actions));
    },
    websocketonerror() {
      //连接建立失败重连
      this.initWebSocket();
    },
    websocketonmessage(e) {
      //数据接收
      const redata = JSON.parse(e.data);

      if (!this.hasStruct) {
        var index = 0;
        this.nodes = [];
        redata.nodes.forEach((e) => {
          this.nodes.push({
            id: e.id,
            name: e.value + "",
            _color: e.active == 1 ? "black" : "",
          });
          this.nidIndexMap[e.id] = index;
          index += 1;
        });
        this.links = redata.edges;
        this.hasStruct = true;
      } else {
        redata.nodes.forEach((e) => {
          // console.log(e)
          this.nodes[this.nidIndexMap[e.id]]._color =
            e.active == 1 ? "black" : "";
          this.nodes[this.nidIndexMap[e.id]].name = e.value + "";
        });
      }
    },
    websocketsend(Data) {
      //数据发送
      this.websock.send(Data);
    },
    websocketclose(e) {
      //关闭
      console.log("断开连接", e);
    },
  },
};
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
</style>
