<template>
  <div class="container">
    <div ref="topdivref" class="top-div">
      <label>起始地点：</label>
      <input type="text" id="start-location" placeholder="请输入起始地点"/>
      <input type="text"
             id="start-city"
             value="北京"/>
      <br/>
      <label>结束地点：</label>
      <input type="text" id="end-location" placeholder="请输入结束地点"/>
      <input type="text" id="end-city" value="北京"/>
      <br/>
      <input type="button" value="点击搜索" @click="onSearch()"/>
      <input type="button" value="输出数据" @click="onExportData()"/>
    </div>
    <div style="display: flex; flex: 1;">
      <div class="map-div" id="allmap"></div>
    </div>
  </div>
</template>
<script>
import BMap from 'BMap'

export default {
  name: 'HelloWorld',
  data () {
    return {
      mapPointArray: [],
      map: null,
      startPoint: null,
      endPoint: null,
      myGeo: null
    }
  },
  components: {},
  methods: {
    creatMap () {
      // 百度地图API功能
      let map = new BMap.Map('allmap')
      let point = new BMap.Point(116.404, 39.915)
      map.centerAndZoom(point, 15)
      this.map = map
    },
    runMap (startPoint, endPoint, map) {
      map.centerAndZoom(new BMap.Point(Math.abs((startPoint.lng + endPoint.lng) / 2), Math.abs((startPoint.lat + endPoint.lat) / 2)), 12)
      let driving = new BMap.DrivingRoute(map) // 驾车实例
      driving.search(startPoint, endPoint)
      driving.setSearchCompleteCallback(function () {
        let pts = driving.getResults().getPlan(0).getRoute(0).getPath() // 通过驾车实例，获得一系列点的数组
        setTimeout(function () {
          this.resetMkPoint(pts)
        }, 100)
      })
    },
    onSearch () {
      let startLocation = document.getElementById('start-location').value
      let startCity = document.getElementById('start-city').value
      let endLocation = document.getElementById('end-location').value
      let endCity = document.getElementById('end-city').value
      if (startLocation && startCity && endLocation && endCity) {
        this.map.clearOverlays()// 清除图层覆盖物
        this.startPoint = null
        this.endPoint = null
        this.mapPointArray = []
        this.parseAddress(startLocation, startCity, endLocation, endCity)
      } else {
        alert('请完善地址')
      }
    },
    addMarker (point) {
      let marker = new BMap.Marker(point)
      this.map.addOverlay(marker)
      this.mapPointArray.push(point)
    },
    resetMkPoint (pts) {
      let i = 0
      let paths = pts.length // 获得有几个点
      let po = new BMap.Point(pts[i].lng, pts[i].lat)
      this.addMarker(po)
      if (i < paths) {
        i++
        this.resetMkPoint(i)
      }
    },
    parseAddress (mStartL, mStartC, mEndL, mEndC) {
      // 创建地址解析器实例
      let myGeo = new BMap.Geocoder()
      myGeo.getPoint(mStartL, function (pstart) {
        if (pstart) {
          this.startPoint = pstart
          if (this.startPoint && this.endPoint) {
            this.runMap(this.startPoint, this.endPoint, this.map)
          }
        } else {
          alert('起始地址没有解析到结果!')
        }
      }, mStartC)

      myGeo.getPoint(mEndL, function (pend) {
        if (pend) {
          this.endPoint = pend
          if (this.startPoint && this.endPoint) {
            this.runMap(this.startPoint, this.endPoint, this.map)
          }
        } else {
          alert('终点地址没有解析到结果!')
        }
      }, mEndC)
    },
    onExportData () {
      alert(JSON.stringify(this.mapPointArray))
    }

  },
  created () {
  },
  mounted () {
    this.creatMap()
  },
  computed: {
    onMapLayout () {
      // let topHeight = this.refs.top-div.width
      console.log(JSON.stringify(this.$refs['topdivref']))
      let wid = window.innerWidth
      let hei = window.innerHeight
      return `height: ${hei}px; width: ${wid}px; paddingTop: 10px;`
    }
  }
}
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .container {
    width: 100%;
    height: 100%;
    background-color: #fff;
    display: flex;
    flex-direction: column;
    overflow: hidden;
    margin: 0;
    font-family: "微软雅黑";
  }
  .map-div {
    width: 100%;
    height: 100%;
  }
  .top-div {

  }
</style>
