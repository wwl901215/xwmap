<template>
  <div class="container">
    <label>起始地点：</label><input type="text" id="start-location" placeholder="请输入起始地点"/>  <input type="text" id="start-city" value="北京" /><br/>
    <label>结束地点：</label><input type="text" id="end-location" placeholder="请输入结束地点"/>  <input type="text" id="end-city" value="北京" /><br/>
    <input type="button" value="点击搜索" onclick="onSearch()"/>
    <input type="button" value="输出数据" onclick="onExportData()"/>
    <div id="allmap"></div>
  </div>
</template>
<script type="text/javascript" src="http://api.map.baidu.com/api?v=2.0&ak=bWNirr8ebCVOwDRlaZlyYF9hI63rIGAm"></script>
<script>
export default {
  name: 'HelloWorld',
  data() {
      return {
        mapPointArray:[],
        map: null,
        startPoint: null,
        endPoint: null,
        myGeo: null,
      }
    },
    components: {
    },
    methods: {
      creatMap() {
             // 百度地图API功能
             let map = new BMap.Map("allmap");
             map.centerAndZoom(new BMap.Point(116.404, 39.915), 15);
             this.map = map;
          // 创建地址解析器实例
          this.myGeo = new BMap.Geocoder();

      },
      runMap(startPoint,endPoint,map){
             map.centerAndZoom(new BMap.Point(Math.abs((startPoint.lng + endPoint.lng)/2), Math.abs((startPoint.lat + endPoint.lat)/2)), 12);
            let driving = new BMap.DrivingRoute(map);    //驾车实例
            driving.search(startPoint, endPoint);
            driving.setSearchCompleteCallback(function(){

              let pts = driving.getResults().getPlan(0).getRoute(0).getPath();    //通过驾车实例，获得一系列点的数组
              let paths = pts.length;    //获得有几个点

              // 编写自定义函数,创建标注
              function addMarker(point){
                let marker = new BMap.Marker(point);
                map.addOverlay(marker);
                mapPointArray.push(point);
              }

              i=0;
              function resetMkPoint(i){

                let po = new BMap.Point(pts[i].lng , pts[i].lat);

                addMarker(po);
                if(i < paths){
                  i++;
                  resetMkPoint(i);
                }
              }
              setTimeout(function(){

                resetMkPoint(0);

              },100)

            });
      },

      onSearch(){
              let startLocation = document.getElementById("start-location").value;
              let startCity = document.getElementById("start-city").value;
              let endLocation = document.getElementById("end-location").value;
              let endCity = document.getElementById("end-city").value;
              if(startLocation && startCity && endLocation && endCity) {
                map.clearOverlays();//清除图层覆盖物
                startPoint = null;
                endPoint = null;
                mapPointArray=[];
                parseAddress(startLocation,startCity,endLocation,endCity);
              }else{
                alert("请完善地址");
              }
            }

    },
    created() {
      this.creatMap();
    },
    mounted() {

    },
    computed: {

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
   overflow: hidden;
   margin:0;
   font-family:"微软雅黑";
 }
</style>
