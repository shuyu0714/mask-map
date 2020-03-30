<template>
  <div class="toolbox" :class="{'active': isCollapse}">
    <a class="collapseBtn" @click="toggleToolbox"></a>
    <div class="topTnfo">
      <div class="head fBalance">
        <h1 class="mb-0">口罩即時地圖</h1>
        <div class="resources">
          <h6 class="resource">資料來源：衛福部</h6>
          <div class="date">2020年02月08日</div>
        </div>
      </div>
      <div class="daily-card">
        <img class="img-fluid" src="@/assets/banner_announce_fix@2x.png" alt />
        <h2 class="today mb-0">{{weekday}}</h2>
        <div class="number">{{idNumber}}</div>
        <div class="info-anchor"></div>
      </div>
      <div class="group">
        <div class="row">
          <div class="col-6 form-group d-flex">
            <label for="cityName" class="mr-2 col-form-label text-right">縣市</label>
            <div class="flex-fill">
              <select id="cityName" class="form-control" v-model="initSelect.city">
                <option value disabled>請選擇</option>
                <option
                  :value="c.CityName"
                  v-for="(c, index) in cityName"
                  :key="index"
                >{{c.CityName}}</option>
              </select>
            </div>
          </div>
          <div class="col-6 form-group d-flex">
            <label for="area" class="mr-2 col-form-label text-right">地區</label>
            <div class="flex-fill">
              <select
                id="area"
                class="form-control"
                v-model="initSelect.area"
                v-if="initSelect.city.length"
                @change="updateSelect"
              >
                <option
                  :value="a.AreaName"
                  v-for="(a, index) in cityName.find((city) => city.CityName === initSelect.city).AreaList"
                  :key="index"
                >{{a.AreaName}}</option>
              </select>
              <select class="form-control" v-else>
                <option value disabled selected>請選擇</option>
              </select>
            </div>
          </div>
        </div>
      </div>
      <div class="group mask-type">
        <div class="row">
          <div class="col-4">
            <div
              class="round big"
              @click="maskType = 'all'"
              :class="{ active: maskType === 'all' }"
            >所有口罩</div>
          </div>
          <div class="col-4">
            <div
              class="round big"
              @click="maskType = 'adult'"
              :class="{ active: maskType === 'adult' }"
            >成人口罩</div>
          </div>
          <div class="col-4">
            <div
              class="round big"
              @click="maskType = 'child'"
              :class="{ active: maskType === 'child' }"
            >兒童口罩</div>
          </div>
        </div>
      </div>
      <div class="group mask-status fBalance">
        <h3>尚有庫存店家</h3>
        <div class="status-btns">
          <div class="round">距離最近</div>
          <div class="round">存量最多</div>
          <div class="round">已標星號</div>
        </div>
      </div>
    </div>
    <ul class="list-group" v-if="initSelect.city !== '' && initSelect.area !== undefined">
      <template v-for="(item, index) in getData">
        <a class="list-item text-left" :key="index"
          v-if="item.properties.county === initSelect.city && item.properties.town === initSelect.area">
          <div class="name">
            <h4>{{item.properties.name}}</h4>
            <span class="distance">10m</span>
          </div>
          <div class="info">
            <div class="item">{{item.properties.address}}</div>
            <div class="item">{{item.properties.phone}}</div>
            <div class="item">今日營業時間：
              <span class="store-staus">Closing soon:</span>
              <span class="time">9am–10pm</span>
            </div>
          </div>
          <div class="mask-num">
            <div class="round" :class="{'empty': item.properties.mask_adult < 1}">成人口罩<label>{{item.properties.mask_adult}}</label></div>
            <div class="round" :class="{'empty': item.properties.mask_child < 1}">兒童口罩<label>{{item.properties.mask_child}}</label></div>
          </div>
          <div class="action-group">
            <a class="favorite" href="#"></a>
            <a class="location" :href="`https://www.google.com.tw/maps/place/${item.properties.address}`" target="_blank" title="Google Map"></a>
          </div>
        </a>
      </template>
    </ul>
    <ul v-else :style="{ textAlign: 'center', fontSize: 1 + 'rem' }">請選擇縣市區域...</ul>
  </div>
</template>
<script>
import cityName from '@/assets/CityCountyData.json'
const date = new Date()
const week = date.getDay()
const weekList = ['星期日', '星期一', '星期二', '星期三', '星期四', '星期五', '星期六']
export default {
  name: 'toolbox',
  props: ['getData', 'initSelect'],
  data: () => ({
    weekday: weekList[week],
    idNumber: String,
    maskType: 'all',
    cityName,
    isCollapse: false
  }),
  created () {
    this.getID()
  },
  methods: {
    getID: function () {
      if ([1, 3, 5].includes(week)) {
        this.idNumber = '1.3.5.7.9'
      } else {
        this.idNumber = '2.4.6.8.0'
      }
    },
    toggleToolbox () {
      this.isCollapse = !this.isCollapse
    },
    updateSelect () {
      this.$emit('change')
    }
  }
}
</script>
