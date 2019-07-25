<template>
    <div>
        <h1>地铁线路实时动态图</h1> {{active}}
        <div class="train">
            <div class="lines" v-for="(lines,index) in all" :key="index">
                <div class="lines_title">{{ lines.line.lineName}}:</div>
                <div class="Train_contains">
                    <div class="stations" :ref="station.tccStationId + index" v-for="(station,index) in lines.stationInfo" :key="index" :style="{width:100/(lines.stationInfo.length) + '%'}">
                        <span class="station_item" >{{ station.stationSname}}</span>
                        <div class="bottom" v-if="index !== lines.stationInfo.length">
                            <div class="bottom1"></div>
                            <div class="circle" @click="get_stationId(station.tccStationId,$event)" :style="{background: active.indexOf(''+ lines.line.lineId + station.tccStationId) > -1? '#3CB371': '#1E90FF'}"></div>
                            <!-- <div class="center" @click="get_stationId(station.tccStationId,$event)" :style="{background: active.indexOf(lines.line.lineId + station.tccStationId)> -1? '#3CB371': '#1E90FF'}"></div> -->
                            <!-- <div class="circle1" @click="get_stationId(station.tccStationId,$event)" :style="{background: active.indexOf(''+ lines.line.lineId + station.tccStationId) > -1 && index === lines.stationInfo.length+1? '#3CB371': '#1E90FF'}"></div> -->
                        </div>
                    </div>
                </div>
                <!-- 车的位置 -->
                <div class="positions">
                    <div class="act-point" v-if="lines.line.lineId === item.lineId" v-for="(item,index) in positions" :style="{left: item.left + 'px', bottom: item.bottom + 'px'}" :title=" '线路Id：' + item.lineId + ',列车编号：' + item.subway + ',车站Id：' + item.stationBId" ></div>
                </div>
            </div> 

        </div>
    </div>
</template>

<script>
import { setInterval } from 'timers';                         
export default {
    data(){
        return{
            all: this.$store.state.parameter.lineStation,
            lines:[],
            stations:[],
            positions: [
                // {left: 0, bottom: 20, lineId: '01', stationAId: '0105', stationBId: '0106', status: 0, subway: 1},
                // {left: 0, bottom: 20, lineId: '01', stationAId: '0124', stationBId: '0125', status: 1, subway: 2},
                {left: 0, bottom: 20, lineId: '02', stationAId: '0202', stationBId: '0201', status: 0, subway: 3},
                // {left: 0, bottom: 20, lineId: '02', stationAId: '0205', stationBId: '0206', status: 1, subway: 3},
                // {left: 0, bottom: 20, lineId: '04', stationAId: '0421', stationBId: '0423', status: 1, subway: 3},
                {left: 0, bottom: 20, lineId: '04', stationAId: '0429', stationBId: '0427', status: 0, subway: 3},
                // {left: 0, bottom: 20, lineId: '05', stationAId: '0521', stationBId: '0523', status: 1, subway: 3},
                // {left: 0, bottom: 20, lineId: '05', stationAId: '0535', stationBId: '0537', status: 0, subway: 3},
            ],
            active: [],
        }
    },
    methods:{
      get_stationId(id,e){
        console.log("id",id,e.target);
        // e.target.style.background = 'red'
        e.target.style.backgroundColor = 'red';
        e.target.style.cursor = 'pointer';
      },
    setPositions() {
      this.active = [];
      this.all.map(line => {
        let _stations = []
        if ( line.stationInfo) {
          line.stationInfo.map(station => {
            _stations.push(station.tccStationId)
          })
        }
        // console.log(_stations)
        // 计算出 left 的初始值
        this.positions.map( v => { 
          if (v.lineId === line.line.lineId) {
              let A_index = _stations.indexOf(v.stationAId)
              let B_index = _stations.indexOf(v.stationBId)
                if (A_index < B_index && B_index <_stations.length+1) {
                  // 正向
                  v.flag = true
                } else  {
                  // 反向
                  v.flag = false
    
                }  
                // 计算A站台到始发站的距离
                let div = this.$refs[v.stationAId+A_index] 
                if (div && div[0]) {
                  // console.log( this.$refs[v.stationAId+A_index][0])
                  let A_offsetLeft = parseInt(this.$refs[v.stationAId + A_index][0].offsetLeft)
                  let B_offsetLeft = parseInt(this.$refs[v.stationBId + B_index][0].offsetLeft)
                  v.left = A_offsetLeft + parseInt(v.status == 0 ? 0 : Math.abs(B_offsetLeft - A_offsetLeft) / 2) - 145- 80
                  v.bottom = v.flag ? 15 : -35;
    
                // console.log('this.$refs', v.lineId, v.stationAId, A_offsetLeft, A_offseWidth, v.left, this.$refs[v.stationAId+A_index][0].left) // 288 /130 -145
                }
                if(v.status == 0 ) {
                  this.active.push('' + v.lineId + v.stationAId);
                }
 
          }
            
        })
      })
        console.log('this.active',this.active )
    },
    // mockDatas() {
    //   let _stations = this.stations.map(v => v.stationName)
    //   this.setPositions();

    //   // 使用定时器模拟 下次路线
    //   this.timer = setInterval(() => {
    //      this.all.map(line => {
    //         let _stations = []
    //         if ( line.stationInfo) {
    //           line.stationInfo.map(station => {
    //             _stations.push(station.tccStationId)
    //           })
    //         }
    //         this.positions.map(v => {
    //            if (v.lineId === line.line.lineId) {
    //              let current_pos = _stations.indexOf(v.stationAId)
    //              let target_pos;
    //              if (current_pos === _stations.length -1) {
    //                  v.flag = false; // 反向
    //              } else if (current_pos === 0) {
    //                v.flag = true;  // 正向
    //              }
    //              if (v.flag) {
    //                  target_pos = current_pos + 1
    //                  // 如果A站是最后一站
    //                  if(target_pos ===_stations.length -1) {
    //                    v.stationBId = _stations[target_pos - 1]

    //                   } else {
    //                    v.stationBId = _stations[target_pos + 1]
    //                  }
    //              } else {
    //                  target_pos = current_pos - 1
    //                  if(target_pos === 0) {
    //                    v.stationBId = _stations[target_pos + 1]
    //                  } else {
    //                    v.stationBId = _stations[target_pos - 1]
    //                  }
   
    //              }
    //              v.stationAId = _stations[target_pos]
   
    //              console.log('下一站',v.stationAId, v.stationBId)
    //            }
    //         })

    //      })
    //     this.setPositions();
    //   }, 1000)

    //   this.$once('hook:beforeDestroy', () => {            
    //       clearInterval(this.timer);                                    
    //   })

    // }

    },
    created(){},
    mounted(){
      this.mockDatas()
        // this.get_lineAndStation();
        this.setPositions();
        console.log("线路和车站：",this.all);
    }
}
</script>

<style type="text/css">
.train{
    margin-top: 20px;
    width:90%;
    /* position: relative;  */
    /* z-index: 9; */
}
.Train_contains{
    width:100%;
    height: 200px;
    /* overflow-x: scroll; */
    white-space:nowrap; 
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
    width: 100%;
    border-bottom: 2px solid #000;
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
    left: -2px;
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
.positions .act-point {
    cursor: pointer;
    position: absolute;
    bottom: 20px;
    left: -6px;
    width: 50px;
    height: 22px;
    background: url('/static/img/tain1.png') no-repeat;
}
</style>
