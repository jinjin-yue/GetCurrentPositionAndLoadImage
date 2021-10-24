<template>
  <div>
    <l-map
      class="map"
      ref="map"
      :min-zoom="minZoom"
      :max-zoom="maxZoom"
      :crs="crs"
      :options="options"
      @update:zoom="updateZoom"
    >
      <l-tile-layer :url="tileLayer.url" :options="tileLayer.options" />
      <l-marker
        ref="marker"
        :lat-lng="currentPosition"
        :draggable="true"
        @update:latLng="updateMarker"
      >
        <l-popup />
      </l-marker>
    </l-map>
    <div>
      <p>Zoom：{{ currentZoom }}</p>
      <v-text-field
        label="現在地"
        append-outer-icon="mdi-map-marker"
        v-model="currentPosition"
        filled
        readonly
        @click:append-outer="getPosition"
      ></v-text-field>
      <v-file-input
        label="写真取込"
        accept="image/*"
        prepend-icon="mdi-camera"
        v-model="imageFiles"
        small-chips
        show-size
        filled
        multiple
      >
        <template v-slot:selection="{ text, index }">
          <v-chip
            small
            text-color="white"
            color="#295671"
            close
            @click:close="removeImage(index)"
          >
            {{ text }}
          </v-chip>
        </template>
      </v-file-input>
      <div>
        <v-row>
          <v-col
            v-for="(imageFile, index) in imageFiles"
            :key="index"
            class="d-flex child-flex"
            cols="4"
          >
            <v-img
              :src="`${imageFile.url}`"
              :lazy-src="`${imageFile.url}`"
              aspect-ratio="1"
              class="grey lighten-2"
            >
              <template v-slot:placeholder>
                <v-row class="fill-height ma-0" align="center" justify="center">
                  <v-progress-circular
                    indeterminate
                    color="grey lighten-5"
                  ></v-progress-circular>
                </v-row>
              </template>
            </v-img>
          </v-col>
        </v-row>
      </div>
    </div>
  </div>
</template>

<script>
import L from "leaflet";
import { LMap, LMarker, LTileLayer, LPopup } from "vue2-leaflet";
// アイコンエラーの対応 https://vue2-leaflet.netlify.app/quickstart/#marker-icons-are-missing
delete L.Icon.Default.prototype._getIconUrl;
L.Icon.Default.mergeOptions({
  iconRetinaUrl: require("leaflet/dist/images/marker-icon-2x.png"),
  iconUrl: require("leaflet/dist/images/marker-icon.png"),
  shadowUrl: require("leaflet/dist/images/marker-shadow.png"),
});

export default {
  name: "basicMap",
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup,
  },
  data() {
    return {
      crs: L.CRS.EPSG3857,
      center: [35.6813, 139.7673], // [Lat, Lon]
      maxZoom: 20, // 最大ズームレベル
      minZoom: 6, // 最小ズームレベル
      options: {
        doubleClickZoom: false, // ダブクリックによるズーム無効化
        touchZoom: true, // タッチによるズーム有効化
        renderer: L.canvas(), // レイヤ表示方式
      },
      tileLayer: {
        url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
        options: {
          maxNativeZoom: 19,
          maxZoom: 20,
          inertiaMaxSpeed: 1000,
          attribution:
            '<a href="//www.openstreetmap.org/copyright">&copy; OpenStreetMap contributors, ODbL</a>',
        },
      },
      currentZoom: 18,
      currentPosition: L.latLng(35.6813, 139.7673),
      currentMarker: L.latLng(35.6813, 139.7673),
      imageFiles: [],
    };
  },
  computed: {},
  watch: {
    imageFiles() {
      if (this.imageFiles && this.imageFiles.length > 0) {
        for (const imageFile of this.imageFiles) {
          imageFile.url = URL.createObjectURL(imageFile);
        }
      }
    },
  },
  methods: {
    updateZoom(zoom) {
      this.currentZoom = zoom;
    },
    updateMarker(latLng) {
      this.currentMarker = latLng;
      this.updateCoordRef(
        this.$refs.map.mapObject,
        this.$refs.marker.mapObject,
        latLng
      );
    },
    formatDigit(num) {
      return ("    " + num.toFixed(7)).substr(-11);
    },
    updateCoordRef(map, marker, latlng) {
      var content =
        '<pre class="coords">現在地（' +
        "X: " +
        this.formatDigit(latlng.lng) +
        ", " +
        "Y: " +
        this.formatDigit(latlng.lat) +
        "）</pre>";
      marker
        .getPopup()
        .setContent(content)
        .openOn(map);
    },
    // 現在地取得処理
    getPosition() {
      if (!navigator.geolocation) {
        // Geolocation APIに対応していない
        alert("この端末では位置情報が取得できません");
      }
      // 現在地を取得
      navigator.geolocation.getCurrentPosition(
        // 取得成功した場合
        (position) => {
          this.currentPosition = [
            position.coords.latitude,
            position.coords.longitude,
          ];
          this.$refs.map.mapObject.setView(this.currentPosition, 18);
        },
        // 取得失敗した場合
        (error) => {
          switch (error.code) {
            case 1: //PERMISSION_DENIED
              alert("位置情報の利用が許可されていません");
              break;
            case 2: //POSITION_UNAVAILABLE
              alert("現在位置が取得できませんでした");
              break;
            case 3: //TIMEOUT
              alert("タイムアウトになりました");
              break;
            default:
              alert("その他のエラー(エラーコード:" + error.code + ")");
              break;
          }
        }
      );
    },
    removeImage(index) {
      this.imageFiles.splice(index, 1);
    },
  },
  mounted() {
    this.$refs.map.mapObject.setView(this.center, this.currentZoom);
    this.getPosition();
  },
};
</script>

<style scoped>
.map {
  height: 60vh;
  z-index: 0;
}
.v-text-field {
  max-width: 480px;
}
.v-file-input {
  flex-direction: row-reverse;
  max-width: 480px;
}
</style>
