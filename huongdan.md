Ok 👍 mình sẽ viết cho bạn **một file hướng dẫn đầy đủ dạng Markdown (.md)** từ lúc setup Node.js/NPM, tạo project Vue, cài ECharts, rồi thêm thanh công cụ vẽ bên trái giống TradingView.

---

# 📘 Hướng Dẫn Tạo Candlestick Chart Giống TradingView Bằng Vue + ECharts

## 1️⃣ Cài đặt môi trường Node.js và NPM

Trước tiên bạn cần Node.js và npm (package manager).

* Tải về tại [Node.js](https://nodejs.org/)
* Sau khi cài xong, kiểm tra:

```bash
node -v
npm -v
```

Ví dụ output:

```
v20.11.1
10.5.0
```

---

## 2️⃣ Tạo project Vue 3

Chạy lệnh:

```bash
npm init vue@latest my-vue-chart
```

Chọn các option theo nhu cầu (TypeScript / Router / Pinia tuỳ bạn).

Sau đó:

```bash
cd my-vue-chart
npm install
npm run dev
```

Mở [http://localhost:5173](http://localhost:5173) để xem app chạy.

---

## 3️⃣ Cài ECharts

Cài thêm thư viện ECharts:

```bash
npm install echarts
```

---

## 4️⃣ Tạo component CandleChart.vue

Tạo file `src/components/CandleChart.vue`:

```vue
<template>
  <div class="p-4 bg-gray-900 text-white">
    <!-- Thanh chọn coin -->
    <div class="p-2 bg-gray-800 text-white flex items-center space-x-2">
      <label>Chọn Coin:</label>
      <select v-model="selectedCoin" @change="renderChart" class="bg-gray-900 p-1 rounded">
        <option value="BTCUSDT">BTC/USDT</option>
        <option value="ETHUSDT">ETH/USDT</option>
        <option value="BNBUSDT">BNB/USDT</option>
      </select>
    </div>

    <!-- Thanh chọn khung thời gian -->
    <div class="flex gap-2 mb-3">
      <button
        v-for="tf in timeframes"
        :key="tf"
        @click="changeInterval(tf)"
        class="px-3 py-1 rounded border"
        :class="interval === tf ? 'bg-blue-600 border-blue-600' : 'bg-gray-700 border-gray-600'"
      >
        {{ tf }}
      </button>
    </div>

    <!-- Chart -->
    <div ref="chartRef" style="height: 600px; width: 100%"></div>
  </div>
</template>

<script setup>
import * as echarts from "echarts"
import { onMounted, ref } from "vue"

const chartRef = ref(null)
let chart = null

const selectedCoin = ref("BTCUSDT")   // mặc định BTC
const interval = ref("1h")            // mặc định 1h

// Các khung thời gian
const timeframes = ["15m", "1h", "4h", "1d", "1w"]

// Hàm format thời gian theo interval
function formatTime(ts, interval) {
  const date = new Date(ts)
  if (interval.endsWith("m") || interval.endsWith("h")) {
    return date.toLocaleString("vi-VN", {
      month: "2-digit",
      day: "2-digit",
      hour: "2-digit",
      minute: "2-digit",
    })
  } else {
    return date.toLocaleDateString("vi-VN", {
      month: "2-digit",
      day: "2-digit",
    })
  }
}

// Fetch dữ liệu từ Binance
async function fetchData(symbol, interval) {
  const url = `https://api.binance.com/api/v3/klines?symbol=${symbol}&interval=${interval}&limit=200`
  const res = await fetch(url)
  const raw = await res.json()

  const dates = raw.map(item => formatTime(item[0], interval))
  const values = raw.map(item => [
    parseFloat(item[1]), // open
    parseFloat(item[4]), // close
    parseFloat(item[3]), // low
    parseFloat(item[2]), // high
  ])
  const volumes = raw.map(item => parseFloat(item[5]))

  return { dates, values, volumes }
}

function calculateMA(dayCount, data) {
  let result = []
  for (let i = 0, len = data.length; i < len; i++) {
    if (i < dayCount) {
      result.push('-')
      continue
    }
    let sum = 0
    for (let j = 0; j < dayCount; j++) {
      sum += data[i - j][1] // lấy giá close
    }
    result.push((sum / dayCount).toFixed(2))
  }
  return result
}

function formatNumber(num, decimals = 2) {
  if (num === '-' || num == null) return '-'
  return parseFloat(num)
    .toFixed(decimals)
    .replace(/\B(?=(\d{3})+(?!\d))/g, ",")
}

// Vẽ chart
async function renderChart() {
  const { dates, values, volumes } = await fetchData(selectedCoin.value, interval.value)

  let ma5 = calculateMA(5, values)
  let ma10 = calculateMA(10, values)
  let ma20 = calculateMA(20, values)
  let ma30 = calculateMA(30, values)

  const option = {
    backgroundColor: "#111827",
    animation: false,
    legend: {
      data: ["Candlestick", "MA5", "MA10", "MA20"],
      top: 0,
      left: 0,
      textStyle: { color: "#fff" },
      formatter: function (name) {
        let lastIndex = values.length - 1
        if (name === 'MA5') return `MA5: ${formatNumber(ma5[lastIndex])}`
        if (name === 'MA10') return `MA10: ${formatNumber(ma10[lastIndex])}`
        if (name === 'MA20') return `MA20: ${formatNumber(ma20[lastIndex])}`
        if (name === 'MA30') return `MA30: ${formatNumber(ma30[lastIndex])}`
        return name
      }
    },
    tooltip: { trigger: "axis", axisPointer: { type: "cross" } },
    axisPointer: { link: [{ xAxisIndex: "all" }], label: { backgroundColor: "#777" } },
    grid: [
      { left: "5%", right: "10%", height: "60%" },
      { left: "5%", right: "5%", top: "75%", height: "15%" },
    ],
    xAxis: [
      {
        type: "category",
        data: dates,
        boundaryGap: false,
        axisLine: { lineStyle: { color: "#aaa" } },
        axisLabel: { color: "#ccc" },
        min: "dataMin",
        max: "dataMax",
        splitLine: {                 // 👉 kẻ dọc
            show: true,
            lineStyle: { color: "#333" } 
        }
      },
      {
        type: "category",
        gridIndex: 1,
        data: dates,
        boundaryGap: false,
        axisLine: { lineStyle: { color: "#aaa" } },
        axisLabel: { color: "#ccc" },
        min: "dataMin",
        max: "dataMax",
      },
    ],
    yAxis: [
      { scale: true, position: "right", axisLine: { lineStyle: { color: "#aaa" } }, splitLine: { show: true, lineStyle: { color: "#333"} }, axisLabel: { color: "#ccc" } },
      { scale: true, gridIndex: 1, position: "right", axisLine: { lineStyle: { color: "#aaa" } }, splitLine: { show: false }, axisLabel: { color: "#ccc" } },
    ],
    dataZoom: [
      { type: "inside", xAxisIndex: [0, 1], start: 80, end: 100 },
      { show: true, type: "slider", xAxisIndex: [0, 1], top: "90%", start: 80, end: 100 },
    ],
    series: [
      {
        name: "Candlestick",
        type: "candlestick",
        data: values,
        itemStyle: {
          color: "#26a69a",
          color0: "#ef5350",
          borderColor: "#26a69a",
          borderColor0: "#ef5350",
        },
        markLine: {
          symbol: "none",
          label: { show: true, position: "end", formatter: params => `Giá: ${params.value}`, color: "#fff", fontWeight: "bold" },
          lineStyle: { type: "dashed", width: 1.5 },
          data: [
            {
              yAxis: values[values.length - 1][1],
              lineStyle: { color: values[values.length - 1][1] > values[values.length - 1][0] ? "#26a69a" : "#ef5350" },
            },
          ],
        },
      },
      { name: "MA5", type: "line", data: ma5, smooth: true, lineStyle: { opacity: 0.8 } },
      { name: "MA10", type: "line", data: ma10, smooth: true, lineStyle: { opacity: 0.8 } },
      { name: "MA20", type: "line", data: ma20, smooth: true, lineStyle: { opacity: 0.8 } },
      {
        name: "Volume",
        type: "bar",
        xAxisIndex: 1,
        yAxisIndex: 1,
        data: volumes,
        itemStyle: {
          color: params => (values[params.dataIndex][1] > values[params.dataIndex][0] ? "#26a69a" : "#ef5350"),
        },
      },
    ]
  }

  chart.setOption(option)
}

// Đổi khung
function changeInterval(tf) {
  interval.value = tf
  renderChart()
}

onMounted(() => {
  chart = echarts.init(chartRef.value)
  renderChart()
  window.addEventListener("resize", () => chart.resize())
})
</script>



```

---

## 5️⃣ Dùng component trong App.vue

Mở `src/App.vue` và thay bằng:

```vue
<template>
  <CandleChart />
</template>

<script setup>
import CandleChart from "./components/CandleChart.vue";
</script>
```

---

## 6️⃣ Chạy thử

```bash
npm run dev
```

Mở [http://localhost:5173](http://localhost:5173) → bạn sẽ thấy:

* Biểu đồ nến (candlestick).
* Thanh công cụ bên trái (Line, Horizontal Line, Rectangle).
* Khi chọn **Horizontal Line** rồi click vào chart → vẽ đường ngang đỏ nét đứt giống TradingView.

---

## 7️⃣ Nâng cao (tuỳ chọn)

* Thêm **MA lines (MA5, MA10, MA20, MA30)**.
* Hiện **giá MA ở góc trên cùng**.
* Thêm **crosshair + tooltip**.
* Tạo **localStorage lưu lại các đường vẽ**.

---

✅ Vậy là bạn đã có một **TradingView mini** bằng Vue 3 + ECharts + Sidebar tool.

---

👉 Bạn có muốn mình bổ sung thêm **MA (Moving Average) + giá hiện tại + crosshair** vào cùng cái chart này để thành bản gần giống TradingView hoàn chỉnh không?
