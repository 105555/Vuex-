<template>
  <div class="mask-map" id="mask-map"></div>
</template>

<script>
import L from "leaflet";
import { mapGetters } from "vuex";
export default {
  name: "maskMap",
  data() {
    return {
      map: {},
      markers: [],
    };
  },
  mounted() {
    // 將地圖初始化
    this.map = L.map("mask-map", {
      center: [25.03, 121.55],
      zoom: 14,
    });
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "© OpenStreetMap", // 商用時必須要有版權出處
      zoomControl: true, // 是否秀出 - + 按鈕
    }).addTo(this.map);
  },
  computed: {
    // 與地圖做連東需要抓取左側選單的資料
    ...mapGetters(["currDistrictInfo", "filteredStores"]),
  },
  watch: {
    // 切換行政區時 指定中心點
    currDistrictInfo(item) {
      if (
        item &&
        typeof item.latitude !== "undefined" &&
        typeof item.longitude !== "undefined"
      ) {
        this.map.panTo(new L.LatLng(item.latitude, item.longitude));
      } else {
        console.error("Invalid latitude or longitude values:", item);
      }
    },

    filteredStores(stores) {
      // 先清除原有 marker
      this.clearMarkers();

      // 根據藥局資訊加上對應 marker
      stores.forEach((element) => this.addMarker(element));
    },
  },
  methods: {
    addMarker(item) {
      // 標記的圖示
      const ICON = {
        iconUrl:
          "https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-violet.png",
        shadowUrl:
          "https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png",
        iconSize: [25, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [41, 41],
      };

      // 將標記放置到地圖上
      const marker = L.marker([item.longitude, item.latitude], ICON)
        .addTo(this.map)
        .bindPopup(`<h2 class="popup-name">${item.name}</h2>`);

      // 替 marker 加入 id 與經緯度資訊
      marker.markerId = item.id;
      marker.lng = item.longitude;
      marker.lat = item.latitude;

      // 將 marker push 到陣列裡
      this.markers.push(marker);
    },
    clearMarkers() {
      // 清除地圖所有標記
      this.map.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          this.map.removeLayer(layer);
        }
      });

      // 清空陣列
      this.markers.length = 0;
    },
    triggerPopup(markerId) {
      const marker = this.markers.find((d) => d.markerId === markerId);

      this.map.flyTo(new L.LatLng(marker.lng, marker.lat), 15);
      marker.openPopup();
    },
  },
};
</script>

<style></style>
