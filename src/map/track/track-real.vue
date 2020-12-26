<template>
    <div class="myTable">
        <div class="input searchNav searchNav2" >
            <el-autocomplete
                    class="inline-input"
                    v-model="iccidOrImei"
                    :fetch-suggestions="querySearch"
                    placeholder=" 请输入 ICCID 或 IMEI "
                    :trigger-on-focus="false"
                    clearable
            ></el-autocomplete>
            <el-button type="primary"  @click="searchInfo" style="float: right; position: relative;left: -20px;" >搜索</el-button>
        </div>
        <div class="amap-page-container" v-loading="loading" >
            <div class="amap-page-container" v-loading="loading">
                <div id="amap-real" class="amap-demo" style="height: 740px;"></div>
            </div>
        </div>
        <div class="track-main" v-if="devInfo.online">
            <p>
                <span><img src="../../assets/img/track-icon1.png" alt=""></span>
                <span>状态：</span>
                <span>{{devInfo.online | showState}}</span>
            </p>
            <p>
                <span><img src="../../assets/img/track-icon2.png" alt=""></span>
                <span>速度：</span>
                <span>{{devInfo.speed | showSpeed}}</span><span>&nbsp;&nbsp;km/h</span>
            </p>
            <p>
                <span><img src="../../assets/img/track-icon3.png" alt=""></span>
                <span>方向角：</span>
                <span>{{devInfo.angle}}</span>
            </p>
            <p>
                <span><img src="../../assets/img/track-icon4.png" alt=""></span>
                <span>经纬度：</span>
                <span>{{devInfo.lonLat}}</span>
            </p>
            <p>
                <span><img src="../../assets/img/track-icon5.png" alt=""></span>
                <span>执行次数：</span>
                <span>{{devInfo.actionNumber}}</span>
            </p>
        </div>
    </div>
</template>

<script>
    // NPM 方式
    import { lazyAMapApiLoaderInstance } from 'vue-amap';
    import carUrl from '../../assets/img/car.png'
    import {axios} from "../../config/axios"
    let setTime;
    let nav;
    export default {
        name: "track-real",
        data () {
            return {
                devInfo:{
                    online:false,
                    speed:'',
                    angle:'',
                    lonLat:'',
                    actionNumber:''
                },
                loading:false, // 点击搜索 时出现的加载状态
                iccid:'',
                iccidOrImei:'', // 临时保存，搜索时将值 赋给 iccid
                restaurants: [],
                currentIndex :-1, // 汽车当前行驶的下标点
                gpsData : [],
                center : '',
            }
        },
        created() {
            this.initMap();
            // 将选择的 数据 从缓存中读取
            this.iccidOrImei = this.getLocalStorageIccid();
        },
        methods:{
            searchInfo(){
                window.clearInterval(setTime); // 清除定时任务
                this.devInfo.actionNumber = 0;
                this.gpsData = []; // 清空数据
                if (this.iccidOrImei !== ""){
                    let paramsLength = this.iccidOrImei.trim().length;
                    if (paramsLength === 20){
                        this.iccid = this.iccidOrImei;
                        this.dealData();
                        return;
                    }
                    if (paramsLength === 15){
                        axios.get("/blackview/api/device-dependent/dev-info/getIccidByImei",{params:{imei:this.iccidOrImei}}).then(result => {
                            if (result.code !== 200) {
                                console.error(result);
                            }
                            this.iccid  = result.data;
                            this.dealData();
                        });
                        return;
                    }
                    this.msg("输入的 字符串长度："+paramsLength+" ，不是 IMEI 也不是ICCID");
                } else {
                    this.msg("请输入 ICCID 或者 IMEI 进行查询");
                }
            },
            dealData(){
                this.setLocalStorageIccid(this.iccidOrImei);
                // 获取已经行驶的轨迹数据
                this.loading = true;
                // 获取设备是否在线
                axios.get("/blackview/contrail/online",{params:{iccid: this.iccid}}).then(result => {
                    this.loading = false;
                    if (result.code !== 200 ) {
                        console.error(result);
                        return;
                    }
                    this.devInfo.online = result.data.online;
                    if ( !this.devInfo.online ) {
                        this.msg("当前设备不在线 ！");
                        this.initMap();
                    }else{
                        this.initPage();
                    }
                }).catch(err => {
                    this.loading = false;
                    console.log(err);
                });
            },
            querySearch(queryString, cb) {  // 模糊查询
                if (queryString.trim().length > 0) {
                    axios.get('/blackview/api/capture/list/device', {
                        params: {
                            iccid: queryString.trim()
                        }
                    }).then(result => {
                        if (result.code != 200) {
                            console.log(result);
                            this.restaurants = [];
                        } else {
                            this.restaurants  = result.data;
                        }
                        cb(this.restaurants);
                    }).catch(err => {
                        console.log(err);
                    });
                }
            },
            initMap(){
                lazyAMapApiLoaderInstance.load().then(() => {
                    this.map = new AMap.Map('amap-real', {
                            center: [114.037939,22.627198],
                            zoom: 14
                        }
                    )
                });
            },
            initPage(){
                console.log("===== init===");
                let _this = this;
                lazyAMapApiLoaderInstance.load().then(() => {
                    this.map = new AMap.Map('amap-real', {
                            center: [120.02572781032985,36.285359429253475],
                            zoom: _this.center
                        },
                        AMapUI.loadUI(['misc/PathSimplifier'], (PathSimplifier) => {

                            if (!PathSimplifier.supportCanvas) {
                                alert('当前环境不支持 Canvas！');
                                return;
                            }

                            //创建组件实例
                            var pathSimplifierIns = new PathSimplifier({
                                zIndex: 100,
                                map: this.map, //所属的地图实例
                                getPath: (pathData) => {
                                    //将gps描述的所有的点数放到全局变量
                                   // pathData.path.length;                                    //返回轨迹数据中的节点坐标信息，[AMap.LngLat, AMap.LngLat...] 或者 [[lng|number,lat|number],...]
                                    return pathData.path;
                                },
                                getHoverTitle: (pathData, pathIndex, pointIndex)=> {
                                    //返回鼠标悬停时显示的信息
                                    if (pointIndex >= 0) {
                                        //鼠标悬停在某个轨迹节点上
                                        return pathData.name + '，点:' + pointIndex + '/' + pathData.path.length;
                                    }
                                    //鼠标悬停在节点之间的连线上
                                    return pathData.name + '，点数量' + pathData.path.length;
                                },
                                renderOptions: {
                                    //轨迹线的样式
                                    pathLineStyle: {
                                        strokeStyle: 'white',
                                        lineWidth: 3,   //线宽度
                                        dirArrowStyle:{
                                            stepSpace:15  //stepSpace: {number} 箭头排布的间隔，单位像素
                                        }
                                    }
                                }
                            });
                            checkNav();
                            trackInterval();
                            function checkNav(){
                                setTime = self.setInterval(trackInterval, 2000); // 3 秒钟获取一次轨迹
                            }
                            let saveOldPosition =[]; // 临时保存经纬度，当这次与上一次相同的时候，就不渲染地图
                            let noMovementNumber = 0;
                            function trackInterval() {
                                    axios.get("/blackview/contrail/latest-location",{params:{iccid: _this.iccid}}).then(result => {
                                        if (result.code !== 200){
                                            console.error(result)
                                            return
                                        }
                                        _this.devInfo.actionNumber++;
                                        let data = result.data;
                                        var gpsArr = data.split(",");
                                        let oneNode = [gpsArr[5],gpsArr[3]];
                                        if ( JSON.stringify(saveOldPosition) === JSON.stringify(oneNode) ) {
                                            console.info("最后定位不变：",oneNode);
                                            noMovementNumber++;
                                            if ( noMovementNumber > 60){  // 如果连续 60 秒位置不变，查询车辆是否离线
                                                onlineState();   // 如果离线则暂停定时任务，否则将统计的  noMovementNumber 置 0；
                                            }
                                            return;
                                        }else{
                                            noMovementNumber = 0;
                                            showDevInfo(data);
                                            saveOldPosition = oneNode;
                                            _this.center = oneNode;
                                            if ( _this.gpsData.length === 2 ){
                                                _this.gpsData.splice(0, 1);
                                            }
                                            _this.gpsData.push(oneNode);
                                            dealPositionStr();
                                        }
                                    });
                            }
                            function onlineState(){
                                axios.get("/blackview/contrail/online",{params:{iccid: _this.iccid}}).then(result => {
                                    if (result.code !== 200) {
                                        console.log(result);
                                        return;
                                    }
                                    if (result.data.online) {
                                           noMovementNumber = 0;
                                    }else {
                                           window.clearInterval(setTime); // 暂停定时任务
                                    }
                                }).catch(err => {
                                    console.log(err);
                                });
                            }
                            function showDevInfo(data){
                                var gpsArr = data.split(",");
                                console.info("=====================",data)
                                _this.devInfo.lonLat =  [gpsArr[5],gpsArr[3]];
                                _this.devInfo.speed=gpsArr[8];
                                _this.devInfo.angle = gpsArr[9];
                            }
                            function dealPositionStr(){
                                pathSimplifierIns.setData([{
                                    name: '轨迹',
                                    path: _this.gpsData
                                }]);
                                nav = pathSimplifierIns.createPathNavigator(0, {
                                    speed:_this.devInfo.speed*3.6 ,
                                    pathNavigatorStyle: {
                                        width: 16,
                                        height: 32,
                                        fillStyle:  'white', //填充色，比如 red, rgb(255,0,0), rgba(0,0,0,1)等
                                        strokeStyle: 'white',  // 描边色
                                        content: PathSimplifier.Render.Canvas.getImageContent(carUrl, onload, onerror),
                                    },
                                    keyPointStyle:{
                                        radius: 9,
                                        fillStyle: "white",  //填充色，比如 red, rgb(255,0,0), rgba(0,0,0,1)等
                                        strokeStyle: "white", //描边色
                                    }
                                });
                                nav.start();
                            }
                        })
                    )
                });
            },
        },
        destroyed: function () {
            window.clearInterval(setTime);
        },
        filters: {
            showState(val){
                if (val){
                    return "在线";
                }
                return "离线"
            },
            showSpeed(val){
                return Math.ceil(val * 3.6)
            }
        }
    }
</script>

<style lang="less" scoped>
    .input input{
        position: relative;
        display: inline-block;
        border-radius: 5px;
        height: 20px;
        margin-right: 20px;
        font-size: 16px;
        line-height: 32px;
        width: 250px;
        padding-left: 8px;
    }
    .searchNav .el-input{
        width: 300px;
    }
    .track-main{
        padding: 25px; background-color:#f4f4f4; border-top: 1px solid #dfe2ea;
        p{
            display: inline-block; padding: 0; margin: 0; margin-right: 6%;
            span{ display: inline-block; vertical-align: middle;}
            img{ width: 18px; vertical-align: middle; margin-right: 8px;}
        }
        p span:nth-child(3){
            color: #333333;
        }
    }
</style>
