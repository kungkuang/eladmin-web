<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">城市级别</label>
        <el-input v-model="query.level" clearable placeholder="城市级别" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">城市名称</label>
        <el-input v-model="query.name" clearable placeholder="城市名称" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">城市编号</label>
        <el-input v-model="query.citycode" clearable placeholder="城市编号" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">父级地理编号</label>
        <el-input v-model="query.pAdcode" clearable placeholder="父级地理编号" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">地理编号</label>
        <el-input v-model="query.adcode" clearable placeholder="地理编号" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">有效标识</label>
        <el-input v-model="query.mark" clearable placeholder="有效标识" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="父级编号">
            未设置字典，请手动设置 Select
          </el-form-item>
          <el-form-item label="城市级别" prop="level">
            <el-select v-model="form.level" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.city_level"
                :key="item.id"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="城市名称" prop="name">
            <el-input v-model="form.name" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="城市编号" prop="citycode">
            <el-input v-model="form.citycode" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="中心点经度" prop="lng">
            <el-input v-model="form.lng" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="中心点纬度" prop="lat">
            <el-input v-model="form.lat" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="排序号" prop="sort">
            <el-input v-model="form.sort" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="有效标识" prop="mark">
            <el-radio v-for="item in dict.city_status" :key="item.id" v-model="form.mark" :label="item.value">{{ item.label }}</el-radio>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="level" label="城市级别">
          <template slot-scope="scope">
            {{ dict.label.city_level[scope.row.level] }}
          </template>
        </el-table-column>
        <el-table-column prop="name" label="城市名称" />
        <el-table-column prop="citycode" label="城市编号" />
        <el-table-column prop="lng" label="中心点经度" />
        <el-table-column prop="lat" label="中心点纬度" />
        <el-table-column prop="sort" label="排序号" />
        <el-table-column prop="mark" label="有效标识">
          <template slot-scope="scope">
            {{ dict.label.city_status[scope.row.mark] }}
          </template>
        </el-table-column>
        <el-table-column v-if="checkPer(['admin','sysCity:edit','sysCity:del'])" label="操作" width="150px" align="center">
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
import crudSysCity from '@/api/sysCity'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, pid: null, level: null, name: null, citycode: null, pAdcode: null, adcode: null, lng: null, lat: null, sort: null, createUser: null, createTime: null, updateUser: null, updateTime: null, mark: null }
export default {
  name: 'SysCity',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['city_level', 'city_status'],
  cruds() {
    return CRUD({ title: '城市管理', url: 'api/sysCity', idField: 'id', sort: 'id,desc', crudMethod: { ...crudSysCity }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'sysCity:add'],
        edit: ['admin', 'sysCity:edit'],
        del: ['admin', 'sysCity:del']
      },
      rules: {
        level: [
          { required: true, message: '城市级别不能为空', trigger: 'blur' }
        ],
        name: [
          { required: true, message: '城市名称不能为空', trigger: 'blur' }
        ],
        citycode: [
          { required: true, message: '城市编号不能为空', trigger: 'blur' }
        ],
        lng: [
          { required: true, message: '中心点经度不能为空', trigger: 'blur' }
        ],
        lat: [
          { required: true, message: '中心点纬度不能为空', trigger: 'blur' }
        ],
        sort: [
          { required: true, message: '排序号不能为空', trigger: 'blur' }
        ],
        mark: [
          { required: true, message: '有效标识不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'level', display_name: '城市级别' },
        { key: 'name', display_name: '城市名称' },
        { key: 'citycode', display_name: '城市编号' },
        { key: 'pAdcode', display_name: '父级地理编号' },
        { key: 'adcode', display_name: '地理编号' },
        { key: 'mark', display_name: '有效标识' }
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
