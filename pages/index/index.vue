<template>
  <v-container fluid :class="$vuetify.breakpoint.xs?'container':'px-12'">
    <h1 class="text-center head">欢迎进入雪中控制台</h1>
    <v-row>
      <v-col cols="12" md="3">
        <v-card flat :loading="loading">
          <v-card-title>栏目管理</v-card-title>
          <v-card-text class="pa-0">
            <div id="chart1"></div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="3">
        <v-card flat :loading="loading">
          <v-card-title>栏目柱状信息</v-card-title>
          <v-card-text class="pa-0">
            <div id="chart2"></div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="6">
        <v-card flat :loading="loading">
          <v-card-title>词云</v-card-title>
          <v-card-text class="pa-0">
            <div id="chart3" class="my-word-box"></div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>
<script>
import Dataset from "@antv/data-set";
import _static from "~/script/static.js";
import { Chart, registerShape, Util } from "@antv/g2"; //或者只引入需要用到的G2组件，我要用Chart
export default {
  vNname: "index",
  data: () => ({
    loading: true,
  }),
  async mounted() {
    let that = this;
    this.getColumnCount();
    // console.log(res);
  },
  methods: {
    async getColumnCount() {
      let that = this;
      try {
        let res = await that.api.getAllColumnCount();
        that.columnData = res.data;
        that.initLineChart();
        that.initPillarChart();
        that.words();
        that.loading = false;
      } catch (e) {
        console.log(e);
        return that.$hint({
          msg: "获取栏目信息失败",
          type: "error",
        });
      }
    },
    initLineChart() {
      let that = this;
      const chart = new Chart({
        container: "chart1",
        autoFit: true,
        height: 150,
        appendPadding: 20,
      });

      chart.data(that.columnData);
      chart.scale({
        num: {
          min: 10,
          nice: true,
        },
        name: {
          range: [0, 1],
        },
      });
      chart.tooltip({
        showCrosshairs: true,
        shared: true,
      });

      chart.axis("num", {
        label: {
          formatter: (val) => {
            return val;
          },
        },
      });

      chart.area().position("name*num").shape("smooth");
      chart.line().position("name*num").shape("smooth");
      chart.point().position("name*num").color("#0094ff").shape("circle");
      chart.render();
    },
    initPillarChart() {
      let that = this;
      const chart = new Chart({
        container: "chart2",
        autoFit: true,
        height: 150,
        appendPadding: 20,
      });
      chart.data(that.columnData);
      chart.scale("num", {
        nice: true,
      });
      chart.axis("name", {
        tickLine: null,
      });

      chart.axis("num", function (val) {
        return val;
      });

      chart.tooltip({
        showMarkers: false,
      });
      chart.interaction("element-active");

      chart.legend(false);
      chart
        .interval()
        .position("name*num")
        .label("num", function (val) {
          return val;
        })
        .color("name", ["#063d8a", "#1770d6", "#47abfc", "#38c060", "#0094ff"]);
      chart.render();
    },
    words() {
      // 给point注册一个词云的shape
      let that = this;
      registerShape("point", "cloud", {
        draw(cfg, container) {
          const attrs = that.getTextAttrs(cfg);
          const textShape = container.addShape("text", {
            attrs: {
              ...attrs,
              x: cfg.x,
              y: cfg.y,
            },
          });
          if (cfg.data.rotate) {
            Util.rotate(textShape, (cfg.data.rotate * Math.PI) / 180);
          }
          return textShape;
        },
      });
      let _w = document.querySelector(".my-word-box").offsetWidth;
      const dv = new Dataset.View().source(_static.words);
      const range = dv.range("value");
      const min = range[0];
      const max = range[1];
      dv.transform({
        type: "tag-cloud",
        fields: ["x", "value"],
        size: [_w, 150],
        font: "Verdana",
        padding: 0,
        timeInterval: 5000, // max execute time
        rotate() {
          let random = ~~(Math.random() * 4) % 4;
          if (random === 2) {
            random = 0;
          }
          return random * 90; // 0, 90, 270
        },
        fontSize(d) {
          if (d.value) {
            return ((d.value - min) / (max - min)) * (80 - 24) + 24;
          }
          return 0;
        },
      });
      const chart = new Chart({
        container: "chart3",
        autoFit: true,
        width: _w,
        height: 150,
        padding: 0,
      });
      chart.data(dv.rows);
      chart.scale({
        x: { nice: false },
        y: { nice: false },
      });
      chart.legend(false);
      chart.axis(false);
      chart.tooltip({
        showTitle: false,
        showMarkers: false,
      });
      chart.coordinate().reflect();
      chart.point().position("x*y").shape("cloud").tooltip("category");
      chart.interaction("element-active");
      chart.render();
    },
    getTextAttrs(cfg) {
      return {
        fontSize: cfg.data.size,
        text: cfg.data.text,
        textAlign: "center",
        fontFamily: cfg.data.font,
        fill: `rgb(${Math.floor(Math.random() * 255)}, ${Math.floor(
          Math.random() * 255
        )},${Math.floor(Math.random() * 255)})`,
        textBaseline: "Alphabetic",
      };
    },
  },
};
</script>
<style lang="scss" scoped>
.container{padding:0;padding-right:12px;padding-top:20px}
#chart1，#chart2,
#chart3 {
  background-color: #fff;
}
.head {
  margin-bottom: 50px;
}
</style>