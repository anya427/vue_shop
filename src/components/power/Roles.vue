<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>权限列表</el-breadcrumb-item>
        </el-breadcrumb>

        <!-- 卡片视图 -->
        <el-card>
            <!-- 添加角色按钮区域 -->
            <el-row>
                <el-col>
                    <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
                </el-col>
            </el-row>

            <!-- 角色列表区域 -->
            <el-table :data="rolelist" border stripe>
                <!-- 展开列 -->
                <el-table-column type="expand">
                    <template v-slot="scope">
                        <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                            <!-- 渲染一级权限 -->
                            <el-col :span="5">
                                <el-tag type="" closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>                             
                            </el-col>
                            <!-- 渲染二级和三级权限 -->
                            <el-col :span="19">
                                <!-- 通过for循环嵌套渲染二级权限 -->
                                <el-row :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                                  <el-col :span="6">
                                      <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                                      <i class="el-icon-caret-right"></i>
                                  </el-col>
                                  <el-col :span="18">
                                      <el-tag type="warning" v-for="(item3, i3) in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                                  </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                        <!-- <pre>
                            {{scope.row}}
                        </pre> -->
                        

                    </template>
                </el-table-column>
                <!-- 索引列 -->
                <el-table-column label="#" type="index"> </el-table-column>
                <el-table-column label="角色名称" type="roleName" prop="roleName"> </el-table-column>
                <el-table-column label="角色描述" type="roleDesc" prop="roleDesc"> </el-table-column>
                <el-table-column label="操作" width="300px">
                    <template v-slot='scope'>
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="editRoles(scope.row.id)">编辑
                        </el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini"
                            @click="removeRolesById(scope.row.id)">删除</el-button>
                        <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>


        <!-- 添加角色的对话框 -->
        <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
            <!-- 内容主体区域 -->
            <el-form :model="addForm" ref="addFormRef" label-width="70px">
                <el-form-item label="角色名称">
                    <el-input v-model="addForm.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述">
                    <el-input v-model="addForm.roleDesc"></el-input>
                </el-form-item>
            </el-form>
            <!-- 底部区域 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addRoles">确 定</el-button>
            </span>
        </el-dialog>


        <!-- 修改角色的对话框 -->
        <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
            <el-form :model="editForm" ref="editFormRef" label-width="70px">
                <el-form-item label="角色名称">
                    <el-input v-model="editForm.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述">
                    <el-input v-model="editForm.roleDesc"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editRolesInfo">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 分配权限的对话框   -->
        <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
            <!-- 树形控件-->
            <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
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
                //所有角色列表数据
                rolelist: [],

                //所有权限的数据
                rightslist:[],

                //查询到的角色信息
                editForm: {},

                //控制添加用户对话框的显示与隐藏
                addDialogVisible: false,

                //控制修改角色对话框的显示与隐藏
                editDialogVisible: false,

                //控制分配权限对话框的显示与隐藏
                setRightDialogVisible: false,

                //表单的验证
                editFormRef: '',
                addFormRef: '',

                //添加角色的表单数据
                addForm: {
                    roleName: '',
                    roleDesc: '',
                },

                //树形控件的属性绑定对象
                treeProps:{
                    label:'authName',
                    children:'children'
                },

                //默认选中的节点id值数组
                defKeys:[],

                //角色本就有的权限id, 当前即将分配权限的角色id
                roleId:'',
            }
        },
        created() {
            this.getRoleList()
        },

        methods: {
            //获取所有角色列表
            async getRoleList() {
                const { data: res } = await this.$http.get('roles')

                if (res.meta.status !== 200) {
                    return this.$message.error('获取角色列表失败！')
                }

                this.rolelist = res.data
                console.log(this.rolelist)
            },
            //添加角色按钮
            addRoles() {
                this.$refs.addFormRef.validate(async valid => {
                    // console.log(valid)
                    if (!valid) return
                    //可以发起添加角色的网络请求
                    const { data: res } = await this.$http.post('roles', this.addForm)

                    if (res.meta.status !== 201)
                        return this.$message.error('添加角色失败!')

                    this.$message.success('添加角色成功!')
                    //隐藏添加用户的对话框
                    this.addDialogVisible = false

                    //重新获取用户的列表数据
                    this.getRoleList()
                })

            },

            //编辑角色列表按钮
            async editRoles(id) {
                console.log(id)
                const { data: res } = await this.$http.get('roles/' + id)
                console.log(res)

                if (res.meta.status !== 200) {
                    return this.$message.error('查询角色信息失败!')
                }

                this.editForm = res.data
                this.editDialogVisible = true
            },

            //监听添加角色对话框的关闭事件
            addDialogClosed() {
                this.$refs.addFormRef.resetFields()
            },

            //监听修改角色对话框的关闭事件
            editDialogClosed() {
                this.$refs.editFormRef.resetFields()
            },

            //修改用户角色信息并提交
            async editRolesInfo() {
                const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, {
                    roleName: this.editForm.roleName,
                    roleDesc: this.editForm.roleDesc
                })

                console.log(res)

                if (res.meta.status !== 200) {
                    return this.$message.error('更新用户信息失败!')
                }

                //关闭对话框
                this.editDialogVisible = false
                
                //提示更新信息成功
                this.$message.success('更新用户信息成功!')

                //更新列表信息
                this.getRoleList()
            },

            //根据id删除对应的角色信息
            async removeRolesById(id) {
                console.log(id)
                const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
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

                const { data: res } = await this.$http.delete('roles/' + id)

                if (res.meta.status !== 200) {
                    return this.$message.error('删除用户失败！')
                }

                this.$message.success('删除用户成功！')

                this.getRoleList()
            },


            //根据id删除对应的权限
            async removeRightById(role, rightId){
                //弹框提示用户是否要删除
                const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).catch(err => {
                    return err
                })

                if (confirmResult !== 'confirm') {
                    return this.$message.info('已取消删除！')
                }
               const {data : res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)

               if(res.meta.status !== 200){
                   return this.$message.error('删除权限失败！')
               }

               // this.getRoleList()
                role.children = res.data
            },

            //展示分配权限的对话框
           async showSetRightDialog(role){

                //获取到的角色本来就有的权限id赋值到roleId中
                this.roleId = role.id
                //获取所有权限的数据
                const {data : res} = await this.$http.get('rights/tree')

               if(res.meta.status !== 200){
                   return this.$message.error('获取权限数据失败！')
               }

               //获取到的权限数据保存到rightslist中
               this.rightslist = res.data
               console.log(this.rightslist)

               //递归获取三级节点的id
               this.getLeafKeys(role, this.defKeys)

               this.setRightDialogVisible = true
            },

            //通过递归的形式，获取角色下所有三级权限的id，并保存到 defKeys 数组中
            getLeafKeys(node ,arr){

                //如果当前node节点不包含 children 属性，则是三级节点
                if(!node.children) {
                    return arr.push(node.id)
                }
                //递归循环找到三级数组
                node.children.forEach(item => this.getLeafKeys(item,arr))
            },

            //监听分配权限对话框的关闭事件
            setRightDialogClosed(){
                this.defKeys = []
            },

            //点击为角色分配权限
            async allotRights(){
                const keys = [
                    ...this.$refs.treeRef.getCheckedKeys(),
                    ...this.$refs.treeRef.getHalfCheckedKeys()
                ]

                // console.log(keys)
                //以英文的逗号进行拼接数组
                const idStr = keys.join(',')

                const {data : res} = await this.$http.post(`roles/${this.roleId}/rights`, {rids: idStr})

                if(res.meta.status !== 200){
                    return this.$message.error('分配权限失败！')
                }

                this.$message.success('分配权限成功！')

                this.getRoleList()

                this.setRightDialogVisible = false
            },

        }

    }


</script>
<style Lang="less" scoped>

    .el-tag{
        margin: 7px;
    }

    .bdtop{
        border-top: 1px solid #eeeeee;
    }

    .bdbottom{
        border-bottom: 1px solid #eeeeee;
    }

    .vcenter {
        display: flex;
        align-items: center;
    }
</style>