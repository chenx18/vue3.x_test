<template>
    <div class="myTable">
        <div class="input searchNav">
            <span>
                <el-switch v-model="radio" inactive-text="辅助定位"></el-switch>
            </span>
            <span style="padding-left: 30px;">
                <el-autocomplete
                        class="inline-input"
                        width="50px"
                        v-model="iccidOrImei"
                        :fetch-suggestions="querySearch"
                        placeholder=" 请输入 ICCID 或 IMEI "
                        :trigger-on-focus="false"
                        clearable
                ></el-autocomplete>
            </span>
            <span><i>时间：</i>
                <el-date-picker
                    v-model="startTime"
                    type="daterange"
                    range-separator="至"
                    start-placeholder="开始日期"
                    end-placeholder="结束日期"
                    value-format="yyyy-MM-dd"
                    align="left"
                    unlink-panels
                    :picker-options="pickerOptions"
                    >
                </el-date-picker>
            </span>
            <a href="javascript:void(0);" style="float: right;position: relative;left: -10px;top: 14px;color: #a59696;" @click="downloadTrack">下载</a>
            <el-button type="primary" @click="searchInfo" style="float: right; position: relative;left: -20px;" >搜索</el-button>

        </div>
        <div class="amap-page-container" v-loading="loading">
            <div id="amap-show" class="amap-demo" style="height: 670px;"></div>
        </div>
    </div>
</template>

<script>
    // NPM 方式
    import { lazyAMapApiLoaderInstance } from 'vue-amap';
    import carUrl from '../../assets/img/car.png'
    import event from './../../utils/event-bus'
    import {axios} from "../../config/axios";
    export default {
        name: "track-show",
        data () {
            return {
                radio:false,
                iccid:'',
                iccidOrImei:'', // 临时存储变量
                startTime:[this.dateFormatter(), this.dateFormatter()],
                loading:false,
                //iccid搜索
                restaurants: [],
                defaultValue:new Date(),
                pickerOptions: {
                    shortcuts: [{
                        text: '今天',
                        onClick(picker) {
                            picker.$emit('pick', [new Date(), new Date()]);
                        }
                    }, {
                        text: '昨天',
                        onClick(picker) {
                            const start = new Date();
                            start.setDate(new Date().getDate()-1);
                            picker.$emit('pick', [start, start]);
                        }
                    }, {
                        text: '最近两天',
                        onClick(picker) {
                            const end = new Date();
                            const start = new Date();
                            start.setDate(end.getDate()-1);
                            picker.$emit('pick', [start, end]);
                        }
                    }]
                },
            }
        },
        created() {
            this.initMap();
            // 将选择的 数据 从缓存中读取
            this.iccidOrImei = this.getLocalStorageIccid();
            this.listenTrackChildren();
            let paramsObj = this.$route.params; // 获取路由的参数对象
            if (Object.keys(paramsObj).length !== 0) {
                this.iccid = paramsObj.iccid;
                this.iccidOrImei = this.iccid;
            }
        },
        methods:{
            searchInfo(){
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
                if (this.startTime === ""){
                    this.msg("请选择轨迹日期 ！");
                    return;
                }
                const start = this.startTime[0];
                const end = this.startTime[1];
                const dayNumber = parseInt(end.replace(/-/g,"")) - parseInt(start.replace(/-/g, ""));
                if (dayNumber === 0){
                    this.getGPSData(start);
                }else if(dayNumber <3){
                    this.getDurationGPSData(start, end);
                }else {
                    this.msg("日期范围不能超过三天！");
                }
            },
            downloadTrack(){
                if (this.iccid.length !==20) {
                    this.msg("ICCID 输入不正确");
                    return;
                }
                const start = this.startTime[0];
                const end = this.startTime[1];
                if (start === ''){
                    this.msg("请选择时间");
                    return;
                }
                if(start !== end){
                    this.msg("开始时间和结束数据必须相同");
                    return;
                }
                let downloadUrl = this.downloadTrackUrl+start.replace(/-/g,"/")+"/"+this.iccid;
                if (start === this.dateFormatter()) { // 判断是否为当天，当天文件是没有后缀，否则 xxx.gz
                    this.existsUrl(downloadUrl);
                }else{
                    downloadUrl+='.gz';
                    this.existsUrl(downloadUrl);
                }
            },
            existsUrl(url){
                axios.get("/blackview/api/util/existsUrl",{params:{url:url}}).then(result => {
                        if (result.code !== 200 ) {
                            console.error("result:",result);
                            return;
                        }
                        if (result.data){
                            window.location.href=url;
                        }else{
                            this.msg("该 ICCID 未有轨迹文件");
                        }
                } )
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
            getGPSData(start){
                this.loading = true;
                axios.get("/blackview/contrail/track-locations",{params:{iccid: this.iccid.trim(),date:start}}).then(result => {
                    let code = result.code;
                    let gpsData = result.data;
                    this.loading = false;
                    if (code !== 200 || gpsData === "" || gpsData === null) {
                        this.msg ("暂无轨迹数据");
                        this.initMap();
                        return
                    }
                    var strArr = gpsData.split(";");
                    var length = strArr.length-1;
                    var resultStr = "";
                    for (var i = 0; i < length; i++) {
                        var gpsArr = strArr[i].split(",");
                        if (!this.radio){ // 是否开启 辅助定位
                            if(gpsArr[11]  !== "1") {
                                continue; // 去除非 GPS 数据
                            }
                        }
                        var jd = gpsArr[3];
                        var wd = gpsArr[5];
                        resultStr += "[" + wd + "," + jd + "],";
                    }
                    if(resultStr === "") {
                        this.msg ("暂无轨迹数据");
                        this.initMap();
                        return;
                    };
                    resultStr = "[" + resultStr + "]";
                    this.initPage(resultStr);
                })
            },
            getDurationGPSData(start, end){
                this.loading = true;
                axios.get("/blackview/contrail/duration-locations",{params:{iccid: this.iccid.trim(), startTime:start, endTime:end }}).then(result => {
                    this.loading = false;
                    let code = result.code;
                    let gpsData = result.data;
                    if (code !== 200 || gpsData === "" || gpsData === null) {
                        this.msg ("暂无轨迹数据");
                        this.initMap();
                        return
                    }
                    var strArr = gpsData.split(";");
                    var length = strArr.length-1;
                    var resultStr = "";
                    for (var i = 0; i < length; i++) {
                        var gpsArr = strArr[i].split(",");
                        if(gpsArr[11]  !== "1") {
                            continue; // 去除非 GPS 数据
                        }
                        var jd = gpsArr[3];
                        var wd = gpsArr[5];
                        resultStr += "[" + wd + "," + jd + "],";
                    }
                    if(resultStr === "") {
                        this.msg("暂无轨迹数据");
                        this.initMap();
                        return;
                    }
                    resultStr = "[" + resultStr + "]";
                    this.initPage(resultStr);
                })
            },
            listenTrackChildren(){
                let _this = this;
                event.$on("lookTrackChildren", function (val) {
                    _this.iccidOrImei = val.iccid;
                    let trackDate = val.date.substr(0,10);
                    _this.startTime=[trackDate,trackDate];
                });
            },
            initMap(){
                lazyAMapApiLoaderInstance.load().then(() => {
                    this.map = new AMap.Map('amap-show', {
                            center: [114.037939,22.627198],
                            zoom: 11
                        }
                    )
                });
            },
            initPage(data){
                console.log("===== init===");
                lazyAMapApiLoaderInstance.load().then(() => {
                    this.map = new AMap.Map('amap-show', {
                            center: [114.037939,22.627198],
                            zoom: 6
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
                                    pathData.path.length;                                    //返回轨迹数据中的节点坐标信息，[AMap.LngLat, AMap.LngLat...] 或者 [[lng|number,lat|number],...]
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
                                        strokeStyle: 'red',

                                        lineWidth: 3,   //线宽度
                                        dirArrowStyle:{

                                            stepSpace:15  //stepSpace: {number} 箭头排布的间隔，单位像素
                                        }
                                    }
                                }
                            });
                            //这里构建两条简单的轨迹，仅作示例
                            var gpsData = eval(data);
                            // let gpsData =  [[116.405289, 39.904987],[113.964458, 40.54664],[111.47836, 41.135964],[108.949297, 41.670904],[106.380111, 42.149509],[103.774185, 42.56996],[101.135432, 42.930601],[98.46826, 43.229964],[95.777529, 43.466798],[93.068486, 43.64009],[90.34669, 43.749086],[87.61792, 43.793308]];
                            pathSimplifierIns.setData([{
                                name: '轨迹0',
                                path: gpsData
                            }]);

                            var nav = pathSimplifierIns.createPathNavigator(0, {
                                loop: false,
                                speed: 4000,
                                pathNavigatorStyle: {
                                    width: 16,
                                    height: 32,
                                    content: PathSimplifier.Render.Canvas.getImageContent(carUrl, onload, onerror),
                                    strokeStyle: null,
                                    fillStyle: null
                                }
                            });
                            nav.start();
                        })
                    )
                });
            },
        }
    }
</script>

<style scoped>
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
    .inline-input{ width: 260px;}
</style>
