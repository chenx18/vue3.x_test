<template>
    <el-tabs v-model="activeName"  type="border-card" >
        <el-tab-pane name="showTrack" label="轨迹展示">
            <TrackShow></TrackShow>
        </el-tab-pane>
        <el-tab-pane label="轨迹列表">
            <TrackList></TrackList>
        </el-tab-pane>
    </el-tabs>
</template>

<script>
    import trackList from './track-list'
    import trackShow from './track-show'
    import event from './../../utils/event-bus'
    export default {
        components:{  // 共用的车辆菜单组件
            "TrackList": trackList,
            "TrackShow": trackShow,
        },
        data(){
            return  {
                //默认第一个选项卡
                activeName:'showTrack'
            }
        },
        created: function () {
            this.listenTrack();
        },
        methods:{
            listenTrack() {
                let _this = this;
                //显示选中的轨迹
                event.$on("lookTrack", function (val) {
                    _this.activeName = "showTrack";
                    // 将参数传给 子容器
                    event.$emit("lookTrackChildren", val);
                });
            }
        }
    }
</script>

<style scoped>

</style>
