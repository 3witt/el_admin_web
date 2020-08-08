<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">id</label>
        <el-input v-model="query.id" clearable placeholder="id" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">基金代码</label>
        <el-input v-model="query.jjCode" clearable placeholder="基金代码" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">基金名称</label>
        <el-input v-model="query.jjName" clearable placeholder="基金名称" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">基金类型</label>
        <el-input v-model="query.jjType" clearable placeholder="基金类型" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">金经理基</label>
        <el-input v-model="query.jjManager" clearable placeholder="金经理基" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker
          v-model="query.equityReturn"
          start-placeholder="equityReturnStart"
          end-placeholder="equityReturnStart"
          class="date-item"
        />
        <date-range-picker
          v-model="query.timeStr"
          start-placeholder="timeStrStart"
          end-placeholder="timeStrStart"
          class="date-item"
        />
        <date-range-picker
          v-model="query.jjEstablishedTime"
          start-placeholder="jjEstablishedTimeStart"
          end-placeholder="jjEstablishedTimeStart"
          class="date-item"
        />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="id">
            <el-input v-model="form.id" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="基金代码" prop="jjCode">
            <el-input v-model="form.jjCode" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="基金名称" prop="jjName">
            <el-input v-model="form.jjName" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="基金类型" prop="jjType">
            <el-select v-model="form.jjType" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.jj_type"
                :key="item.id"
                :label="item.label"
                :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="金经理基">
            <el-input v-model="form.jjManager" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="基金最新涨值">
            <el-input v-model="form.equityReturn" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="基金最新涨值时间">
            <el-date-picker v-model="form.timeStr" type="datetime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="基金成立时间">
            <el-date-picker v-model="form.jjEstablishedTime" type="datetime" style="width: 370px;" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="id" label="id" />
        <el-table-column prop="jjCode" label="基金代码" />
        <el-table-column prop="jjName" label="基金名称" />
        <el-table-column prop="jjType" label="基金类型">
          <template slot-scope="scope">
            {{ dict.label.jj_type[scope.row.jjType] }}
          </template>
        </el-table-column>
        <el-table-column prop="jjManager" label="金经理基" />
        <el-table-column prop="equityReturn" label="基金最新涨值" />
        <el-table-column prop="timeStr" label="基金最新涨值时间" />
        <el-table-column prop="jjEstablishedTime" label="基金成立时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.jjEstablishedTime) }}</span>
          </template>
        </el-table-column>
        <el-table-column v-permission="['admin','jjData:edit','jjData:del']" label="操作" width="150px" align="center">
          <template slot-scope="scope">
            <udOperation
              :data="scope.row"
              :permission="permission"
            />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudJjData from '@/api/system/jjData'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, jjCode: null, jjName: null, jjType: null, jjManager: null, equityReturn: null, timeStr: null, jjEstablishedTime: null }
export default {
  name: 'JjData',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['jj_type'],
  cruds() {
    return CRUD({ title: 'jj/api', url: 'api/jjData', sort: 'id,desc', crudMethod: { ...crudJjData }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'jjData:add'],
        edit: ['admin', 'jjData:edit'],
        del: ['admin', 'jjData:del']
      },
      rules: {
        jjCode: [
          { required: true, message: '基金代码不能为空', trigger: 'blur' }
        ],
        jjName: [
          { required: true, message: '基金名称不能为空', trigger: 'blur' }
        ],
        jjType: [
          { required: true, message: '基金类型不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'id', display_name: 'id' },
        { key: 'jjCode', display_name: '基金代码' },
        { key: 'jjName', display_name: '基金名称' },
        { key: 'jjType', display_name: '基金类型' },
        { key: 'jjManager', display_name: '金经理基' }
      ]
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped>

</style>
