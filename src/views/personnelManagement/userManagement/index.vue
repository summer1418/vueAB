<template>
  <div class="userManagement-container">
    <vab-query-form>
      <vab-query-form-left-panel :span="12">
        <el-button icon="el-icon-plus" type="primary" @click="handleEdit">
          添加
        </el-button>
        <el-button icon="el-icon-delete" type="danger" @click="handleDelete">
          批量删除
        </el-button>
      </vab-query-form-left-panel>
      <vab-query-form-right-panel :span="12">
        <el-form :inline="true" :model="queryForm" @submit.native.prevent>
          <el-form-item>
            <el-input
              v-model.trim="queryForm.username"
              placeholder="请输入用户名"
              clearable
            />
          </el-form-item>
          <el-form-item>
            <el-button icon="el-icon-search" type="primary" @click="queryData">
              查询
            </el-button>
          </el-form-item>
        </el-form>
      </vab-query-form-right-panel>
    </vab-query-form>

    <el-table
      v-loading="listLoading"
      :data="list"
      :element-loading-text="elementLoadingText"
      @selection-change="setSelectRows"
    >
      <el-table-column show-overflow-tooltip type="selection"></el-table-column>
      <el-table-column
        show-overflow-tooltip
        prop="REPORTNAME"
        label="id"
      ></el-table-column>
      <el-table-column
        show-overflow-tooltip
        prop="REPORTNAME"
        label="报表名称"
      ></el-table-column>
      <el-table-column
        show-overflow-tooltip
        prop="FLOWSNO"
        label="流程号"
      ></el-table-column>

      <!-- <el-table-column show-overflow-tooltip label="数据类型" :prop="DATATYPE">
        <template v-slot="{ row }">
          <el-tag :prop="row.DATATYPE">
            {{ item }}
          </el-tag>
        </template>
      </el-table-column> -->
      <el-table-column label="数据类型" width="180">
        <template slot-scope="scope">
          <!-- <el-tag size="medium" type="warning"> -->
          <el-tag
            size="medium"
            :type="scope.row.DATATYPE == '' ? 'danger' : 'warning'"
          >
            {{ scope.row.DATATYPE == "" ? "XML" : scope.row.DATATYPE }}
          </el-tag>
        </template>
      </el-table-column>

      <el-table-column
        show-overflow-tooltip
        prop="INPUT_DATE"
        label="设计时间"
        :formatter="dataFormat"
      ></el-table-column>
      <el-table-column
        show-overflow-tooltip
        fixed="right"
        label="操作"
        width="200"
      >
        <template v-slot="scope">
          <el-button type="text" @click="handleEdit(scope.row)">编辑</el-button>
          <el-button type="text" @click="handleDelete(scope.row)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      background
      :current-page="queryForm.pageNo"
      :page-size="queryForm.pageSize"
      :layout="layout"
      :total="total"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    ></el-pagination>
    <edit ref="edit" @fetchData="fetchData"></edit>
  </div>
</template>

<script>
  import { getList, doDelete } from "@/api/userManagement";
  import Edit from "./components/UserManagementEdit";
  import moment from "moment";

  export default {
    name: "UserManagement",
    components: { Edit },
    data() {
      return {
        list: null,
        listLoading: true,
        layout: "total, sizes, prev, pager, next, jumper",
        total: 0,
        selectRows: "",
        elementLoadingText: "正在加载...",
        queryForm: {
          pageNo: 1,
          pageSize: 10,
          username: "",
          pageSearch: "",
        },
      };
    },
    created() {
      this.fetchData();
    },
    mounted() {
      getList(this.queryForm).then((value) => {
        const data = value.data;
      });
    },
    methods: {
      //日期格式化
      dataFormat(val, column) {
        var date = val[column.property];
        if (date == "undefined") {
          return "";
        }
        return moment(date).format("YYYY-MM-DD");
      },

      setSelectRows(val) {
        this.selectRows = val;
      },
      handleEdit(row) {
        if (row.REPORTNAME) {
          this.$refs["edit"].showEdit(row);
        } else {
          this.$refs["edit"].showEdit();
        }
      },
      handleDelete(row) {
        if (row.id) {
          this.$baseConfirm("你确定要删除当前项吗", null, async () => {
            const { msg } = await doDelete({ ids: row.id });
            this.$baseMessage(msg, "success");
            this.fetchData();
          });
        } else {
          if (this.selectRows.length > 0) {
            const ids = this.selectRows.map((item) => item.id).join();
            this.$baseConfirm("你确定要删除选中项吗", null, async () => {
              const { msg } = await doDelete({ ids });
              this.$baseMessage(msg, "success");
              this.fetchData();
            });
          } else {
            this.$baseMessage("未选中任何行", "error");
            return false;
          }
        }
      },
      handleSizeChange(val) {
        this.queryForm.pageSize = val;
        this.fetchData();
      },
      handleCurrentChange(val) {
        this.queryForm.pageNo = val;
        this.fetchData();
      },
      queryData() {
        this.queryForm.pageNo = 1;
        this.fetchData();
      },
      async fetchData() {
        this.listLoading = true;
        const { data, totalCount } = await getList(this.queryForm);
        this.list = data;
        this.total = parseInt(totalCount);
        // console.log(await getList(this.queryForm));
        // console.log(getList(this.queryForm));
        // getList(this.queryForm).then((value) => {
        //   this.list = value.data;
        //   this.total = parseInt(value.totalCount);
        // });

        setTimeout(() => {
          this.listLoading = false;
        }, 300);
      },
    },
  };
</script>
