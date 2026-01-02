<template>
  <v-app>

    <v-app-bar color="primary" elevation="2" class="ps-6">

      <template v-slot:prepend><v-icon icon="mdi-speedometer"></v-icon></template>
      <v-app-bar-title>  迅客科技拨测</v-app-bar-title>
      <v-spacer></v-spacer>
    </v-app-bar>



    <v-main class="bg-grey-lighten-4">
      <v-container>
        <p>运维工具：</p>
        <v-tabs grow>
          <v-tab value="0">在线Ping</v-tab>
          <v-tab value="1">在线TCPing</v-tab>
          <v-tab value="2">网站测速</v-tab>
          <v-tab value="3">路由追踪</v-tab>
          <v-tab value="4">DNS查询</v-tab>
          <v-tab value="5">IPv4查询</v-tab>
          <v-tab value="6">IPv6查询</v-tab>
        </v-tabs>

        <v-card class="mb-6 pa-4" elevation="1">
          <v-row align="center">
            <v-col cols="12" md="8">
              <v-text-field v-model="targetUrl" color="primary" label="请输入测试域名" variant="outlined" hide-details density="comfortable" prepend-inner-icon="mdi-earth"></v-text-field>
            </v-col>
            <v-col cols="12" md="2">
              <v-btn  color="primary" size="large" elevation="6" block prepend-icon="mdi-play" @click="startTest">快速测试</v-btn>
            </v-col>
            <v-col cols="12" md="2">
              <v-btn  color="primary" variant="outlined" size="large" block prepend-icon="mdi-play-circle-outline" @click="startslowTest">
                缓慢测试
                <v-tooltip activator="parent" location="top">适合网站并发性能较差时使用</v-tooltip>
              </v-btn>
            </v-col>
          </v-row>
          <!--
          <v-row>
            <v-col cols="2"><v-checkbox :model-value="true" label="全部"></v-checkbox></v-col>
            <v-col cols="2"><v-checkbox :model-value="true" label="中国电信"></v-checkbox></v-col>
            <v-col cols="2"><v-checkbox :model-value="true" label="中国移动"></v-checkbox></v-col>
            <v-col cols="2"><v-checkbox :model-value="true" label="中国联通"></v-checkbox></v-col>
            <v-col cols="2"><v-checkbox :model-value="true" label="港澳台、海外"></v-checkbox></v-col>
          </v-row>
          -->
          <v-sheet class="bg-grey-lighten-4 rounded pa-3 mt-4">
            <div class="text-caption text-grey-darken-1 mb-1 ml-1">ISP 运营商选择</div>
            <v-row dense>
              <v-col cols="6" sm="4" md="2">
                <v-checkbox label="全选" color="primary" hide-details density="compact"></v-checkbox>
              </v-col>
              <v-col cols="6" sm="4" md="2">
                <v-checkbox label="中国电信" color="info" hide-details density="compact"></v-checkbox>
              </v-col>
              <v-col cols="6" sm="4" md="2">
                <v-checkbox label="中国移动" color="success" hide-details density="compact"></v-checkbox>
              </v-col>
              <v-col cols="6" sm="4" md="2">
                <v-checkbox label="中国联通" color="warning" hide-details density="compact"></v-checkbox>
              </v-col>
              <v-col cols="12" sm="8" md="4">
                <v-checkbox label="港澳台、海外节点" color="error" hide-details density="compact"></v-checkbox>
              </v-col>
            </v-row>
          </v-sheet>
        </v-card>

        <v-row class="mb-2">
          <v-col cols="6" md="3">
            <v-card class="pa-4 text-center" variant="outlined">
              <div class="text-caption text-grey">平均延迟</div>
              <div class="text-h4 font-weight-bold text-success">--</div>
            </v-card>
          </v-col>
          <v-col cols="6" md="3">
            <v-card class="pa-4 text-center" variant="outlined">
              <div class="text-caption text-grey">最快节点</div>
              <div class="text-h4 font-weight-bold text-success">--</div>
            </v-card>
          </v-col>
          <v-col cols="6" md="3">
            <v-card class="pa-4 text-center" variant="outlined">
              <div class="text-caption text-grey">最慢节点</div>
              <div class="text-h4 font-weight-bold text-error">--</div>
            </v-card>
          </v-col>
          <v-col cols="6" md="3">
            <v-card class="pa-4 text-center" variant="outlined">
              <div class="text-caption text-grey">状态</div>
              <div class="text-h4 font-weight-bold text-info">待机</div>
            </v-card>
          </v-col>
        </v-row>

        <v-row>
          <v-col cols="12" md="7">
            <v-card class="fill-height">
              <v-card-title class="d-flex align-center">
                <v-icon icon="mdi-map-marker-radius" class="mr-2" size="small"></v-icon>
                全国延迟
              </v-card-title>
              <v-divider></v-divider>
              <v-card-text>
                <div ref="mapContainer" class="china-map-container"></div>
              </v-card-text>
            </v-card>
          </v-col>

          <v-col cols="12" md="5">
            <v-card class="fill-height d-flex flex-column" height="550">
              <v-card-title>
                <v-icon icon="mdi-format-list-bulleted" class="mr-2" size="small"></v-icon>
                节点列表
              </v-card-title>
              <v-divider></v-divider>
              <div class="d-flex align-center justify-center flex-grow-1 text-grey">
                暂无数据
              </div>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref, onMounted, onUnmounted, nextTick } from 'vue';
import * as echarts from 'echarts';

const targetUrl = ref('www.baidu.com');
const mapContainer = ref(null);
let mapInstance = null;

const startTest = () => {
  if (!targetUrl.value) {
    alert("请输入域名");
    return;
  }
};
const startslowTest = () => {
  if (!targetUrl.value) {
    alert("请输入域名");
    return;
  }
};

//地图
const initMap = async () => {
  if (!mapContainer.value) return;
  mapInstance = echarts.init(mapContainer.value);
  mapInstance.showLoading();
  try {
    const response = await fetch('https://cdn.jsdmirror.com/npm/echarts@4.9.0/map/json/china.json');
    const chinaJson = await response.json();
    mapInstance.hideLoading();
    echarts.registerMap('china', chinaJson);
    const option = {
      tooltip: {
        trigger: 'item',
        formatter: '{b}' //仅显示省份名称
      },
      series: [
        {
          name: '中国地图',type: 'map',map: 'china',roam: false,zoom: 1.2,
          itemStyle: {
            areaColor: '#24aa1d', //默认绿色
            borderColor: '#ecf0f1', //白色边框
            borderWidth: 0.7
          },
          emphasis: {
            label: { show: false, color: '#fff' },
            itemStyle: { areaColor: '#ffdf33' } //悬停：深绿
          },
          data: []
        }
      ]
    };
    mapInstance.setOption(option);
  } catch (error) {
    console.error("地图加载失败:", error);
    mapInstance.hideLoading();
  }
};
//窗口大小变化,重绘地图
const handleResize = () => {mapInstance && mapInstance.resize();};
//生命周期
onMounted(() => {
  nextTick(() => {
    initMap();
    window.addEventListener('resize', handleResize);
  });
});
onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
  mapInstance && mapInstance.dispose();
});
</script>

<style scoped>
.china-map-container {width: 100%;height: 450px;}
</style>