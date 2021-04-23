<template>
  <div class="app-container">
    <el-tabs v-model="activeName" @tab-click="handleClick">
      <el-tab-pane label="退费" name="first">
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
                <el-button type="primary" @click="handleRefund">退费</el-button>
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
              <el-table-column label="收费项目" width="110px" align="left">
                <template slot-scope="scope">
                  {{ scope.row.name }}
                </template>
              </el-table-column>
              <el-table-column label="类型" width="110px" align="left">
                <template slot-scope="scope">
                  {{ chargeHandleMap[scope.row.chargeHandle] }}
                </template>
              </el-table-column>
              <el-table-column label="价格" width="110px" align="left">
                <template slot-scope="scope">
                  {{ scope.row.price }}
                </template>
              </el-table-column>
              <el-table-column label="数量" width="110px" align="left">
                <template slot-scope="scope">
                  {{ scope.row.quantity }}
                </template>
              </el-table-column>
              <el-table-column label="金额" width="110px" align="left">
                <template slot-scope="scope">
                  {{ scope.row.amount }}
                </template>
              </el-table-column>
              <el-table-column label="退费时间" width="110px" align="left">
                <template slot-scope="scope">
                  {{ scope.row.createTime }}
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
            </el-table>

            <!-- 表单 -->
            <refundForm ref="formRefund" :checkin-no="checkinQuery.checkinNo" @fetchCheckinList="fetchCheckinList" />
          </el-col>
        </el-row>
      </el-tab-pane>
      <el-tab-pane label="退费查询" name="second">
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
          <el-table-column label="床位" width="130px" align="left">
            <template slot-scope="scope">
              {{ scope.row.buildingName + scope.row.floorName + scope.row.bedName }}
            </template>
          </el-table-column>
          <el-table-column label="护理等级" width="110px" align="left">
            <template slot-scope="scope">
              {{ scope.row.nursingLevelName }}
            </template>
          </el-table-column>
          <el-table-column label="收费项目" width="110px" align="left">
            <template slot-scope="scope">
              {{ scope.row.name }}
            </template>
          </el-table-column>
          <el-table-column label="类型" width="110px" align="left">
            <template slot-scope="scope">
              {{ chargeHandleMap[scope.row.chargeHandle] }}
            </template>
          </el-table-column>
          <el-table-column label="价格" width="110px" align="left">
            <template slot-scope="scope">
              {{ scope.row.price }}
            </template>
          </el-table-column>
          <el-table-column label="数量" width="110px" align="left">
            <template slot-scope="scope">
              {{ scope.row.quantity }}
            </template>
          </el-table-column>
          <el-table-column label="金额" width="110px" align="left">
            <template slot-scope="scope">
              {{ scope.row.amount }}
            </template>
          </el-table-column>
          <el-table-column label="退费时间" width="200px" align="left">
            <template slot-scope="scope">
              {{ scope.row.createTime }}
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
import { findRefundByCheckinNo, findRefundPage } from '@/api/billdetail'
import { getByCheckinNo } from '@/api/checkin'
import waves from '@/directive/waves'
import { pickerOptions, getMonth } from '@/utils'
import refundForm from './refund-form'

export default {
  name: 'RefundManager',
  components: { refundForm },
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
      checkinQuery: {
        checkinNo: '',
        endDate: '',
        startDate: ''
      },
      selesctCheckinDateRange: [getMonth('s', 0), getMonth('e', 0)],
      selesctDateRange: [getMonth('s', 0), getMonth('e', 0)],
      listQuery: {
        page: 1,
        psize: 10,
        checkinNo: '',
        endDate: '',
        startDate: ''
      },
      chargeHandleMap: {
        charge: '收费',
        refund: '退费',
        deductions: '扣除'
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
  },
  methods: {
    handleClick(tab, event) {
      console.log(tab, event)
    },
    fetchCheckinList() {
      this.checkinListLoading = true
      this.checkinQuery.startDate = this.selesctCheckinDateRange[0]
      this.checkinQuery.endDate = this.selesctCheckinDateRange[1]
      findRefundByCheckinNo(this.checkinQuery).then(response => {
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
      this.listQuery.endDate = this.selesctDateRange[1]
      findRefundPage(this.listQuery).then(response => {
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
        checkinNo: '',
        endDate: '',
        startDate: ''
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
    handleRefund() {
      this.$refs.formRefund.handleRefund()
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
