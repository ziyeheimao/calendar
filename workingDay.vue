
<template>
  <section class="ctn">
    <div class="title">
      <el-form label-width="90px" :model="form" :rules="rules" ref="form"  :inline='true'>
        <el-form-item label="查询年份:" prop="">
          <el-select style='width: 280px;' v-model="form.year" placeholder="请选择年份">
            <el-option v-for="(v, k) in res.ExistingYear" :key="k" :label="v" :value="v"></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="查询月份:" prop="" v-show="model.model[model.index] === 'month'">
          <el-date-picker style='width: 280px;' v-model="month" type="month" placeholder="选择月份"></el-date-picker>
        </el-form-item>

        <el-form-item label="添加年份:" prop="">
          <el-input style='width: 280px;' placeholder="请输入年份" v-model="form.addYear" class="input-with-select">
            <el-button slot="append" icon="el-icon-search" @click='addYear'>添加</el-button>
          </el-input>
        </el-form-item>
      </el-form>

    </div>

    <div class="style">
      <span>今日 <i style="background-color: rgb(255, 224, 183);"></i> </span>
      <span>节假日 <i style="background-color: rgb(227, 255, 235);"></i> </span>
      <span>工作日 <i style="background-color: rgb(255, 255, 255);"></i> </span>
    </div>

    <!-- 年组件 -->
    <div class="year" v-if="model.model[model.index] === 'year' && calendarData2.length === 12">
      <calendar
        v-for="(v, k) in 12" :key="k"
        :model="model.model[model.index]"
        :year='calendarData2[res.monthVal - 1][0].year'
        :month='v'
        :calendarData='calendarData2[v-1]'
        @childRefresh='getYear'
        @childFn='childFn'
        @monthDetail='monthDetail'></calendar>
    </div>

    <!-- 月组件 -->
    <div class="month" v-if="model.model[model.index] === 'month' && calendarData2[res.monthVal - 1]">
      <calendar
        :model="model.model[model.index]"
        :year='calendarData2[res.monthVal - 1][0].year'
        :month='calendarData2[res.monthVal - 1][0].month'
        :calendarData='calendarData2[res.monthVal - 1]'
        @childRefresh='getYear'
        @childFn='childFn'
        @monthDetail='monthDetail'></calendar>
    </div>
  </section>
</template>

<script>
import { clone, openInfo, getVlaue } from '@/utils'
import calendar from './calendar'

export default {
  components: {
    calendar
  },
  // props: [''],
  computed: {

  },
  data () {
    return {
      calendarData: [], // 日历数据
      calendarData2: [], // 日历二维数组
      currentDate: new Date(), // 当前初始化时间
      month: new Date(), // 当前月份:组件用
      form: {
        year: '', // 搜索的年份
        addYear: '' // 添加年份
      },
      res: {
        ExistingYear: [], // 现有的年份
        currentDate: '',
        monthVal: 1 // 当前月份
      },
      rules: {},
      model: {
        model: ['year', 'month'],
        index: 0
      }
    }
  },
  methods: {
    init: async function () {
      this.res.currentDate = this.currentDate.getFullYear() // 当前年份
      this.res.monthVal = this.currentDate.getMonth() + 1 // 当前年份
      await this.getExistingYear()
      this.getYear()
    },
    getExistingYear () { // 获取现有年份
      return new Promise((resolve, reject) => {
        this.$api.systemWorkCalendarManageGetYear().then(res => {
          this.res.ExistingYear = res || []
          if (this.res.ExistingYear.includes(this.res.currentDate)) {
            this.form.year = this.res.currentDate
            resolve()
          }
        })
      })
    },

    getYear () { // 获取默年份
      this.$api.systemWorkCalendarManageFindWorkCalendar(this.form).then(res => {
        if (res.code === 0) {
          this.calendarData = res.bean
          let arr = [[], [], [], [], [], [], [], [], [], [], [], []]

          for (let i of res.bean) {
            if (i.month) {}
            arr[i.month - 1].push(i)
          }

          this.calendarData2 = arr
        } else {
          openInfo(res.msg, 'w')
        }
      })
    },
    childFn (type) { // 子组件事件监听
      switch (type) {
        case 0: // 返回全年
          this.model.index = 0
          break
        case 1: // 上个月
          // console.log(this.month.getMonth() + 1)
          this.month = new Date(`${this.month.getFullYear}/${this.month.getMonth()}/1`)
          break
        case 2: // 今天
          this.init() // 洗牌
          this.month = new Date()
          break
        case 3: // 下个月
          this.month = new Date(`${this.month.getFullYear}/${this.month.getMonth() + 2}/1`)
          break
      }
    },
    monthDetail (month) { // 查看某个月详情
      this.model.index = 1
      this.month = new Date(`${this.month.getFullYear}/${month}/1`)
    },
    addYear () { // 初始化 添加年份
      if (isNaN(Number(this.form.addYear))) {
        this.$message({ message: '年份只能输入数字', type: 'warning' })
        return
      }
      if (this.form.addYear.length !== 4) {
        this.$message({ message: '年份格式为4位数字', type: 'warning' })
        return
      }
      this.$api.systemWorkCalendarManageInitWorkCalendarByYear(this.form.addYear).then(res => {
        if (res.code === 0) {
          this.$message({ message: res.msg, type: 'success' })
          this.init()
        } else {
          this.$message({ message: res.msg, type: 'warning' })
        }
      })
    }
  },
  beforeCreate () {},
  created () {
    this.init()
  },
  beforeMount () {},
  mounted () {
  },
  beforeUpdate () {},
  updated () {},
  beforeDestroy () {},
  deactivated () {},
  watch: {
    month () {
      this.res.monthVal = this.month.getMonth() + 1
    },
    'form.year' () {
      this.month = new Date(`${this.form.year}/1/1`)
      this.getYear()
    }
  }
}
</script>

<style scoped lang='scss'>
.ctn{
  width: 100%;
  &>.title{
    padding: 15px 0 0;
    &>span{
      font-size: 20px;
      font-weight: bold;
      margin: 0 20px;
    }
  }
  &>.style{
    padding: 0 20px 5px;
    &>span{
      margin-right: 15px;
      &>i{
        border: 1px solid #aaa;
        border-radius: 3px;
        padding: 5px 12px;
        width: 10px;
        display: inline-block;
      }
    }
  }
  .year{
    width: 100%;
    &>section{
      padding: 10px;
      display: inline-block;
    }
  }
}
@media screen and (max-width:1280px){
  .year>section{
    width: 100%
  }
}
@media screen and (min-width:1281px) and (max-width:1919px){
  .year>section{
    width: 50%
  }
}
@media screen and (min-width:1920px) and (max-width:2559px){
  .year>section{
    width: 33.33%;
  }
}
@media screen and (min-width:2560px){
  .year>section{
    width: 25%;
  }
}
</style>
