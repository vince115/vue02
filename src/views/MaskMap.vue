<template>
  <div class="maskmap">
<div class="row">
    <div class="col-sm-3">
      <div class="toolbox">

        <div class="sticky-top bg-white shadow-sm p-2">
          <div class="form-group d-flex">
            <label for="cityName" class="mr-2 col-form-label text-right">縣市</label>
            <div class="flex-fill">

               <select id="cityName" class="form-control"
              v-model="select.city" @change="removeMarker(); updateMap()" >
                <option value="">-- Select One --</option>
                <option :value="c.CityName" v-for="c in cityName" :key="c.CityName">
                  {{ c.CityName }}
                </option>
              </select>

            </div>
          </div>

          <div class="form-group d-flex">
            <label for="cityArea" class="mr-2 col-form-label text-right">地區</label>
            <div class="flex-fill">
              <select id="cityArea" class="form-control">
                <option>1</option>
                <option>2</option>
                <option>3</option>
              </select>

            </div>
          </div>
          <p class="mb-0 small text-muted text-right">請先選擇區域查看（綠色表示還有口罩）</p>
        </div>
        <ul class="list-group">
          <template>
            <a class="list-group-item text-left">
              <h3>藥局名稱</h3>
              <p class="mb-0">成人口罩： 1| 兒童口罩：2</p>
              <p class="mb-0">
                地址：
                <a></a>
              </p>
            </a>
          </template>
        </ul>
      </div>
    </div>
    <div class="col-sm-9">
      <div id="map"></div>
    </div>
  </div>
</div>
</template>

<script>
import L from 'leaflet';
import cityName from '../assets/cityName.json';

let osmMap = {};

console.log(L);

export default {
  name: 'MaskMap',
  data: () => ({
    data: [],
    cityName,
    select: {
      city: '臺北市',
    },
  }),
  methods: {
    updateMap() {
      const pharmacies = this.data.filter((pharmacy) => (
        pharmacy.properties.county === this.select.city));
        // console.log(pharmacies);
      pharmacies.forEach((pharmacy) => {
        const { properties, geometry } = pharmacy;
        L.marker([
          geometry.coordinates[1],
          geometry.coordinates[0],
        ]).addTo(osmMap).bindPopup(`
        <strong>${properties.name}</strong><br/>
        口罩剩餘：<strong>成人 - ${properties.mask_adult ? `${properties.mask_adult} 個` : '未取得資料'}/ 兒童 - ${properties.mask_child ? `${properties.mask_child} 個` : '未取得資料'}</strong><br>
        地址: <a href="https://www.google.com.tw/maps/place/${properties.address}" target="_blank">${properties.address}</a><br>
        電話: ${properties.phone}<br>
        <small>最後更新時間: ${properties.updated}</small>`);
        // console.log(properties);
      });
      this.penTo(pharmacies[0]);
    },
    removeMarker() {
      osmMap.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          osmMap.removeLayer(layer);
        }
      });
    },
    penTo(item) {
      const { properties, geometry } = item;
      console.log(properties);
      osmMap.panTo([geometry.coordinates[1], geometry.coordinates[0]]);
    },
  },
  mounted() {
    const url = 'https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json';
    this.$http.get(url).then((response) => {
      this.data = response.data.features;
      this.updateMap();
    });

    osmMap = L.map('map', {
      center: [25.04, 121.52],
      zoom: 15,
    });
    L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 18,
    }).addTo(osmMap);
    L.marker([25.03, 121.55]).addTo(osmMap);
  },
};
</script>
<style lang="scss">
@import "bootstrap/scss/bootstrap";

#map {
  height: 100vh;
}
.highlight {
  background: #e9ffe3;
}
.toolbox {
  overflow-y: auto;
  a {
    cursor: pointer;
  }
}
</style>
