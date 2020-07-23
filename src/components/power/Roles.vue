<template>
    <div>
           <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>


        <!-- 卡片区域 -->
        <el-card>
            <!-- 头部  添加角色按钮-->
            <el-row>
                <el-button type="primary">添加角色</el-button>
            </el-row>
            <!-- 列表区域 -->
            <el-table :data="roleList" border stripe >
                <!-- 配置》展开按钮 -->
              <el-table-column type="expand">
                  <template slot-scope="scope">
                      <el-row 
                      :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" 
                      v-for="(item1,i1) in scope.row.children" 
                      :key="item1.id">
                        <!-- 渲染一级权限 -->
                        <el-col :span="5">
                            <el-tag
                            @click="removeRightById(scope.row, item1.id)"
                            closable 
                            >
                            {{item1.authName}}</el-tag>
                            <i class="el-icon-caret-right"></i>                         
                        </el-col> 
                        <!-- 渲染二级、三级权限 -->
                        <el-col :span="19">
                            <!-- 通过for循环 嵌套渲染二级权限 -->
                            <el-row 
                            :class="[i2 === 0 ? '': 'bdtop', 'vcenter']" 
                            v-for="(item2,i2) in item1.children" 
                            :key="item2.id">
                              <el-col :span="6">
                                  <el-tag type="success"
                                  closable 
                                  @click="removeRightById(scope.row, item2.id)"
                                  > {{item2.authName}}</el-tag>
                                  <i class="el-icon-caret-right"></i> 
                              </el-col>

                              <el-col :span="18">
                                  <el-tag 
                                  closable 
                                  type="warning" 
                                  v-for="(item3, i3) in item2.children" 
                                  :key="item3.id"
                                  @click="removeRightById(scope.row, item3.id)"
                                  >
                                      {{ item3.authName}}
                                  </el-tag>
                                  
                              </el-col>
                            </el-row>
                        </el-col>
                      </el-row>
                     

                  </template>
              </el-table-column>

              <el-table-column label="#" type="index"></el-table-column>
              <el-table-column label="角色名称" prop="roleName"></el-table-column>
              <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
              <el-table-column label="操作" width="300px">
                  <template slot-scope="scope" >
                      <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
                      <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
                      <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                  </template>
              </el-table-column>
            </el-table>

        </el-card>

        <!-- 分配用户权限对话框 -->
        <el-dialog
            title="分配用户权限"
            :visible.sync="setRightDialogVisible"
            width="50%"
            @close="setRightDialogClosed"
            >
            <!-- 树形结构哦空间 -->
            <el-tree 
            :data="rightsList" 
            :props="treeProps" 
            show-checkbox 
            node-key="id"
            :default-checked-keys = "defkeys" 
            default-expand-all
            ref="treeRef"
            >
            </el-tree>
            <span slot="footer" class="dialog-footer">
              <el-button @click="setRightDialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="allotRights">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>
<script>
export default {
    data() {
        return {
            // 用户角色列表
            roleList: [],
            //控制分配权限对话框的显示与隐藏
            setRightDialogVisible:false,
            //存储分配用户权限列表对话框中的数据
            rightsList:[],
            treeProps:{
                children: 'children',
                label: 'authName'
            },
            defkeys:[],
            //当前即将分配权限的角色id
            roleId: ''

        }
    },
    created() {
        this.getRoleList()
        
    },
    methods: {
        async getRoleList() {
         const {data: res} = await this.$http.get('roles')
         console.log(res);
         if(res.meta.status !== 200) {
            return this.$message.error("获取用户角色列表失败")
         }
         this.roleList = res.data
        },
        // 根据id删除对应的三级权限
        async removeRightById(role, rightId) {
           const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
           }).catch((error) => {
               console.log(error);                                
           });
           if(confirmResult !== 'confirm') {
              return this.$message.info('已取消')

           }
        //    this.$message.success('删除成功')
         const {data:res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
         console.log(res);
         if(res.meta.status !== 200) {
             this.$message.error('删除失败')
         }

        //  如果删除成功了，重新获取角色列表
        
        // this.getRoleList()
        role.children = res.data




        },
        async showSetRightDialog(role) {
            this.roleId = role.id
            //弹出用户分配权限对话框之前获取所有的数据
            const {data: res} = await this.$http.get('rights/tree')
            if(res.meta.status !== 200) {
                return this.$message.error('获取用户分配权限列表失败')
            }
            //如果成功则将分配用户权限列表中的数据存储起来
            this.rightsList = res.data
            console.log(this.rightsList)

            //递归三级节点的id
            this.getLeafkeys(role, this.defkeys)

             //弹出分配用户权限对话框
            this.setRightDialogVisible = true

        },

        //通过递归的形式，获取角色下所有三级权限的id,并保存到defkeys数组中
        getLeafkeys(node, arr) {
            //如果当前节点不包含children属性，则是三级节点
           if(!node.children) {
               //如果是
               return arr.push(node.id)

           }
           //如果不是则继续递归,直到找到为止
           node.children.forEach(item => 
              this.getLeafkeys(item, arr))

        },
        //监听分配权限对话框关闭事件
        setRightDialogClosed() {
            this.defkeys = []
        },
        //点击为角色分配权限
        async allotRights() {
            //获取勾选按钮所在权限的id
            const keys = [
                // getCheckedKeys、getHalfCheckedKeys都是tree树形结构给定的方法
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ]
            
            // console.log(keys);
            // 将获取到的id数组转成
           const idStr = keys.join(',')

           const {data: res} = await this.$http.post(`roles/${this.roleId}/rights`, {rids:idStr})
        //    const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`,{ rids: idStr })
           console.log(res);

           if(res.meta.status !== 200) {
               return this.$message.error('分配权限失败')
           }

           this.$message.success('分配权限成功')

           //重新刷新列表
           this.getRoleList()
           //关闭对话框
           
           this.setRightDialogVisible = false


        }
        


    

        

        
    },

    
}
</script>
<style lang="less" scoped>
.el-tag {
    margin: 7px;  

}
.bdtop {
    border-top: 1px solid #eee;

}
.bdbottom {
    border-bottom: 1px solid #eee;

}
.vcenter {
    display: flex;
    align-items: center;
}


</style>