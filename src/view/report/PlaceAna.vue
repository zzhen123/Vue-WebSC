<template>
  <div id="PlaceAna">
    <b-container fluid>
      <!-- User Interface controls -->
      <b-row>
        <b-col sm="4" class="my-1">
          <el-date-picker v-show="!isYear" v-model="reportdate" type="month" placeholder="选择月" :picker-options="datepickerOptions">
          </el-date-picker>
          <el-date-picker v-show="isYear" v-model="reportdate" type="year" placeholder="选择年" :picker-options="datepickerOptions">
          </el-date-picker>
        </b-col>
        <b-col sm="4" class="my-1">
          <el-switch v-model="isYear" @change="typeChange"
            active-text="年报"
            inactive-text="月报">
          </el-switch>
        </b-col>
        <b-col sm="4" class="my-1">
          <b-form-group class="mb-0">
            <b-button @click="getreportdata" variant="primary">查询</b-button>
            <b-button @click="exportexcel" variant="success">导出</b-button>
          </b-form-group>
        </b-col>
      </b-row>
      <el-table
        id="datatable"
        ref="datatable"
        :data="reportDatas"
        border
        show-summary
        :summary-method="getSummaries"
        :row-class-name="tableRowClassName"
        style="width: 100%" :max-height="tableHeight">
        <el-table-column
          v-for="item in fildes"
          :prop="item.prop"
          :label="item.label"
          :width="item.width"
          :sortable="item.sortable"
          :class-name="item.classname"
          :show-overflow-tooltip="item.showTip" :key="item.prop">
        </el-table-column>
      </el-table>
    </b-container>
  </div>
</template>

<script>
  import XLSX from 'xlsx'
  import Vue from 'vue'
  import { mapGetters, mapMutations } from 'vuex'
  import methodinfo from '../../config/MethodConst'

  let reportDatas = []
  const fildes = [
    {  prop: 'code', label:  '场地代码',width:'90',sortable:false ,showTip:true},
    {  prop: 'descript', label:  '场地名称',width:'',sortable:false,showTip:true},
    {  prop: 'year', label:  '年',width:'50',sortable:false},
    {  prop: 'month', label:  '月',width:'30',sortable:false},
    {  prop: 'covers', label:  '容纳数',width:'60',sortable:false,"classname":"text-right"},
    {  prop: 'number', label:  '场次',width:'50',sortable:false,"classname":"text-right" },
    {  prop: 'people', label:  '人数',width:'50',sortable:false,"classname":"text-right" },
    {  prop: 'userate', label:  '利用率',width:'60',sortable:false,"classname":"text-right"},
    {  prop: 'amount', label:  '营收',width:'90',sortable:false,"classname":"text-right" },
    {  prop: 'avgamount', label:  '场均营收',width:'90',sortable:false,"classname":"text-right" },
    {  prop: 'lastnumber', label:  '同期场次',width:'80',sortable:false,"classname":"text-right" },
    {  prop: 'lastpeople', label:  '同期人数',width:'80',sortable:false,"classname":"text-right" },
    {  prop: 'lastuserate', label:  '同期利用率',width:'100',sortable:false,"classname":"text-right"},
    {  prop: 'lastamount', label:  '同期营收',width:'80',sortable:false,"classname":"text-right" },
    {  prop: 'lastavgamount', label:  '同期场均营收',width:'100',sortable:false,"classname":"text-right" }
  ]

  export default {
    data () {
      return {
        reportDatas:reportDatas,
        fildes :fildes,
        totalRows: reportDatas.length,
        sortBy: null,
        sortDesc: false,
        isYear:false,
        saleid: '',
        eloptions: [],
        reportdate: '',
        sums: [],
        tableHeight: document.body.clientHeight-190,//减去header的60px
      }
    },
    computed: {
      ...mapGetters([
      ]),
      datepickerOptions() {
        return {
          disabledDate:this.getDisableDate
        }
      },
    },
    methods: {
      exportexcel:function () {
        //跳过json的key,手工添加输出标题和合计行
        let sheetopt = {skipHeader:true};
        let exportitems = this.reportDatas.slice(0);
        let fia = {};
        for(let items of this.fi ){
          fia[items.prop] = items.label;
        }
        let sums = {};
        let i=0;
        for(let key in fia ){
          sums[key] = this.sums[i];
          i++;
        }
        exportitems.push(sums);
        exportitems.unshift(fia);

        //json转换为表格
        let worksheet = XLSX.utils.json_to_sheet(exportitems,sheetopt);
        worksheet['!cols'] = [{width:10},{width:15},{width:7},{width:7},{width:10},{width:10},{width:10},{width:10},{width:10},{width:10},{width:10},{width:10},{width:13},{width:10},{width:14}]
        let new_workbook = XLSX.utils.book_new();
        XLSX.utils.book_append_sheet(new_workbook, worksheet, "Sheet1");
        /* generate file and force a download*/
        XLSX.writeFile(new_workbook, "PlaceAna.xlsx");
      },
      getreportdata(){
        if(!this.reportdate){
          let message="请选择报表查询月份"
          if(this.isYear){
            message="请选择报表查询年"
          }
          this.$message.error({
            message:message
          });
        }
        else{
          let year = this.reportdate.getFullYear();
          let month = this.reportdate.getMonth()+1;
          this.$store.dispatch('encrypttoken').then(() => {
            this.$http.defaults.headers.common['username'] = this.$store.getters.username
            this.$http.defaults.headers.common['signature'] = this.$store.getters.signature
            this.$http.defaults.headers.common['timestamp'] = new Date().getTime();

            let method = methodinfo.getplaceanadata;
            if(this.isYear){
              method = methodinfo.getplaceanadatabyyear;
            }

            this.$http.post(method, {
              year:year,
              month:month
            }).then((response)=> {
              if (response.data.errorCode==="0") {
                this.reportDatas = response.data.placeanadata;
                this.reportDatas.forEach((data, index)=>{
                  data.userate = (Math.round(data.userate * 10000) / 100) + '%';
                  data.lastuserate = (Math.round(data.lastuserate * 10000) / 100) + '%';
                })
              }
              else{
                this.reportDatas = [];
              }
            })
          })
        }
      },
      getDisableDate(time){
        return time > new Date();
      },
      getSummaries(param){
        const { columns, data } = param;
        const sums = [];
        columns.forEach((column, index) => {
          if (index === 0) {
            sums[index] = '合计:';
            return;
          } else if (index < 4) {
            sums[index] = '';
            return;
          } else {
            const values = data.map(item => Number(item[column.property]));
            if (!values.every(value => isNaN(value))) {
              sums[index] = values.reduce((prev, curr) => {
                let value = Number(curr);
                if (!isNaN(value)) {
                  value =  prev + curr;
                } else {
                  value =  prev;
                }
                if([8,9,13,14].indexOf(index)>=0){
                  return value.toFixed(2);
                }else{
                  return value;
                }
              }, 0);
            }
          }
        });
        let cover = sums[4];
        let num = sums[5];
        let people = sums[6];
        let lastnum = sums[10];
        let lastpeople = sums[11];

        let userate = (people/num)/cover;
        let lastuserate = (lastpeople/lastnum)/cover;

        sums[7] = (Math.round(userate * 10000) / 100).toFixed(0) + '%';
        sums[12] = (Math.round(lastuserate * 10000) / 100).toFixed(0) + '%';

        this.sums = sums;
        return sums;
      },
      tableRowClassName({row, rowIndex}) {
        if (row.code === '合计') {
          return 'success-row';
        }
        return '';
      },
      typeChange(){
        if(this.isYear){

        }
        this.reportDatas = [];
      }
    },
    watch:{
      reportdate(val,oldval){
        if(val!==oldval){
          this.reportDatas = [];
        }
      }
    },
    created(){
    }
  }
</script>
<style lang="scss"  type="text/scss">
  #PlaceAna{
    thead{
      .cell{
        text-align:center !important;
      }
    }
    .el-date-editor .el-range-separator{
      padding: 0;
    }
    caption{
      caption-side: top;width: 100%;
      span{
        padding-left:5px
      }
    }
    .btn{
      width: 100px;
    }
    .table{
      border-color: #dee2e6;
      th{
        border-color: #dee2e6;
      }
    }
    .el-input__inner{
      height: 36px;
    }
    .el-table td, .el-table th{
      padding: 0;
    }
    .el-table__header-wrapper.el-table td, .el-table th{
      padding: 0;
      background: linear-gradient(#fff, #F4F5F6);
    }
    .el-table .caret-wrapper{
      width: 20px;
    }
    .row{
      margin-right: 0;
    }
    .el-table .warning-row {
      background: #e5e5e5;
    }
    .el-switch{
      padding-top: 8px;
    }
    .el-table .success-row {
      background: #CCC;
    }
  }
</style>
