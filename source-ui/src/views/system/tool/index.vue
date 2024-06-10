<template>
  <div class="app-container">
    <el-row :gutter="10" class="mb5">
      <el-col :span="1.5">
        <el-button
          plain
          icon="el-icon-plus"
          size="mini"
          @click="handleAdd"
          v-hasPermi="['system:tool:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          plain
          icon="el-icon-edit"
          size="mini"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['system:tool:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          plain
          icon="el-icon-delete"
          size="mini"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['system:tool:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          plain
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
          v-hasPermi="['system:tool:export']"
        >导出</el-button>
      </el-col>
      <el-form :model="queryParams" ref="queryForm" :inline="true" v-show="showSearch" label-width="68px" class="el-form-search">
                        <el-form-item label="工具类型" prop="toolType" class="el-form-search-item">
                            <el-select v-model="queryParams.toolType" placeholder="请选择工具类型" clearable size="mini">
                                <el-option
                                        v-for="dict in dict.type.sys_tool_type"
                                        :key="dict.value"
                                        :label="dict.label"
                                        :value="dict.value"
                                />
                            </el-select>
                        </el-form-item>
                        <el-form-item label="工具名称" prop="name" class="el-form-search-item">
                            <el-input
                                    v-model="queryParams.name"
                                    placeholder="请输入工具名称"
                                    clearable
                                    size="mini"
                                    @keyup.enter.native="handleQuery"
                            />
                        </el-form-item>
                        <el-form-item label="是否热点" prop="isHot" class="el-form-search-item">
                            <el-select v-model="queryParams.isHot" placeholder="请选择是否热点" clearable size="mini">
                                <el-option
                                        v-for="dict in dict.type.sys_normal_disable"
                                        :key="dict.value"
                                        :label="dict.label"
                                        :value="dict.value"
                                />
                            </el-select>
                        </el-form-item>
                        <el-form-item label="审核状态" prop="auditStatus" class="el-form-search-item">
                            <el-select v-model="queryParams.auditStatus" placeholder="请选择审核状态" clearable size="mini">
                                <el-option
                                        v-for="dict in dict.type.sys_audit_status"
                                        :key="dict.value"
                                        :label="dict.label"
                                        :value="dict.value"
                                />
                            </el-select>
                        </el-form-item>
            <el-form-item class="el-form-search-item">
                <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
                <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
            </el-form-item>
        </el-form>
    </el-row>

    <el-table :height="tableHeight" v-loading="loading" :data="toolList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
        <el-table-column label="#" type="index" width="50" align="center">
            <template scope="scope">
          <span>{{
            (queryParams.pageNum - 1) * queryParams.pageSize + scope.$index + 1
          }}</span>
            </template>
        </el-table-column>
      <el-table-column label="ID" align="center" prop="id" />
      <el-table-column label="工具类型" align="center" prop="toolType">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.sys_tool_type" :value="scope.row.toolType"/>
        </template>
      </el-table-column>
      <el-table-column label="工具名称" align="center" prop="name" />
      <el-table-column label="工具缩略图" align="center" prop="thumb" width="100">
        <template slot-scope="scope">
          <image-preview :src="scope.row.thumb" :width="30" :height="30"/>
        </template>
      </el-table-column>
      <el-table-column label="工具版本" align="center" prop="toolVersion" />
      <el-table-column label="是否热点" align="center" prop="isHot">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.sys_normal_disable" :value="scope.row.isHot"/>
        </template>
      </el-table-column>
      <el-table-column label="审核状态" align="center" prop="auditStatus">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.sys_audit_status" :value="scope.row.auditStatus"/>
        </template>
      </el-table-column>
      <el-table-column label="排序" align="center" prop="sort" />
      <el-table-column label="备注" align="center" prop="remark" />
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width" width="150">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['system:tool:edit']"
          >修改</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['system:tool:remove']"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改工具管理对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="800px" append-to-body :close-on-click-modal="false" v-dialogDrag>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="工具类型">
          <el-radio-group v-model="form.toolType">
            <el-radio
              v-for="dict in dict.type.sys_tool_type"
              :key="dict.value"
:label="dict.value"
            >{{dict.label}}</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="工具名称" prop="name">
          <el-input v-model="form.name" placeholder="请输入工具名称" />
        </el-form-item>
        <el-form-item label="工具介绍" prop="intro">
          <el-input v-model="form.intro" type="textarea" placeholder="请输入内容" />
        </el-form-item>
        <el-form-item label="工具缩略图">
          <image-upload v-model="form.thumb"/>
        </el-form-item>
        <el-form-item label="工具地址" prop="url">
          <el-input v-model="form.url" placeholder="请输入工具地址" />
        </el-form-item>
        <el-form-item label="提取密码" prop="extractPassword">
          <el-input v-model="form.extractPassword" placeholder="请输入提取密码" />
        </el-form-item>
        <el-form-item label="工具内容">
          <editor v-model="form.content" :min-height="192"/>
        </el-form-item>
        <el-form-item label="工具版本" prop="toolVersion">
          <el-input v-model="form.toolVersion" placeholder="请输入工具版本" />
        </el-form-item>
        <el-form-item label="是否热点">
          <el-radio-group v-model="form.isHot">
            <el-radio
              v-for="dict in dict.type.sys_normal_disable"
              :key="dict.value"
:label="dict.value"
            >{{dict.label}}</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="审核状态">
          <el-radio-group v-model="form.auditStatus">
            <el-radio
              v-for="dict in dict.type.sys_audit_status"
              :key="dict.value"
:label="dict.value"
            >{{dict.label}}</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="排序" prop="sort">
          <el-input v-model="form.sort" placeholder="请输入排序" />
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input v-model="form.remark" placeholder="请输入备注" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { listTool, getTool, delTool, addTool, updateTool } from "@/api/system/tool";

export default {
  name: "Tool",
  dicts: ['sys_tool_type', 'sys_audit_status', 'sys_normal_disable'],
  data() {
    return {
      // 表格高度
      tableHeight: document.documentElement.clientHeight - 180,
      // 遮罩层
      loading: false,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 工具管理表格数据
      toolList: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 20,
        orderByColumn: "create_time",
        isAsc: "desc",
        toolType: null,
        name: null,
        isHot: null,
        auditStatus: null,
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
        toolType: [
          { required: true, message: "工具类型不能为空", trigger: "blur" }
        ],
        name: [
          { required: true, message: "工具名称不能为空", trigger: "blur" }
        ],
        intro: [
          { required: true, message: "工具介绍不能为空", trigger: "blur" }
        ],
        thumb: [
          { required: true, message: "工具缩略图不能为空", trigger: "blur" }
        ],
        url: [
          { required: true, message: "工具地址不能为空", trigger: "blur" }
        ],
        extractPassword: [
          { required: true, message: "提取密码不能为空", trigger: "blur" }
        ],
        content: [
          { required: true, message: "工具内容不能为空", trigger: "blur" }
        ],
        toolVersion: [
          { required: true, message: "工具版本不能为空", trigger: "blur" }
        ],
        isHot: [
          { required: true, message: "是否热点不能为空", trigger: "blur" }
        ],
        auditStatus: [
          { required: true, message: "审核状态不能为空", trigger: "blur" }
        ],
        sort: [
          { required: true, message: "排序不能为空", trigger: "blur" }
        ],
      }
    };
  },
  created() {
    this.getList();
  },
  methods: {
    /** 查询工具管理列表 */
    getList() {
      this.loading = true;
      listTool(this.queryParams).then(response => {
        this.toolList = response.rows;
        this.total = response.total;
        this.loading = false;
      });
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
    // 表单重置
    reset() {
      this.form = {
        id: null,
        toolType: "0",
        name: null,
        intro: null,
        thumb: null,
        url: null,
        extractPassword: null,
        content: null,
        toolVersion: null,
        isHot: "0",
        auditStatus: "0",
        viewCount: null,
        downloadCount: null,
        sort: null,
        createTime: null,
        createBy: null,
        updateTime: null,
        updateBy: null,
        remark: null
      };
      this.resetForm("form");
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.resetForm("queryForm");
      this.handleQuery();
    },
    // 多选框选中数据
    handleSelectionChange(selection) {
      this.ids = selection.map(item => item.id)
      this.single = selection.length!==1
      this.multiple = !selection.length
    },
    /** 新增按钮操作 */
    handleAdd() {
      this.reset();
      this.open = true;
      this.title = "添加工具管理";
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset();
      const id = row.id || this.ids
      getTool(id).then(response => {
        this.form = response.data;
        this.open = true;
        this.title = "修改工具管理";
      });
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs["form"].validate(valid => {
        if (valid) {
          if (this.form.id != null) {
            updateTool(this.form).then(response => {
              this.$modal.msgSuccess("修改成功");
              this.open = false;
              this.getList();
            });
          } else {
            addTool(this.form).then(response => {
              this.$modal.msgSuccess("新增成功");
              this.open = false;
              this.getList();
            });
          }
        }
      });
    },
    /** 删除按钮操作 */
    handleDelete(row) {
      const ids = row.id || this.ids;
      this.$modal.confirm('是否确认删除记录？').then(function() {
        return delTool(ids);
      }).then(() => {
        this.getList();
        this.$modal.msgSuccess("删除成功");
      }).catch(() => {});
    },
  }
};
</script>
