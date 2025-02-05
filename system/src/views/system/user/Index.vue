<template>
  <div class="app-container">
    <el-row :gutter="20">
      <!--部门数据-->
      <el-col
        :span="4"
        :xs="24"
      >
        <div class="head-container">
          <el-input
            v-model="deptName"
            placeholder="请输入部门名称"
            clearable
            size="small"
            prefix-icon="el-icon-search"
            style="margin-bottom: 20px"
          />
        </div>
        <div class="head-container">
          <el-tree
            ref="treeRef"
            :data="deptOptions"
            :props="defaultProps"
            :expand-on-click-node="false"
            :filter-node-method="filterNode"
            default-expand-all
            @node-click="handleNodeClick"
          />
        </div>
      </el-col>
      <!--用户数据-->
      <el-col
        :span="20"
        :xs="24"
      >
        <el-form
          v-show="showSearch"
          ref="queryForm"
          :model="queryParams"
          :inline="true"
          label-width="68px"
        >
          <el-form-item
            label="用户名称"
            prop="userName"
          >
            <el-input
              v-model="queryParams.userName"
              placeholder="请输入用户名称"
              clearable
              size="small"
              style="width: 240px"
              @keyup.enter="handleQuery"
            />
          </el-form-item>
          <el-form-item
            label="手机号码"
            prop="phonenumber"
          >
            <el-input
              v-model="queryParams.phonenumber"
              placeholder="请输入手机号码"
              clearable
              size="small"
              style="width: 240px"
              @keyup.enter="handleQuery"
            />
          </el-form-item>
          <el-form-item
            label="状态"
            prop="status"
          >
            <el-select
              v-model="queryParams.status"
              placeholder="用户状态"
              clearable
              size="small"
              style="width: 240px"
            >
              <el-option
                v-for="dict in statusOptions"
                :key="dict.dictValue"
                :label="dict.dictLabel"
                :value="dict.dictValue"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="创建时间">
            <el-date-picker
              v-model="dateRange"
              size="small"
              style="width: 240px"
              value-format="yyyy-MM-dd"
              type="daterange"
              range-separator="-"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
            />
          </el-form-item>
          <el-form-item>
            <el-button
              type="primary"
              icon="el-icon-search"
              size="mini"
              @click="handleQuery"
            >
              搜索
            </el-button>
            <el-button
              icon="el-icon-refresh"
              size="mini"
              @click="resetQuery"
            >
              重置
            </el-button>
          </el-form-item>
        </el-form>

        <el-row
          :gutter="10"
          class="mb8"
        >
          <el-col :span="1.5">
            <el-button
              v-hasPermi="['system:user:add']"
              type="primary"
              plain
              icon="el-icon-plus"
              size="mini"
              @click="handleAdd"
            >
              新增
            </el-button>
          </el-col>
          <el-col :span="1.5">
            <el-button
              v-hasPermi="['system:user:edit']"
              type="success"
              plain
              icon="el-icon-edit"
              size="mini"
              :disabled="single"
              @click="handleUpdate"
            >
              修改
            </el-button>
          </el-col>
          <el-col :span="1.5">
            <el-button
              v-hasPermi="['system:user:remove']"
              type="danger"
              plain
              icon="el-icon-delete"
              size="mini"
              :disabled="multiple"
              @click="handleDelete"
            >
              删除
            </el-button>
          </el-col>
          <el-col :span="1.5">
            <el-button
              v-hasPermi="['system:user:import']"
              type="info"
              plain
              icon="el-icon-upload2"
              size="mini"
              @click="handleImport"
            >
              导入
            </el-button>
          </el-col>
          <el-col :span="1.5">
            <el-button
              v-hasPermi="['system:user:export']"
              type="warning"
              plain
              icon="el-icon-download"
              size="mini"
              @click="handleExport"
            >
              导出
            </el-button>
          </el-col>
          <right-toolbar
            v-model:show-search="showSearch"
            :columns="columns"
            @queryTable="getList"
          />
        </el-row>

        <el-table
          v-loading="loading"
          :data="userList"
          @selection-change="handleSelectionChange"
        >
          <el-table-column
            type="selection"
            width="50"
            align="center"
          />
          <el-table-column
            v-if="columns[0].visible"
            key="userId"
            label="用户编号"
            align="center"
            prop="userId"
          />
          <el-table-column
            v-if="columns[1].visible"
            key="userName"
            label="用户名称"
            align="center"
            prop="userName"
            :show-overflow-tooltip="true"
          />
          <el-table-column
            v-if="columns[2].visible"
            key="nickName"
            label="用户昵称"
            align="center"
            prop="nickName"
            :show-overflow-tooltip="true"
          />
          <el-table-column
            v-if="columns[3].visible"
            key="deptName"
            label="部门"
            align="center"
            prop="dept.deptName"
            :show-overflow-tooltip="true"
          />
          <el-table-column
            v-if="columns[4].visible"
            key="phonenumber"
            label="手机号码"
            align="center"
            prop="phonenumber"
            width="120"
          />
          <el-table-column
            v-if="columns[5].visible"
            key="status"
            label="状态"
            align="center"
          >
            <template #default="scope">
              <el-switch
                v-model="scope.row.status"
                active-value="0"
                inactive-value="1"
                @change="handleStatusChange(scope.row)"
              />
            </template>
          </el-table-column>
          <el-table-column
            v-if="columns[6].visible"
            label="创建时间"
            align="center"
            prop="createTime"
            width="120"
          >
            <template #default="scope">
              <span>{{ parseTime(scope.row.createTime,'{y}-{m}-{d}' ) }}</span>
            </template>
          </el-table-column>
          <el-table-column
            label="操作"
            align="center"
            width="160"
            class-name="small-padding fixed-width"
          >
            <template #default="scope">
              <el-button
                v-hasPermi="['system:user:edit']"
                size="mini"
                type="text"
                icon="el-icon-edit"
                @click="handleUpdate(scope.row)"
              >
                修改
              </el-button>
              <el-button
                v-if="scope.row.userId !== 1"
                v-hasPermi="['system:user:remove']"
                size="mini"
                type="text"
                icon="el-icon-delete"
                @click="handleDelete(scope.row)"
              >
                删除
              </el-button>
              <el-button
                v-hasPermi="['system:user:resetPwd']"
                size="mini"
                type="text"
                icon="el-icon-key"
                @click="handleResetPwd(scope.row)"
              >
                重置
              </el-button>
            </template>
          </el-table-column>
        </el-table>

        <pagination
          v-show="total>0"
          :total="total"
          v-model:page="queryParams.pageNum"
          v-model:limit="queryParams.pageSize"
          @pagination="getList"
        />
      </el-col>
    </el-row>

    <!-- 添加或修改参数配置对话框 -->
    <el-dialog
      :title="title"
      v-model="open"
      width="600px"
      append-to-body
      @opened="showDialog"
    >
      <el-form
        ref="addForm"
        :model="formVal"
        :rules="rules"
        label-width="80px"
      >
        <el-row>
          <el-col :span="12">
            <el-form-item
              label="用户昵称"
              prop="nickName"
            >
              <el-input
                v-model="formVal.nickName"
                placeholder="请输入用户昵称"
              />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item
              label="归属部门"
              prop="deptId"
            >
              <Treeselect
                :treeProps="props"
                :options="deptOptions"
                placeholder="请选择归属部门"
                :originOptions="originOptions"
                :defalut="formVal.deptId"
                :accordion="isAccordion"
                @getValue="getParentValue($event)"
                :user="true"
              />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="12">
            <el-form-item
              label="手机号码"
              prop="phonenumber"
            >
              <el-input
                v-model="formVal.phonenumber"
                placeholder="请输入手机号码"
                maxlength="11"
              />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item
              label="邮箱"
              prop="email"
            >
              <el-input
                v-model="formVal.email"
                placeholder="请输入邮箱"
                maxlength="50"
              />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="12">
            <el-form-item
              v-if="formVal.userId === undefined"
              label="用户名称"
              prop="userName"
            >
              <el-input
                v-model="formVal.userName"
                placeholder="请输入用户名称"
              />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item
              v-if="formVal.userId === undefined"
              label="用户密码"
              prop="password"
            >
              <el-input
                v-model="formVal.password"
                placeholder="请输入用户密码"
                type="password"
              />
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="12">
            <el-form-item label="用户性别">
              <el-select
                v-model="formVal.sex"
                placeholder="请选择"
              >
                <el-option
                  v-for="dict in sexOptions"
                  :key="dict.dictValue"
                  :label="dict.dictLabel"
                  :value="dict.dictValue"
                />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="状态">
              <el-radio-group v-model="formVal.status">
                <el-radio
                  v-for="dict in statusOptions"
                  :key="dict.dictValue"
                  :label="dict.dictValue"
                >
                  {{ dict.dictLabel }}
                </el-radio>
              </el-radio-group>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="12">
            <el-form-item label="岗位">
              <el-select
                v-model="formVal.postIds"
                multiple
                placeholder="请选择"
              >
                <el-option
                  v-for="item in postOptions"
                  :key="item.postId"
                  :label="item.postName"
                  :value="item.postId"
                  :disabled="item.status === 1"
                />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="角色">
              <el-select
                v-model="formVal.roleIds"
                multiple
                placeholder="请选择"
              >
                <el-option
                  v-for="item in roleOptions"
                  :key="item.roleId"
                  :label="item.roleName"
                  :value="item.roleId"
                  :disabled="item.status === 1"
                />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="24">
            <el-form-item label="备注">
              <el-input
                v-model="formVal.remark"
                type="textarea"
                placeholder="请输入内容"
              />
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <template #footer>
        <div
          class="dialog-footer"
        >
          <el-button
            type="primary"
            @click="submitForm"
          >
            确 定
          </el-button>
          <el-button @click="cancel">
            取 消
          </el-button>
        </div>
      </template>
    </el-dialog>

    <!-- 用户导入对话框 -->
    <el-dialog
      :title="upload.title"
      v-model="upload.open"
      width="400px"
      append-to-body
    >
      <el-upload
        :action="upload.url + '?updateSupport=' + upload.updateSupport"
        :disabled="upload.isUploading"
        :on-progress="handleFileUploadProgress"
        :on-success="handleFileSuccess"
        accept=".xlsx, .xls"
        :headers="upload.headers"
        :auto-upload="false"
        drag
        ref="uploadRef"
        :limit="1"
      >
        <i class="el-icon-upload" />
        <div class="el-upload__text">
          将文件拖到此处，或
          <em>点击上传</em>
        </div>

        <template #tip>
          <div
            class="el-upload__tip"
          >
            <el-checkbox v-model="upload.updateSupport" />是否更新已经存在的用户数据
            <el-link
              type="info"
              style="font-size:12px"
              @click="importTemplateDown"
            >
              下载模板
            </el-link>
          </div>
        </template>

        <div
          class="el-upload__tip"
          style="color:red"
        >
          提示：仅允许导入“xls”或“xlsx”格式文件！
        </div>
      </el-upload>
      <!-- <el-upload

      >

      </el-upload> -->
      <template #footer>
        <div
          class="dialog-footer"
        >
          <el-button
            type="primary"
            @click="submitFileForm"
          >
            确 定
          </el-button>
          <el-button @click="upload.open = false">
            取 消
          </el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script lang='ts'>
import { listUser, getUser, delUser, addUser, updateUser, exportUser, resetUserPwd, changeUserStatus, importTemplate } from '@/apis/system/user'
import { getToken } from '@/utils/cookies'
import { treeselect } from '@/apis/system/dept'

import Treeselect from '@/components/tree-select/Index.vue'

// import Treeselect from '@riophae/vue-treeselect'
// import '@riophae/vue-treeselect/dist/vue-treeselect.css'
import { defineComponent, reactive, toRefs, ref, unref, onMounted, watchEffect } from 'vue'
import { ElMessage, ElMessageBox, ElTree } from 'element-plus'
import { download, parseTime } from '@/utils/ruoyi'
import { getDicts, getConfigKey } from '@/apis/system/system'

export default defineComponent({
  components: {
    Treeselect
  },
  setup () {
    const treeRef = ref(ElTree)
    const queryForm = ref<HTMLInputElement | null>(null)
    const addForm = ref<HTMLInputElement | null>(null)
    const dataMap = reactive({
      props: { // 配置项（必选）
        value: 'id',
        label: 'label',
        children: 'children'
        // disabled:true
      },
      addformFlag: false,
      // 阻止触发switch  change事件
      tigger: false,
      // 遮罩层
      loading: true,
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
      originOptions: [],
      // 用户表格数据
      userList: null,
      // 弹出层标题
      title: '',
      // 部门树选项
      deptOptions: [],
      // 是否显示弹出层
      open: false,
      // 部门名称
      deptName: '',
      // 默认密码
      initPassword: '',
      // 日期范围
      dateRange: [],
      // 状态数据字典
      statusOptions: [],
      // 性别状态字典
      sexOptions: [],
      // 岗位选项
      postOptions: [],
      // 角色选项
      roleOptions: [],
      // 表单参数
      formVal: {
        userId: undefined,
        deptId: '',
        userName: undefined,
        nickName: undefined,
        password: '',
        phonenumber: undefined,
        email: undefined,
        sex: undefined,
        status: '0',
        remark: undefined,
        postIds: [],
        roleIds: []

      },
      defaultProps: {
        children: 'children',
        label: 'label'
      },
      // 用户导入参数
      upload: {
        // 是否显示弹出层（用户导入）
        open: false,
        // 弹出层标题（用户导入）
        title: '',
        // 是否禁用上传
        isUploading: false,
        // 是否更新已经存在的用户数据
        updateSupport: 0,
        // 设置上传的请求头部
        headers: { Authorization: 'Bearer ' + getToken() },
        // 上传的地址
        url: process.env.VUE_APP_BASE_API + '/system/user/importData'
      },
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        userName: undefined,
        phonenumber: undefined,
        status: undefined,
        deptId: undefined
      },
      // 列信息
      columns: [
        { key: 0, label: '用户编号', visible: true },
        { key: 1, label: '用户名称', visible: true },
        { key: 2, label: '用户昵称', visible: true },
        { key: 3, label: '部门', visible: true },
        { key: 4, label: '手机号码', visible: true },
        { key: 5, label: '状态', visible: true },
        { key: 6, label: '创建时间', visible: true }
      ],
      // 表单校验
      rules: {
        userName: [
          { required: true, message: '用户名称不能为空', trigger: 'blur' }
        ],
        nickName: [
          { required: true, message: '用户昵称不能为空', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '用户密码不能为空', trigger: 'blur' }
        ],
        email: [
          {
            type: 'email',
            message: "'请输入正确的邮箱地址",
            trigger: ['blur', 'change']
          }
        ],
        phonenumber: [
          {
            pattern: /^1[3|4|5|6|7|8|9][0-9]\d{8}$/,
            message: '请输入正确的手机号码',
            trigger: 'blur'
          }
        ]
      }
    })

    /** 查询用户列表 */
    const getList = () => {
      dataMap.loading = true
      dataMap.tigger = true
      listUser(dataMap.queryParams).then(response => {
        dataMap.userList = response.rows
        dataMap.total = response.total
        dataMap.loading = false
      }
      )
    }

    const flatten = (origin: any) => {
      let result: any = []
      for (let i = 0; i < origin.length; i++) {
        const item = origin[i]
        if (Array.isArray(item.children)) {
          result = result.concat(flatten(item.children))
        } else {
          result.push(item)
        }
      }
      return result
    }
    /** 查询部门下拉树结构 */
    const getTreeselect = () => {
      treeselect().then(response => {
        dataMap.deptOptions = response?.data
        dataMap.originOptions = flatten(response?.data)
      })
    }

    // 筛选节点
    const filterNode = (value: string, data: any) => {
      if (!value) return true
      return data.label.indexOf(value) !== -1
    }
    // 节点单击事件
    const handleNodeClick = (data: {[key: string]: any}) => {
      dataMap.queryParams.deptId = data.id
      getList()
    }
    // 用户状态修改
    const handleStatusChange = (row: {[key: string]: any}) => {
      if (dataMap.tigger) {
        const text = row.status === '0' ? '启用' : '停用'
        ElMessageBox.confirm('确认要"' + text + '""' + row.userName + '"用户吗?', '警告', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(function () {
          return changeUserStatus(row.userId, row.status)
        }).then(() => {
          ElMessage.success(text + '成功')
        }).catch(function () {
          row.status = row.status === '0' ? '1' : '0'
        })
      }
    }

    /** 搜索按钮操作 */
    const handleQuery = () => {
      dataMap.queryParams.pageNum = 1
      getList()
    }
    /** 重置按钮操作 */
    const resetQuery = () => {
      (queryForm.value as any).resetFields()
      handleQuery()
    }

    const resetForm = () => {
      (addForm.value as any).resetFields()
    }
    // 多选框选中数据
    const handleSelectionChange = (selection: any) => {
      dataMap.ids = selection.map((item: any) => item.userId)
      dataMap.single = selection.length !== 1
      dataMap.multiple = !selection.length
    }
    /** 新增按钮操作 */
    const handleAdd = () => {
      dataMap.addformFlag = true
      dataMap.formVal = {
        userId: undefined,
        deptId: '',
        userName: undefined,
        nickName: undefined,
        password: '',
        phonenumber: undefined,
        email: undefined,
        sex: undefined,
        status: '0',
        remark: undefined,
        postIds: [],
        roleIds: []
      }
      dataMap.originOptions = []
      getTreeselect()
      getUser().then(response => {
        dataMap.postOptions = response.posts
        dataMap.roleOptions = response.roles
        dataMap.open = true
        dataMap.title = '添加用户'
        dataMap.formVal.password = dataMap.initPassword
      })
    }

    /** 修改按钮操作 */
    const handleUpdate = (row: {[key: string]: any}) => {
      const userId = row.userId || dataMap.ids
      getUser(userId).then(response => {
        dataMap.formVal = response.data
        dataMap.postOptions = response.posts
        dataMap.roleOptions = response.roles
        dataMap.formVal.postIds = response.postIds
        dataMap.formVal.roleIds = response.roleIds
        dataMap.open = true
        dataMap.title = '修改用户'
        dataMap.formVal.password = ''
        dataMap.formVal.deptId = response.data.deptId
      })
    }
    /** 重置密码按钮操作 */
    const handleResetPwd = (row: {[key: string]: any}) => {
      ElMessageBox.prompt('请输入"' + row.userName + '"的新密码', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消'
      }).then(({ value }) => {
        resetUserPwd(row.userId, value).then(() => {
          ElMessage.success('修改成功，新密码是：' + value)
        })
      }).catch((err: any) => console.log(err))
    }
    /** 提交按钮 */
    const submitForm = () => {
      (queryForm.value as any).validate((valid: any) => {
        if (valid) {
          if (dataMap.formVal.userId !== undefined) {
            updateUser(dataMap.formVal).then(() => {
              ElMessage.success('修改成功')
              dataMap.open = false
              getList()
            })
          } else {
            addUser(dataMap.formVal).then(() => {
              ElMessage.success('x新增成功')
              dataMap.open = false
              getList()
            })
          }
        }
      })
    }
    /** 删除按钮操作 */
    const handleDelete = (row: {[key: string]: any}) => {
      const userIds = row.userId || dataMap.ids
      ElMessageBox.confirm('是否确认删除用户编号为"' + userIds + '"的数据项?', '警告', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(function () {
        return delUser(userIds)
      }).then(() => {
        getList()
        ElMessage.success('刪除成功')
      })
    }

    /** 导出按钮操作 */
    const handleExport = () => {
      const queryParams = dataMap.queryParams
      ElMessageBox.confirm('是否确认导出所有用户数据项?', '警告', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(function () {
        return exportUser(queryParams)
      }).then(response => {
        download(response.msg)
      })
    }
    /** 导入按钮操作 */
    const handleImport = () => {
      dataMap.upload.title = '用户导入'
      dataMap.upload.open = true
    }
    /** 下载模板操作 */
    const importTemplateDown = () => {
      importTemplate().then((response: {[key: string]: any}) => {
        download(response.msg)
      })
    }
    // 文件上传中处理
    const handleFileUploadProgress = () => {
      dataMap.upload.isUploading = true
    }
    // 文件上传成功处理
    const handleFileSuccess = (response: {[key: string]: any}) => {
      dataMap.upload.open = false
      dataMap.upload.isUploading = false
      //   dataMap.$refs.upload.clearFiles()
      ElMessageBox.alert(response.msg, '导入结果', { dangerouslyUseHTMLString: true })
      getList()
    }
    // 提交上传文件
    const submitFileForm = () => {
    //   this.$refs.upload.submit()
      console.log('upload')
    }

    watchEffect(() => {
      if (dataMap.deptName) {
        const tree = unref(treeRef)
        tree.filter(dataMap.deptName)
      }
    })
    onMounted(() => {
      getList()
      getTreeselect()
      getDicts('sys_normal_disable').then(response => {
        dataMap.statusOptions = response?.data
      })
      getDicts('sys_user_sex').then(response => {
        dataMap.sexOptions = response?.data
      })
      getConfigKey('sys.user.initPassword').then(response => {
        dataMap.initPassword = String(response?.msg)
      })
    })

    const showDialog = () => {
      getTreeselect()
    }
    return { ...toRefs(dataMap), resetForm, addForm, showDialog, flatten, parseTime, queryForm, treeRef, handleQuery, handleExport, submitFileForm, handleImport, handleFileSuccess, handleStatusChange, handleFileUploadProgress, importTemplateDown, handleNodeClick, filterNode, getTreeselect, getList, resetQuery, handleAdd, handleSelectionChange, handleUpdate, handleResetPwd, submitForm, handleDelete }
  }
})
</script>
<style lang="scss" scoped>
.small-padding {
  .cell {
    padding-left: 5px;
    padding-right: 5px;
  }
}

.fixed-width {
  .el-button--mini {
    padding: 7px 10px;
    width: 60px;
  }
}

</style>
