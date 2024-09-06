<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios';

onMounted(() => {
  getCurrentMatchInfo()
  getBlockInfo()
})

const matchInfo = ref({})
const ticketInfos = ref([
  { "name": "价位一", "child": [{ "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }] },
  { "name": "价位二", "child": [{ "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }] },
  { "name": "价位三", "child": [{ "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }] }
])

const ticketTotalCount = ref(0)
let groupedTicketInfos = ref([])
let ticketInfoMap = ref(new Map())
let ticketInfosWithLevel = ref([])
let levels = ref([])
const timerId = ref()
const currentTime = ref(new Date().toLocaleTimeString())
const listenBtn = ref({
  listenStatus: false,
  startStyle: 'inline-block',
  stopStyle: 'none'
})
const listenLoad = ref(false)
const serverHost = 'https://oryjk.cn:82'

function listenBtnClick() {

  if (!listenBtn.value.listenStatus) {
    listenLoad.value = true
    listenBtn.value.listenStatus = true
    listenBtn.value.stopStyle = 'inline-block'
    listenBtn.value.startStyle = 'none'
    setTimeout(() => {
      listenLoad.value = false
    }, 1000)
    timerId.value = setInterval(() => {
      const now = new Date();
      currentTime.value = now.toLocaleTimeString();
      fetchRealtimeBlockInfo()
    }, 300)
    // listenLoad.value=false
  } else {
    listenBtn.value.listenStatus = false
    listenBtn.value.startStyle = 'inline-block'
    listenBtn.value.stopStyle = 'none'
    clearInterval(timerId.value);
  }
}

function fetchRealtimeBlockInfo() {
  getData('/api/general-ticket/blockInfo/' + matchInfo.value.matchId).then(res => {
    res.forEach(item => {
      let ticketInfo = ticketInfoMap.get(item.name);
      ticketInfo.occurrences = item.occurrences
      if (item.occurrences > 0) {
        ticketInfo.hasOccur = 'hasOccur'
        ticketTotalCount.value += item.occurrences
      }
    })
  })
}


function getBlockInfo() {
  getData('/api/general-ticket/ticketInfo').then(res => {
    ticketInfos.value = []
    res.forEach(item => {
      item.sites.forEach(site => {

        ticketInfos.value.push({
          name: site,
          level: item.level,
          tower: item.tower,
          inventory: 0,
          occurrences: 0,
          hasOccur: ''
        })
        groupedTicketInfos.value = ticketInfos.value.reduce(function (result, ticketInfo) {
          let level = ticketInfo.level.level;
          if (!result[level]) {
            result[level] = [];
          }
          result[level].push(ticketInfo);
          return result;
        }, {})



      })
    })
  }).then(() => {
    ticketInfoMap.value = ticketInfos.value.reduce(function (map, ticketInfo) {
      let name = ticketInfo.name;
      map.set(name, ticketInfo)
      return map;
    }, new Map())

    levels = [...new Set(ticketInfos.value.map(ticketInfo => {
      return { level: ticketInfo.level.level, price: ticketInfo.level.price };
    }))]
      .filter(function (value, index, self) {
        return index === self.findIndex(function (obj) {
          return obj.level === value.level && obj.price === value.price;
        });
      })
      .sort(function (a, b) {
        return a.price - b.price;
      }).map(function (level) {
        return level;
      });

    levels.forEach(level => {
      let levelStr = level.level
      let price = level.price

      let ticketInfos = groupedTicketInfos.value[levelStr];
      // ticketInfos.forEach(ticketInfo => {
      //   ticketInfosWithLevel.value.push(ticketInfo)
      // })

      ticketInfosWithLevel.value.push({ level: levelStr, price: price, ticketInfos: ticketInfos })
    })
    console.log(ticketInfosWithLevel)
  }
  )
}

function getCurrentMatchInfo() {
  getData('/api/schedule/list').then(res => {
    const currentMatchData = res[res.length - 1];
    console.log(currentMatchData)
    matchInfo.value.homeName = currentMatchData.homeName
    matchInfo.value.opposingName = currentMatchData.awayName
    matchInfo.value.opposingColor = currentMatchData.awayName == '北京国安' ? 'green' : 'blue'
    matchInfo.value.color = 'red'
    matchInfo.value.round = currentMatchData.round
    matchInfo.value.competitionDate = formattedDate(currentMatchData.date)
    matchInfo.value.competitionHour = currentMatchData.hour
    matchInfo.value.competitionMinute = currentMatchData.minute
    matchInfo.value.matchId = currentMatchData.matchId
  })
}

function formattedDate(date) {
  const dateObj = new Date(date);
  const year = dateObj.getFullYear();
  const month = dateObj.getMonth() + 1;
  const day = dateObj.getDate();
  return `${year}年${month}月${day}日`;
}


async function getData(url) {
  try {
    const response = await axios.get(serverHost + url);
    return response.data;
  } catch (error) {
    console.error(error);
  }
}
</script>

<template>
  <div class="container">
    <div class="header">
      <div :class="matchInfo.color">
        <h2>{{ matchInfo.homeName }}</h2>
      </div>
      <div>
        <h2>VS</h2>
      </div>
      <div :class="matchInfo.opposingColor">
        <h2>{{ matchInfo.opposingName }}</h2>
      </div>
    </div>
    <div class="content">
      <div class="block-content" v-for="block in ticketInfosWithLevel" :key="block.level">
        <nut-divider :style="{ color: '#1989fa', borderColor: '#1989fa', padding: '0 16px' }"> {{ block.level }}价格区域{{
          block.price }}
        </nut-divider>
        <div class="block">
          <div class="box" v-for="child in block.ticketInfos" :key="child">
            <h2>{{ child.name }}</h2>
            <p>出现余票{{ child.inventory }}次</p>
            <p>{{ child.tower.value }}</p>
          </div>
        </div>
      </div>
    </div>

    <div class="footer">
      <div class="current-time">
        <h2>当前时间：{{ currentTime }}</h2>
      </div>
      <div class="listen-btn">
        <div :class="listenBtn.startStyle">
          <nut-button type="primary" v-on:click="listenBtnClick">开始监控</nut-button>
        </div>
        <div :class="listenBtn.stopStyle">
          <nut-button plain type="primary" v-on:click="listenBtnClick">停止监控</nut-button>
        </div>
      </div>
    </div>
    <nut-overlay v-model:visible="listenLoad" :close-on-click-overlay="false">
      <div class="overlay-body">
        <div class="overlay-content" @click.stop="listenLoad = false">开始同步数据</div>
      </div>
    </nut-overlay>
  </div>

  <!-- <nut-popup v-model:visible="listenLoad" :style="{ padding: '30px 50px' }"> 加载中…… </nut-popup> -->
</template>

<style scoped lang="scss">
body {
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;

  .container {
    max-width: 1200px;
    margin: 0 auto;

    .header {
      background-color: #333;
      color: #fff;
      // padding: 20px;
      text-align: center;
      display: flex;
      justify-content: center;
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      z-index: 1000;
      height: 80px;


      div {
        flex: 1 1 30%;
        // margin: 10px;
      }

      .red {
        background-color: red;
      }

      .blue {
        background-color: blue;
      }

      .green {
        background-color: green;
      }
    }

    .footer {
      background-color: #f4f4f4;
      display: flex;
      text-align: center;
      justify-content: center;
      position: fixed;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 70px;
      z-index: 1000;

      div {
        display: flex;

        align-items: center;
        justify-content: center;
        // margin: 10px;
      }

      .current-time {
        flex: 1 1 70%;
      }

      .listen-btn {
        flex: 1 1 30%;

        .inline-block {
          display: inline-block;
        }

        .none {
          display: none;
        }
      }

    }



    .content {
      margin-top: 100px;
      margin-bottom: 80px;

      .block {
        display: flex;
        flex-wrap: wrap;
        justify-content: space-between;

        .box {
          flex: 1 1 20%;
          margin: 1px;
          background-color: #f4f4f4;
          text-align: center;
          border-radius: 20px;

        }
      }

    }

    .overlay-body {
      display: flex;
      height: 100%;
      align-items: center;
      justify-content: center;

      .overlay-content {
        display: flex;
        width: 150px;
        height: 150px;
        background: #fff;
        border-radius: 8px;
        align-items: center;
        justify-content: center;
        color: red;
      }
    }


  }
}

/* 响应式布局 */
@media (max-width: 768px) {
  .content .box {
    flex: 1 1 100%;
  }
}
</style>
