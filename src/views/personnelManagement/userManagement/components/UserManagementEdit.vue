<template>
  <el-dialog
    :title="title"
    :visible.sync="dialogFormVisible"
    width="500px"
    @close="close"
  >
    <el-form ref="form" :model="form" :rules="rules" label-width="80px">
      <el-form-item label="报表名称" prop="REPORTNAME">
        <el-input v-model.trim="form.REPORTNAME" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item label="数据类型" prop="DATATYPE">
        <el-select
          v-model.trim="form.DATATYPE"
          placeholder="请选择"
          autocomplete="off"
        >
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="close">取 消</el-button>
      <el-button type="primary" @click="save">确 定</el-button>
    </div>
  </el-dialog>
</template>

<script>
  import { doEdit } from "@/api/userManagement";

  export default {
    name: "UserManagementEdit",
    data() {
      return {
        options: [
          { value: "", label: "请选择" },
          { value: "XML", label: "XML" },
          { value: "JSON", label: "JSON" },
        ],
        DATATYPE: "",
        form: {
          username: "",
          password: "",
          email: "",
          permissions: [],
        },
        rules: {
          username: [
            { required: true, trigger: "blur", message: "请输入用户名" },
          ],
          DATATYPE: [
            { required: true, trigger: "blur", message: "请选择数据类型" },
          ],
          // password: [
          //   { required: true, trigger: "blur", message: "请输入密码" },
          // ],
          // email: [{ required: true, trigger: "blur", message: "请输入邮箱" }],
          // permissions: [
          //   { required: true, trigger: "blur", message: "请选择权限" },
          // ],
        },
        title: "",
        dialogFormVisible: false,
      };
    },
    created() {},
    methods: {
      showEdit(row) {
        if (!row) {
          this.title = "添加";
        } else {
          if (row.DATATYPE == "") {
            row.DATATYPE = "XML";
          }
          this.title = "编辑";
          this.form = Object.assign({}, row);
        }
        this.dialogFormVisible = true;
      },
      close() {
        this.$refs["form"].resetFields();
        this.form = this.$options.data().form;
        this.dialogFormVisible = false;
      },
      save() {
        this.$refs["form"].validate(async (valid) => {
          if (valid) {
            const { msg } = await doEdit(this.form);
            this.$baseMessage(msg, "success");
            this.$emit("fetchData");
            this.close();
          } else {
            return false;
          }
        });
      },
    },
  };
</script>
