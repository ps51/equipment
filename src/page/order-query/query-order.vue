<template>
  <div class="query-order">
    <div class="count-order">
      <div class="search-condition" ref="countOrder">
        <el-form ref="form" labelPosition="left" label-width="100px" size="large">
          <el-row :gutter="30">
            <el-col :span="12">
              <el-form-item label="订单号">
                <el-input v-model="form.serialNumber" maxlength="30"></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="落地票号">
                <el-input v-model="form.qrInfo" maxlength="30"></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="6">
              <el-form-item label="彩种">
                <el-select v-model="form.subPlayType" placeholder="请选择">
                  <el-option
                    v-for="item in subPlayTypeSelect"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
                  </el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="出票状态">
                <el-select v-model="form.printFlag" placeholder="请选择">
                  <el-option
                    v-for="item in printFlagSelect"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
                  </el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="中奖状态">
                <el-select v-model="form.awardFlag" placeholder="请选择">
                  <el-option
                    v-for="item in awardFlagSelect"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
                  </el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="结算状态">
                <el-select v-model="form.settleStatus" placeholder="请选择">
                  <el-option
                    v-for="item in settleFlagSelect"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
                  </el-option>
                </el-select>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="30">
            <el-col :span="12">
              <el-form-item label="出票时间" class="creatTime">
                <el-date-picker
                  v-model="createDate"
                  type="datetimerange"
                  range-separator="至"
                  start-placeholder="选择开始时间"
                  end-placeholder="选择结束时间"
                  :default-time="['00:00:00', '23:59:59']"
                  format="yyyy-MM-dd"
                  value-format="yyyy-MM-dd HH:mm:ss"
                  class="tick-time">
                </el-date-picker>
              </el-form-item>
            </el-col>
            <el-col :span="12" class="btn-box">
              <el-button @click="empty" type="success">清空</el-button>
              <el-button @click="queryOrder" type="primary">查询</el-button>
            </el-col>
          </el-row>
        </el-form>
      </div>
    </div>

    <!-- 查询列表 -->
    <el-table
      :height="winHeight"
      ref="multipleTable"
      :data="tableData"
      tooltip-effect="dark"
      border
      style="width: 100%"
      v-loading="loading"
      element-loading-text="拼命加载中..."
      element-loading-spinner="el-icon-loading"
      @row-click="getOutPopover">
      <el-table-column v-for="(item, index) in tableColumn"
        :key="index"
        :prop="item.prop"
        :label="item.label"
        :min-width="item.width"
        align="center">
      </el-table-column>
    </el-table>

    <!-- 查询列表 -->
    <el-pagination
      v-if="tableData.length"
      class="page"
      background
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pageIndex"
      :page-size="pageSize"
      :page-sizes="[10, 50, 100, 200, 300, 500]"
      layout="total, sizes, prev, pager, next, jumper"
      :total="totalCount">
    </el-pagination>

    <!-- 票详情页弹出框 -->
    <el-dialog
      :visible.sync="showOutPopover"
      width="80%"
      center
      class="orderNum-popover"
      id="outPopover"
      title="订单详情">
      <div class="hoverContent">
        <el-row :gutter="10">
          <el-col :span="12">出票店铺：<div class="grid-content">{{orderInfo.storeName}}</div></el-col>
          <el-col :span="12">系统票号：<div class="grid-content">{{orderInfo.ticketInfoNumber || '暂无'}}</div></el-col>
        </el-row>
        <el-row :gutter="10">
          <el-col :span="12">出票时间：<div class="grid-content">{{orderInfo.uploadTime}}</div></el-col>
          <el-col :span="12">最迟出票时间：<div class="grid-content">{{orderInfo.lastPrintDate}}</div></el-col>
        </el-row>
        <el-row class="tip" :gutter="20">
          <el-col :span="5">彩种：<div class="grid-content">{{orderInfo.lotterykinds}}</div></el-col>
          <el-col :span="5">过关方式：<div class="grid-content">{{orderInfo.betTypeWord}}</div></el-col>
          <el-col :span="4">倍数：<div class="grid-content">{{orderInfo.multiple}}倍</div></el-col>
          <el-col :span="5">金额：<div class="grid-content">{{orderInfo.amount}}元</div></el-col>
          <el-col :span="5">预计奖金：<div class="grid-content red">{{orderInfo.maxMoney || 0.00}}元</div></el-col>
        </el-row>
      </div>
      <div class="contentBox">
        <el-table :data="hoverData" border style="width: 70%">
          <el-table-column v-for="(item, index) in hoverTableColumn"
            :key="index"
            :prop="item.prop"
            :label="item.label"
            :width="item.width"
            align="center">
          </el-table-column>
          <el-table-column
            prop="assumption"
            label="预设"
            align="center"
            v-if="orderInfo.assumptionShow">
          </el-table-column>
          <el-table-column prop="assumption" label="投注项" width="240" align="center">
            <template slot-scope="scope">
              <span>{{scope.row.subPlayTypeWord}}</span>
              <span v-for="(item1, index1) in scope.row.betItemsObj" :key="index1">[{{item1.key}}&nbsp;({{item1.odds}})]</span>
            </template>
          </el-table-column>
        </el-table>
        <div class="uploadImg">
          <img class="img" :src="imgStr" alt="" @click="enlarge">
        </div>
      </div>
    </el-dialog>

    <!-- 图片放大功能 -->
    <div class="Mask" v-if="Mask" @click="maskClick"></div>
    <div class="enlarge" v-if="enlargeImg">
      <img :src="imgStr" alt="" @click="narrow">
    </div>
    <i-dialog-error :dialogVisible="errorDialogVisible" :content="errorContent"></i-dialog-error>
  </div>
</template>

<script>
import {mul} from '../../utils/fastCombine'
import ChangeBetContext from '../../utils/changeBetContext.js'
import req from '../../api/order-list/index.js'
import iDialogError from '../../components/common/i-dialog-error'
export default {
  components: {
    iDialogError
  },
  data () {
    return {
      tableColumn: [
        {prop: 'serialNumber', label: '系统编号', width: '250'},
        {prop: 'typeWords', label: '彩种类型', width: '130'},
        {prop: 'multiple', label: '倍数', width: '80'},
        {prop: 'amountWord', label: '金额', width: '100'},
        {prop: 'awardAmountWord', label: '奖金', width: '100'},
        {prop: 'awardFlagWord', label: '中奖', width: '80'},
        {prop: 'printFlagWord', label: '出票', width: '90'},
        {prop: 'settleStatusWord', label: '结算', width: '100'}
      ],
      // 列表数据
      tableData: [],
      pageIndex: 1,
      pageSize: 10,
      // 列表总条数
      totalCount: 0,
      // 控制票详情页是否打开
      showOutPopover: false,
      // 票详情页的表格表头
      hoverTableColumn: [
        {prop: 'orderNum', label: '编号'},
        {prop: 'host', label: '主队'},
        {prop: 'guest', label: '客队'}
      ],
      // 票详情页的表格数据
      hoverData: [],
      // 票详情页的基本信息
      orderInfo: {},
      // 扫码获取的落地票号
      scanTicket: '',
      // 扫码枪的定时器
      timer: null,
      // 图片
      imgStr: '',
      Mask: false,
      enlargeImg: false,
      loading: false,
      // 查询的表单数据
      form: {
        // 订单号
        serialNumber: '',
        // 落地票号
        qrInfo: '',
        // 彩种
        subPlayType: '',
        // 出票状态
        printFlag: '',
        // 中奖状态
        awardFlag: '',
        // 结算状态
        settleStatus: '',
        // 出票时间
        beginUploadTime: '',
        endUploadTime: ''
      },
      // 彩种
      subPlayTypeSelect: [
        {value: '51', label: '足球胜平负'},
        {value: '52', label: '足球比分'},
        {value: '53', label: '足球总进球'},
        {value: '54', label: '足球半全场'},
        {value: '56', label: '足球让球胜平负'},
        {value: '59', label: '足球混合过关'},
        {value: '61', label: '篮球让分胜负'},
        {value: '62', label: '篮球胜负'},
        {value: '63', label: '篮球胜分差'},
        {value: '64', label: '篮球大小分'},
        {value: '69', label: '篮球混合过关'}
      ],
      // 出票状态
      printFlagSelect: [
        {value: 2, label: '已出票'},
        {value: 3, label: '出票失败'}
      ],
      // 中奖状态
      awardFlagSelect: [
        {value: 1, label: '已中奖'},
        {value: 2, label: '未中奖'}
      ],
      // 结算状态
      settleFlagSelect: [
        {value: 0, label: '未可结算'},
        {value: 1, label: '可结算'},
        {value: 2, label: '待审核'},
        {value: 3, label: '审核通过'}
      ],
      searchFlag: false,
      // 创建时间
      createDate: [],
      // 落地票号
      lastqrInfo: '',
      // 屏幕高度
      winHeight: 0,
      // 系统票号
      ticketInfoNumber: '',
      // 扫描枪初始化标志
      scanInitFlag: true,
      errorContent: '',
      errorDialogVisible: false
    }
  },
  watch: {
    // 监控扫码枪扫码获取的落地票号
    scanTicket (val) {
      console.log(val, '落地票号')
      this.$nextTick(() => {
        this.$set(this.form, 'qrInfo', val)
        this.queryOrder()
      })
    },
    // 出票时间
    createDate (val) {
      if (val && val.length) {
        this.form.beginUploadTime = val[0]
        this.form.endUploadTime = val[1]
      }
    }
  },
  created () {
    if (!this.$store.state.setActiveIndex) {
      this.$store.commit('setActiveIndex', localStorage.getItem('setActiveIndex'))
    }
    let setMenuDisabled = {
      orderList: false,
      accountOrder: false,
      queryOrder: false,
      quitSystem: false
    }
    this.$store.commit('setMenuDisabled', setMenuDisabled)
    localStorage.setItem('setMenuDisabled', JSON.stringify(setMenuDisabled))
    this.getData()
    document.addEventListener('click', () => {
      this.showOutPopover = false
    })
  },
  mounted () {
    this.$nextTick(() => {
      this.winHeight = localStorage.getItem('winHeight') - 250 - this.$refs.countOrder.offsetHeight
    })
    document.getElementById('outPopover').addEventListener('click', (event) => {
      event.stopPropagation()
    })
    const _this = this
    document.onkeydown = function (e) {
      // console.log('获取', e.keyCode)
      if (e.keyCode === 144) {
        return
      }
      switch (e.keyCode) {
        case 0:
          if (_this.errorDialogVisible) {
            _this.errorDialogVisible = false
            return
          }
          if (_this.$store.state.managerFlag || _this.$store.state.originalPrizeFlag || _this.$store.state.prizeExitFlag) {
            _this.$store.commit('setkeyboardCode', 0)
          } else {
            _this.queryOrder()
          }
          break
        case 111:
          _this.$store.commit('setkeyboardCode', 111)
          break
        case 106:
          _this.$store.commit('setkeyboardCode', 106)
          break
        case 107:
          _this.$store.commit('setkeyboardCode', 107)
          return false
          break // eslint-disable-line
        case 109:
          _this.$store.commit('setkeyboardCode', 109)
          return false
          break // eslint-disable-line
          break
        default:
          break
      }
    }
  },
  methods: {
    // 获取列表
    getData () {
      this.searchFlag || Object.keys(this.form).map(key => {
        this.form[key] = ''
      })
      let nullFlag = false
      Object.keys(this.form).map(key => {
        this.form[key] !== '' && (nullFlag = true)
      })
      nullFlag || (this.searchFlag = false)
      let memberParams = {
        ...this.form,
        page: this.pageIndex,
        pageSize: this.pageSize,
        printFlags: '1,2,3'
      }
      this.loading = true
      req('getOrderList', memberParams)
        .then(res => {
          if (res.code === '00000') {
            this.loading = false
            // 表格数据
            res.data.orderList.result.map(val => {
              val.lotteryTypeWord = ChangeBetContext.lotteryType(val.lotteryType)
              val.changeSettleStatus = val.settleStatus
              val.subPlayTypeWord = ChangeBetContext.subPlayType(val.subPlayType)
              val.settleStatusWord = ChangeBetContext.settleStatus(val.changeSettleStatus)
              val.printFlagWord = ChangeBetContext.printFlag(val.printFlag)
              val.amount = val.amount / 100
              val.calAwardAmount = val.calAwardAmount / 100
              val.awardFlagWord = ChangeBetContext.awardFlag(val.awardFlag)
              val.amountWord = (val.amount).toFixed(2)
              val.awardAmountWord = (val.calAwardAmount).toFixed(2)
              val.flag = false
              val.typeWords = `${val.lotteryTypeWord}${val.subPlayTypeWord}`
            })
            this.tableData = res.data.orderList.result
            this.totalCount = res.data.orderList.totalCount
            if (this.scanInitFlag) {
              try {
                this.scan()
              } catch (error) {
                console.log('条码枪扫描错误', error)
              }
              this.scanInitFlag = false
            }
          } else {
            this.$message({
              type: 'error',
              message: res.msg
            })
          }
        })
        .catch(error => {
          console.log(error)
        })
    },
    // 查询列表
    queryOrder () {
      this.pageIndex = 1
      this.searchFlag = true
      this.getData()
    },
    // 清空列表
    empty () {
      Object.keys(this.form).map(key => {
        this.form[key] = ''
      })
      this.createDate = []
      this.searchFlag = false
      this.pageIndex = 1
      this.getData()
    },
    // 获取订单信息
    getOutPopover (rows, event, column) {
      this.orderInfo = {}
      this.hoverData = []
      this.ticketInfoNumber = ''
      // 打开对应的订单形心
      this.tableData.map(item => {
        if (item.serialNumber === rows.serialNumber) {
          this.$set(item, 'flag', true)
        }
      })
      req('queryTicketList', {serialNumber: rows.serialNumber})
        .then(res => {
          if (res.code === '00000') {
            if (res.data.ticketInfoVoPage.result && res.data.ticketInfoVoPage.result.length) {
              this.ticketInfoNumber = res.data.ticketInfoVoPage.result[0].ticketInfoNumber
            } else {
              this.$message({
                type: 'error',
                message: '此订单无票'
              })
              return
            }
            this.getPopoverData(rows)
          } else {
            this.$message({
              type: 'error',
              message: res.msg
            })
          }
        })
        .catch(error => {
          console.log(error)
        })
    },
    // 获取票面信息
    getPopoverData () {
      req('getTicketInfo', {ticketInfoNumber: this.ticketInfoNumber})
        .then(res => {
          if (res.code === '00000') {
            let maxMoney = 0
            let calcData = JSON.parse(JSON.stringify(res.data))
            // 计算最高奖金
            try {
              if (calcData.orderInfo.betType === 'single') {
                maxMoney = ChangeBetContext.returnFloat((ChangeBetContext.getSingleMaxMoney(JSON.parse(calcData.orderInfo.betContextOdds), calcData.orderInfo.multiple)))
              } else {
                let dataInfo = ChangeBetContext.getPassMaxMoney(calcData)
                maxMoney = mul(dataInfo.price, calcData.orderInfo.multiple)
                maxMoney = ChangeBetContext.returnFloat(ChangeBetContext.returnEvenRound(maxMoney))
              }
              maxMoney = (maxMoney > 1000000 && maxMoney % 1000000 === 0) ? maxMoney / 10000 + '万' : ChangeBetContext.getQianfenWei(maxMoney)
            } catch (error) {
              console.log(error)
            }
            // 获取信息
            let orderInfo = res.data.orderInfo
            orderInfo.maxMoney = maxMoney
            orderInfo.lotteryTypeWord = ChangeBetContext.lotteryType(orderInfo.lotteryType)
            orderInfo.subPlayTypeWord = ChangeBetContext.subPlayType(orderInfo.subPlayType)
            orderInfo.amount = (orderInfo.amount / 100).toFixed(2)
            orderInfo.uploadTime = orderInfo.uploadTime ? orderInfo.uploadTime : '暂无'
            orderInfo.lastPrintDate = orderInfo.lastPrintDate ? orderInfo.lastPrintDate : '暂无'
            if (orderInfo.betType === 'single') {
              orderInfo.betTypeWord = '单关'
            } else if (Object.prototype.toString.call(orderInfo.betType) === '[object String]') {
              orderInfo.betTypeWord = orderInfo.betType
            } else {
              orderInfo.betTypeWord = JSON.parse(orderInfo.betType)
            }
            this.orderInfo = orderInfo
            this.orderInfo.lotterykinds = `${orderInfo.lotteryTypeWord}${orderInfo.subPlayTypeWord}`
            // 竞彩篮球的让球胜负和大小分彩种才需要显示预设值
            if (this.orderInfo.subPlayType === '61' || this.orderInfo.subPlayType === '64' || this.orderInfo.subPlayType === '69') {
              this.orderInfo.assumptionShow = true
            } else {
              this.orderInfo.assumptionShow = false
            }
            this.orderInfo.printResult && (this.imgStr = this.orderInfo.printResult)
            // 投注项
            let betContextList = res.data.betContextList
            betContextList.map(val => {
              val.assumption = !val.score ? '—' : val.score
              val.assumptionFlag = false
              val.orderNum = ChangeBetContext.changeMatchUniqueId(val.matchUniqueId)
              // 转换票详情页列表数据
              val.betItemsObj.map(val1 => {
                for (let oddsKey in val1) {
                  let obj = {}
                  obj[oddsKey] = val1[oddsKey]
                  val1[val.matchUniqueId] = obj
                  val1.odds = val1[oddsKey]
                  val1.flag = false
                  val1.key = ChangeBetContext.bet(val.subPlayType, oddsKey)
                }
              })
              val.lotteryTypeWord = ChangeBetContext.lotteryType(val.lotteryType)
              val.subPlayTypeWord = ChangeBetContext.subPlayType(val.subPlayType)
            })
            this.hoverData = betContextList
            this.tableData.map(item => {
              if (this.orderInfo.serialNumber === item.serialNumber) {
                this.$set(item, 'flag', false)
              }
            })
            this.showOutPopover = true
          } else {
            this.$message({
              type: 'error',
              message: res.msg
            })
          }
        })
        .catch(error => {
          console.log(error)
        })
    },
    // 图片放大
    enlarge () {
      if (this.imgStr === '') {
        return
      }
      this.Mask = true
      this.enlargeImg = true
    },
    // 图片缩小
    narrow (event) {
      event.stopPropagation()
      this.Mask = false
      this.enlargeImg = false
    },
    maskClick (event) {
      event.stopPropagation()
      this.Mask = false
      this.enlargeImg = false
    },
    // 更改查询条数
    handleSizeChange (val) {
      this.pageSize = val
      this.getData()
    },
    // 分页查询
    handleCurrentChange (val) {
      this.pageIndex = val
      this.getData()
    },
    scan () {
      // 条码枪初始化
      let code = LA.BCRInitFromJS() // eslint-disable-line
      if (code === 0) {
        // 条码枪设置扫描模式 参数： 1 手动模式， 2 自动模式
        if (LA.BCRSetScanModeFromJS(1) === true) { // eslint-disable-line
          // 条码枪开始扫描
          // LA.BCRStartScanFromJS() // eslint-disable-line
          // console.log(1222222)
          const _this = this
          _this.timer = setInterval(function () {
            // 判断扫描枪是否扫描完
            if (LA.BCRScanIsCompleteFromJS() === true) { // eslint-disable-line
              // clearInterval(_this.timer)
              _this.scanTicket = LA.BCRGetTicketInfoFromJS() // eslint-disable-line
              LA.BCRStopScan() // eslint-disable-line
              if (_this.lastqrInfo === _this.scanTicket) {
                _this.$message({
                  type: 'error',
                  message: '此单已扫码，请扫码新的单据！'
                })
              } else {
                _this.lastqrInfo = _this.scanTicket
              }
            }
          }, 200)
        }
      } else {
        this.errorDialogVisible = true
        this.errorContent = `扫描枪异常，异常code: ${code}`
        // this.$confirm(`扫描枪异常，异常code: ${code}`, '错误', {
        //   confirmButtonText: '确定',
        //   showCancelButton: false,
        //   type: 'error',
        //   closeOnClickModal: false,
        //   beforeClose: (action, instance, done) => {
        //     done()
        //   },
        //   callback: action => {
        //   }
        // })
      }
    }
  },
  destroyed () {
    try {
      clearInterval(this.timer)
      LA.BCRDisableFromJS() // eslint-disable-line
    } catch (error) {
      console.log('条码枪停止错误', error)
    }
  }
}
</script>

<style lang="less">
.el-select-dropdown__item{
  font-size: 20px;
}
.query-order{
  padding-top: 92px!important;
  .count-order{
    box-sizing: border-box;
    width: calc(100% - 60px);
    overflow: hidden;
    padding:15px 20px 5px;
    background: #ffffff;
    position: fixed;
    top: 110px;
    left: 30px;
    z-index: 998;
    border-bottom: 1px solid #4dafdb;
    .el-row{
      margin-bottom: 10px;
      *{
        font-size: 20px!important;
      }
      .grid-content{
        display: inline-block;
      }
      .el-form-item{
        margin-bottom: 0;
        .el-date-editor{
          &.el-input__inner{
            width: 100%!important;
          }
        }
      }
      .btn-box{
        text-align: right;
      }
    }
  }
  .page{
    margin-top: 15px;
    margin-left: 100px;
  }
  .el-pagination{
    text-align:center;
    margin:20px 0;
    *{
      font-size: 20px!important;
    }
    .el-select .el-input {
      width: 200px;
    }
  }
  .el-table{
    font-size: 20px;
  }
  // 禁用的颜色
  .el-table .disabled-row {
    background: #D3D3D3;
  }
  // 订单详情
  .orderNum-popover{
    .el-dialog{
      margin-top: 0 !important;
      margin-bottom: 0;
    }
    .el-dialog__header{
      padding: 20px;
      .el-dialog__title{
        font-size: 24px;
      }
      .el-dialog__close{
        font-size: 28px;
      }
    }
    .el-dialog__body{
      padding: 0 20px 20px;
      .hoverContent{
        font-size: 20px;
        .tip{
          .el-col{
            margin-bottom: 15px;
            .grid-content{
              color: #FE4C40;
            }
          }
        }
        .el-row{
          width: 100%;
          margin: 0 0 10px 0!important;
          .el-col{
            padding: 0!important;
          }
          .grid-content{
            display: inline-block;
            color: #0485c1;
            &.red{
              color: #FE4C40;
            }
          }
        }
      }
      .contentBox{
        margin-top: 10px;
        overflow: hidden;
        .el-table{
          float: left;
        }
        .uploadImg{
          float: left;
          width: 30%;
          .el-button.is-disabled{
            color: #ffffff;
            background: #909399;
            border: 1px solid #909399;
          }
          .btn{
            text-align: center;
          }
          .no-image{
            margin: 0 auto;
            width: 200px;
            height: 250px;
            line-height: 250px;
            text-align: center;
            border: 1px solid #cccccc;
          }
          .img{
            margin: 0 auto;
            width: 250px;
            height: 500px;
            display: block;
          }
        }
      }
    }
  }
  .managerLogin-right{
    width: 100px;
    height: 50px;
    line-height: 50px;
    background: #409EFF;
    color: #fff;
    letter-spacing: 6px;
    cursor:pointer;
  }
  .Mask{
    position: fixed;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    opacity: .5;
    background: #000;
    z-index: 9999;
  }
  .enlarge{
    width: 300px;
    position: absolute;
    top: 0;
    bottom: 0;
    left: 50%;
    margin-left: -150px;
    z-index: 99999;
    overflow: auto;
    &::-webkit-scrollbar {
    display: none;
    }
    img{
      width: 100%;
    }
  }
}
.el-picker-panel__body{
  .el-input__inner{
    font-size: 20px;
  }
  .el-picker-panel__icon-btn{
    font-size: 20px;
  }
  div{
    font-size: 20px;
  }
}
.el-date-range-picker{
    table{
      font-size: 20px!important;
    }
}
.el-button{
  font-size: 20px;
}
.el-button+.el-button {
    margin-left: 40px;
}
.creatTime{
  input { pointer-events: none; }
}
.el-picker-panel__body{
  .el-date-range-picker__time-header{
    .el-date-range-picker__editors-wrap{
      .el-date-range-picker__time-picker-wrap:nth-of-type(2){
        display: none;
      }
      .el-date-range-picker__time-picker-wrap:nth-of-type(1){
        input{
          text-align: center;
        }
      }
    }
  }
}
</style>
