<template>
  <div class="app-container">
    <el-tabs v-model="activeName" @tab-click="handleClick">
      <el-tab-pane label="缴款退款" name="first">
        <el-row>
          <el-col :span="4" style="padding-right:10px;">
            <el-card class="box-card" shadow="never">
              <div slot="header">
                <span>入住人</span>
                <el-button style="float: right; padding: 3px 0" type="text">选择</el-button>
              </div>
              <div class="input-wapper">
                <el-input v-model="checkinRecord.checkinNo" @change="checkinNoChange">
                  <template slot="prepend"><span class="input-label">入住号</span></template>
                </el-input>
              </div>
              <div class="input-wapper">
                <el-input v-model="checkinRecord.name" :readonly="true">
                  <template slot="prepend"><span class="input-label">姓名</span></template>
                </el-input>
              </div>
              <div class="input-wapper">
                <el-input v-model="checkinRecord.bedName" :readonly="true">
                  <template slot="prepend"><span class="input-label">床位</span></template>
                </el-input>
              </div>
              <div class="input-wapper">
                <el-input v-model="checkinRecord.nursingLevelName" :readonly="true">
                  <template slot="prepend"><span class="input-label">护理等级</span></template>
                </el-input>
              </div>
            </el-card>
          </el-col>
          <el-col :span="20">

            <el-form :inline="true" :model="checkinQuery" class="form-inline">
              <el-form-item label="">
                <el-date-picker
                  v-model="selesctCheckinDateRange"
                  type="daterange"
                  range-separator="至"
                  start-placeholder="开始日期"
                  end-placeholder="结束日期"
                  :picker-options="pickerOptions"
                />
              </el-form-item>
              <el-form-item>
                <el-button v-waves type="primary" icon="el-icon-search" size="small" @click="fetchCheckinList">搜索</el-button>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="handlePayment">缴款</el-button>
              </el-form-item>
            </el-form>
            <!-- 表格 -->
            <el-table
              :key="tableKey"
              v-loading="checkinListLoading"
              :data="checkinList"
              border
              fit
              highlight-current-row
              style="width: 100%;"
            >
              <el-table-column label="缴款编号" width="110px" align="left">
                <template slot-scope="scope">
                  {{ scope.row.payNo }}
                </template>
              </el-table-column>
              <el-table-column label="类型" width="110px" align="left">
                <template slot-scope="scope">
                  {{ operateTypeMap[scope.row.operateType] }}
                </template>
              </el-table-column>
              <el-table-column label="付款方式" width="110px" align="left">
                <template slot-scope="scope">
                  {{ paymentWayMap[scope.row.paymentWayId] }}
                </template>
              </el-table-column>
              <el-table-column label="金额" width="110px" align="left">
                <template slot-scope="scope">
                  {{ scope.row.amount }}
                </template>
              </el-table-column>
              <el-table-column label="操作人" width="110px" align="left">
                <template slot-scope="scope">
                  {{ scope.row.createName }}
                </template>
              </el-table-column>
              <el-table-column label="备注" width="110px" align="left">
                <template slot-scope="scope">
                  {{ scope.row.remarks }}
                </template>
              </el-table-column>
              <el-table-column label="操作" align="center" width="150px" class-name="small-padding">
                <template slot-scope="scope">
                  <el-tooltip content="打印" placement="top">
                    <el-button v-waves type="primary" size="mini" @click="handlePrint(scope.$index,scope.row.id)">打印</el-button>
                  </el-tooltip>
                  <el-tooltip v-if="scope.row.refundFlag==='0' && scope.row.expendFlag==='0'" content="退款" placement="top">
                    <el-button v-waves :loading="scope.row.delete" type="danger" size="mini" @click="handleRefund(scope.$index,scope.row.id)">退款</el-button>
                  </el-tooltip>
                </template>
              </el-table-column>
            </el-table>

            <!-- 表单 -->
            <paymentForm ref="formPayment" :checkin-no="checkinQuery.checkinNo" :payment-way="paymentWay" @updateRow="updateRow" />
          </el-col>
        </el-row>
      </el-tab-pane>
      <el-tab-pane label="缴款查询" name="second">
        <div class="filter-container">
          <el-form :inline="true" :model="listQuery" class="form-inline">
            <el-form-item label="">
              <el-input v-model="listQuery.checkinNo" placeholder="入住号" clearable size="small" />
            </el-form-item>
            <el-form-item label="">
              <el-date-picker
                v-model="selesctDateRange"
                type="daterange"
                range-separator="至"
                start-placeholder="开始日期"
                end-placeholder="结束日期"
                :picker-options="pickerOptions"
              />
            </el-form-item>
            <el-form-item>
              <el-button v-waves type="primary" icon="el-icon-search" size="small" @click="fetchList">搜索</el-button>
            </el-form-item>
          </el-form>
        </div>
        <!-- 表格 -->
        <el-table
          :key="tableKey"
          v-loading="listLoading"
          :data="list"
          border
          fit
          highlight-current-row
          style="width: 100%;"
        >
          <el-table-column label="入住号" width="110px" align="left">
            <template slot-scope="scope">
              {{ scope.row.checkinNo }}
            </template>
          </el-table-column>
          <el-table-column label="姓名" width="110px" align="left">
            <template slot-scope="scope">
              {{ scope.row.personName }}
            </template>
          </el-table-column>
          <el-table-column label="缴款编号" width="110px" align="left">
            <template slot-scope="scope">
              {{ scope.row.payNo }}
            </template>
          </el-table-column>
          <el-table-column label="类型" width="110px" align="left">
            <template slot-scope="scope">
              {{ operateTypeMap[scope.row.operateType] }}
            </template>
          </el-table-column>
          <el-table-column label="付款方式" width="110px" align="left">
            <template slot-scope="scope">
              {{ paymentWayMap[scope.row.paymentWayId] }}
            </template>
          </el-table-column>
          <el-table-column label="金额" width="110px" align="left">
            <template slot-scope="scope">
              {{ scope.row.amount }}
            </template>
          </el-table-column>
          <el-table-column label="操作人" width="110px" align="left">
            <template slot-scope="scope">
              {{ scope.row.createName }}
            </template>
          </el-table-column>
          <el-table-column label="备注" width="110px" align="left">
            <template slot-scope="scope">
              {{ scope.row.remarks }}
            </template>
          </el-table-column>
          <el-table-column label="操作" align="center" width="150px" class-name="small-padding">
            <template slot-scope="scope">
              <el-tooltip content="打印" placement="top">
                <el-button v-waves type="primary" size="mini" @click="handleUpdate(scope.$index,scope.row.id)">打印</el-button>
              </el-tooltip>
              <el-tooltip v-if="scope.row.refundFlag==='0'" content="退款" placement="top">
                <el-button v-waves :loading="scope.row.delete" type="danger" size="mini" :v-if="scope.row.operateType === 'payin'" @click="handleDelete(scope.$index,scope.row.id)">退款</el-button>
              </el-tooltip>
            </template>
          </el-table-column>
        </el-table>

        <!-- 分页 -->
        <div class="pagination-container">
          <el-pagination v-show="total>0" :current-page="listQuery.page" :page-sizes="[10,20,30, 50]" :page-size="listQuery.psize" :total="total" background layout="total, sizes, prev, pager, next, jumper" @size-change="handleSizeChange" @current-change="handleCurrentChange" />
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import { findListByCheckinNo, findPage } from '@/api/account-record'
import { getByCheckinNo } from '@/api/checkin'
import { findByOrg as findPaymentWay } from '@/api/payment-way'
import waves from '@/directive/waves'
import { pickerOptions, getMonth } from '@/utils'
import paymentForm from './account-payment'

export default {
  name: 'AccountRecord',
  components: { paymentForm },
  directives: {
    waves
  },
  filters: {
  },
  data() {
    return {
      activeName: 'first',
      tableKey: 0,
      checkinList: null,
      list: null,
      total: null,
      selectedRows: null,
      checkinListLoading: false,
      listLoading: false,
      showSearch: false,
      paymentWay: [],
      paymentWayMap: {},
      checkinQuery: {
        checkinNo: '',
        endDate: '',
        startDate: ''
      },
      selesctCheckinDateRange: [getMonth('s', 0), getMonth('e', 0)],
      selesctDateRange: [getMonth('s', 0), getMonth('e', 0)],
      listQuery: {
        checkinNo: '',
        endDate: '',
        startDate: ''
      },
      operateTypeMap: {
        payin: '缴款存入',
        refund: '退款取出',
        deductions: '费用扣除',
        restitution: '退费返还'
      },
      checkinRecord: {
        checkinNo: '',
        name: '',
        bedName: '',
        nursingLevelName: ''
      },
      buttonDisabled: true,
      deleting: false,
      pickerOptions: pickerOptions,
      currentIndex: -1
    }
  },
  watch: {
  },
  created() {
    this.findPaymentWay()
  },
  methods: {
    handleClick(tab, event) {
      console.log(tab, event)
    },
    findPaymentWay() {
      findPaymentWay().then(response => {
        this.paymentWay = response.data
        if (this.paymentWay.length > 0) {
          for (let i = 0; i < this.paymentWay.length; i++) {
            const payWay = this.paymentWay[i]
            this.paymentWayMap[payWay.id] = payWay.name
          }
        }
      })
    },
    fetchCheckinList() {
      this.checkinListLoading = true
      this.checkinQuery.startDate = this.selesctCheckinDateRange[0]
      this.checkinQuery.endDate = this.selesctCheckinDateRange[1]
      findListByCheckinNo(this.checkinQuery).then(response => {
        if (response.data) {
          this.checkinList = response.data
        }
        this.checkinListLoading = false
      })
    },
    checkinNoChange(val) {
      this.checkinQuery.checkinNo = val
      this.checkinRecord = {
        checkinNo: this.checkinQuery.checkinNo,
        name: '',
        bedName: '',
        nursingLevelName: ''
      }
      this.getByCheckinNo()
      this.fetchCheckinList()
    },
    getByCheckinNo() {
      getByCheckinNo(this.checkinQuery.checkinNo).then(response => {
        if (response.data) {
          this.checkinRecord = response.data
        }
      })
    },
    fetchList() {
      this.listLoading = true
      this.listQuery.startDate = this.selesctDateRange[0]
      this.listQuery.endDate = this.selesctDateRange[0]
      findPage(this.listQuery).then(response => {
        this.list = response.data.list
        this.total = response.data.totalRow
        this.listLoading = false
      })
    },
    handleFilter() {
      this.listQuery.page = 1
      this.fetchList()
    },
    handleFilterClear() {
      this.listQuery = {
        page: 1,
        psize: 10,
        sort: 0,
        name: ''
      }
      this.fetchList()
    },
    handleSizeChange(val) {
      this.listQuery.psize = val
      this.fetchList()
    },
    handleCurrentChange(val) {
      this.listQuery.page = val
      this.fetchList()
    },
    handleSelectionChange(val) {
      if (val.length > 0) {
        this.buttonDisabled = false
      } else {
        this.buttonDisabled = true
      }
      this.selectedRows = val
    },
    handlePayment() {
      this.$refs.formPayment.handlePayment()
    },
    handlePring(id) {

    },
    handleRefund(index, id) {
      this.$refs.formPayment.handleRefund(id)
    },
    updateRow(temp) {
      if (this.currentIndex >= 0 && temp.id) {
        this.checkinList.splice(this.currentIndex, 1, temp)
      } else {
        this.checkinList.push(temp)
      }
      this.currentIndex = -1
    }
  }
}
</script>
<style rel="stylesheet/scss" lang="scss">
	.text-red{
		color: red;
		cursor: pointer;
	}
	.text-green{
		color: green;
		cursor: pointer;
	}
  .el-card__header{
    padding:8px 15px;
    font-size: 14px;
  }
  .input-wapper{
    width:100%;
    margin-bottom:10px;
    &:last-child {
      margin-bottom: 0;
    }
  }
  .input-label{
    display: block;
    text-align: right;
    width:40px;
    line-height:24px;
    font-size: 12px;
  }
</style>
