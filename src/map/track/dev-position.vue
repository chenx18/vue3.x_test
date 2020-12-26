<template>
    <div class="myTable flex-column">
        <div class="input searchNav searchNav2" >
            <el-autocomplete class="inline-input" v-model="iccidOrImei" :fetch-suggestions="querySearch" placeholder=" 请输入 ICCID 或 IMEI " 
            :trigger-on-focus="false" clearable></el-autocomplete>
            <el-button type="primary" @click="findPosition" style="float: right; position: relative;left: -20px;" >搜索</el-button>
        </div>
        <div class="amap-page-container flex1" v-loading="loading">
            <!-- 地图容器 -->
            <template v-for="item in mapOptions" >
                <div v-if="item.selected" :key="item.value" :id="item.value" style="width:100%;height:100%"></div>
            </template>
            <!-- 操作 -->
            <div class="map-tool">
                <el-dropdown trigger="click" @command="handelTileSel">
                    <span class="tool-item" > 
                        {{selectMap.tile.label}} 
                        <i class="el-icon-arrow-down el-icon--right"></i>
                    </span>
                    <el-dropdown-menu slot="dropdown">
                        <el-dropdown-item v-for="(item,index) in selectMap.tiles" :key="item.library" :command="index" @click="handelTileSel(item)">
                            {{item.label}}
                        </el-dropdown-item>
                    </el-dropdown-menu>
                </el-dropdown>
                <el-dropdown trigger="click" @command="handelMapSel">
                    <span class="tool-item"> 
                        {{selectMap.label}} 
                        <i class="el-icon-arrow-down el-icon--right"></i>
                    </span>
                    <el-dropdown-menu slot="dropdown">
                        <el-dropdown-item v-for="(item,index) in mapOptions" :key="item.library" :command="index" @click="handelMapSel(item)">
                            {{item.label}}
                        </el-dropdown-item>
                    </el-dropdown-menu>
                </el-dropdown>
            </div>
            <!-- 弹窗 -->
            <div class="mapInfo" id="mapInfo" ref="mapInfo" style="display:none">
                <div class="info_header">
                    <span class="close" id="close">x</span>
                    <div class="header_item imei">
                        <strong>IMEI：</strong> {{infoData[1]}}
                    </div>
                    <div class="header_item imei">
                        <strong>ICCID：</strong> {{infoData[0]}}
                    </div>
                </div>
                <div class="info_body">
                    <div class="body_item">
                        <strong>时间：</strong> {{infoData[2]}}
                    </div>
                    <div class="body_item flex-row"> 
                        <p class="mr-auto"><strong>速度：</strong> {{(infoData[8] * 3.6).toFixed(2)}} km/h </p>
                        <p><strong>状态：</strong> {{this.online}} </p>
                    </div>
                    <div class="body_item">
                        <strong>方向：</strong> {{infoData[9]}}
                    </div>
                    <div class="body_item">
                        <strong>位置：</strong> {{this.positionStr}}
                    </div>
                </div>
                <ul id="info_footer" ref="info_footer" class="info_footer flex flex-space-between">
                    <li data-name="track">轨迹回放</li>
                    <li data-name="picture">抓拍照片</li>
                    <li data-name="video">抓拍视频</li>
                    <li data-name="remote-video">视频直播</li>
                </ul>
                <div class="info-sharp flex-center"><div></div></div>
            </div>
        </div>
    </div>
</template>

<script>
    import {axios} from "../../config/axios";
    import carUrl from '../../assets/img/car.png'
    import mapConfig from '@/assets/LDMap/config'
    import map_js from '@/assets/LDMap/index'
    export default {
        name: 'dev-position',
        data () {
            return {
                loading: false,
                map: null, 						    // 地图对象
                mapApi: null,					    // 导入地图方法
                mapOptions:[],						// 地图选项
                selectMap: {}, 						// 选择的地图
                markers: [],                        // 添加地图markder
                positionStr:'',                     // 经纬度解析地址
                infoWindow: null,                   // 弹窗对象
                infoData: '',                       // 弹窗信息
                online:'离线',                      // 设备状态
                iccid:'',
                iccidOrImei:'',                     // 搜索输入框
                //iccid搜索
                restaurants: [],
            }
        },
        created(){
            let paramsObj = this.$route.params; // 获取路由的参数对象
            if (Object.keys(paramsObj).length !== 0) {
                this.iccidOrImei = paramsObj.iccid;
                this.findPosition();
            }else{
                // 将选择的 数据 从缓存中读取
                this.iccidOrImei = this.getLocalStorageIccid() || '';
            }
        },
        beforeMount(){
            let option = (mapConfig.map || {}).options || [];
            let map = option.find(e => e.selected)
            map.tile = map.tiles.find(m => m.selected);
            this.mapOptions = option;
            this.selectMap = map;
            this.mapApi = map_js.init(this.selectMap.library);  //加载地图api
            this.initMap();
		},
        methods:{
            
            // 初始化地图
            initMap(){
                this.$nextTick(() => {
                    const { value, tile}  = this.selectMap;
                    this.map = this.mapApi.createMap(value, value, tile.value, this.selectMap);
                    // this.map = this.mapApi.AddTrafficLayer()
					// this.map.zoomend(this.zoomend); // 地图缩放级别
				});
            },

            // 地图类型选择(baidu,gaode)
            handelMapSel(index){
                if(this.mapOptions[index].library === this.selectMap.library) return;
                let val = this.mapOptions[index];
                val.tile = val.tiles.find(m => m.selected);
                this.mapOptions.forEach(item => {
                    item.tile = item.tiles.find(m => m.selected);
                    if(item.library === val.library) item.selected = true;
                    else item.selected = false
                })
                this.selectMap = val;
                this.map.clearOverlays();
                this.mapApi.dispose();
                this.mapApi = null;
                this.map = null;
                this.mapApi = map_js.init(this.selectMap.library);  //加载地图api
                this.initMap();
            },
            // 切换地图底图(Plan:2D平面图, Satellite:卫星图)
			handelTileSel(index) {
                const {tile, tiles} = this.selectMap;
                if(tiles[index].value === tile.value) return;
                let val = this.selectMap.tiles[index]
                this.selectMap.tile = val;
				this.mapApi.setMapType(val.value);
            },
            // 搜索
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
            // 查询iccid
            findPosition(){
                if (this.iccidOrImei !== ""){
                    let paramsLength = this.iccidOrImei.trim().length;
                    if (paramsLength === 20){
                        this.iccid = this.iccidOrImei;
                        this.getPosition();
                        return;
                    }
                    if (paramsLength === 15){
                        axios.get("/blackview/api/device-dependent/dev-info/getIccidByImei",{params:{imei:this.iccidOrImei}}).then(res => {
                            console.log("result:",res);
                            if (res.code == 200) {
                                this.iccid = res.data;
                                this.getPosition();
                            }
                        });
                        return;
                    }
                    this.msg("输入的 字符串长度："+paramsLength+" ，不是 IMEI 也不是ICCID");
                } else {
                    this.msg("请输入 ICCID 或者 IMEI 进行查询");
                }
            },
            // 最后一次定位
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
                    this.AddMarkers(oneLogLat);
                });
                axios.get("/blackview/contrail/online",{params:{iccid: this.iccid}}).then(result => {
                    this.loading=false;
                    if (result.code === 200 ) {
                        let onlineState = result.data.online;
                        this.online = onlineState?"在线":"离线";
                    }
                    
                }).catch(err => {
                    this.loading = false;
                    console.log(err);
                });
            },
            // 地址解析
            getSite(lng, lat){
                this.positionStr = '';
                let params = { lonlat: `${lng},${lat}`};
                axios.get("/blackview/contrail/change-to-site",{params})
                .then(res => {
                    if (res.code == 200) this.positionStr = res.data;
                });
            },

            // 添加覆盖物
            AddMarkers(val){
                this.markers.length = 0; // 清空上一次位置信息
                var carInfo = val.split(",");
                this.getSite(carInfo[5],carInfo[3]);
                let icon = this.mapApi.SetIcon(
                    '/static/img/car.png', this.mapApi.SetSize(16,34), 
                    { imageSize: this.mapApi.SetSize(16,34), anchor: this.mapApi.SetSize(16,34) }, 
                );
                console.log(icon)
                let point = this.mapApi.SetPoint(carInfo[5],carInfo[3])
                let marker = this.mapApi.SetMarker(point,{icon});
                marker.active = false;
                marker.point = point;
                marker.title = this.iccid;
                marker.info = carInfo
                marker.addClick((e)=>{this.markerEvent(e)});
                this.markerEvent(marker)
                this.markers.push(marker,{icon});
                this.map.setBestView({
                    points: [point],
                    markers:[marker], 
                    margin:[60, 60, 60, 60]
                })
            },
            //  覆盖物点击事件
            markerEvent(val){
                console.log(val)
                const content = this.$refs.mapInfo;
                const close = document.getElementById('close');
                if(close) close.onclick = (e) => this.closeInfoWindow(e);
                const footer = this.$refs.info_footer;
                if(footer) footer.onclick = (e) => this.handelInfoMore(e);
                this.infoWindow = this.mapApi.SetInfoWindow(val.point,content, {offset: this.mapApi.SetSize(-25, 0)});
                this.infoWindow.openInfo();
                this.infoData = val.info;
            },

            // 关闭弹窗
            closeInfoWindow(val){
                if(this.infoWindow){
                    this.infoWindow.closInfo();
                }
            },
            // 弹窗更多操作
            handelInfoMore(e) {
                if (e.target.nodeName.toLowerCase() === 'li') {
                    const name = e.target.dataset.name+'';
                    this.$router.push({name, params: {iccid: this.infoData[0]}})
                }
            },
        },
        computed:{
           
        },
        watch:{
           
        }
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
        position: relative;
        /* height: 740px; */
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
