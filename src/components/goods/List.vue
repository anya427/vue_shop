<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品列表</el-breadcrumb-item>
        </el-breadcrumb>

<!--        卡片视图区域-->
        <el-card>

            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">

                        <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="goAddpage">
                        添加商品
                    </el-button>
                </el-col>
            </el-row>

<!--            表格区域-->
            <el-table :data="goodsList" border stripe>
                <el-table-column label="#" type="index" ></el-table-column>
                <el-table-column label="商品名称" prop="goods_name" width="500px"></el-table-column>
                <el-table-column label="商品价格(元)" prop="goods_price" width="100px"></el-table-column>
                <el-table-column label="商品重量" prop="goods_weight" width="70px"></el-table-column>
                <el-table-column label="创建时间" prop="add_time" width="140px">
                    <template v-slot="scope">
                        {{scope.row.add_time | dateFormat}}
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="130px">
                    <template v-slot="scope">
                        <el-button type="primary" size="mini" icon="el-icon-edit" @click="editById(scope.row.goods_id)"></el-button>
                        <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeById(scope.row.goods_id)"></el-button>
                    </template>
                </el-table-column>

            </el-table>

            <!--        修改参数对话框-->
            <el-dialog
                    title="修改商品信息"
                    :visible.sync="editDialogVisible"
                    width="50%"
                    @close="editDialogClose">
                <el-form :model="editForm"  ref="editFormRef" label-width="100px">
                    <el-form-item label="商品名称">
                        <el-input v-model="editForm.goods_name"></el-input>
                    </el-form-item>
                    <el-form-item label="商品价格">
                        <el-input v-model="editForm.goods_price"></el-input>
                    </el-form-item>
                    <el-form-item label="商品重量">
                        <el-input v-model="editForm.goods_weight"></el-input>
                    </el-form-item>
                    <el-form-item label="商品数量">
                        <el-input v-model="editForm.goods_number"></el-input>
                    </el-form-item>
                </el-form>
                <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editParams">确 定</el-button>
  </span>
            </el-dialog>

            <!-- 分页 -->
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
                           :current-page="queryInfo.pagenum" :page-sizes="[5, 10, 15, 20]" :page-size="queryInfo.pagesize"
                           layout="total, sizes, prev, pager, next, jumper" :total="total">
            </el-pagination>
        </el-card>
    </div>



</template>

<script>
    export default {
        name: "List",
        data(){
            return {
                //查询参数对象
                queryInfo:{
                    query: '',
                    pagenum: 1,
                    pagesize: 10
                },
                //商品列表
                goodsList:[],
                //总数据条数
                total:0,
                //控制修改对话框的显示与隐藏
                editDialogVisible: false,
                //修改参数的表单验证对象
                editForm: {
                    goods_id:'',
                    goods_name:'',
                    goods_price:'',
                    goods_number:'',
                    goods_weight:'',
                    goods_introduce:'',
                    pics:{},
                    attrs:[],
                },
                editFormRef:'',

            }
        },

        created() {
            this.getGoodsList()
        },

        methods:{
            //获取商品列表参数
            async getGoodsList(){
                const {data : res} = await this.$http.get('goods',{params:this.queryInfo })

                if (res.meta.status !== 200){
                    return this.$message.error('获取商品列表失败！')
                }

                // this.$message.success('获取商品列表成功！')
                console.log(res.data)
                this.goodsList = res.data.goods
                this.total = res.data.total
            },

            handleSizeChange(newSize){
                this.queryInfo.pagesize = newSize
                this.getGoodsList()
            },

            handleCurrentChange(newPage){
                this.queryInfo.pagenum = newPage
                this.getGoodsList()
            },

            //删除商品信息
           async removeById(goods_id){
                console.log(goods_id)
                const confirmResult = await this.$confirm('此操作将永久删除该的商品, 是否继续?', '提示', {
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

                const { data: res } = await this.$http.delete(  `goods/${goods_id}`)

                if (res.meta.status !== 200) {
                    return this.$message.error('删除参数失败！')
                }

                this.$message.success('删除参数成功！')

                this.getGoodsList()
            },

            editDialogClose(){
                this.$refs.editFormRef.resetFields()
            },

            //修改商品
           async editById(goods_id){
               const {data :res} = await this.$http.get('goods/' + goods_id)

               if (res.meta.status !== 200){
                   return this.$message.error('获取商品失败!')
               }
               this.editForm = res.data
               this.editDialogVisible = true

               console.log(res.data)

            },

            //提交修改商品参数
           async editParams(){
                const {data :res} = await this.$http.put('goods/' + this.editForm.goods_id, {
                    goods_name: this.editForm.goods_name,
                    goods_price: this.editForm.goods_price,
                    goods_number: this.editForm.goods_number,
                    goods_weight: this.editForm.goods_weight,
                    goods_introduce: this.editForm.goods_introduce,
                    pics: this.editForm.pics,
                    attrs: this.editForm.attrs
                    })

               console.log(res)
            },




            //添加商品
            goAddpage(){
                this.$router.push('/goods/add')
            }




        },
    }
</script>

<style Lang="less" scoped>

</style>