<script setup>

import LineChart from "./components/LineChart.vue";
import {reactive, ref} from "vue";

let logs = reactive(["日志位置"])

let pitch = reactive([1])
let yaw = reactive([1])
let roll = reactive([1])
let pressure = reactive([1])
let motorValue = reactive( [ reactive( [1]),reactive( [1]),reactive( [1]),reactive( [1])]);

let curPitch = reactive([1])
let curYaw = reactive([1])
let curRoll = reactive([1])
let curPressure = reactive([1])
let curMotorValue = reactive([[1, 2, 3, 4]])

let temperature = ref(1)

const source = new EventSource("http://localhost:8080/serial/data");

source.onopen = function () {
  logs.push("成功连接SSE")
}

source.onmessage = function (event) {
  let src = JSON.parse(event.data);
  src.forEach(function (item) {
    if (item.type === "STATUS") {
      curRoll[0] = item.roll;
      curPitch[0] = item.pitch;
      curYaw[0] = item.yaw;
      curPressure[0] = item.pressure;
      // curMotorValue[0] = src.motorValue;
      temperature.value = item.temperature;
      pitch.push(item.pitch);
      roll.push(item.roll);
      yaw.push(item.yaw);
      pressure.push(item.pressure);

      motorValue[0].push(item.motorValue[0]);
      motorValue[1].push(item.motorValue[1]);
      motorValue[2].push(item.motorValue[2]);
      motorValue[3].push(item.motorValue[3]);
    }
  });
  const len = 1000;
  if (pitch.length > len) {
    pitch = pitch.slice(-len);
    roll = roll.slice(-len);
    yaw = yaw.slice(-len);
    pressure = pressure.slice(-len);



    motorValue[0].splice(0,motorValue[0].length-1000);

    motorValue[1].splice(0,motorValue[1].length-1000);

    motorValue[2].splice(0,motorValue[2].length-1000);

    motorValue[3].splice(0,motorValue[3].length-1000);

  }


}

function test() {
  for (let i = 0; i < 1000; i++) {
    pitch.push(Math.random() * 10);
  }
}


</script>


<template>
  <div class="banner">标题五个字</div>

  <div class="content">

    <div class="chart-area">
      <line-chart :data=pitch :subtitle="curPitch" title="Pitch" :max=180 :min=-180 :interval=30></line-chart>
      <line-chart :data="roll" :subtitle="curRoll" title="Roll" :max=180 :min=-180 :interval=30></line-chart>
      <line-chart :data="yaw" :subtitle="curYaw" title="Yaw" :max=180 :min=-180 :interval=30></line-chart>
      <line-chart :data="pressure" :subtitle="curPressure" title="压力" :max=102000 :min=101000
                  :interval=100></line-chart>

      <line-chart :data=motorValue :subtitle="curMotorValue" title="四电机油门值" :max=2000 :min=0
                  :interval=500></line-chart>
    </div>
    <div class="content-right">
      <div class="log-area">
        <div v-for="(item,key) in logs" :key="key" class="output-item">{{ item }}</div>
      </div>
    </div>
  </div>

  <div class="under">
    <div class="input-area">

      <button @click="test">测试</button>
    </div>

    <div class="output-area">
      <span>温度:{{ temperature }}℃</span>

    </div>
  </div>

</template>


<style scoped>

.under {
  width: 100%;
  height: 10vh;
}

.banner {
  width: 100%;
  height: 16%;
  text-align: center;
  font-weight: bold;

  letter-spacing: 2em;

}

.chart-area {
  width: 80vw;
  height: 60vh;
  display: flex;
  flex-wrap: wrap;
  overflow-y: scroll;
}

.content-right {
  width: 20vw;
  height: 60vh;

  overflow-y: scroll;
}

.content {
  width: 100vw;
  height: 60vh;
  display: flex;
}

.log-area {
  background-color: black;
}

.output-item {
  color: white;
}
</style>
