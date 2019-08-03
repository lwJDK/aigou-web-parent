<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-input v-model="filters.keyword" placeholder="关键字"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getBrands">查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">新增</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="brands" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
            <el-table-column type="selection" width="55">
            </el-table-column>
            <el-table-column type="index" width="60">
            </el-table-column>
            <el-table-column prop="name" label="品牌名称" width="120" sortable>
            </el-table-column>
            <el-table-column prop="englishName" label="英文名称" width="120" sortable>
            </el-table-column>
            <el-table-column prop="logo" label="logo" width="150" sortable>
                <template scope="scope">
                    <img :src="'http://192.168.43.184'+scope.row.logo" height="50"/>
                </template>
            </el-table-column>
            <el-table-column prop="productType.name" label="类型" width="150" sortable>
            </el-table-column>
            <el-table-column prop="description" label="描述" min-width="180" sortable>
            </el-table-column>
            <el-table-column label="操作" width="150">
                <template scope="scope">
                    <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>

        <!--工具条-->
        <el-col :span="24" class="toolbar">
            <el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
            <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="pageSize" :total="total" style="float:right;">
            </el-pagination>
        </el-col>

        <!--编辑界面-->
        <el-dialog title="编辑" v-model="editFormVisible" :close-on-click-modal="false">
            <el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
                <el-form-item label="品牌名称" prop="name">
                    <el-input v-model="editForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="英文名称" prop="englishName">
                    <el-input v-model="editForm.englishName" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="logo" prop="logo">
                    <el-input v-model="editForm.logo" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="类型" prop="productTypeId">
                    <el-input v-model="editForm.productTypeId" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="描述">
                    <el-input type="textarea" v-model="editForm.description"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
            </div>
        </el-dialog>

        <!--新增界面-->
        <el-dialog title="新增" v-model="addFormVisible" :close-on-click-modal="false">
            <el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm" size="tiny">
                <el-form-item label="品牌名称" prop="name">
                    <el-input v-model="addForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="英文名称" prop="englishName">
                    <el-input v-model="addForm.englishName" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="类型"  prop="productTypeId">
                    <el-cascader
                            :options="productTypes"
                            :props="props"
                            v-model="addForm.productTypeId">
                    </el-cascader>
                </el-form-item>
                <el-form-item label="logo">
                    <!--
                        action  必选参数，上传的地址
                    -->
                    <el-upload
                            class="upload-demo"
                            action="http://localhost:4000/service/common/fastdfs"
                            :on-remove="handleRemove"
                            :on-success="handleSucces"
                            :file-list="logoList"
                            :before-upload="handleBeforeUpload"
                            list-type="picture">
                        <el-button size="small" type="primary">点击上传</el-button>
                        <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                    </el-upload>
                </el-form-item>
                <el-form-item label="描述">
                    <el-input type="textarea" v-model="addForm.description"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="addFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>
    import util from '../../common/js/util'
    //import NProgress from 'nprogress'
    //import { getUserListPage, removeUser, batchRemoveUser, editUser, addUser } from '../../api/api';

    export default {
        data() {
            return {
                logoList:[{
                   name:'',
                   url:'http://192.168.43.184/group1/M00/00/01/wKgruF1Eq_uAYJr7AAExwhewTXc293.jpg',
                 }],
                productTypes:[],
                props:{
                    label:"name",
                    value:"id"
                },
                filters: {
                    keyword: ''
                },
                pageSize:10,
                brands: [],
                total: 0,
                page: 1,
                listLoading: false,
                sels: [],//列表选中列

                editFormVisible: false,//编辑界面是否显示
                editLoading: false,
                editFormRules: {
                    name: [
                        { required: true, message: '请输入品牌名称', trigger: 'blur' }
                    ]
                },
                //编辑界面数据
                editForm: {
                    id: 0,
                    name: '',
                    englishName: '',
                    logo: '',
                    productTypeId: [],
                    description: ''
                },

                addFormVisible: false,//新增界面是否显示
                addLoading: false,
                addFormRules: {
                    name: [
                        { required: true, message: '请输入品牌名称', trigger: 'blur' }
                    ]
                },
                //新增界面数据
                addForm: {
                    name: '',
                    englishName: '',
                    productTypeId: null,
                    description: '',
                    options:[],
                    logo:''
                }

            }
        },
        methods: {
            //上传之前的回调
            handleBeforeUpload(){
                if(this.logoList.length>0){
                    this.$message({
                        message:"只能上传一个文件",
                        success:"error"
                    })
                    return false;
                }
            },

            //上传文件成功的回调
            handleSucces(response, file, fileList){
                this.addForm.logo = response.obj;
                console.debug("fileList",fileList);
                this.logoList = fileList;
            },

            //移除图片的回调
            handleRemove(file, fileList){
                console.debug("fileList",fileList);
                this.logoList = fileList;
            },

            //性别显示转换
            formatSex: function (row, column) {
                return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
            },
            handleCurrentChange(val) {
                this.page = val;
                this.getBrands();
            },
            //获取品牌列表
            getBrands() {
                this.listLoading = true;
                //NProgress.start();
                this.$http.post("/product/brand/json",{
                    pageNum: this.page,
                    pageSize: this.pageSize,
                    keyword: this.filters.keyword
                }).then(res=>{
                    this.listLoading = false;
                    let pageList = res.data;
                    this.brands = pageList.rows;
                    this.total = pageList.total;
                })
            },
            //获取类型数据
            loadProductType(){
                this.$http.get("/product/productType/list")
                    .then(res=>{
                        this.listLoading = false;
                        this.productTypes = res.data;
                    })
            },
            //删除
            handleDel: function (index, row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    this.$http.delete("/product/brand/delete/"+row.id)
                        .then(res=>{
                            this.listLoading = false;
                            //NProgress.done();
                            //成功的提示
                            if(res.data.success){
                                this.$message({
                                    message: '删除成功',
                                    type: 'success'
                                });
                            }else{
                                //失败的提示
                                this.$message({
                                    message: res.data.message,
                                    type: 'error'
                                });
                            }
                            this.getBrands();
                        })
                    /*let para = { id: row.id };
                    removeUser(para).then((res) => {
                        this.listLoading = false;
                        //NProgress.done();
                        this.$message({
                            message: '删除成功',
                            type: 'success'
                        });
                        this.getBrands();
                    });*/
                });
            },
            //显示编辑界面
            handleEdit: function (index, row) {
                this.editFormVisible = true;
                this.editForm = Object.assign({}, row);
            },
            //显示新增界面
            handleAdd: function () {
                this.addFormVisible = true;
                this.logoList = [],
                this.addForm = {
                    name: '',
                    englishName: '',
                    productTypeId: '',
                    description: '',
                    logo:''
                };
            },
            //编辑
            editSubmit: function () {
                this.$refs.editForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            //NProgress.start();
                            let para = Object.assign({}, this.editForm);
                            this.$http.post("/product/brand/add",para)
                                .then((res)=>{
                                    //成功之后的操作
                                    if(res.data.success){
                                        this.$message({
                                            message: "修改成功",
                                            type: "success"
                                        });
                                        //关闭模态框表单
                                        this.editFormVisible = false;
                                        //重新加载数据
                                        this.getBrands();
                                    }
                                })
                            /*para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
                            editUser(para).then((res) => {
                                this.editLoading = false;
                                //NProgress.done();
                                this.$message({
                                    message: '提交成功',
                                    type: 'success'
                                });
                                this.$refs['editForm'].resetFields();
                                this.editFormVisible = false;
                                this.getBrands();
                            });*/
                        });
                    }
                });
            },
            //新增
            addSubmit: function () {
                this.$refs.addForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            //NProgress.start();
                            //对象的复制
                            let para = Object.assign({}, this.addForm);
                            //级联选择器的结果是一个数组
                            para.productTypeId = para.productTypeId[para.productTypeId.length-1];
                            this.$http.post("/product/brand/add",para)
                                .then((res)=>{
                                    //成功之后的操作
                                    if(res.data.success){
                                        this.addLoading = false;
                                        this.$message({
                                            message: '提交成功',
                                            type: 'success'
                                        });
                                        //清空表单
                                        this.$refs['addForm'].resetFields();
                                        //关闭模态框
                                        this.addFormVisible = false;
                                        //重新加载数据列表
                                        this.getBrands();
                                    }else{
                                        //失败之后的操作
                                        this.$message({
                                            message: res.data.message,
                                            type: 'error'
                                        });
                                    }
                                })
                            /*addUser(para).then((res) => {
                                this.addLoading = false;
                                //NProgress.done();
                                this.$message({
                                    message: '提交成功',
                                    type: 'success'
                                });
                                this.$refs['addForm'].resetFields();
                                this.addFormVisible = false;
                                this.getBrands();
                            });*/
                        });
                    }
                });
            },
            selsChange: function (sels) {
                this.sels = sels;
            },
            //批量删除
            batchRemove: function () {
                var ids = this.sels.map(item => item.id).toString();
                this.$confirm('确认删除选中记录吗？', '提示', {
                    type: 'warning'
                }).then(() => {
                    //NProgress.start();
                    let para = { ids: ids };
                    batchRemoveUser(para).then((res) => {
                        //NProgress.done();
                        this.$message({
                            message: '删除成功',
                            type: 'success'
                        });
                        this.getBrands();
                    });
                }).catch(() => {

                });
            }
        },
        mounted() {
            this.getBrands();
            this.loadProductType();
        }
    }

</script>

<style scoped>

</style>