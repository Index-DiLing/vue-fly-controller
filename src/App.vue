<script setup>

import LineChart from "./components/LineChart.vue";
import {reactive, ref} from "vue";

let logs = reactive(["Terminal"])

let pitch = reactive([1])
let yaw = reactive([1])
let roll = reactive([1])
let pressure = reactive([1])
let motorValue = reactive( [ reactive( [1]),reactive( [1]),reactive( [1]),reactive( [1])]);

let status = reactive({})

const source = new EventSource("http://localhost:8080/serial/data");

source.onopen = function () {
  logs.push("成功连接SSE")
}

source.onmessage = function (event) {
  let src = JSON.parse(event.data);
  src.forEach(function (item) {

    if (item.type === "STATUS") {
      console.log(item);

      for (let i in item){
        status[i] = item[i];
      }
      pitch.push(item.pitch);
      roll.push(item.roll);
      yaw.push(item.yaw);
      pressure.push(item.pressure);

      motorValue[0].push(item.motorValue[0]);
      motorValue[1].push(item.motorValue[1]);
      motorValue[2].push(item.motorValue[2]);
      motorValue[3].push(item.motorValue[3]);
    }
    if (item.type === "LOG"){
      logs.push(item.message);
    }
  });
  const len = 1000;
  if (pitch.length > len) {
    pitch.splice(0,pitch.length-len)

    yaw.splice(0,yaw.length-len)

    roll.splice(0,roll.length-len)
    pressure.splice(0,pressure.length-len)

    motorValue[0].splice(0,motorValue[0].length-len);

    motorValue[1].splice(0,motorValue[1].length-len);

    motorValue[2].splice(0,motorValue[2].length-len);

    motorValue[3].splice(0,motorValue[3].length-len);

  }
}

function test() {
  // for (let i = 0; i < 1000; i++) {
  //   pitch.push(Math.random() * 10);
  // }
  status.a = true;
}
function wait(){
  fetch("http://localhost:8080/sync").then(res=>{
      res.json().then(data=>{
        logs.push("解锁状态:"+data)
      })
  });
}


</script>


<template>
  <div class="banner">飞控地面站</div>
  <div class="content">
    <div class="chart-area">
      <line-chart :data=pitch title="Pitch" :max=180 :min=-180 :interval=30></line-chart>
      <line-chart :data="roll"  title="Roll" :max=180 :min=-180 :interval=30></line-chart>
      <line-chart :data="yaw"  title="Yaw" :max=180 :min=-180 :interval=30></line-chart>
      <line-chart :data="pressure"  title="压力" :max=102000 :min=101000
                  :interval=100></line-chart>

      <line-chart :data=motorValue title="四电机油门值" :max=2000 :min=0
                  :interval=500></line-chart>
    </div>

    <div class="content-right">
      <div class="status-item" v-for="(value,key) in status">{{key}} : {{value}}</div>
    </div>

  </div>

  <div class="under">
    <div class="terminal">
      <div class="log-area">
        <div v-for="(item,key) in logs" :key="key" class="output-item">{{ item }}</div>
      </div>
      <div class="input-area">
        <input class="terminal-input">
        <span class="terminal-buttons">
          <button @click="test">测试</button>
          <button @click="wait">同步/解锁</button>
        </span>
      </div>
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
  width: 64vw;
  height: 48vh;
  display: flex;
  flex-wrap: wrap;
  overflow-y: scroll;
}
.content-right {
  width: 36vw;
  height: 48vh;
  overflow-y: scroll;
}

.content {
  width: 100vw;
  height: 48vh;
  display: flex;
}

.log-area {
  background-color: #3b3a3a;
  height: 85%;
  overflow: scroll;
}

.output-item {
  color: white;
}

.terminal-buttons{

}
.input-area{
  height: 15%;
}

.terminal-input{
  width: 60%;
}

.terminal{
  width: 64vw;
  height: 52vh;
}
</style>
