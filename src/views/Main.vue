<template>
  <div>
    <el-card>
      <el-form :inline="true" :model="mapLocation" :rules="rules" ref="mapLocation" class="demo-form-inline"
               style="height: 40px">
        <el-form-item label="地址" prop="address">
          <el-autocomplete
              v-model="mapLocation.address"
              :fetch-suggestions="querySearch"
              placeholder="请输入您现处详细地址"
              style="width: 300px"
              :trigger-on-focus="false"
              @select="handleSelect"
              prop="address"
          />
        </el-form-item>
        <el-form-item label="体温" prop="temperature">
          <el-input placeholder="请输入您的体温" style="width: 300px" v-model="mapLocation.temperature" prop="temperature">
            <template slot="append">℃</template>
          </el-input>
        </el-form-item>
        <el-form-item label="打卡时间" prop="date">
          <el-date-picker
              v-model="mapLocation.date"
              type="date"
              value-format="yyyy-MM-dd"
              placeholder="选择日期">
          </el-date-picker>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSubmit('mapLocation')">提交</el-button>
        </el-form-item>
      </el-form>
    </el-card>
    <el-card>
      <div class="app-container">
        <div style="margin: 5px">
          <baidu-map class="bm-view" :center="mapCenter" :zoom="mapZoom" :scroll-wheel-zoom="true" ak="baidu-ak"
                     @ready="handlerBMap">
            <bm-geolocation anchor="BMAP_ANCHOR_BOTTOM_RIGHT" :showAddressBar="true"
                            :autoLocation="true"></bm-geolocation>
          </baidu-map>
        </div>
      </div>
    </el-card>
  </div>
</template>

<script>
import BaiduMap from 'vue-baidu-map/components/map/Map.vue'
import axios from "axios";
import Cookies from "js-cookie";

export default {
  name: 'Main',
  components: {
    BaiduMap
  },
  data() {
    var checkAge = (rule, value, callback) => {
      if (!value) {
        return callback(new Error('请输入您的体温'));
      }
      if (value < 36 || value > 44) {
        callback(new Error('体温异常！'));
      } else {
        callback();
      }

    };
    return {
      mapZoom: 15,
      mapCenter: {lng: 0, lat: 0},
      mapLocation: {
        address: undefined,
        coordinate: undefined,
        date: '',
        temperature: '',
      },
      rules: {
        temperature: [
          {validator: checkAge, required: true, trigger: 'blur'}
        ],
        date: [
          {required: true, message: '请选择日期', trigger: 'blur'}
        ],
        address: [
          {required: true, message: '请输入您的地址', trigger: 'blur'}
        ],
      }
    }
  },
  computed: {
    showUsername() {
      return Cookies.get("username")
    }
  },
  methods: {
    handlerBMap({BMap, map}) {
      this.BMap = BMap
      this.map = map
      if (this.mapLocation.coordinate && this.mapLocation.coordinate.lng) {
        this.mapCenter.lng = this.mapLocation.coordinate.lng
        this.mapCenter.lat = this.mapLocation.coordinate.lat
        this.mapZoom = 15
        map.addOverlay(new this.BMap.Marker(this.mapLocation.coordinate))
      } else {
        this.mapCenter.lng = 116.404
        this.mapCenter.lat = 39.915
        this.mapZoom = 10
      }
    },
    querySearch(queryString, cb) {
      var that = this
      var myGeo = new this.BMap.Geocoder()
      myGeo.getPoint(queryString, function (point) {
        if (point) {
          that.mapLocation.coordinate = point
          that.makerCenter(point)
        } else {
          that.mapLocation.coordinate = null
        }
      }, this.locationCity)
      var options = {
        onSearchComplete: function (results) {
          if (local.getStatus() === 0) {
            // 判断状态是否正确
            var s = []
            for (var i = 0; i < results.getCurrentNumPois(); i++) {
              var x = results.getPoi(i)
              var item = {value: x.address + x.title, point: x.point}
              s.push(item)
              cb(s)
            }
          } else {
            cb()
          }
        }
      }
      var local = new this.BMap.LocalSearch(this.map, options)
      local.search(queryString)
    },
    handleSelect(item) {
      var {point} = item
      this.mapLocation.coordinate = point
      this.makerCenter(point)
    },
    makerCenter(point) {
      if (this.map) {
        this.map.clearOverlays()
        this.map.addOverlay(new this.BMap.Marker(point))
        this.mapCenter.lng = point.lng
        this.mapCenter.lat = point.lat
        this.mapZoom = 15
      }
    },
    onSubmit(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          axios.post('http://localhost:8088/signinfo/punch', {
            temperature: this.mapLocation.temperature,
            username: this.showUsername,
            date: this.mapLocation.date,
            address: this.mapLocation.address
          }).then((result) => {
            console.log(result.data)
            if (result.data === "success") {
              this.$message({
                message: '打卡成功',
                type: 'success'
              });
              this.mapLocation = ''
            }
          })
        } else {
          return false
        }
      })
    }
  }
}
</script>

<style>
.bm-view {
  width: 100%;
  height: 300px;
}
</style>