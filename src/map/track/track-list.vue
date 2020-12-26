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
            <span><i>日期：</i>
                 <el-date-picker
                    v-model="searchInfo.date"
                    type="date"
                    value-format="yyyy-MM-dd"
                    clearable
                    placeholder="选择日期">
                 </el-date-picker></span>
            <!--<span><i>里程：</i><el-input v-model="searchInfo.mileage" clearable placeholder="请输入"></el-input></span>-->
            <el-button type="primary" @click="search" style="float: right; position: relative;left: -20px;" >搜索</el-button>
        </div>
        <div class="tableStyle">
            <el-table class="el-table-model" v-loading="loading" :data="tableData" style="width: 100%;border-bottom: 1px solid #ccc;" :row-class-name="tableRowClassName" @sort-change="tableSort">
                <el-table-column prop="iccid" label="ICCID"></el-table-column>
                <el-table-column label="日期" sortable>
                    <template slot-scope="scope">
                        {{ scope.row.date | showDate }}
                    </template>
                </el-table-column>
                <el-table-column prop="mileage" label="里程"></el-table-column>
                <el-table-column label="文件大小">
                    <template slot-scope="scope">
                        {{ scope.row.size | showSize }}
                    </template>
                </el-table-column>
                <el-table-column prop="createTime" label="插入时间" sortable></el-table-column>
                <el-table-column fixed="right" label="操作" width="150">
                    <template slot-scope="scope">
                        <el-button @click="download(scope.row)" type="text" size="small">下载</el-button>
                        <el-button type="text" size="small" @click="listenTrackChildren(scope.row)">查看轨迹</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <el-pagination
                @current-change="handleCurrentChange"
                :current-page.sync="currentPage"
                :page-size="searchInfo.pageSize"
                background
                layout="total, prev, pager, next"
                :total="total">
            </el-pagination>
        </div>
    </div>

</template>

<script>
    import {axios} from "../../config/axios";
    import event from './../../utils/event-bus'
    export default {
        name: 'track-list',
        data () {
            return {
                iccidOrImei:'',
                currentPage:1,
                inputSome: '',
                total:1,
                tableData: [],
                //iccid搜索
                restaurants: [],
                loading: false,
                imei:'',
                searchInfo:{
                    iccid:'',
                    date:'',
                   // mileage:'',
                    pageNumber:1,
                    pageSize:11,
                    sortName: 'id',
                    sortBy: 'desc',
                },
            }
        },
        created: function () {
            // 将选择的 数据 从缓存中读取
            this.iccidOrImei = this.getLocalStorageIccid();
            this.searchInfo.iccid = this.iccidOrImei;
            this.pageInfo();
        },
        methods:{
            search(){
                if (this.iccidOrImei !== ""){
                    let paramsLength = this.iccidOrImei.trim().length;
                    if (paramsLength === 20){
                        this.pageInfo();
                        return;
                    }
                    if (paramsLength === 15){
                        axios.get("/blackview/api/device-dependent/dev-info/getIccidByImei",{params:{imei:this.iccidOrImei}}).then(result => {
                            if (result.code !== 200) {
                                console.error(result);
                            }
                            this.searchInfo.iccid  = result.data;
                            console.log("result:",result);
                            this.pageInfo();
                        });
                        return;
                    }
                    this.msg("输入的 字符串长度："+paramsLength+" ， 不是IMEI也不是ICCID");
                } else {
                    this.pageInfo();
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
            // 排序需要
            tableSort({ column, prop, order }) {
                this.searchInfo.sortName = prop;
                this.searchInfo.sortBy = order;
                this.currentPage = 1;
                this.handleCurrentChange(1);
            },
            tableRowClassName({row, rowIndex}) {
                if (rowIndex%2 ===0) {
                    return 'success-row';
                }
                return '';
            },
            handleCurrentChange(val) {
                this.searchInfo.pageNumber = val -1;
                this.currentPage = val;
                this.pageInfo();
            },
            pageInfo(){
                this.setLocalStorageIccid(this.iccidOrImei);
                if (this.loading)  return; // 防止重复提交
                this.loading=true;
                axios.get('/blackview/contrail',{params: this.searchInfo}).then(result => {
                    this.loading=false;
                    if (result.code !== 200){
                        console.log(result);
                    }
                    let page = result.data;
                    this.total = page.total;
                    this.tableData = page.list;
                }).catch(err => {
                    console.log(err);
                    this.loading=false;
                });
            },
            download(val){
                let downloadUrl = this.downloadTrackUrl+val.date.substr(0,10).replace(/-/g,"/")+"/"+val.iccid+".gz";
                console.log("============",downloadUrl);
                axios.get("/blackview/api/util/existsUrl",{params:{url:downloadUrl}}).then(result => {
                    if (result.code !== 200 ) {
                        console.error("result:",result);
                        return;
                    }
                    if (result.data){
                        window.open(downloadUrl);
                    }else{
                        this.msg("该路径 为空");
                    }
                } )
            },
            listenTrackChildren(val){
                event.$emit("lookTrack", val);
            }
        },
        filters: {
            showDate(val){
                return val.substr(0,10);
            },
            showSize(val){
                if (val < 1024){
                    return "1 KB";
                } else if (val< 1024*1024){
                    return Math.ceil(val/1024) +" KB";
                }else {
                    return Math.ceil(val/1024/1024) + " MB";
                }
            }
        }
    }
</script>
