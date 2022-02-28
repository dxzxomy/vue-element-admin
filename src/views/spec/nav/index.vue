<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input v-model="listQuery._filter.name" placeholder="导航名称" class="search-input-large filter-item" @keyup.enter.native="handleFilter" />
      <el-select v-model="listQuery._filter['type']" clearable filterable placeholder="节点层级" class="search-input-medium filter-item">
        <el-option v-for="(st, index) in navList" :key="index" :label="st.level" :value="st.label" />
      </el-select>
      <el-select v-model="listQuery._filter['type']" clearable filterable placeholder="父节点" class="search-input-medium filter-item">
        <el-option v-for="(st, index) in navList" :key="index" :label="st.pnode" :value="st.pnode" />
      </el-select>
      <el-button v-waves class="filter-item" style="margin-left: 10px" type="primary" icon="el-icon-search" @click="handleFilter">查刷</el-button>
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-delete" @click="clearFilter">清除</el-button>
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-edit" @click="handleCreate">新建</el-button>
    </div>
    <el-table :key="tableKey" v-loading="listLoading" :data="navList" border fit highlight-current-row style="width: 100%;" @sort-change="sortChange">
      <el-table-column fixed="left" label="ID" prop="id" sortable="custom" align="center" width="80" :class-name="getSortClass('id')">
        <template slot-scope="{row}">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="导航名" min-width="200px" align="center" fixed="left">
        <template slot-scope="{$index, row}">
          <span class="link-type" @click="handleUpdate($index, row)">{{ row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="导航地址" min-width="250px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.url }}</span>
        </template>
      </el-table-column>
      <el-table-column label="导航图标" width="200px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.icon }}</span>
        </template>
      </el-table-column>
      <el-table-column label="父节点" min-width="250px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.pnode }}</span>
        </template>
      </el-table-column>
      <el-table-column label="更新时间" width="155" align="center">
        <template slot-scope="{row}">
          <span>{{ row.utime }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" :disabled="true" fixed="right" align="center" width="145px" class-name="small-padding fixed-width">
        <template slot-scope="{$index, row}">
          <el-button :disabled="true" size="mini" type="primary">备用</el-button>
          <el-button :disabled="true" size="mini" type="danger" @click="handleDelete($index, row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <pagination v-show="total>0" :page-sizes="page_sizes" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getNavList" />
  </div>
</template>

<script>
import { getNavList, deleteNav } from '@/api/spec'
import waves from '@/directive/waves' // waves directive
import Pagination from '@/components/Pagination'
import { success, error } from '@/views/common/conf'

export default {
  name: 'Nav',
  components: { Pagination },
  directives: { waves },
  data() {
    return {
      navList: [],
      total: 0,
      tableKey: 0,
      page_sizes: [15, 30, 100, 500],
      listLoading: true,
      listQuery: {
        _filter: {},
        page: 1,
        limit: 15,
        sort: '-id'
      }
    }
  },
  created() {
    this.getNavList()
  },
  methods: {
    getQueryFilter() {
      for (const k in this.listQuery._filter) {
        if (this.listQuery._filter[k] === '') {
          delete this.listQuery._filter[k]
        }
      }
    },
    getNavList() {
      this.listLoading = true
      this.getQueryFilter()
      getNavList(this.listQuery).then(response => {
        this.suitList = response.data.items
        this.total = response.data.total
        setTimeout(() => {
          this.listLoading = false
        }, 1.5 * 1000)
      })
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getNavList()
    },
    clearFilter() {
      this.listQuery._filter = {}
    },
    sortChange(data) {
      const { prop, order } = data
      if (prop === 'id') {
        this.sortByID(order)
      }
    },
    sortByID(order) {
      if (order === 'ascending') {
        this.listQuery.sort = '+id'
      } else {
        this.listQuery.sort = '-id'
      }
      this.handleFilter()
    },
    getSortClass: function(key) {
      const sort = this.listQuery.sort
      return sort === `+${key}` ? 'ascending' : 'descending'
    },
    handleCreate() {
      this.$refs.suitmodel.handleCreate()
    },
    handleUpdate(index, row) {
      this.$refs.suitmodel.handleUpdate(index, row)
    },
    handleDelete(index, row) {
      const id = row.id
      deleteNav(id, row).then((response) => {
        if (response.code === 20200) {
          this.NavList.splice(index, 1)
          this.total -= 1
          success(response.msg)
        } else {
          error(response.msg)
        }
      })
    },
    modelCallback(index, item) {
      console.log('parent', index, item)
      if (index > -1) {
        this.suitList.splice(index, 1, item)
      } else {
        const total = this.suitList.unshift(item)
        this.total = total
      }
    },
    apiNotify(resp) {
      let ntitle, ntype
      const sucCode = [20200, 20201]
      const warnCode = [20404, 20408]
      const errCode = [20409, 20500]
      if (sucCode.includes(resp.code)) {
        ntitle = 'Success'
        ntype = 'success'
      } else if (warnCode.includes(resp.code)) {
        ntitle = 'Warning'
        ntype = 'warning'
      } else if (errCode.includes(resp.code)) {
        ntitle = 'Error'
        ntype = 'error'
      }
      this.$notify({
        title: ntitle,
        message: resp.msg,
        type: ntype,
        duration: 2000
      })
    }
  }
}
</script>

<style scoped>
  .search-input-medium {
    width: 110px;
    margin-left: 1px;
  }
  .search-input-large {
    width: 150px;
    margin-left: 1px;
  }
</style>
