<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>

        <!-- 卡片视图区域 -->
        <el-card>
<!--            添加分类按钮-->
            <el-row>
                <el-col>
                    <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
                </el-col>
            </el-row>


        <!-- 表格  tree-table是另外引入的一个组件,然后注册进去的-->
            <tree-table :data="cateList" :columns="columns"
                        :selection-type="false"
                        :expand-type="false"
                        show-index
                        index-text="#"
                        border
                        class="treeTable">
<!--                是否有效-->
                <template v-slot:isok="scope">
                    <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: lightgreen;"></i>
                    <i class="el-icon-error" v-else style="color: red;"></i>
                </template>
                <!--排序-->
                <template v-slot:order="scope">
                    <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
                    <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
                    <el-tag type="warning" size="mini" v-else>三级</el-tag>
                </template>
                <!--操作-->id
                <template v-slot:opt="scope">
                    <el-button type="primary" icon="el-icon-edit" size="mini" @click="editCate(scope.row.cat_id)">编辑</el-button>
                    <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeCateById(scope.row.cat_id)">删除</el-button>
                </template>

            </tree-table>

            <!-- 分页 -->
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
                           :current-page="queryInfo.pagenum" :page-sizes="[3, 5, 10, 15]" :page-size="queryInfo.pagesize"
                           layout="total, sizes, prev, pager, next, jumper" :total="total">
            </el-pagination>
        </el-card>

        <!-- 添加分类对话框 -->
        <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
            <!-- 内容主体区域 -->
            <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
                <el-form-item label="分类名称：" prop="cat_name">
                    <el-input v-model="addCateForm.cat_name"></el-input>
                </el-form-item>
<!--               级联选择器-->
                <el-form-item label="父级分类：">
                    <div class="block">
                        <el-cascader
                                v-model="selectedKeys"
                                :options="parentCateList"
                                :props="{   expandTrigger: 'hover',
                                            value: 'cat_id',
                                            label: 'cat_name',
                                            children: 'children',
                                            checkStrictly: true}"
                                @change="parentCateChange"
                                clearable></el-cascader>
                    </div>
                </el-form-item>
            </el-form>
            <!-- 底部区域 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="addCateDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addCate">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 修改分类的对话框 -->
        <el-dialog title="修改用户" :visible.sync="editCateDialogVisible" width="50%" @close="editCateDialogClosed">
            <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
                <el-form-item label="分类名称：" prop="cat_name">
                    <el-input v-model="addCateForm.cat_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editCateDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editCateInfo">确 定</el-button>
            </span>
        </el-dialog>
    </div>
    
</template>

<script>
    export default {
        name: "Cate",

        data(){
            return {
                //分页查询条件
                queryInfo: {
                    type:3,
                    pagenum:1,
                    pagesize:5
                },
                //商品分类的数据列表
                cateList:[],
                //总数据条数
                total:0,
                //为table指定列的定义
                columns:[
                    {
                    label:'分类名称',
                    prop:'cat_name'
                },{
                    label: '是否有效',
                    //表示将当前列定义为模板列
                    type:'template',
                    //表示当前这一列使用模板名称
                    template: 'isok',

                    },{
                        label: '排序',
                        //表示将当前列定义为模板列
                        type:'template',
                        //表示当前这一列使用模板名称
                        template: 'order',

                    },{
                        label: '操作',
                        //表示将当前列定义为模板列
                        type:'template',
                        //表示当前这一列使用模板名称
                        template: 'opt',
                    }
                ],

                //控制添加分类对话框的显示与隐藏
                addCateDialogVisible: false,

                //添加分类的数据表单对象
                addCateForm:{
                    //将要添加的分类名称
                    cat_name: '',
                    //父级分类的ID
                    cat_pid: 0,
                    //分类的等级，默认要添加的是1级分类
                    cat_level:0
                },

                //添加分类的数据表单的验证规则对象
                addCateFormRules:{
                    cat_name: [
                        { required: true, message: '请输入分类名称', trigger: 'blur' },
                    ]
                },

                //父级分类的列表
                parentCateList:[],

                //选中的父级分类的id数组
                selectedKeys:[],

                //控制修改分类界面的关闭
                editCateDialogVisible: false


            }
        },

        created() {
            this.getCateList()
        },

        methods:{

            //获取商品分类的数据
           async getCateList(){
                const {data : res} = await this.$http.get('categories', {params:this.queryInfo})

               if (res.meta.status !== 200){
                   return this.$message.error('获取商品分类数据失败!')
               }
               console.log(res.data)
               //赋值数据列表
               this.cateList = res.data.result
               //为总数据条数赋值
               this.total = res.data.total
            },


            //监听 pagesize 的改变
            handleSizeChange(newSize){
               this.queryInfo.pagesize = newSize
               this.getCateList()
            },

            //监听pagenum的改变
            handleCurrentChange(newNum){
                this.queryInfo.pagenum = newNum
                this.getCateList()
            },

            //点击按钮，展示添加分类的对话框
            showAddCateDialog(){
               //先获取父级分类的数据列表
                this.getParentCateList()
                //展示出对话框
               this.addCateDialogVisible = true
            },

            //获取父级分类的数据列表
            async getParentCateList(){
              const {data: res} = await this.$http.get('categories',{params:{
                   type:2,
                   }})
                if (res.meta.status !== 200){
                    return this.$message.error('获取父级分类数据失败!')
                }

                console.log(res.data)

                this.parentCateList = res.data
            },

            //选择项发生变化触发这个函数
            parentCateChange(){
               console.log(this.selectedKeys)

                //如果selectedKeys数组中的length大于0，证明选中的父级分类
                //反之，就说明没有选中任何父级选项
                if(this.selectedKeys.length > 0){
                    //父级分类的ID
                    this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
                    //为当前分类的等级赋值
                    this.addCateForm.cat_level = this.selectedKeys.length
                    return
                }else {
                    //父级分类的ID
                    this.addCateForm.cat_pid = 0
                    //为当前分类的等级赋值
                    this.addCateForm.cat_level = 0
                }
            },

            //点击按钮，添加新的分类
            addCate(){
                this.$refs.addCateFormRef.validate(async valid => {
                    if(!valid) return
                  const {data :res} = await this.$http.post('categories', this.addCateForm)

                    if (res.meta.status !== 201){
                        return this.$message.error('添加分类失败!')
                    }

                    this.$message.success('添加分类成功！')
                    this.getCateList()
                    this.addCateDialogVisible = false
                })
               console.log(this.addCateForm)
            },

            //点击添加分类,并重置数据
           async addCateDialogClosed(){
               //重置表单数据
               this.$refs.addCateFormRef.resetFields()
               //清空数据
               this.selectedKeys = []
               this.addCateForm.cat_level = 0
               this.addCateForm.cat_pid = 0

            },

            //根据id查询分类信息
           async editCate(id){
                console.log(id)
                const { data: res } = await this.$http.get('categories/' + id)
                console.log(res)

                if (res.meta.status !== 200) {
                    return this.$message.error('查询分类信息失败!')
                }

                this.addCateForm = res.data
                this.editCateDialogVisible = true

            },

            //关闭修改分类编辑界面
            editCateDialogClosed(){
               this.$refs.addCateForm.resetFields()
            },

            //点击确定按钮，提交编辑分类并关闭对话框
            async editCateInfo(){
                const { data: res } = await this.$http.put('categories/' + this.addCateForm.cat_id, {
                    cat_name: this.addCateForm.cat_name
                })

                console.log(res)

                if (res.meta.status !== 200) {
                    return this.$message.error('更新分类失败!')
                }

                //关闭对话框
                this.editCateDialogVisible = false

                //提示更新信息成功
                this.$message.success('更新分类成功!')

                //更新列表信息
                this.getCateList()
            },

            //根据id删除对应的角色信息
            async removeCateById(id) {
                console.log(id)
                const confirmResult = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).catch(err => {
                    return err
                })

                //如果用户取消删除,则返回结果为字符串 cancel
                //如果用户确认删除,则返回结果为字符串 confirm
                // console.log(confirmResult)
                if (confirmResult !== 'confirm') {
                    return this.$message.info('已取消删除！')
                }

                const { data: res } = await this.$http.delete('categories/' + id)

                if (res.meta.status !== 200) {
                    return this.$message.error('删除分类失败！')
                }

                this.$message.success('删除分类成功！')

                this.getCateList()
            },





        }


    }
</script>

<style Lang="less" scoped>
    .treeTable{
        margin-top: 15px;
    }

    .el-cascader{
        width: 100%;
    }

</style>