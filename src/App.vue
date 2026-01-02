<template>
  <v-app class="bg-grey-lighten-5">
    <v-app-bar color="white" elevation="1" density="comfortable">
      <template v-slot:prepend>
        <v-avatar color="primary" variant="tonal" size="32" class="ms-2">
          <v-icon icon="mdi-speedometer" size="small"></v-icon>
        </v-avatar>
      </template>
      <v-app-bar-title class="font-weight-bold text-h6 text-primary">讯客科技拨测</v-app-bar-title>
      <v-spacer></v-spacer>
      <v-btn icon="mdi-github" variant="text" color="grey"></v-btn>
    </v-app-bar>

    <v-main>
      <v-container class="py-6">

        <v-card elevation="0" border class="mb-6 rounded-lg">
          <v-tabs v-model="activeTab" color="primary" bg-color="grey-lighten-5" slider-color="primary">
            <v-tab v-for="(tool, index) in tools" :key="index" :value="index" class="text-capitalize">
              <v-icon :icon="tool.icon" start size="small"></v-icon> {{ tool.name }}
            </v-tab>
          </v-tabs>

          <v-divider></v-divider>

          <div class="pa-6">
            <v-row align="center" dense>
              <v-col cols="12" md="8">
                <v-text-field
                    v-model="targetUrl" color="primary" variant="outlined"
                    label="请输入测试域名 / IP" hide-details clearable="clearable"
                    density="comfortable" prepend-inner-icon="mdi-web" @keyup.enter="startTest"
                    class="rounded-input"
                ></v-text-field>
              </v-col>
              <v-col cols="6" md="2">
                <v-btn color="primary" height="48" block elevation="2" prepend-icon="mdi-lightning-bolt" @click="startTest">
                  快速测试
                </v-btn>
              </v-col>
              <v-col cols="6" md="2">
                <v-btn color="secondary" variant="tonal" height="48" block prepend-icon="mdi-tortoise" @click="startslowTest">
                  慢速拨测
                  <v-tooltip activator="parent" location="top">并发较低，适合性能较弱的目标</v-tooltip>
                </v-btn>
              </v-col>
            </v-row>

            <div class="mt-6">
              <div class="text-caption text-grey-darken-1 mb-2 font-weight-medium">监测节点选择</div>
              <v-chip-group v-model="selectedISPs" column multiple filter selected-class="text-primary">
                <v-chip v-for="isp in ispList" :key="isp.value" :value="isp.value" variant="outlined" :ripple="false" label size="small">
                  {{ isp.label }}
                </v-chip>
              </v-chip-group>
            </div>
          </div>
        </v-card>

        <v-row class="mb-2" dense>
          <v-col cols="6" md="3" v-for="(stat, idx) in statistics" :key="idx">
            <v-card class="pa-4 d-flex align-center rounded-lg" elevation="0" border>
              <v-avatar :color="stat.color" variant="tonal" size="48" class="mr-4 rounded-lg">
                <v-icon :icon="stat.icon"></v-icon>
              </v-avatar>
              <div>
                <div class="text-caption text-grey">{{ stat.label }}</div>
                <div class="text-h5 font-weight-bold" :class="`text-${stat.color}`">{{ stat.value }}</div>
              </div>
            </v-card>
          </v-col>
        </v-row>

        <v-row>
          <v-col cols="12" md="7">
            <v-card class="fill-height rounded-lg" elevation="0" border>
              <v-card-title class="d-flex align-center py-3 px-4 bg-grey-lighten-5">
                <v-icon icon="mdi-map-marker-radius-outline" class="mr-2" size="small" color="primary"></v-icon>
                <span class="text-body-1 font-weight-bold">全国延迟热力图</span>
              </v-card-title>
              <v-divider></v-divider>
              <v-card-text class="pa-0 position-relative">
                <div ref="mapContainer" class="china-map-container"></div>
                <div class="position-absolute bottom-0 left-0 pa-4 text-caption text-grey">
                  * 颜色越深代表延迟越高
                </div>
              </v-card-text>
            </v-card>
          </v-col>

          <v-col cols="12" md="5">
            <v-card class="fill-height d-flex flex-column rounded-lg" height="500" elevation="0" border>
              <v-card-title class="d-flex align-center py-3 px-4 bg-grey-lighten-5">
                <v-icon icon="mdi-format-list-bulleted" class="mr-2" size="small" color="primary"></v-icon>
                <span class="text-body-1 font-weight-bold">实时监测结果</span>
                <v-spacer></v-spacer>
                <v-chip size="x-small" color="grey" variant="flat">0 / 0</v-chip>
              </v-card-title>
              <v-divider></v-divider>

              <div class="d-flex px-4 py-2 bg-grey-lighten-5 text-caption font-weight-bold text-grey-darken-1">
                <div style="width: 30%">监测点</div>
                <div style="width: 25%">IP地址</div>
                <div style="width: 20%" class="text-end">延迟</div>
                <div style="width: 25%" class="text-end">状态</div>
              </div>
              <v-divider></v-divider>

              <div class="d-flex align-center justify-center flex-grow-1 flex-column text-grey-lighten-1">
                <v-icon icon="mdi-server-network-off" size="64" class="mb-2 opacity-50"></v-icon>
                <div class="text-body-2">等待开始测试...</div>
              </div>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted, nextTick } from 'vue';
import * as echarts from 'echarts';

const targetUrl = ref('qiqinb.cn');
const activeTab = ref(0);
const selectedISPs = ref(['all']);

const tools = [
  { name: '在线Ping', icon: 'mdi-lan-connect' },
  { name: '在线TCPing', icon: 'mdi-handshake-outline' },
  { name: '网站测速', icon: 'mdi-speedometer' },
  { name: '路由追踪', icon: 'mdi-map-marker-path' },
  { name: 'DNS查询', icon: 'mdi-dns' },
  { name: 'IPv4/6查询', icon: 'mdi-ip-network' },
];

const ispList = [
  { label: '全选', value: 'all' },
  { label: '中国电信', value: 'ct' },
  { label: '中国移动', value: 'cm' },
  { label: '中国联通', value: 'cu' },
  { label: '港澳台、海外', value: 'overseas' },
];

const statistics = reactive([
  { label: '平均延迟', value: '--', unit: 'ms', color: 'primary', icon: 'mdi-timer-outline' },
  { label: '最快节点', value: '--', unit: '', color: 'success', icon: 'mdi-rocket-launch' },
  { label: '最慢节点', value: '--', unit: '', color: 'warning', icon: 'mdi-tortoise' },
  { label: '当前状态', value: '待机', unit: '', color: 'info', icon: 'mdi-list-status' },
]);
const startTest = () => {
  if (!targetUrl.value) return alert("请输入域名");
  statistics[3].value = "测试中...";
  //
};

const startslowTest = () => {
  if (!targetUrl.value) return alert("请输入域名");
};

//ECharts地图逻辑
const mapContainer = ref(null);
let mapInstance = null;

const initMap = async () => {
  if (!mapContainer.value) return;

  mapInstance = echarts.init(mapContainer.value);
  mapInstance.showLoading({ color: '#1867C0', maskColor: 'rgba(255, 255, 255, 0.8)' });

  try {
    const response = await fetch('https://cdn.jsdmirror.com/npm/echarts@4.9.0/map/json/china.json');
    const chinaJson = await response.json();

    mapInstance.hideLoading();
    echarts.registerMap('china', chinaJson);

    const option = {
      backgroundColor: '#fff', //纯白背景融合卡片
      tooltip: {
        trigger: 'item',
        backgroundColor: 'rgba(0,0,0,0.7)',
        textStyle: { color: '#fff' },
        formatter: params => `${params.name}<br/>延迟: ${params.value || '--'} ms`
      },
      geo: {
        map: 'china',
        roam: false,
        zoom: 1.2,
        itemStyle: {
          areaColor: '#f3f4f6', //默认浅灰
          borderColor: '#cfd8dc', //浅蓝灰边框
          borderWidth: 1
        },
        emphasis: {
          label: { show: false },
          itemStyle: { areaColor: '#bbdefb' } //悬停浅蓝
        }
      },
      series: [
        {
          name: '延迟数据',
          type: 'map',
          geoIndex: 0,
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
const handleResize = () => mapInstance?.resize();
onMounted(() => {
  nextTick(() => {
    initMap();
    window.addEventListener('resize', handleResize);
  });
});
onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
  mapInstance?.dispose();
});
</script>

<style scoped>
.china-map-container {
  width: 100%;
  height: 450px;
}
:deep(.v-field--variant-outlined .v-field__outline__start) {
  border-top-left-radius: 8px;
  border-bottom-left-radius: 8px;
}
:deep(.v-field--variant-outlined .v-field__outline__end) {
  border-top-right-radius: 8px;
  border-bottom-right-radius: 8px;
}
</style>
