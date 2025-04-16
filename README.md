<!-- pages/index/index.vue 主页面 -->
<template>
  <view class="container">
    <button @click="handleCamera">1. 拍摄/上传小票</button>
    <button @click="handleHistory">2. 历史数据查看</button>
    <button @click="handleSearch">3. 查询与评价</button>
    <button @click="handleLanguage">4. 语言切换（当前：{{ currentLanguage }}）</button>
  </view>
</template>

<script>
export default {
  data() {
    return {
      currentLanguage: '中文'
    }
  },
  methods: {
    handleCamera() {
      uni.chooseImage({
        success: async (res) => {
          // 模拟OCR识别过程
          const mockData = {
            date: '2025-04-13',
            total: 2896,
            items: [
              {name: '雪印 特濃', price: 268},
              // ...其他商品
            ]
          };
          
          uni.navigateTo({
            url: '/pages/receipt/receipt?data=' + JSON.stringify(mockData)
          });
        }
      });
    },
    handleHistory() {
      uni.navigateTo({ url: '/pages/history/history' });
    },
    handleSearch() {
      uni.navigateTo({ url: '/pages/search/search' });
    },
    handleLanguage() {
      this.currentLanguage = this.currentLanguage === '中文' ? '日本語' : '中文';
      uni.setStorageSync('language', this.currentLanguage);
    }
  }
}
</script>

<!-- pages/receipt/receipt.vue 小票详情页 -->
<template>
  <view>
    <button @click="saveData">保存记录</button>
    <!-- 显示识别结果 -->
  </view>
</template>

<script>
export default {
  methods: {
    saveData() {
      const records = uni.getStorageSync('records') || [];
      records.push(this.data);
      uni.setStorageSync('records', records);
      uni.navigateBack();
    }
  }
}
</script>

<!-- pages/history/history.vue 历史页面 -->
<template>
  <view>
    <picker @change="changeView">
      <view>{{ viewType }}</view>
    </picker>
    
    <!-- 日视图 -->
    <view v-if="viewType === 'day'">
      <text>当日总金额：{{ dailyTotal }}</text>
      <!-- 商品列表 -->
    </view>

    <!-- 月视图 -->
    <view v-if="viewType === 'month'">
      <canvas canvas-id="monthChart"></canvas>
    </view>
  </view>
</template>

<!-- pages/search/search.vue 搜索页面 -->
<template>
  <view>
    <input v-model="keyword" placeholder="输入商品名称"/>
    <button @click="search">搜索</button>
    
    <view v-for="item in results" :key="item.id">
      <text @click="showDetail(item)">{{ item.name }}</text>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      keyword: '',
      results: []
    }
  },
  methods: {
    search() {
      const allItems = uni.getStorageSync('records').flatMap(r => r.items);
      this.results = allItems.filter(item => 
        item.name.includes(this.keyword)
      );
    },
    showDetail(item) {
      uni.navigateTo({
        url: `/pages/review/review?id=${item.id}`
      });
    }
  }
}
</script>

<!-- 国际化实现 -->
<script>
// 在App.vue中
export default {
  globalData: {
    i18n: {
      '中文': {
        total: '总金额',
        // 其他翻译
      },
      '日本語': {
        total: '合計金額',
        // 其他翻译
      }
    }
  }
}
</script>
