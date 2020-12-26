<template>
    <div class="myTable">
        <div class="input searchNav searchNav2" >
            <el-autocomplete class="inline-input" v-model="iccidOrImei" :fetch-suggestions="querySearch" placeholder=" 请输入 ICCID 或 IMEI " 
            :trigger-on-focus="false" clearable></el-autocomplete>
            <el-button  type="primary" @click="findPosition" style="float: right; position: relative;left: -20px;" >搜索</el-button>
        </div>
        <div class="amap-page-container" v-loading="loading">
            <el-amap vid="amapDemo" :zoom="zoom" :center="center" class="amap-demo">
                <el-amap-marker v-for="(marker, index) in markers" :position="marker.position" :angle="marker.angle" :events="marker.events" :title="marker.title" :zIndex="marker.zIndex" :content-render="marker.contentRender" :isShow="marker.isShow" :key="index"></el-amap-marker>
            </el-amap>
        </div>
    </div>
</template>

<script>
    import {axios} from "../../config/axios";
    import carUrl from '../../assets/img/car.png'
    export default {
        name: 'dev-position',
        data () {
            return {
                iccid:'',
                iccidOrImei:'',
                zoom: 15,
                online:'离线',
                //iccid搜索
                restaurants: [],
                markers: [],
                center: [114.037651, 22.627568],
                loading: false,
                positionStr:''
            }
        },
        created(){
            let paramsObj = this.$route.params; // 获取路由的参数对象
            if (Object.keys(paramsObj).length !== 0) {
                this.iccidOrImei = paramsObj.iccid;
                this.findPosition();
            }else{
                // 将选择的 数据 从缓存中读取
                this.iccidOrImei = this.getLocalStorageIccid();
            }
        },
        methods:{
            findPosition(){
                if (this.iccidOrImei !== ""){
                    let paramsLength = this.iccidOrImei.trim().length;
                    if (paramsLength === 20){
                        this.iccid = this.iccidOrImei;
                        this.getPosition();
                        return;
                    }
                    if (paramsLength === 15){
                        axios.get("/blackview/api/device-dependent/dev-info/getIccidByImei",{params:{imei:this.iccidOrImei}}).then(result => {
                            if (result.code !== 200) {
                                console.error(result);
                            }
                            this.iccid = result.data;
                            console.log("result:",result);
                            this.getPosition();
                        });
                        return;
                    }
                    this.msg("输入的 字符串长度："+paramsLength+" ，不是 IMEI 也不是ICCID");
                } else {
                    this.msg("请输入 ICCID 或者 IMEI 进行查询");
                }
            },
            querySearch(queryString, cb) {  // 模糊查询
                if (queryString.trim().length > 0) {
                    axios.get('/blackview/api/capture/list/device', {
                        params: {
                            iccid: queryString.trim()
                        }
                    }).then(result => {
                        if (result.code != 200) {
                            console.error(result);
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
            getPosition(){
                this.loading=true;
                this.markers.length = 0; // 清空已经定位的数据
                this.setLocalStorageIccid(this.iccidOrImei);
                axios.get("/blackview/contrail/latest-location",{params:{iccid:this.iccid}}).then(result => {
                    this.loading=false;
                    if (result.code === 404) {
                        this.msg(result.message);
                        return
                    }
                    if (result.code !== 200) {
                        console.error(result);
                        return
                    }
                    var oneLogLat = result.data;
                    if (oneLogLat === null || oneLogLat === '' ) {
                        this.msg("暂无定位");
                        return
                    }
                    this.myMarkers(oneLogLat);
                });
                axios.get("/blackview/contrail/online",{params:{iccid: this.iccid}}).then(result => {
                    if (result.code !== 200 ) {
                        console.error(result);
                        return;
                    }
                    let onlineState = result.data.online;
                    this.online = onlineState?"在线":"离线";
                }).catch(err => {
                    this.loading = false;
                    console.log(err);
                });
            },
            showInfo(carInfo){
                let isShow = this.markers[0].isShow;
                if (isShow) {
                    this.markers[0].contentRender=(h, instance) => {
                        return <div>
                        <a href="javascript:void(0)" onClick={() => { this.showInfo(carInfo)}} > <img src="http://test.blackview4g.com:8280/images/car.png" /></a>
                            </div>
                    }
                    this.markers[0].isShow=false;
                }else {
                    this.markers[0].contentRender=(h, instance) => this.returnPositionHtml(carInfo)
                    this.markers[0].isShow=true;

                }
            },
            returnPositionHtml(carInfo){
                // 将速度 m/s 转成 km/h 同时 保留两位数字
                let speed = (carInfo[8] * 3.6).toFixed(2);
                return <div>
                    <div class="carInfo">
                    <div class="carInfo-header">
                    <div class="carInfo-header-left">
                    <div><span><strong>IMEI：</strong></span><span>{carInfo[1]}</span></div>
                    <div><span><strong>ICCID：</strong></span><span>{carInfo[0]}</span></div>
                    </div>
                    <div class="carInfo-header-right">
                        <div class="carInfo-time">{carInfo[2]}</div>
                        <div class="cencleInfo"><i class="el-icon-circle-close" href="javascript:void(0)" onClick={() => {this.showInfo(carInfo)}} ></i></div>
                    </div>
                    </div>
                    <div class="carInfo-box">
                        <div class="carInfo-txt">
                        <p><span><strong>速度：</strong>{speed}&nbsp;&nbsp;km/h</span><span><strong>状态：</strong>{this.online}</span><span><strong>方向角：</strong>{carInfo[9]}</span></p>
                    <p><span><strong>方位：</strong>{this.positionStr}</span></p>
                    </div>
                    <div class="carInfo-router">
                        <span>
                        <a href="javascript:void(0)" onClick={() => this.routerOther(carInfo[0],'track')} > 轨迹回放</a>
                    </span>
                    <span>
                    <a href="javascript:void(0)" onClick={() => this.routerOther(carInfo[0],'picture')} > 抓拍照片</a>
                    </span>
                    <span>
                    <a href="javascript:void(0)" onClick={() => this.routerOther(carInfo[0],'video')} > 抓拍视频</a>
                    </span>
                    <span>
                    <a href="javascript:void(0)" onClick={() => this.routerOther(carInfo[0],'remote-video')} > 视频直播</a>
                    </span>
                    </div>
                    </div>
                    </div>
                    <div class="car-css"><a href="javascript:void(0)" onClick={() => {this.showInfo(carInfo)}} > <img src="http://test.blackview4g.com:8280/images/car.png" /></a></div>
                    </div>
            },
            routerOther(iccid, url){
                this.$router.push({
                    name: url,  // 路由的名称
                    params: {
                        iccid:iccid
                    }
                })
            },
            getSite(paramsPosition){
                axios.get("/blackview/contrail/change-to-site",{params:{lonlat:paramsPosition}}).then(result => {
                    if (result.code !== 200) {
                        console.error(result);
                        return
                    }
                    this.positionStr = result.data;
                });
            },
            myMarkers(oneLogLat){
                this.markers.length = 0; // 清空上一次位置信息
                var carInfo = oneLogLat.split(",");
                //const dir = carInfo[9]; // 方向角
                let paramsPosition = carInfo[5] + ',' + carInfo[3];
                this.getSite(paramsPosition);
                let position =[carInfo[5],carInfo[3]];
                this.center=position;
                let marker = {
                    position: position,
                    events: {
                        click: () => {
                            console.log("click marker")
                        },
                    },
                    contentRender: (h, instance) => {
                        return this.returnPositionHtml(carInfo);
                    },
                    //angle:dir,
                    title:this.iccid,
                    zIndex:100, //属性使级别较高的点标记在上层显示默认zIndex：100
                    isShow:true,   // 这个属性用于 区别展示的状态  是单独的车还是有车也有汽车信息
                };
                this.markers.push(marker);
            },
        },
    }
</script>

<style scoped >
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
    .amap-page-container {
        height: 740px;
    }
</style>

<style lang="less">
    .carInfo-all{

    }
    .car-css{
        position: fixed;
        top: 0px;
    }
    .carInfo{
        position: fixed;
        top: -235px;
        left: -15px;
        // background-color: #ccc;
        // padding: 10px 15px;
        // line-height: 1.5;
        // margin-bottom: 20px;
        // z-index: 9999;

        width: 460px;
        background-color:rgba(255,255,255,0.85);
        border-radius: 10px;
        cursor: context-menu;
        z-index: 9999;
        font-size: 14px;
        line-height: 24px;
        -moz-box-shadow: 0 0 10px #c2c2c2;
		-webkit-box-shadow: 0 0 10px #c2c2c2;
        box-shadow: 0 0 10px #c2c2c2;
        &:before{
            content: '';
            width: 33px;
            height: 28px;
            position:absolute;
            bottom:-27px;
            left:34px;
            opacity:0.85;
            background: url(../../assets/img/triangle.png) no-repeat;
        }
        .carInfo-header{
            height: 70px; background-color:rgba(14,137,245,0.85); color: #ffffff; border-top-left-radius: 10px;; border-top-right-radius: 10px; position: relative; padding-left: 30px;
        }
        .carInfo-header-left{
            float: left; padding-top: 16px;
            .car-state{
                float: left; color: #fff; margin-right: 15px; text-align: center;
                &:first-child{ text-align: right; padding-top: 3px;}
                .car-state-img{
                    width:23px; height: 19px;
                    // background: url(../../assets/images/ico-car.png) no-repeat;
                }
            }
            .car-txt{ overflow: hidden;}
        }
        .carInfo-header-right{
            float: right;
            .carInfo-time{
                padding-top: 16px; padding-right: 35px; width: 120px; text-align: right;
            }
            .el-icon-circle-close{
                color: #ffffff; cursor: pointer; position: absolute; right: 8px; top: 7px; font-size: 18px;
            }
        }
        .carInfo-box{
            padding:15px 30px 26px 30px;
            .carInfo-txt{
                p{ width: 100%; padding: 0; margin: 0; padding-bottom: 5px;}
                span{ display: inline-block; width: 190px;}
                p:nth-child(2) span{ width:auto;}
            }
            .carInfo-router{
                display: table; width: 100%; padding-top: 10px;
                span{ display: table-cell;}
                a{
                    display: inline-block; text-decoration:none; width: 80px; height: 24px; line-height: 23px; text-align: center; color: #0e89f5; border: 1px solid #0e89f5; border-radius: 50px;
                }
                a:hover{ background-color: #cbe4f9;}
            }
        }
    }
</style>
