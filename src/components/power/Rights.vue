<template>
    <div>
            <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>权限列表</el-breadcrumb-item>
        </el-breadcrumb>

        <!-- 卡片区域 -->
        <el-card>
            <el-table :data="rightsList" border stripe >

              <el-table-column label="#" type="index"></el-table-column>
              <el-table-column label="权限名称" prop="authName"></el-table-column>
              <el-table-column label="路径" prop="path"></el-table-column>
              <el-table-column label="等级" prop="level">
                  <template slot-scope="scope">
                      <el-tag v-if="scope.row.level === '0'">等级一</el-tag>
                      <el-tag v-else-if="scope.row.level === '1'" type="success">等级二</el-tag>
                      <el-tag v-else type="danger">等级三</el-tag>
                  </template>
              </el-table-column>
            </el-table>
        
            
             
        </el-card>
    </div>    
    
</template>
<script>
export default {
    data() {
        return {
            //权限列表中的数据
            rightsList: []
        }
    },
    created() {
        this.getRightsList() 
    },
    methods: {
        async getRightsList() {
            const  {data: res} = await this.$http.get('rights/list')
            console.log(res);
            if(res.meta.status !== 200) {
                return this.$message.error('获取权限列表失败')
            }
            this.rightsList = res.data
        }

        
    },
    
}
</script>
<style lang="less" scoped>
.el-breadcrumb {
    margin-bottom: 15px;
}

</style>