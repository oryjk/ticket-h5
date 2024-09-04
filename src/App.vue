<script setup>
import { ref } from 'vue'
import axios from 'axios';
const opposingName = ref("山东泰山")

const color = ref("red")
const opposingColor = ref("blue")

const boxCount = ref(6)

const ticketInfos = ref([
  { "name": "价位一", "child": [{ "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }] },
  { "name": "价位二", "child": [{ "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }] },
  { "name": "价位三", "child": [{ "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }, { "name": "101", "count": 10 }] }
])

let groupedTicketInfos = ref([])
let ticketInfoMap = ref(new Map())
let ticketInfosWithLevel = ref([])
let levels = ref([])

const serverHost = 'https://oryjk.cn:82'

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
      <div :class="color">
        <h2>成都</h2>
      </div>
      <div>
        <h2>VS</h2>
      </div>
      <div :class="opposingColor">
        <h2>{{ opposingName }}</h2>
      </div>
    </div>
    <div class="content">
      <div class="block-content" v-for="block in ticketInfosWithLevel" :key="block.level">
        <nut-divider :style="{ color: '#1989fa', borderColor: '#1989fa', padding: '0 16px' }"> 价格区域{{ block.level }}{{
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
      <div :class="color">
        <h2>成都</h2>
      </div>
      <div>
        <h2>VS</h2>
      </div>
      <div :class="opposingColor">
        <h2>{{ opposingName }}</h2>
      </div>
    </div>
  </div>
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
      // z-index: 1000;
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
      div {
        flex: 1 1 30%;
        // margin: 10px;
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
  }
}

/* 响应式布局 */
@media (max-width: 768px) {
  .content .box {
    flex: 1 1 100%;
  }
}
</style>
