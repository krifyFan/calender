<template>
<div id="calendar">
    <!-- 年份 月份 -->
    <div class="month">
        <ul>
            <!--点击会触发pickpre函数，重新刷新当前日期 @click(vue v-on:click缩写) -->
            <!-- <li class="arrow" @click="pickPre(currentYear,currentMonth)">❮</li> -->
            <li class="year-month" @click="pickYear(currentYear,currentMonth)">
                <span class="choose-year">{{ currentYear }}年</span>
                <span class="choose-month">{{ currentMonth }}月</span>
            </li>
            <!-- <li class="arrow" @click="pickNext(currentYear,currentMonth)">❯</li> -->
        </ul>
    </div>
    <!-- 星期 -->
    <ul class="weekdays">
        <li style="color:red">日</li>
        <li>一</li>
        <li>二</li>
        <li>三</li>
        <li>四</li>
        <li>五</li>
        <li style="color:red">六</li>
    </ul>
    <!-- 日期 -->
    <ul class="days">
        <!-- 核心 v-for循环 每一次循环用<li>标签创建一天 -->
        <li v-for="(dayobject,index) in days" :key="index">
            <!--本月-->
            <!--如果不是本月  改变类名加灰色-->
            <span v-if="dayobject.day.getMonth()+1 != currentMonth" class="other-month">{{ dayobject.day.getDate() }}</span>
            <!--是本月-->
            <span v-else ref="dayLi" @click="selectDay($event)"  class="curMonthDay">{{ dayobject.day.getDate() }}
        </li>
    </ul>
</div>
</template>
<script>
export default {
    data() {
        return {
            currentDay: 1,
            // currentMonth: 1,
            // currentYear: 1970,
            currentWeek: 1,
            // 日曆顯示日期數
            showDayNum: 42,
            days: [],
            class: 'active',
            arrWeek: [],
            selectedWeek: [],
            inspectionTimeCon: []
        }
    },
    props:{
        currentMonth: null,
        currentYear: null,
    },
    watch:{
        'currentMonth':function(curVal,oldVal){
            if(curVal){
                this.initData(null)
            }
        },
        'currentYear': function(curVal,oldVal){
            if(curVal){
                this.initData(null)
            }
        }
    },
    created: function() { //在vue初始化时调用
        this.initData(null);
    },
    methods: {
        initData: function(cur) {
            var leftcount = 0; //存放剩余数量
            var date;
            if (cur) {
                date = new Date(cur);
            } else {
                var now = new Date();
                var d = new Date(this.formatDate(this.currentYear, this.currentMonth-1, 1));
                d.setDate(this.showDayNum);
                date = new Date(this.formatDate(this.currentYear, this.currentMonth, 1));
            }
            this.currentDay = date.getDate();

            this.currentWeek = date.getDay(); // 1...6,0
            var str = this.formatDate(this.currentYear, this.currentMonth, this.currentDay);
            this.days.length = 0;
            // 今天是周日，放在第一行第7个位置，前面6个
            //初始化本周
            for (var i = this.currentWeek; i >= 0; i--) {
                var d = new Date(str);
                d.setDate(d.getDate() - i);
                var dayobject = {}; //用一个对象包装Date对象  以便为以后预定功能添加属性
                dayobject.day = d;
                this.days.push(dayobject); //将日期放入data 中的days数组 供页面渲染使用
            }
            //其他周
            for (var i = 1; i < this.showDayNum - this.currentWeek; i++) {
                var d = new Date(str);
                d.setDate(d.getDate() + i);
                var dayobject = {};
                dayobject.day = d;
                this.days.push(dayobject);
            }
        },
        pickYear: function(year, month) {
            alert(year + "," + month);
        },

        // 返回 类似 2016-01-02 格式的字符串
        formatDate: function(year, month, day) {
            var y = year;
            var m = month;
            if (m < 10) m = "0" + m;
            var d = day;
            if (d < 10) d = "0" + d;
            return y + "-" + m + "-" + d
        },

        //取消所有的active
        cancelActive() {
            var arr = this.$refs.dayLi;
            if(arr!=undefined){
                for (var i = 0; i < arr.length; i++) {
                    if (arr[i].getAttribute("class") == 'active') {
                        arr[i].setAttribute('class', '');
                    }
                }
            }
            this.selectedWeek = [];
        },

        //输出所有的高亮显示内容
        outputHeighLight() {
            var arr = this.$refs.dayLi;
            let activeText = [];
            for (var i = 0; i < arr.length; i++) {
                if (arr[i].getAttribute("class") == 'active') {
                    let HeighLightDay = new Date(this.currentYear, this.currentMonth-1, Number(arr[i].innerText));
                    activeText.push(HeighLightDay);
                }
            }
            this.$emit('childByValue', activeText)
        },

        //显示当前点击对象
        selectDay(event) {
            if (event.currentTarget.getAttribute("class") == 'active') {
                event.currentTarget.setAttribute('class', '');
            } else {
                event.currentTarget.setAttribute('class', 'active');
            }
            this.outputHeighLight();
        },
        //间隔模式
        interVal(startNum, endNum, intervalNum) {
            let arr = this.$refs.dayLi; 
            for(let i=0; i<arr.length; i++){
                for(let j=i+1; j< arr.length; j++){
                    if(Number(arr[i].innerText) > Number(arr[j].innerText)){
                        let t = arr[i]
                        arr[i] = arr[j]
                        arr[j] = t
                    }
                }
            }
            if (endNum > arr.length) 
                endNum = arr.length;
            while (Number(startNum)<=Number(endNum)){
                arr[startNum-1].setAttribute("class", "active");
                startNum = Number(startNum) + Number(intervalNum) + Number(1);
            }
        },
        //选择星期
        chooseWeek(arrWeek) {  
            //循环所有的天数
            var arr = this.$refs.dayLi;
            for (var i = 0; i < arr.length; i++) {             
                let t = new Date(this.currentYear, this.currentMonth-1, Number(arr[i].innerText));
                // 判断当前星期在不在列表中          
                if (arrWeek.indexOf(t.getDay()) != -1) {
                    arr[i].setAttribute("class", "active");
                }
            }
        },
        //回显选中的日期
        shwoActiveDate(inspectionTimeCon){
            this.$nextTick(()=>{
                let list = []
                for(let k = 0; k<inspectionTimeCon.length; k++){
                    list.push(new Date(inspectionTimeCon[k].taskTime).getDate())
                }
                let arr = this.$refs.dayLi;          
                for (let i = 0; i < arr.length; i++) {
                    for(let j = 0 ; j<list.length; j++){
                        if(arr[i].innerText==list[j]){
                            arr[i].setAttribute("class", "active");
                        }
                    }
                }
            })
        }
    }
}
</script>

<style scoped>
* {
    box-sizing: border-box;
}

ul {
    list-style-type: none;
}

body {
    font-family: Verdana, sans-serif;
    background: #e8f0f3;
}

#calendar {
    width: 350px;
    margin-top: 20px;
    /* box-shadow: 0 2px 2px 0 rgba(0, 0, 0, 0.14), 0 3px 1px -2px rgba(0, 0, 0, 0.1), 0 1px 5px 0 rgba(0, 0, 0, 0.12); */
    background: url("../../assets/UM/cardBG.png") no-repeat;
    background-size: 100% 100%; 
}

.month {
    width: 100%;
    /* background: #1d5f87; */
}

.month ul {
    margin: 0;
    padding: 0;
    justify-content: space-between;
}

.year-month {
    flex-direction: column;
    align-items: center;
    justify-content: space-around;
    padding-top: 10px;
}

.year-month:hover {
    background: rgba(150, 2, 12, 0.1);
}
.choose-month,.choose-year {
    text-align: center;
    font-size: 1.5rem;
}

.arrow {
    padding: 30px;
}

.arrow:hover {
    background: rgba(100, 2, 12, 0.1);
}

.month ul li {
    color: white;
    font-size: 20px;
    text-transform: uppercase;
    letter-spacing: 3px;
    text-align: center;
}

.weekdays {
    margin: 0;
    padding: 10px 0;
    background-color: #1d5f87;
    display: flex;
    flex-wrap: wrap;
    color: #ffffff;
    justify-content: space-around;
}

.weekdays li {
    display: inline-block;
    width: 13.6%;
    text-align: center;
}

.days {
    padding: 0;
    /* background: #ffffff; */
    margin: 0;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
}

.days li {
    list-style-type: none;
    display: inline-block;
    width: 45px;
    text-align: center;
    line-height: 45px;
    font-size: 1rem;
    color: #FFF;
    cursor: pointer;
}

.days li .curDay {
    border-radius: 50%;
    color: #009bfc;
    font-weight: 700
}

.days li .active {
    border-radius: 50%;
    background: #1d5f87;
    color: #fff;
    display: block;
    height: 32px;
    width: 32px;
    line-height: 32px;
    margin-top: 7px;
    margin-left: 7px;
}

.days li .other-month {
    padding: 5px;
    color: #827979;
}
/* 在屏幕尺寸大于2200px的情况下使用以下样式 */
@media (min-width: 2200px){
    #calendar{
        width: 42vmin;
    }
    .days li{
        width: 6vmin;
        line-height: 6vmin;
        font-size: 1.8vmin;
    }
    .choose-month, .choose-year{
        font-size: 2.5vmin;
    }
    .days li .active{
        height: 5vmin;
        width: 5vmin;
        line-height: 5vmin;
    }
}
</style>
