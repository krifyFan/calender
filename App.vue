<template>
  <div id="app">
      <!-- iview年份和月份选择的插件 -->
     <DatePicker type="month" placeholder="请选择巡检月份" style="width: 100%" v-model="uploadPlan.inspectTime" @on-change="getChooseMonth()"></DatePicker>
     <!-- 三种模式的选择 -->
    <Tabs value="name1">
      <TabPane label="间隔模式" name="name1">
        <div>
          开始日期：
          <input  v-model="intervalMode.startDay" type="number" min="1" max="31" value="1">
          结束日期：<input  v-model="intervalMode.endDay" type="number" min="1" max="31" value="30">
          间隔天数：
          <input  v-model="intervalMode.interval" type="number" min="1" max="31" value="1">
          <Button type="default" @click="getDay" class="tabBtn">确定</Button>
          <Button type="default" @click="defaultBtn" class="tabBtn">重置</Button>
        </div>
      </TabPane>
      <TabPane label="星期模式" name="name2">
        <Checkbox v-for="(item, index) in weekDay" v-model="item.value" :value='item.value' :key='index' @on-change="getWeek">{{item.name}}</Checkbox>
      </TabPane>
      <TabPane label="自定义" name="name3" style="text-align: center">
        请选择您需要的时间
      </TabPane>
    </Tabs>
    <!-- 日历插件 -->
    <calender ref="calender" :currentMonth="currMonth"  :currentYear="currYear" v-on:childByValue="getActiveText" style="margin: 10px auto;margin-bottom: 0px;"></calender>
  </div>
</template>

<script>
import calender from "./components/calender.vue";
export default {
  name: "App",
  components: {
    calender
  },
  data(){
    return{
      currMonth: '',
      currYear: '',
      weekDay:[
        {
          name:"星期日",
          value: false
        },
        {
          name:"星期一",
          value: false
        },
        {
          name:"星期二",
          value: false
        },
        {
          name:"星期三",
          value: false
        },
        {
          name:"星期四",
          value: false
        },
        {
          name:"星期五",
          value: false
        },
        {
          name:"星期六",
          value: false
        }
      ],
      intervalMode:{
        startDay: 1,
        endDay: 31,
        interval: 3
      },
      //用来存储选中的日期，传给后台
      uploadPlan:{
        tasks:[]
      }
    }
  },
  mounted(){
    this.getChooseMonth()
  },
  methods:{
    //获取所选年份和月份
    getChooseMonth(){
      if(this.currMonth==undefined||this.currMonth==null||this.currMonth==''||this.currYear==undefined||this.currYear==null||this.currYear==''){
        this.currMonth=new Date().getMonth()+1
        this.currYear = new Date().getFullYear()
      }else{
        this.currMonth=this.uploadPlan.inspectTime.getMonth()+1
        this.currYear=this.uploadPlan.inspectTime.getFullYear()
      }
    },
    //间隔模式 输入的开始时间、结束时间、间隔天数
    getDay() {
      this.$refs.calender.cancelActive();
      this.$refs.calender.interVal(this.intervalMode.startDay, this.intervalMode.endDay, this.intervalMode.interval);
      this.$refs.calender.outputHeighLight();
    },
    //间隔模式的取消按钮
    defaultBtn() {
      this.$refs.calender.cancelActive();
      this.$refs.calender.outputHeighLight();
    },
    //获取选中的星期数
    getWeek(){
      this.$refs.calender.cancelActive();
      var week = [];
      for (let index = 0; index < this.weekDay.length; index++) {
        if (this.weekDay[index].value) week.push(index);
      }
      this.$refs.calender.chooseWeek(week);
      this.$refs.calender.outputHeighLight();
    },
    //获取选中的时间
    getActiveText(childValue){
      this.uploadPlan.tasks=[];
      for(var i=0;i<childValue.length;i++){
          this.uploadPlan.tasks.push({taskTime:childValue[i]})
      }
    },
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  /* margin-top: 20px; */
  width: 500px;
  margin: 20px auto;
}
</style>
