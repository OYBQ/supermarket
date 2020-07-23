<template>
    <div>
          <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        
        <!-- 卡片视图区域 -->
        <el-card>
            <!-- 添加商品按钮 -->
            <el-row>
              <el-col>
                  <el-button type="primary" @click="showAddCateDialog">添加商品</el-button>
              </el-col>
            </el-row>
            <!-- 表格区域 -->
            <tree-table 
            class="treeTable"
            :data="catelist" 
            :columns="columns" 
            :selection-type ="false" 
            :expand-type="false" 
            show-index 
            index-text="#"
            :border = 'true'
            :show-row-hover ="false"
            >    
                <!-- 是否有效列 -->
                <template slot="isOk" slot-scope="scope">
                    <i 
                    style="color: lightgreen"
                    class="el-icon-success" 
                    v-if="scope.row.cat_deleted === false"
                    
                    ></i>
                    <i class="el-icon-error" v-else style="color: red"></i>

                </template>
                <!-- 排序列 -->
                <template slot="order" slot-scope="scope">
                    <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
                    <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
                    <el-tag type="warning" size="mini" v-else>三级</el-tag>
                </template>

                <!-- 操作 -->
                <template slot="opt" slot-scope="scope">
                    <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                    <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>

                </template>
            </tree-table>
          


            <!-- 分页区域 -->
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[3, 5, 10, 15]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>
        </el-card>

        <!-- 添加分类的对话框 -->
        <el-dialog
            title="添加分类"
            :visible.sync="addCateDialogVisible"
            width="50%"
            @close="addCateDialogClosed"
            >
            <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
                <el-form-item label="分类名称：" prop="cat_name">
                  <el-input v-model="addCateForm.cat_name"></el-input>
                </el-form-item>
                <el-form-item label="父级分类">
                    <el-cascader 
                    
                    :options="parentCateList" 
                    :props="cascaderProps" 
                    v-model="selectedKeys" 
                    @change="parentCateChanged" 
                    clearable 
                     
                    >
                    </el-cascader>
                  
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
              <el-button @click="addCateDialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="addCate">确 定</el-button>
            </span>
        </el-dialog>

    </div>
</template>
<script>
export default {
    data() {
        return {
            // 查询条件
            queryInfo: {
                type:3,
                pagenum:1,
                pagesize:5

            },
            // 商品分类的数据列表
            catelist: [],
            // 总数据条数
            total: 0,
            // 为talbe指定的列数
            columns: [
                { 
                   label: '分类名称',
                   prop: 'cat_name'
                },
                {
                   label: '是否有效',
                   prop: 'cat_deleted',
                   //表示将当前列定义为模板列
                   type: 'template',
                   //模板列的名称
                   template: 'isOk',
                },
                {
                   label: '排序',
                   prop: 'cat_level',
                   //表示将当前列定义为模板列
                   type: 'template',
                   //模板列的名称
                   template: 'order',
                },
                {
                   label: '操作',
                   //表示将当前列定义为模板列
                   type: 'template',
                   //模板列的名称
                   template: 'opt',
                },
                
            ],
            //控制添加分类的对话框的显示与隐藏
            addCateDialogVisible: false,
            //添加分类的表单数据
            addCateForm: {
                //将要添加的分类名称
                cat_name: '',
                //父级分类的id
                cat_pid: 0,
                //分类的等级，默认要添加的是1级分类
                cat_level: 0

            },
            //添加分类表单的验证规则对象
            addCateFormRules: {
                cat_name: [
                    { require: true, message:'请输入分类名称', trigger: 'blur'}
                ]

            },
            //父级分类的列表
            parentCateList: [],
            //指定级联选择器的配置对象
            cascaderProps: {
                value: 'cat_id',
                label: 'cat_name',
                children: 'children',
                // 次级菜单的展开方式
                expandTrigger:'hover',
                checkStrictly: 'true'
                
            },
            //选中的父级分类的id数组
            selectedKeys: []


            
        }
    },
    created() {
        this.getCatelist()
        
    },
    methods: {
       async getCatelist() {
         const {data:res}  = await this.$http.get(`categories`, { params: this.queryInfo })
        //  console.log(res)
         if(res.meta.status !== 200) {
             this.$message.error('获取商品列表失败')
         }
         this.catelist = res.data.result
        //  为总数据条数赋值
         this.total = res.data.total
        //  console.log(this.catelist);

        },
        // 监听pagesize改变
        handleSizeChange(newSize) {
            console.log("newSize" + newSize);
            this.queryInfo.pagesize = newSize
            this.getCatelist()

        },
        //监听pagenum的改变
        handleCurrentChange(newPage) {
            // console.log("newPage" + newPage);
            this.queryInfo.pagenum = newPage
            this.getCatelist()
        },
        //点击按钮展示添加分类对话框
        showAddCateDialog() {
            //先获取父级分类数据列表
            this.getParentCateList()
            
            //然后再展示添加分类对话框
            this.addCateDialogVisible = true
            
        },
        // 获取父级分类的数据列表
        async getParentCateList() {
            // const { data: res } = await this.$http.get('categories', {
            //     params: this.querInfo
            // })
            const {data: res} = await this.$http.get(`categories`, { params: { type: 2}})
           
            if(res.meta.status !== 200) {
                return this.$message.error('获取父级分类数据失败')
            }
            
            this.parentCateList = res.data
            
            
        },
        //选择项发生变化时触发这个函数
        parentCateChanged() {
            console.log(this.selectedKeys);
            //如果selectedkeys数组中的length大于0，证明选中了父级分类
            //反之，就说明没有选中任何父级分类
            if(this.selectedKeys.length > 0) {
                //父级分类的id
                this.addCateForm.car_pid = this.selectedKeys[this.selectedKeys.length -1]
                //为当前分类的等级赋值
                this.addCateForm.cat_level = this.selectedKeys.length
                return
                
            }else {
                //父级分类的id
                this.addCateForm.cat_pid = 0
                //为当前分类的等级赋值
                this.addCateForm.cat_level = 0
            }
            
        },
        //点击按钮，添加新的分类,并对添加的数据进行预验证
        addCate() {
            this.$refs.addCateFormRef.validate(async valid => {
                if (!valid) return
               const {data: res} = await this.$http.post(`categories`, this.addCateForm)
            //    console.log(res);
               if(res.meta.status !== 201) {
                   return this.$message.error("添加分类失败")
               }

               this.$message.success("添加分类成功")
               this.getCatelist()
               this.addCateDialogVisible = false
                
            })
            // console.log(this.addCateForm);
        },
        //监听对话框关闭事件，重置表单数据
        addCateDialogClosed() {
           

            // this.$refs.addCateFormRef.resetFields()
            this.$refs.addCateFormRef.resetFields()
            //  console.log("123");
            this.selectedKeys = []
            this.addCateForm.cat_level = 0
            this.addCateForm.cat_pid = 0

        }


    
        
    },
    
    
}
</script>
<style lang="less" scoped>
.el-breadcrumb {
    margin-bottom: 15px;
}
.treeTable {
    margin-top: 15px;
}
.el-cascader {
    width: 100%;
}



</style>