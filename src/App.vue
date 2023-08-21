<script setup>
import { ref, onMounted, watchEffect} from 'vue'
import Chart from 'chart.js/auto';
import axios from 'axios'

// dom
const ctx1 = ref(null)
const donut1 = ref(null)
const donut2 = ref(null)
const donut3 = ref(null)
const donut4 = ref(null)
const donutWrap1 = ref(null)
const donutWrap2 = ref(null)
const donutWrap3 = ref(null)
const donutWrap4 = ref(null)
// chart
let chart1
let donutChart1
let donutChart2
let donutChart3
let donutChart4
// 綁在圖表顯示的資料
const chartData = ref({
  birth_total: [],
  death_total: [],
  divorce_total: [],
  marry_total: [],
})
const donutChart1Data = ref([])
const donutChart2Data = ref([])
const donutChart3Data = ref([])
const donutChart4Data = ref([])
// 將api取得的資料改成想要的資料結構
const zone = ['松山區','信義區','大安區','中山區','中正區','大同區','萬華區','文山區','南港區','內湖區','士林區','北投區']
const apiData = {}
function process(url, res){
  apiData[url]=[]
  for(let i=0; i<zone.length; i++){
    let obj = {
      site_id: zone[i],
      birth_total: 0,
      birth_total_f: 0,
      birth_total_m: 0,
      death_f: 0,
      death_m: 0,
      death_total: 0,
      district_code: 0,
      divorce_pair_OppositeSex: 0,
      divorce_pair_SameSex: 0,
      divorce_pair_SameSex_f: 0,
      divorce_pair_SameSex_m: 0,
      marry_pair_OppositeSex: 0,
      marry_pair_SameSex: 0,
      marry_pair_SameSex_f: 0,
      marry_pair_SameSex_m: 0,
    }
    res.data.responseData.filter(d=>d.site_id.includes(zone[i])).forEach(i=>{ //把每個里依照行政區累加起來
      obj.birth_total+=Number(i.birth_total)
      obj.birth_total_f+=Number(i.birth_total_f)
      obj.birth_total_m+=Number(i.birth_total_m)
      obj.death_f+=Number(i.death_f)
      obj.death_m+=Number(i.death_m)
      obj.death_total+=Number(i.death_total)
      obj.district_code+=Number(i.district_code)
      obj.divorce_pair_OppositeSex+=Number(i.divorce_pair_OppositeSex)
      obj.divorce_pair_SameSex+=Number(i.divorce_pair_SameSex)
      obj.divorce_pair_SameSex_f+=Number(i.divorce_pair_SameSex_f)
      obj.divorce_pair_SameSex_m+=Number(i.divorce_pair_SameSex_m)
      obj.marry_pair_OppositeSex+=Number(i.marry_pair_OppositeSex)
      obj.marry_pair_SameSex+=Number(i.marry_pair_SameSex)
      obj.marry_pair_SameSex_f+=Number(i.marry_pair_SameSex_f)
      obj.marry_pair_SameSex_m+=Number(i.marry_pair_SameSex_m)
    })
    apiData[url].push(obj)
  }
  let obj = {
    site_id: '臺北市總計',
    birth_total: apiData[url].reduce((total,item)=>total+item.birth_total,0),
    birth_total_f: apiData[url].reduce((total,item)=>total+item.birth_total_f,0),
    birth_total_m: apiData[url].reduce((total,item)=>total+item.birth_total_m,0),
    death_f: apiData[url].reduce((total,item)=>total+item.death_f,0),
    death_m: apiData[url].reduce((total,item)=>total+item.death_m,0),
    death_total: apiData[url].reduce((total,item)=>total+item.death_total,0),
    district_code: apiData[url].reduce((total,item)=>total+item.district_code,0),
    divorce_pair_OppositeSex: apiData[url].reduce((total,item)=>total+item.divorce_pair_OppositeSex,0),
    divorce_pair_SameSex: apiData[url].reduce((total,item)=>total+item.divorce_pair_SameSex,0),
    divorce_pair_SameSex_f: apiData[url].reduce((total,item)=>total+item.divorce_pair_SameSex_f,0),
    divorce_pair_SameSex_m: apiData[url].reduce((total,item)=>total+item.divorce_pair_SameSex_m,0),
    marry_pair_OppositeSex: apiData[url].reduce((total,item)=>total+item.marry_pair_OppositeSex,0),
    marry_pair_SameSex: apiData[url].reduce((total,item)=>total+item.marry_pair_SameSex,0),
    marry_pair_SameSex_f: apiData[url].reduce((total,item)=>total+item.marry_pair_SameSex_f,0),
    marry_pair_SameSex_m: apiData[url].reduce((total,item)=>total+item.marry_pair_SameSex_m,0),
  }
  apiData[url].push(obj)
}
// 打多個API
const api_base = 'https://www.ris.gov.tw/rs-opendata/api/v1/datastore/ODRP060/'
const api_Url = ['11201', '11202', '11203', '11204', '11205', '11206', '11207']
async function api0 () {
  const res = await axios.get(api_base+api_Url[0]);
  process(api_Url[0], res)
}
async function api1 () {
  const res = await axios.get(api_base+api_Url[1]);
  process(api_Url[1], res)
}
async function api2 () {
  const res = await axios.get(api_base+api_Url[2]);
  process(api_Url[2], res)
}
async function api3 () {
  const res = await axios.get(api_base+api_Url[3]);
  process(api_Url[3], res)
}
async function api4 () {
  const res = await axios.get(api_base+api_Url[4]);
  process(api_Url[4], res)
}
async function api5 () {
  const res = await axios.get(api_base+api_Url[5]);
  process(api_Url[5], res)
}
async function api6 () {
  const res = await axios.get(api_base+api_Url[6]);
  process(api_Url[6], res)
}
// 當selectedDistrict.value或 selectedMonth更新時 把每個月的資料放到各個donutChart裡面並更新甜甜圈圖表
const selectedDistrict = ref('臺北市總計') // 會因template click事件改變
const selectedMonth = ref('11201') // 會因template click事件改變
function setDonutChart(){
  donutChart1Data.value=[]
  donutChart1Data.value.push(apiData[selectedMonth.value].find(i=>i.site_id===selectedDistrict.value).marry_pair_OppositeSex)
  donutChart1Data.value.push(apiData[selectedMonth.value].find(i=>i.site_id===selectedDistrict.value).marry_pair_SameSex_m)
  donutChart1Data.value.push(apiData[selectedMonth.value].find(i=>i.site_id===selectedDistrict.value).marry_pair_SameSex_f)
  donutChart1.data.datasets[0].data = donutChart1Data.value
  donutChart1.update()

  donutChart2Data.value=[]
  donutChart2Data.value.push(apiData[selectedMonth.value].find(i=>i.site_id===selectedDistrict.value).divorce_pair_OppositeSex)
  donutChart2Data.value.push(apiData[selectedMonth.value].find(i=>i.site_id===selectedDistrict.value).divorce_pair_SameSex_m)
  donutChart2Data.value.push(apiData[selectedMonth.value].find(i=>i.site_id===selectedDistrict.value).divorce_pair_SameSex_f)
  donutChart2.data.datasets[0].data = donutChart2Data.value
  donutChart2.update()

  donutChart3Data.value=[]
  donutChart3Data.value.push(apiData[selectedMonth.value].find(i=>i.site_id===selectedDistrict.value).birth_total_m)
  donutChart3Data.value.push(apiData[selectedMonth.value].find(i=>i.site_id===selectedDistrict.value).birth_total_f)
  donutChart3.data.datasets[0].data = donutChart3Data.value
  donutChart3.update()

  donutChart4Data.value=[]
  donutChart4Data.value.push(apiData[selectedMonth.value].find(i=>i.site_id===selectedDistrict.value).death_m)
  donutChart4Data.value.push(apiData[selectedMonth.value].find(i=>i.site_id===selectedDistrict.value).death_f)
  donutChart4.data.datasets[0].data = donutChart4Data.value
  donutChart4.update()
}
// 當selectedDistrict.value更新時 把每個月的資料放到chartData裡面
function setChart(){ 
  // 更新資料
  chartData.value.marry_total=[]
  chartData.value.divorce_total=[]
  chartData.value.birth_total=[]
  chartData.value.death_total=[]
  for(let i=0; i<api_Url.length; i++){
    chartData.value.marry_total.push(apiData[api_Url[i]].find(i=>i.site_id===selectedDistrict.value).marry_pair_OppositeSex + apiData[api_Url[i]].find(i=>i.site_id===selectedDistrict.value).marry_pair_SameSex)
    chartData.value.divorce_total.push(apiData[api_Url[i]].find(i=>i.site_id===selectedDistrict.value).divorce_pair_OppositeSex + apiData[api_Url[i]].find(i=>i.site_id===selectedDistrict.value).divorce_pair_SameSex)
    chartData.value.birth_total.push(apiData[api_Url[i]].find(i=>i.site_id===selectedDistrict.value).birth_total)
    chartData.value.death_total.push(apiData[api_Url[i]].find(i=>i.site_id===selectedDistrict.value).death_total)
  }
  // 更新圖表
  chart1.data.datasets[0].data = chartData.value.marry_total
  chart1.data.datasets[1].data = chartData.value.divorce_total
  chart1.data.datasets[2].data = chartData.value.birth_total
  chart1.data.datasets[3].data = chartData.value.death_total
  chart1.options.plugins.title.text = selectedDistrict.value
  chart1.update();
  setDonutChart()
}

async function callMutiApi () {
  // console.time();
  const promises = [api0(), api1(), api2(), api3(), api4(), api5(), api6()]; //打API
  Promise.allSettled(promises).then(values => {
    // console.log(values);
    // console.timeEnd();
    // console.log(apiData);
    setChart()
    setDonutChart()
  });
}
callMutiApi();

// template
const buttons = ['臺北市總計','松山區','信義區','大安區','中山區','中正區','大同區','萬華區','文山區','南港區','內湖區','士林區','北投區']
function monthTrans(month){
  let mon = month.slice(3,5)
  if(mon==='01')return '一月'
  else if(mon==='02')return '二月'
  else if(mon==='03')return '三月'
  else if(mon==='04')return '四月'
  else if(mon==='05')return '五月'
  else if(mon==='06')return '六月'
  else if(mon==='07')return '七月'
  else if(mon==='08')return '八月'
  else if(mon==='09')return '九月'
  else if(mon==='10')return '十月'
  else if(mon==='11')return '十一月'
  else if(mon==='12')return '十二月'
}

onMounted(()=>{
  chart1 = new Chart(ctx1.value, {
    // type: 'bar',
    data: {
      datasets: [
        {
          type: 'bar',
          label: '結婚對數',
          data: chartData.value.marry_total,
          borderColor: '#FA5A4B',
          backgroundColor: '#DB4C40',
        },{
          type: 'bar',
          label: '離婚對數',
          data: chartData.value.divorce_total,
          borderColor: '#3499C4',
          backgroundColor: '#2E86AB',
        }, {
          type: 'line',
          label: '出生數',
          data: chartData.value.birth_total,
          borderColor: '#79C99E',
          backgroundColor: '#79C99E',
      }, {
          type: 'line',
          label: '死亡數',
          data: chartData.value.death_total,
          borderColor: '#564946',
          backgroundColor: '#605B56',
      }],
      labels: ['一月', '二月', '三月', '四月', '五月', '六月', '七月']
    },
    options: {
      scales: {
        y: {
          beginAtZero: true,
          // min: 0,
          // max: 50
        }
      },
      plugins: {
        title: {
          display: true,
          text: selectedDistrict.value,
        }
      }
    }
  }); 
  
  donutChart1 = new Chart(donut1.value, {
    type: 'doughnut',
    data: {
      labels: ['結婚對數_異性', '結婚對數_同性_男', '結婚對數_同性_女'],
      datasets: [
        {
          // label: '甜甜圈',
          data: donutChart1Data.value,
          borderWidth: 0,
          backgroundColor: [
            'rgb(255, 99, 132)',
            'rgb(54, 162, 235)',
            'rgb(255, 205, 86)',
          ],
        }
      ]
    },
    options: {
      responsive: true,
      plugins: {
        legend: {
          position: 'bottom',
        },
        title: {
          display: true,
          text: '結婚對數結構分布',
        }
      }
    },
  });

  donutChart2 = new Chart(donut2.value, {
    type: 'doughnut',
    data: {
      labels: ['離婚對數_異性', '離婚對數_同性_男', '離婚對數_同性_女'],
      datasets: [
        {
          data: donutChart2Data.value,
          borderWidth: 0,
          backgroundColor: [
            '#E1B07E',
            '#AF929D',
            '#D2D6EF',
          ],
        }
      ]
    },
    options: {
      responsive: true,
      plugins: {
        legend: {
          position: 'bottom',
        },
        title: {
          display: true,
          text: '離婚對數結構分布',
        }
      }
    },
  });
  
  donutChart3 = new Chart(donut3.value, {
    type: 'doughnut',
    data: {
      labels: ['出生數_男', '出生數_女'],
      datasets: [
        {
          data: donutChart3Data.value,
          borderWidth: 0,
          backgroundColor: [
            'rgb(54, 162, 235)',
            'rgb(255, 99, 132)',
          ],
        }
      ]
    },
    options: {
      responsive: true,
      plugins: {
        legend: {
          position: 'bottom',
        },
        title: {
          display: true,
          text: '出生數結構分布',
        }
      }
    },
  });

  donutChart4 = new Chart(donut4.value, {
    type: 'doughnut',
    data: {
      labels: ['死亡數_男', '死亡數_女'],
      datasets: [
        {
          data: donutChart4Data.value,
          borderWidth: 0,
          backgroundColor: [
            'rgb(54, 162, 235)',
            'rgb(255, 99, 132)',
          ],
        }
      ]
    },
    options: {
      responsive: true,
      plugins: {
        legend: {
          position: 'bottom',
        },
        title: {
          display: true,
          text: '死亡數結構分布',
        }
      }
    },
  });


  function RWD(){
    if (window.innerWidth < 768) {
      donutWrap1.value.style.width =
      donutWrap1.value.style.height =
      donutWrap2.value.style.width =
      donutWrap2.value.style.height =
      donutWrap3.value.style.width =
      donutWrap3.value.style.height =
      donutWrap4.value.style.width =
      donutWrap4.value.style.height = '60vw' 
    }else{
      donutWrap1.value.style.width =
      donutWrap1.value.style.height =
      donutWrap2.value.style.width =
      donutWrap2.value.style.height =
      donutWrap3.value.style.width =
      donutWrap3.value.style.height =
      donutWrap4.value.style.width =
      donutWrap4.value.style.height = '15vw'
    }
  }
  RWD()
  window.addEventListener("resize", RWD);
})
</script>

<template>
<h1>112年臺北市出生死亡人口統計</h1>
<div class="wrap">
  <div class="buttons">
    <button v-for="button in buttons" @click="selectedDistrict=button; setChart()" :class="{active:selectedDistrict===button}" :key="button">{{ button }}</button>
  </div>
  <div class="chart" style="position: relative; height:40vh; width:90vw">
    <canvas ref="ctx1" id="ctx1"></canvas>
  </div>
  <div class="buttons">
    <button v-for="month in api_Url" @click="selectedMonth=month; setDonutChart()" :class="{active:selectedMonth===month}" :key="month">{{ monthTrans(month) }}</button>
  </div>
  <div class="donutWrap">
    <div class="chart" ref="donutWrap1" style="position: relative; height:15vw; width:15vw">
      <canvas ref="donut1" id="donut1"></canvas>
    </div>
    <div class="chart" ref="donutWrap2" style="position: relative; height:15vw; width:15vw">
      <canvas ref="donut2" id="donut2"></canvas>
    </div>
    <div class="chart" ref="donutWrap3" style="position: relative; height:15vw; width:15vw">
      <canvas ref="donut3" id="donut3"></canvas>
    </div>
    <div class="chart" ref="donutWrap4" style="position: relative; height:15vw; width:15vw">
      <canvas ref="donut4" id="donut4"></canvas>
    </div>
  </div>
  <div class="info">資料來源：<a href="https://data.gov.tw/dataset/131138">政府資料開放平台-各村（里）人口統計月報表（含同婚）</a></div>
</div>
</template>

<style scoped lang="scss">
@mixin mobile{
  @media(max-width: 768px){
    @content;
  }
}
@mixin flex_center{
  display: flex;
  justify-content: center;
  align-items: center;
}
h1{
  margin-bottom: 10px;
  font-size: 36px;
  @include mobile{
    font-size: 6vw;
  }
}
.wrap{
  width: 100%;
  @include flex_center;
  flex-direction: column;
  .info{
    // margin-top: 10px;
    width: 100%;
    text-align: end;
  }
}
.buttons{
  width: 100%;
  padding: 0 5px;
  margin-bottom: 10px;
  @include flex_center;
  flex-wrap: nowrap;
  overflow-x: auto;
  @include mobile{
    justify-content: start;
  }
  button{
    flex-shrink: 0;
    padding: 15px 20px;
    margin: 10px;
    background: #181818;
    border: 1px solid rgba(203, 203, 203, 0.708) ; 
    color: aquamarine;
    cursor: pointer;
    &.active{
      box-shadow: 0 0 2px 2px aquamarine;
    }
  }
}
.donutWrap{
  @include flex_center;
  @include mobile{
    flex-direction: column;
  }
}
.chart{
  @include flex_center;
}
</style>