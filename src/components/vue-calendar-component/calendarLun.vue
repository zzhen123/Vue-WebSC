<style lang="scss">
  .wh_containerlun {
    width: 450px;
    margin: auto;
    * {
      margin: 0;
      padding: 0;
    }
    li {
      list-style-type: none;
    }

    .wh_content_all {
      font-family: -apple-system, BlinkMacSystemFont, 'PingFang SC', 'Helvetica Neue', STHeiti, 'Microsoft Yahei', Tahoma,
      Simsun, sans-serif;
      background-color: #fff;
      width: 100%;
      overflow: hidden;
      padding-bottom: 25px;
      box-shadow: 0 6px 12px rgba(0,0,0,.175);
      border: 1px solid rgba(0,0,0,.15);
      border-radius: 4px;
      &:after {
        position: absolute;
        top: -9px;
        left: 70px;
        z-index: 23;
        display: inline-block;
        border-right: 7px solid transparent;
        border-bottom: 10px solid #f3f3f3;
        border-left: 7px solid transparent;
        content: '';
      }
    }
    .wh_content {
      display: flex;
      flex-wrap: wrap;
      padding: 0 3% 0 3%;
      width: 100%;
    }
    .weeken{
      background: #f3f3f3;
    }
    .wh_content:first-child .wh_content_item {
      color: #ddd;
      font-size: 16px;
      height: 15%;
      line-height: 15%;
    }

    .wh_content_item {
      font-size: 12px;
      width: 14%;
      padding-bottom: 10.14%;
      text-align: center;
      color: #000;
      position: relative;
      height: 35px;
      div {
        position: absolute;
        top: 0;
        width: 100%;
        height: 100%;
        left: 0;
        display: flex;
        justify-content: center;
        align-items: center;
      }
      li {
        width: 45px;
        height: 40px;
      }
    }
  }
  @media screen and (min-width: 460px) {
    .wh_content_item li:hover {
      background: #F5F5F5;
      cursor: pointer
    }
  }

  .wh_top_changge {
    display: flex;
    li {
      cursor: pointer;
      display: flex;
      color: #000;
      font-size: 18px;
      flex: 1;
      justify-content: center;
      align-items: center;
      height: 47px;
    }
    .wh_content_li {
      cursor: auto;
      flex: 2.5;
    }
  }

  .wh_jiantou1 {
    width: 12px;
    height: 12px;
    border-top: 2px solid #000;
    border-left: 2px solid #000;
    transform: rotate(-45deg);
  }

  .wh_jiantou1:active,
  .wh_jiantou2:active {
    border-color: #ddd;
  }

  .wh_content_item div{
    .wh_isToday {
      margin: auto;
      background-color: #6fb3e0;
      color: #fff;
      text-align: center;
    }
    .wh_isMark {
      margin: auto;
      border-radius: 100px;
      border: 1px solid blue;
      z-index: 2;
    }
    .wh_nextDayShow {
      color: #bfbfbf;
    }
  }
  .isTodayNow{
    background: rgb(255, 255, 204);
  }
  .wh_jiantou2 {
    width: 12px;
    height: 12px;
    border-top: 2px solid #000;
    border-right: 2px solid #000;
    transform: rotate(45deg);
  }
</style>
<template>
  <section class="wh_containerlun">
    <div class="wh_content_all">
      <div class="wh_top_changge">
        <li @click="PreMonth()">
          <div class="wh_jiantou1"></div>
        </li>
        <li class="wh_content_li">{{dateTop}}</li>
        <li @click="NextMonth()">
          <div class="wh_jiantou2"></div>
        </li>
      </div>
      <div class="wh_content weeken">
        <div class="wh_content_item" v-for="tag in textTop">
          <div>
            {{tag}}
          </div>
        </div>
      </div>
      <div class="wh_content">
        <div class="wh_content_item" v-for="(item,index) in list" @click="clickDay(item,index)">
          <div>
            <li class="wh_nextDayShow" v-if="(isHideOtherday&&item.nextDayShow)||item.otherMonth||item.dayHide" :class="{wh_isToday:item.isTodayNow}">
              {{item.id}}<br/>{{item.lun}}
            </li>
            <li v-else="(isHideOtherday&&item.nextDayShow)||item.otherMonth||item.dayHide" :class="{ wh_isToday: item.isToday,wh_isMark:item.isMark,isTodayNow:item.isTodayNow}">
              {{item.id}}<br/>{{item.lun}}
            </li>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
<script>
  import calendarjs from './../../common/calendar'
  export default {
    data() {
      return {
        textTop: ['一', '二', '三', '四', '五', '六', '日'],
        myData: [],
        list: [],
        dateTop: '',
        datatop: new Date(),
      };
    },
    props: {
      markArray: { default: '[]' },
      agoDayHide: { default: '0' },
      futureDayHide: { default: '15181550670000' },
      isHideOtherday: { default: false },
      datalunsign:{ default: ''}
    },
    created() {
      if(this.datalunsign==''){
        this.myData=new Date()
      }else{
        this.myData = new Date(this.datalunsign);
      }
    },
    beforeUpdate: function () {

    },
    methods: {
      clickDay: function (item, index) {
        if (!(this.isHideOtherday && item.nextDayShow) && !item.dayHide) {
          this.$emit('choseDayLun', item.date);
        }
        if (item.otherMonth) {
          item.otherMonth < 0 ? this.PreMonth(item.date) : this.NextMonth(item.date);
        } else {
          if (!(this.isHideOtherday && item.nextDayShow) && !item.dayHide) {
            for (var i = 0; i < this.list.length; i++) {
              if (i == index) {
                this.list[i].isToday = true;
              } else {
                this.list[i].isToday = false;
              }
            }
          }
        }
      },
      ChoseMonth: function (date, isChosedDay = true) {
        date = this.dateFormat(date);
        this.myData = new Date(date);
        this.$emit('changeMonth', this.dateFormat(this.myData));
        this.getList(this.myData, date, isChosedDay);
      },
      PreMonth: function (date, isChosedDay = true) {
        date = this.dateFormat(date);
        this.myData = this.getPreMonth(this.myData);
        this.$emit('changeMonth', this.dateFormat(this.myData));
        this.getList(this.myData, date, isChosedDay);
      },
      NextMonth: function (date, isChosedDay = true) {
        date = this.dateFormat(date);
        this.myData = this.getNextMonth(this.myData);
        this.$emit('changeMonth', this.dateFormat(this.myData));
        this.getList(this.myData, date, isChosedDay);
      },
      getPreMonth: function (date) {
        var timeArray = this.dateFormat(date).split('/');
        var year = timeArray[0];
        var month = timeArray[1];
        var day = timeArray[2];
        var days = new Date(year, month, 0);
        days = days.getDate();
        var year2 = year;
        var month2 = parseInt(month) - 1;
        if (month2 == 0) {
          year2 = parseInt(year2) - 1;
          month2 = 12;
        }
        var day2 = day;
        var days2 = new Date(year2, month2, 0);
        days2 = days2.getDate();
        if (day2 > days2) {
          day2 = days2;
        }
        if (month2 < 10) {
          month2 = '0' + month2;
        }
        var t2 = year2 + '/' + month2 + '/' + day2;
        return new Date(t2);
      },
      getNextMonth: function (date) {
        var arr = this.dateFormat(date).split('/');
        var year = arr[0]; //获取当前日期的年份
        var month = arr[1]; //获取当前日期的月份
        var day = arr[2]; //获取当前日期的日
        var days = new Date(year, month, 0);
        days = days.getDate(); //获取当前日期中的月的天数
        var year2 = year;
        var month2 = parseInt(month) + 1;
        if (month2 == 13) {
          year2 = parseInt(year2) + 1;
          month2 = 1;
        }
        var day2 = day;
        var days2 = new Date(year2, month2, 0);
        days2 = days2.getDate();
        if (day2 > days2) {
          day2 = days2;
        }
        if (month2 < 10) {
          month2 = '0' + month2;
        }

        var t2 = year2 + '/' + month2 + '/' + day2;
        return new Date(t2);
      },
      getDaysInOneMonth: function (date) {//天数
        var getyear = date.getFullYear();
        var getmonth = date.getMonth() + 1;
        getmonth = parseInt(getmonth, 10);
        var d = new Date(getyear, getmonth, 0);
        return d.getDate();
      },
      getMonthweek: function (date) {
        var getyear = date.getFullYear();
        var getmonth = date.getMonth() + 1;
        var dateOne = new Date(getyear + '/' + getmonth + '/1');
        return dateOne.getDay() == 0 ? 6 : dateOne.getDay() - 1;
      },
      getList: function (date, chooseDay, isChosedDay = true) {
        var mygetMonth = date.getMonth() + 1 < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1;
        var mygetDay= date.getDate()< 10 ? '0' + (date.getDate()) : date.getDate();
        this.dateTop = date.getFullYear() + '年' + mygetMonth + '月';
        this.datatop= date.getFullYear() + '-' + mygetMonth + '-'+mygetDay
        var array = [];
        var timenowList=this.dateFormat(new Date())
        var datatimechoose=this.datalunsign==''?this.signFormat(timenowList):this.datalunsign
        for (var i = 0; i < this.getDaysInOneMonth(date); i++) {
          var nowTime = date.getFullYear() + '/' + (date.getMonth() + 1) + '/' + (i + 1);
          var timeList=this.dateFormat(new Date(date.getFullYear() + '/' + (date.getMonth() + 1) + '/' + (i + 1)))
          timeList=this.signFormat(timeList)
          var lunar1=calendarjs.solar2lunar( date.getFullYear(), (date.getMonth() + 1),i+1)
          if (
            datatimechoose == timeList
          ) {
            array = array.concat({
              id: i + 1,
              lun:lunar1.IDayCn,
              date: nowTime,
              isTodayNow: true,
              isToday: true,
              isMark: this.markArray.indexOf(i + 1) >= 0,
              dayHide: new Date(nowTime).getTime() < parseInt(this.agoDayHide) || new Date(nowTime).getTime() > parseInt(this.futureDayHide),
              nextDayShow:
              new Date(nowTime).getTime() >
              new Date().getTime()
            });
            this.$emit(
              'isToday',
              this.dateFormat(new Date(date.getFullYear() + '/' + (date.getMonth() + 1) + '/' + (i + 1)))
            );
          } else {
            array = array.concat({
              id: i + 1,
              lun:lunar1.IDayCn,
              date: nowTime,
              isTodayNow: false,
              isToday: chooseDay == nowTime && isChosedDay,
              isMark: this.markArray.indexOf(i + 1) >= 0,
              dayHide: new Date(nowTime).getTime() < parseInt(this.agoDayHide) || new Date(nowTime).getTime() > parseInt(this.futureDayHide),
              nextDayShow:
              new Date(nowTime).getTime() >
              new Date().getTime()
            });
          }
        }
        var array2 = [];
        var num = this.getDaysInOneMonth(this.getPreMonth(date)) - this.getMonthweek(date) + 1;
        var preDate = this.getPreMonth(date);
        var nextDate = this.getNextMonth(date);
        //上个月多少开始
        for (var i = 0; i < this.getMonthweek(date); i++) {
          var nowTime = preDate.getFullYear() + '/' + (preDate.getMonth() + 1) + '/' + (num + i);
          var lunar=calendarjs.solar2lunar( preDate.getFullYear(), (preDate.getMonth() + 1),num + i)
          array2 = array2.concat(
            {
              id: num + i,
              date: nowTime,
              lun:lunar.IDayCn,
              dayHide: new Date(nowTime).getTime() < parseInt(this.agoDayHide) || new Date(nowTime).getTime() > parseInt(this.futureDayHide),
              nextDayShow:
              new Date(nowTime).getTime() >
              new Date().getTime(),
              otherMonth: -1
            });
        }
        array = array2.concat(array);
        var _length = 7 - array.length % 7;
        if (_length < 7) {
          var nowTime = nextDate.getFullYear() + '/' + (nextDate.getMonth() + 1) + '/' + (i + 1);
          for (let i = 0; i < _length; i++) {
            var lunar=calendarjs.solar2lunar( nextDate.getFullYear(), (nextDate.getMonth() + 1),i+1)
            array.push({
              id: i + 1,
              lun:lunar.IDayCn,
              date: nextDate.getFullYear() + '/' + (nextDate.getMonth() + 1) + '/' + (i + 1),
              dayHide: new Date(nowTime).getTime() < parseInt(this.agoDayHide) || new Date(nowTime).getTime() > parseInt(this.futureDayHide),
              nextDayShow:
              new Date(nowTime).getTime() >
              new Date().getTime(),
              otherMonth: 1
            });
          }
        }
        this.list = array;
      },
      dateFormat: function (date) {
        date = new Date(date)
        return date.getFullYear() + '/' + (date.getMonth() + 1) + '/' + date.getDate();
      },
      signFormat:function (date) {
        var datefor=date.split('/')
        var m;
        var d;
        if(datefor[1]<10){m="0"+datefor[1]}else{m=datefor[1]}
        if(datefor[2]<10){d="0"+datefor[2]}else{d=datefor[2]}
        datefor=datefor[0]+'-'+m+'-'+d
        return datefor
      },
    },
    mounted() {
      this.getList(this.myData);
    },
    watch: {
      markArray(val, oldVal) {
        var list = this.list;
        for (var i = 0; i < list.length; i++) {
          list[i].isMark = false;
          if (list[i].date) {
            for (var n = 0; n < val.length; n++) {
              if (list[i].id == val[n]) {
                list[i].isMark = true;
              }
            }
          }
        }
        this.list = list;
      },
    }
  };
</script>
