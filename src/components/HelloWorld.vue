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
      mapStepArray: [],
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
      let _that = this
      map.centerAndZoom(new BMap.Point(Math.abs((startPoint.lng + endPoint.lng) / 2), Math.abs((startPoint.lat + endPoint.lat) / 2)), 12)
      let driving = new BMap.DrivingRoute(map) // 驾车实例
      driving.search(startPoint, endPoint)
      driving.getResults()// 检索最近一次搜索结果 返回DrivingRouteResult
      driving.clearResults() // 清除最近一次检索的结果
      // driving.setPolicy(policy:DrivingPolicy) // 设置路线规划策略，参数为策略常量
      driving.setPolylinesSetCallback(function (event) {
        alert(JSON.stringify(event))
      })
      driving.setSearchCompleteCallback(function () {
        let pts = driving.getResults().getPlan(0).getRoute(0).getPath() // 通过驾车实例，获得一系列点的数组
        let plansNum = driving.getResults().getNumPlans() // Number 表示驾车几种方案，每种方案中有多条路线；
        let plans = driving.getResults().getPlan(0) // 表示第一条方法；
        let routeNum = plans.getNumRoutes() // 表示每条方案中的路线数量；
        console.log('plan-distance:' + plans.getDistance())
        console.log('plan-dragpois:' + plans.getDragPois())
        console.log('plan-duration:' + plans.getDuration())
        let route = plans.getRoute(0) // 表示获取第一条路线；
        console.log('route-distance:' + route.getDistance(0))
        console.log('route-index:' + route.getIndex())
        console.log('route-numsteps:' + route.getNumSteps())
        console.log('route-polyline:' + route.getPolyline())
        console.log('route-routeType:' + route.getRouteType())
        console.log('route-step:' + route.getStep(1))
        console.log('route-distance:' + route.getDistance())
        let stepNums = route.getNumSteps()
        for (let i = 0; i < stepNums; i++) {
          let step = route.getStep(i)
          // console.log('step-distance:' + step.getDistance(true))
          // console.log('step-description:' + step.getDescription(true)) // 描述信息中是否带html信息
          // console.log('step-index:' + step.getIndex())
          // console.log('step-position.lat:' + step.getPosition().lat)
          let start = `<span class="navtrans-navlist-icon "></span><div class='navtrans-navlist-content'>从起点向正西方向出发</div>`
          let dec = `<span class="navtrans-navlist-icon s-1"></span><div class='navtrans-navlist-content'>行驶90米，到达终点</div>`
          let dec2 = `<span class="navtrans-navlist-icon s-2"></span><div class='navtrans-navlist-content'>沿同成街行驶340米，右前方转弯进入<span>G6辅路</span></div>`
          _that.dealString(step.getDescription())
          _that.mapStepArray.push(step.getPosition())
        }
        // console.log('route-path:' + route.getPath())
        // let poly = route.getPolyline() // 仅当结果自动添加到地图上有效，拐点图标信息
        // let des = route.getPath()
        _that.resetMkPoint(pts)
      })
    },
    /**
     * 解析字符串
     */
    dealString (dataString) {
      let iconTypeReg = /(?<=<span class="navtrans-navlist-icon ).*?(?="><\/span>)/
      let nextRouteReg = /(?<=<span>).*?(?=<\/span>)/
      let iconType = dataString.match(iconTypeReg) || []
      let nextRoute = dataString.match(nextRouteReg) || []
      let dirReg = ''
      if (nextRoute) {
        dirReg = /(?<=米，).*?(?=进入<span>)/
      } else {
        dirReg = /(?<=，).*?(?=<\/div>)/
      }
      let dirString = dataString.match(dirReg) || []
      let tempReg = /(?<=<div class='navtrans-navlist-content'>).*?(?=<\/div>)/
      let tempString = dataString.match(tempReg) || []
      console.log(iconType[0] + '<--->' + tempString[0])
      let resul = {
        stepType: '',
        iconType: ''
      }
      return resul
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
      /**
         * 该方法是在地图上绘制图标，如果路径太远会导致页面卡死，可去掉该绘点功能；
         */
      // this.map.addOverlay(marker)
      this.mapPointArray.push(point)
    },
    /**
     *该方法用来画点或者线
     * @param point
     */
    resetMkPoint (pts) {
      let leng = pts.length // 获得有几个点
      for (let i = 0; i < leng; i++) {
        let po = new BMap.Point(pts[i].lng, pts[i].lat)
        this.mapLine(pts, i)
        this.addMarker(po)
      }
    },
    /**
     * this method is used to draw line;
     * @param pts
     * @param i
     */
    mapLine (pts, i = 0) {
      if (!pts || !pts[i] || !pts[i + 1] || !pts[i].lng || !pts[i].lat || !pts[i + 1].lng || !pts[i + 1].lat) return
      let sP = pts[i]
      let eP = pts[i + 1]
      let polyline = new BMap.Polyline(
        [
          sP,
          eP
        ],
        {
          strokeColor: 'red',
          strokeWeight: 7,
          strokeOpacity: 1
        }) // 创建折线
      this.map.addOverlay(polyline)
    },
    parseAddress (mStartL, mStartC, mEndL, mEndC) {
      // 创建地址解析器实例
      let myGeo = new BMap.Geocoder()
      let _that = this
      myGeo.getPoint(mStartL, function (pstart) {
        if (pstart) {
          _that.startPoint = pstart
          if (_that.startPoint && _that.endPoint) {
            _that.runMap(_that.startPoint, _that.endPoint, _that.map)
          }
        } else {
          alert('起始地址没有解析到结果!')
        }
      }, mStartC)

      myGeo.getPoint(mEndL, function (pend) {
        if (pend) {
          _that.endPoint = pend
          if (_that.startPoint && _that.endPoint) {
            _that.runMap(_that.startPoint, _that.endPoint, _that.map)
          }
        } else {
          alert('终点地址没有解析到结果!')
        }
      }, mEndC)
    },
    onExportData () {
      let allPoint = this.mapPointArray
      let stepPoint = this.mapStepArray
      console.log('allPoint-length:' + allPoint.length)
      console.log('stepPoint-length:' + stepPoint.length)
      let k = 0
      for (let i = 0; i < allPoint.length; i++) {
        console.log('allData:' + i)
        let allItem = allPoint[i]
        for (let j = 0; j < stepPoint.length; j++) {
          let stepItem = stepPoint[j]
          if (allItem.lat === stepItem.lat && allItem.lng === stepItem.lng) {
            k++
            console.log(k)
          }
        }
      }
      alert(JSON.stringify(this.mapPointArray))
    }

  },
  created () {
  },
  mounted () {
    this.creatMap()
  },
  computed: {}
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
