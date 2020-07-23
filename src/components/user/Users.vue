<template>
    <div>
     <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>
     <!-- 卡片区域 -->
        <el-card class="box-card">
           <!-- 搜索区域 -->         
           <el-row :gutter="20">
                  <el-col :span="10">
                     <el-input placeholder="请输入内容" class="input-with-select" v-model="queryInfo.query" clearable @clear="getUserList">
                           <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                     </el-input>
                  </el-col>
                  <el-col :span="5">
                      <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
                  </el-col>              
           </el-row>
           <!-- 用户列表区域 -->
           <el-table :data="userlist" border stripe >
              <el-table-column label="#" type="index"></el-table-column> 
             <el-table-column label="姓名" prop="username"></el-table-column>
             <el-table-column label="邮箱" prop="email"></el-table-column>
             <el-table-column label="电话" prop="mobile"></el-table-column>
             <el-table-column label="角色" prop="role_name"></el-table-column>
             <el-table-column label="状态">
                 <!-- 用作用域插槽拿到状态中的布尔值 这个scope就是当前行对象-->
                 <template slot-scope="scope">
                     <el-switch
                       v-model="scope.row.mg_state"
                       active-color="#13ce66"
                       inactive-color="#ff4949" @change="userStateChanged(scope.row)">
                     </el-switch>
                 </template>
             </el-table-column>

             <el-table-column label="操作" width="180px">
                 <template slot-scope="scope">
                     <!-- 修改用户按钮 -->
                   <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
                   <!-- 删除按钮 -->
                   <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
                   <!-- 分配角色按钮 -->
                   <el-tooltip content="分配角色" placement="top" :enterable="false" >
                       <el-button 
                       type="warning" 
                       icon="el-icon-setting" 
                       size="mini"
                       @click="setRole(scope.row)"
                       ></el-button>
                   </el-tooltip>
                 </template>
             </el-table-column>


           </el-table>

           <!-- 分页区域-->
           <el-pagination
               @size-change="handleSizeChange"
               @current-change="handleCurrentChange"
               :current-page="queryInfo.pagenum"
               :page-sizes="[1, 2, 3, 4]"
               :page-size="queryInfo.pagesize"
               layout="total, sizes, prev, pager, next, jumper"
               :total="total">
           </el-pagination>
            
        </el-card>

    <!-- 添加用户对话框 -->
        <el-dialog
            title="提示"
            :visible.sync="addDialogVisible"
            width="50%" @close="addDialogClosed"
            >
            <!-- 主体区域 -->
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
            <el-form-item label="用户名" prop="username">
              <el-input type="text" v-model="addForm.username" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="密码" prop="password">
              <el-input type="text" v-model="addForm.password" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="邮箱" prop="email">
              <el-input type="text" v-model="addForm.email" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="手机" prop="mobile">
              <el-input type="text" v-model="addForm.mobile" autocomplete="off"></el-input>
            </el-form-item>           
            </el-form>
            <!-- 底部区域 -->
            <span slot="footer" class="dialog-footer">
              <el-button @click="addDialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="addUser">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 编辑用户对话框 -->
        <el-dialog
            title="修改用户"
            :visible.sync="editDialogVisible"
            width="50%"
            @close="editFormClosed"
            >
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
                <el-form-item label="用户名">
                    <el-input v-model="editForm.username" disabled></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="editForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="editForm.mobile"></el-input>
                </el-form-item>
            </el-form>    
            <span slot="footer" class="dialog-footer">
              <el-button @click="editDialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="editUserInfo">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 分配角色对话框 -->
        <el-dialog
            title="分配角色"
            :visible.sync="setRoleDialogVisble"
            width="50%"
            @close="setRoleDialogColsed"

            >
            <div>
                <p>{{userInfo.username}}</p>
                <p>{{userInfo.role_name}}</p>
                <p>分配新角色
                    <el-select v-model="selectedRoleId" placeholder="请选择">
                        <el-option
                          v-for="item in rolesList"
                          :key="item.id"
                          :label="item.roleName"
                          :value="item.id">
                        </el-option>
                   </el-select>
                </p>
            </div>
            <span slot="footer" class="dialog-footer">
              <el-button @click="setRoleDialogVisble = false">取 消</el-button>
              <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
            </span>
       </el-dialog>
        
    </div>
</template>

<script>
export default {
    data() {
        // 验证邮箱的规则
        let checkEmail = (rules, value, callback) => {
            // 验证邮箱是否合法的正则表达式
            const regEmail = /^([a-zA-Z]|[0-9])(\w|\-)+@[a-zA-Z0-9]+\.([a-zA-Z]{2,4})$/;
            if(regEmail.test(value)) return callback()
            // 如果邮箱不合法
            callback(new Error("请输入合法邮箱"))



        }
        //    验证手机号的规则
        let checkMobile = (rules, value, callback) => {
            // 验证手机号是否合法的正则表达式
            const regMobile = /^1(2|3|4|5|6|7|8|9)\d{9}$/; 
            if(regMobile.test(value)) return callback()
            // 如果邮箱不合法
            callback(new Error("请输入合法手机号"))



        }
        return {

            //获取用户列表的数据
            queryInfo: {
                query: '',
                //当前页数
                pagenum: 1,
                // 当前每页显示多少条数据
                pagesize: 2 
            },
            userlist:[],
            total: 0,

            //添加用户提示框的显示与隐藏
            addDialogVisible:false,
            //编辑用户对话框的显示与隐藏
            editDialogVisible: false,
            // 添加用户的表单数据
            addForm:{
                username: '',
                password: '',
                email: '',
                mobile: ''

            },
            
             // 添加用户的表单数据的验证规则
            addFormRules: {
               username: [
                   { require: true, message:'请输入用户名', trigger: 'blur' },
                   { min: 3, max: 10, message:"请输入3~10位的用户名", trigger: 'blur'}
               ],
               password: [
                   { require: true, message:'请输入密码', trigger: 'blur'},
                   { min: 6, max: 10, message:"请输入6~10位的密码", trigger: 'blur'}
               ],
               email: [
                   { require: true, message:'请输入邮箱', trigger: 'blur'},
                   { validator: checkEmail, trigger: 'blur' },
                   { min: 6, max: 10, message:"请输入6~10位的邮箱", trigger: 'blur'}
               ],
               mobile: [
                   { require: true, message:'请输入手机号', trigger: 'blur' },
                   { validator: checkMobile, trigger: 'blur' },
                   { min: 10, max: 11, message:"请输入11位的手机号", trigger: 'blur'}
               ],      
            },
            //查询到的用户信息
            editForm: {},
            // 编辑用户的表单验证规则对象
            editFormRules:{
                email: [
                    {require: true, message:'邮箱不能为空', trigger: 'blur'},
                    {validator: checkEmail, trigger:'blur'}
                ],
                mobile: [
                    {require: true, message: '手机号不能为空', trigger: 'blur'},
                    {validator: checkMobile, trigger: 'blur'}

                ]
            },
            // 控制分配角色对话框的显示与隐藏、
            setRoleDialogVisble:false,
            // 需要被分配角色的用户信息
            userInfo: {},
            // 所有角色的数据列表
            rolesList:[],
            // 下拉框已选中的角色id值
            selectedRoleId: ''
      }
    },
    created () {
        this.getUserList()
        
    },
    methods: {
        //获取用户列表
        async getUserList() {
            const { data: res } = await this.$http.get('users', {
              params: this.queryInfo
            })
            if (res.meta.status !== 200) {
              return this.$message.error('获取用户列表失败！')
            }
            this.userlist = res.data.users
            this.total = res.data.total
            // console.log(res)
        },
        //监听pageSize改变事件
        handleSizeChange(newSize) {
            // console.log(newSize);
            this.queryInfo.pagesize = newSize;
            this.getUserList()

        },
        //监听页码值改变事件
        handleCurrentChange(newPage) {
            this.queryInfo.pagenum = newPage;
            this.getUserList()

        },
        //监听switch开关状态的改变
        async userStateChanged(userinfo) {
            // console.log(userinfo);
            const {data : res }= await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
            if(res.meta.status !== 200) {
                userinfo.mg_state = !userinfo.mg_state; 
                return this.$message.error('更新用户状态失败')
            }
            this.$message.success('更新用户状态成功')

        },
        // 监听添加用户对话框关闭事件
        addDialogClosed() {
            this.$refs.addFormRef.resetFields()

        },
        // 点击按钮添加新用户
        addUser() {
            this.$refs.addFormRef.validate(async vali => {
                if(!vali) return
                //可以发起网络用户请求 
                const {data: res} = await this.$http.post('users',this.addForm)
                if(res.meta.status != 201) {
                    this.$message.error('添加用户失败')
                }
                this.$message.success('添加用户成功')
                //添加成功后隐藏添加用户的对话框
                this.addDialogVisible = false
                // 重新获取用户列表
                this.getUserList()

            })         
        },
        // 展示编辑用户的对话框
        async showEditDialog(id) {
            // console.log(id);
            const {data: res} = await this.$http.get('users/'+ id,)
            if(res.meta.status != 200) return this.$message.error('查询用户失败')
            // 如果查询成功
            this.editForm = res.data
            this.editDialogVisible = true

        },
        //c重置编辑用户对话框
        editFormClosed() {
            // console.log(this.$refs);
            this.$refs.editFormRef.resetFields()
        },
        // 编辑用户对话框确认按钮提交前数据校验
        editUserInfo() {
            this.$refs.editFormRef.validate(async valid => {
                if(!valid) return 
                //如果数据符合要求则提交修改
               const {data: res} = await this.$http.put('users/'+ this.editForm.id,{email: this.editForm.email, mobile: this.editForm.mobile})
               
               if(res.meta.status != 200 ) return this.$message.error('修改用户数据失败')

               //如果修改成功
                 //g关闭编辑用户对话框,并提示修改成功，重新刷新数据列表
                this.editDialogVisible = false;
                
                this.getUserList()
                this.$message.success('修改用户数据成功')




            })
            
        },

        //根据id、
        // 删除用户
        async removeUserById(id) {
            // console.log(id);
            const confirmResult = await this.$confirm('此操作将永久删除该用户', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch( err => {
                // 用户点击取消后，返回的则是cancle字符串
                return err
            })

            //如果用户点击确认删除按钮，则返回的是字符串confirm
            // console.log(confirmResult)
            if(confirmResult != 'confirm') {
                return this.$message.info('已取消删除')
            }
            // this.$message.success('删除成功')
            const {data: res} = await this.$http.delete('users/' + id)
            console.log(res);
            
            if(res.meta.status !== 200) {
                return this.$message.error('删除用户失败')
            }
            

            this.$message.success('删除用户成功')
            this.getUserList()
 


        },
        // 展示分配角色的对话框
        async setRole(userInfo) {
            // console.log(userInfo);
            this.userInfo = userInfo
            //展示对话框之前，获取所有角色的列表
            const {data: res} = await this.$http.get(`roles`)
            if(res.meta.status !== 200) {
                return this.$message.error('获取角色列表失败')
            }

            //如果成功则接收
            this.rolesList = res.data;
            this.setRoleDialogVisble = true;       
        },
        async saveRoleInfo() {

            if(!this.selectedRoleId) {
                return this.$message.error('请选择要分配的角色')

            }

            const { data: res} = await this.$http.put(`users/${this.userInfo.id}/role`, { rid: this.selectedRoleId})
            if(res.meta.status !== 200) {
                return this.$message.error('更新角色失败')
            }
            this.$message.success('更新角色成功')
            this.getUserList()
            // 关闭对话框
            this.setRoleDialogVisble = false
        },
        // 监听角色分配对话框关闭事件,将其重置为空
        setRoleDialogColsed() {
            this.selectedRoleId = ''
            this.userInfo = {}


        }
    
    }
}

</script>

<style lang="less" scoped>
.el-breadcrumb {
    margin-bottom: 15px;
}
.el-table {
    margin-top: 15px;
    font-size: 12px;
}


</style>