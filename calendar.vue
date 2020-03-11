<template>
  <section class="calendarCtn">
    <div class="title">
      <h4 @click="detail">{{year}}年 {{month}}月</h4>

      <div class="btn" v-show="model === 'month'">
        <el-button size='mini' type="primary" icon="el-icon-arrow-left" @click="btnClick(0)">返回全年</el-button>
        <el-button-group>
          <el-button size='mini' type="primary" icon="el-icon-arrow-left" @click="btnClick(1)" :disabled="month === 1">上个月</el-button>
          <el-button size='mini' type="primary" icon="el-icon-share" @click="btnClick(2)">今天</el-button>
          <el-button size='mini' type="primary" @click="btnClick(3)" :disabled="month === 12">下个月 <i class="el-icon-arrow-right"></i> </el-button>
        </el-button-group>
      </div>
    </div>
    <table>
      <thead>
        <tr class="">
          <td v-for="(v, k) in weekText" :key="k">{{v}}</td>
        </tr>
      </thead>

      <tbody>
        <tr v-for="(arr, k) in tableData" :key="k">
          <td v-for="(date, k2) in arr" :key="k2" @click="dateClick(date)"
            :style="currentDate.getFullYear() == year && currentDate.getMonth() + 1 == month && currentDate.getDate() == date.day ? 'background-color: rgb(255, 224, 183);' : date.holiday ? 'background-color: rgb(227, 255, 235);' : ''">
            <div class="data-box">
              <div>
                <span>{{date.day}} {{date.day === '' ? '' : '日'}}</span>
              </div>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </section>
</template>

<script>
export default {
  components: {
    // x
  },
  props: ['year', 'month', 'calendarData', 'model'],
  computed: {

  },
  data () {
    return {
      weekText: ['日', '一', '二', '三', '四', '五', '六'],
      tableData: [],
      weekOfMonth1: [], // 当前月中的第几周
      weekOfMonth2: [],
      weekOfMonth3: [],
      weekOfMonth4: [],
      weekOfMonth5: [],
      weekOfMonth6: [], // 最多六周
      Interval: null, // 定时器
      currentDate: new Date() // 当前初始化时间
    }
  },
  methods: {
    initInterval () {
      if (!this.calendarData) {
        this.Interval = setInterval(() => {
          this.init()
        }, 300)
      } else {
        this.init()
      }
    },

    init () { // 将传入进来的整月的日期数据按星期分组
      // console.log('全部', this.calendarData)
      clearInterval(this.Interval) // 销毁定时器
      for (let i of this.calendarData) { // 将一个月的数据分成6周
        // i.weekOfMonth - 1
        if (i.weekOfMonth === 1) this.weekOfMonth1.push(i)
        if (i.weekOfMonth === 2) this.weekOfMonth2.push(i)
        if (i.weekOfMonth === 3) this.weekOfMonth3.push(i)
        if (i.weekOfMonth === 4) this.weekOfMonth4.push(i)
        if (i.weekOfMonth === 5) this.weekOfMonth5.push(i)
        if (i.weekOfMonth === 6) this.weekOfMonth6.push(i)
      }

      let obj = { // <周> 的空对象
        id: '',          // 编号：1
        actualDate: '',  // 实际日期：“2020-01-01T00:00:00.000+0800”
        year: '',        // 年份：2020
        month: '',       // 月：1
        day: '',         // 天：1
        dayOfWeek: '',   // 星期三 (一周中的第几天)
        dayOfYear: '',   // 年份：1 (一年中的第几天)
        weekOfMonth: '', // 当前月中的第几周
        weekOfYear: '',  // 当前年中的第几周
        holiday: '',     // 假日：假 (是否节假日)
        createDate: '',  // 创建日期：“2020-01-01T15:20:29.000+0800”
        updateDate: '',  // 更新日期：空
        createUser: '',  // 创建用户：1
        updateUser: ''   // 更新用户：空
      };

      (() => { // 补齐第一周的空位
        let len = 7 - this.weekOfMonth1.length
        for (let i = 0; i < len; i++) {
          this.weekOfMonth1.unshift(obj)
        }
      })();

      (() => { // 补齐最后一周的空位
        if (this.weekOfMonth6.length > 0) { // 最后一周是第六周
          let len = 7 - this.weekOfMonth6.length
          for (let i = 0; i < len; i++) {
            this.weekOfMonth6.push(obj)
          }
        } else if (this.weekOfMonth5.length > 0) { // 最后一周是第五周
          let len = 7 - this.weekOfMonth5.length
          for (let i = 0; i < len; i++) {
            this.weekOfMonth5.push(obj)
          }
        }
      })()

      for (let i = 0; i < 6; i++) { // 把周塞到月的二维数组中
        if (i + 1 === 1 && this.weekOfMonth1.length > 0) this.tableData.push(this.weekOfMonth1)
        if (i + 1 === 2 && this.weekOfMonth2.length > 0) this.tableData.push(this.weekOfMonth2)
        if (i + 1 === 3 && this.weekOfMonth3.length > 0) this.tableData.push(this.weekOfMonth3)
        if (i + 1 === 4 && this.weekOfMonth4.length > 0) this.tableData.push(this.weekOfMonth4)
        if (i + 1 === 5 && this.weekOfMonth5.length > 0) this.tableData.push(this.weekOfMonth5)
        if (i + 1 === 6 && this.weekOfMonth6.length > 0) this.tableData.push(this.weekOfMonth6)
      }
    },
    reset () { // 复位
      this.tableData = []
      this.weekOfMonth1 = [] // 当前月中的第几周
      this.weekOfMonth2 = []
      this.weekOfMonth3 = []
      this.weekOfMonth4 = []
      this.weekOfMonth5 = []
      this.weekOfMonth6 = [] // 最多六周
      clearInterval(this.Interval) // 组件销毁时清除定时器
    },
    dateClick (v) {
      if (!v.id) return
      this.$confirm('确认切换 工作日 / 休息日?', '提示', { confirmButtonText: '确定', cancelButtonText: '取消', type: 'warning' }).then(() => {
        this.submit(v)
      }).catch(() => {
        this.$message({ type: 'info', message: '已取消' })
      })
    },
    submit (v) {
      if (v.holiday) { // 当前是节假日 设置工作日
        this.$api.systemWorkCalendarManageSetAsWeekDay(v.id).then(res => {
          if (res.code === 0) {
            this.$message({ message: res.msg, type: 'success' })
            this.refresh() // 通知父组件刷新
          } else {
            this.$message({ message: res.msg, type: 'warning' })
          }
        })
      } else { //  当前是工作日 设置节假日
        this.$api.systemWorkCalendarManageSetAsHoliday(v.id).then(res => {
          if (res.code === 0) {
            this.$message({ message: res.msg, type: 'success' })
            this.refresh() // 通知父组件刷新
          } else {
            this.$message({ message: res.msg, type: 'warning' })
          }
        })
      }
    },
    btnClick (type) {
      this.$emit('childFn', type)
    },
    // 通知父组件刷新数据
    refresh () {
      this.$emit('childRefresh')
    },
    detail () { // 通知父组件 放大查看某个月份
      if (this.model !== 'month') {
        this.$emit('monthDetail', this.month)
      }
    }
  },
  beforeCreate () {},
  created () {},
  beforeMount () {},
  mounted () {
    this.initInterval()
    // this.$emit('childFn', 2) // 初始到今天
  },
  beforeUpdate () {},
  updated () {},
  beforeDestroy () {
    clearInterval(this.Interval) // 组件销毁时清除定时器
  },
  deactivated () {},
  watch: {
    calendarData () {
      this.reset()
      this.initInterval()
    }
  }

}
</script>

<style scoped lang='scss'>
$borderColor: #ddd;
.calendarCtn{
  width: 100%;
  &>.title{
    padding: 15px;
    border-top: 1px solid $borderColor;
    border-left: 1px solid $borderColor;
    border-right: 1px solid $borderColor;
    h4{
      margin: 0;
      font-size: 20px;
      text-align: center;
      padding: 15px;
      cursor: pointer;
    }
    h4:hover {
      background-color: rgb(215, 235, 255) !important;
    }
    &>.btn{
      display: flex;
      justify-content: space-between;

      padding-top: 10px;
    }
  }
  &>table{
    width: 100%;
    border-collapse:collapse;
    & > thead > tr >td{
      padding: 25px;
      width: 14.2857%;
      text-align: center;
      border: 1px solid $borderColor;
    }
    & > tbody > tr{
      &>td{
        width: 14.2857%;
        padding: 15px 5px;
        text-align: center;
        border: 1px solid $borderColor;
        cursor: pointer;
        & > .data-box{}
      }
      &>td:hover{
        background-color: rgb(215, 235, 255) !important;
      }
    }
  }

}
</style>
