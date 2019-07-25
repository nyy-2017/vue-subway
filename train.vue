<template>
    <div>
        <h1>地铁线路实时动态图</h1>
        <!-- <div>{{active}}</div> -->
        <div class="train">
            <div class="lines" v-for="(lines,index) in all" :value="lines.line.lineId" :key="index">
                <div class="lines_title">{{ lines.line.lineName}}:</div>
                <div class="Train_contains">
                    <div class="stations" :ref="station.tccStationId + index" v-for="(station,index) in lines.stationInfo" :value="station.tccStationId" :key="index" :style="{width:100/(lines.stationInfo.length) + '%'}">
                        <span class="station_item">{{ station.stationSname}}</span>
                        <div class="bottom" v-if="index !== lines.stationInfo.length">
                            <div class="bottom1"></div>
                            <div class="circle" @click="get_stationId(station.tccStationId,$event)" :style="{background: active.indexOf( station.tccStationId) > -1 ? '#3CB371': '#1E90FF'}"></div>
                            <!-- <div class="center"></div> -->
                            <!-- <div class="circle1" @click="get_stationId(station.tccStationId,$event)" :style="{background: active.indexOf( station.tccStationId) > -1 ? '#3CB371': '#1E90FF'}"></div> -->
                        </div>
                    </div>
                </div>
                <!-- 小车的位置 -->
                <div class="positions">
                    <div id="act_point"
                        v-if="lines.line.lineId === item.lineId"
                        v-for="(item,index) in positions" :key="index"
                        :style="{left: item.left + 'px', bottom: item.bottom + 'px'}"
                        :class="{'bg1':item.flag == true,'bg2':item.flag == false }"
                        :title=" '线路Id：' + item.lineId + ',列车编号：' + item.trainId + ',车站Id：' + item.stationBId"></div>
                </div>
            </div> 
        </div>
    </div>
</template>

<script>
import data from './lineAndStation.json';
export default {
    data(){
        return{
            send_NO:'',
            all:data,
            lines:[],
            stations:[],
            positions: [
                // {left: 200, bottom: 20, line: '1号线',lineId:'01', stationAId:'0103', stationBId:'0104', stationA: '苹果园', stationB: '古城路', time:'2019-07-18',locationFlag:'0',trainId: 5},
                // {left: 200, bottom: 20, line: '2号线',lineId:'02', stationAId:'0206', stationBId:'0207', stationA: '宣武门', stationB: '和平门', time:'2019-07-18',locationFlag:'0',trainId: 1},
                // {left: 200, bottom: 20, line: '2号线',lineId:'02', stationAId:'0202', stationBId:'0203', stationA: '车公庄', stationB: '阜成门', time:'2019-07-18',locationFlag:'1', trainId: 2 },
                // {left: 200, bottom: 20, line: '5号线',lineId:'05', stationAId:'0525', stationBId:'0527', stationA: '天通苑南', stationB: '立水桥',time:'2019-07-18',locationFlag:'0', trainId: 3},
                // {left: 200, bottom: 20, line: '5号线',lineId:'05', stationAId:'0541', stationBId:'0539', stationA: '和平里北街', stationB: '和平西桥',time:'2019-07-18',locationFlag:'1', trainId: 4}
            ],
            active: [],
        }
    },
    methods:{
        // get_lineAndStation(){    
        // }
        //点击获取当前车的位置的Id
        get_stationId(id,e){
            console.log("id",id,e.target,e);
            e.target.style.backgroundColor = 'red';
            e.target.style.cursor = 'pointer';
        },
        setPositions() {
            this.active = [];
            this.all.map(line => {
                let _stations = [];
                if ( line.stationInfo) {
                    line.stationInfo.map(station => {
                        _stations.push(station.tccStationId)
                    })
                }
                // 计算出 left 的初始值
                // 判断socket接口里面返回的数组
                this.positions.map( v => { 
                    if (v.lineId === line.line.lineId) {
                        let A_index = _stations.indexOf(v.stationAId)
                        let B_index = _stations.indexOf(v.stationBId)
                        if (A_index < B_index && B_index <_stations.length + 1) {
                            // 正向
                            v.flag = true
                        } else  {
                            // 反向
                            v.flag = false
                        }
                        // 计算A站台到始发站的距离
                        let div = this.$refs[v.stationAId + A_index];
                        if (div && div[0]) {
                            // console.log("div", this.$refs[v.stationAId+A_index][0]); // html
                            let A_offsetLeft = parseInt(this.$refs[v.stationAId + A_index][0].offsetLeft);
                            let B_offsetLeft = parseInt(this.$refs[v.stationBId + B_index][0].offsetLeft);
                            v.left = A_offsetLeft + parseInt(v.locationFlag == 0 ? 0 : Math.abs(B_offsetLeft - A_offsetLeft) / 2);
                            v.bottom = v.flag ? 15 : -35; //小车在 线上正反方向的位置
                            // console.log('this.$refs',A_index, B_index, v.lineId, v.stationAId, v.stationBId, A_offsetLeft, B_offsetLeft, v.left,v.bottom);
                        }
                        if(v.locationFlag == 0 ) {
                            this.active.push(v.stationAId);
                        }

                    }
                })
            })
            // console.log('this.active', this.active );
        },
        /**
         * webstocket 开启长连接
         */
        initWebSocket() {
            //页面刚进入时开启长连接
            // const http_ip = "ws://" + this.$store.state.user.Websocket_BAreaStasrv_ip + ":" + this.$store.state.user.Websocket_BAreaStasrv_port;
            const http_ip = "ws://192.168.25.51:4545";
            this.webstock = new WebSocket(http_ip);
            console.log("ip:",http_ip);
            this.webstock.onopen = this.webstockonopen;
            this.webstock.onmessage = this.webstockonmessage;
            this.webstock.onclose = this.webstockclose;
            this.webstock.onerror = this.webstockonerror;
        },
        /**
         * webstock连接成功
         */
        webstockonopen(onopen) {
            console.log("websocket open 连接成功", onopen);
            // 定义全局变量
            this.send_NO = '0x01000103';
            this.webstock.send(this.send_NO);
        },
        /**
         * webstock接收消息
         */
        webstockonmessage(message) {
            // console.log( '返回的message', message);
            // console.log( '返回2', message.data);
            let returnArr1 = JSON.parse(message.data);
                    console.log( 'returnArr1', returnArr1);
                if(returnArr1.length <=0){
                    return false;
                }else if(returnArr1.length >0 && returnArr1.length <=1 ){
                    returnArr1.map(v=>{
                        v.left = -11, v.bottom = -8,v.flag = true
                    })
                    this.positions = returnArr1;
                    this.setPositions();
                } else{
                    if(this.positions.length >0){
                        returnArr1.map(v=>{
                            v.left = -11, v.bottom = -8,v.flag = true
                        })
                        let new_arr = []; // 放处理后的结果
                        this.positions.map(item => { 
                            let new_list = {};
                                new_list.trainId = item.trainId 
                                new_list.lineId = item.lineId 
                                new_list.left = item.left 
                                new_list.bottom = item.bottom
                                new_list.time = item.time 
                                new_list.stationAId = item.stationAId
                                new_list.stationBId = item.stationBId
                                new_list.locationFlag = item.locationFlag 
                            returnArr1.map(v => {
                                if (item.trainId === v.trainId) {
                                    item.lineId = v.lineId;
                                    item.left = v.left;
                                    item.bottom = v.bottom;
                                    item.time = v.time;
                                    item.stationAId = v.stationAId;
                                    item.stationBId = v.stationBId;
                                    item.locationFlag = v.locationFlag;
                                } else {
                                    new_list.trainId = item.trainId 
                                    new_list.lineId = item.lineId 
                                    new_list.left = item.left 
                                    new_list.bottom = item.bottom
                                    new_list.time = item.time 
                                    new_list.stationAId = item.stationAId
                                    new_list.stationBId = item.stationBId
                                    new_list.locationFlag = item.locationFlag 
                                }
                            })
                            new_arr.push(new_list);
                        })
                        this.positions = new_arr;
                        console.log("new_arr",new_arr);
                        this.setPositions();
                    }
                    else{
                        this.positions = returnArr1;
                        this.setPositions();
                    }
                }
        },
        /**
         * webstock关闭
         */
        webstockclose(event) {
            console.log("websocket close"); //监听关闭
        },
        /**
         * webstock出错
         */
        webstockonerror(error) {
            console.log("websocket error");
            this.webstockclose();//先关闭连接
            this.initWebSocket(); // 开启WebSocket长连接
        },
    },
    created(){},
    mounted(){
        // this.get_lineAndStation();
        this.initWebSocket();
        console.log("线路和车站：",this.$store.state.parameter.lineStation);
        // console.log("线路和车站：",data);
    },
    destroyed() {
　　    //页面销毁时关闭长连接
        if(this.webstock !== null){
            this.webstock.close(); //关闭websocket
        }
    },
}
</script>

<style type="text/css">
.train{
    margin-top: 20px;
    position: relative; 
    z-index: 9;
}
.Train_contains{
    width:100%;
    height: 200px;
    /* overflow-x: scroll; */
    /* border: 1px solid #ccc; */
    border-radius: 4px;
    white-space:nowrap; /* 不换行 */
}
/* 线路 --标题 */
.lines .lines_title{
    margin-bottom:10px;
    margin-top: 20px;
    font-size:16px;
    color:#000;
    font-weight:bold;
}
/* 每条线的所有车站 */
.lines .stations {
    margin-top: 20px;
    display: inline-block;
    /* width: 100px; */
}
/* p:nth-child(even){} //偶数行 */
.lines .stations:nth-child(even) .station_item {
    position: relative;
    top: 100px;
    left:0;
}
.lines .stations .station_item{
    display: inline-block;
    font-size: 12px;
    margin-bottom: 20px;
    word-wrap: break-word;
    word-break: break-all;
    overflow: hidden;
}
/* 小圆圈---以及底部线 */
.lines .stations .bottom {
    position: relative;
    /* width: 100px; */
    border-bottom: 2px solid #000;
    /* border-radius: 4px; */
    margin-top: 15px;
    margin-left:10px;
}
.lines .stations .bottom .bottom1{
    position: relative;
    top: -2px;
    border-bottom: 2px solid green;
    z-index: 0;
}
/* .lines .stations .bottom .center {
    position: absolute;
    right: 35px;
    bottom: -5px;
    content: '';
    display: inline-block;
    width: 12px;
    height: 12px;
    border-radius: 50%;
    background-color: #0089ff;
    opacity: 1;
    z-index: 9;
} */
.lines .stations .bottom .circle {
    position: absolute;
    left: -11px;
    bottom: -8px;
    content: '';
    display: inline-block;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background-color: #0089ff;
    opacity: 1;
}
.lines .stations .bottom .circle:hover {
  cursor: pointer;
}
.stations:nth-last-child(1) .bottom {
  border-width:0; 
  height: 2px;
}
.stations:nth-last-child(1) .bottom .bottom1{
  border-width:0;
  height: 4px; 
}
/* .stations:nth-last-child(2) .bottom .circle1{
    position: absolute;
    right: -20px;
    bottom: -8px;
    content: '';
    display: inline-block;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background-color: #0089ff;
    opacity: 1;
    z-index: 9;
} */
.lines .bottom::after :hover {
    bottom: -8px;
    width: 20px;
    height: 20px;
    opacity: 1;
    cursor: pointer;
}
/* 小车 */
.positions {
    width: 100%;
    position: relative;
    top: -120px;
}
.positions #actpoint {
    position: absolute;
    bottom: 20px;
    left: -6px;
    width: 50px;
    height: 22px;
    /* background: url('/static/img/tain1.png') no-repeat; */
    cursor: pointer;
}
.bg1{
    position: absolute;
    bottom: 20px;
    left: -6px;
    width: 50px;
    height: 22px;
    background: url('/static/img/tain1.png') no-repeat;
    cursor: pointer;
}
.bg2{
    position: absolute;
    bottom: 20px;
    left: -6px;
    width: 50px;
    height: 22px;
    background: url('/static/img/tain.png') no-repeat;
    cursor: pointer;
}
</style>