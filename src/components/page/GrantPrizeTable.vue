<template>
    <div class="table">
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-lx-cascades"></i> 发放礼品管理</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="handle-box">
                <!--<el-button type="danger" icon="delete" class="handle-del mr10" @click="delAll">批量删除</el-button>-->
                <!--<el-select v-model="select_cate" placeholder="筛选省份" class="handle-select mr10">-->
                    <!--<el-option key="1" label="广东省" value="广东省"></el-option>-->
                    <!--<el-option key="2" label="湖南省" value="湖南省"></el-option>-->
                <!--</el-select>-->
                <el-input v-model="select_word" placeholder="筛选关键词" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="search" @click="search">搜索</el-button>
                <el-button type="primary" style="float: right" @click="handleEdit(undefined, undefined, 1)">发放奖品</el-button>
            </div>
            <el-table :data="data" border class="table" ref="multipleTable" @selection-change="handleSelectionChange">
                <el-table-column type="selection" width="55" align="center"></el-table-column>
                <el-table-column prop="id" label="ID" width="70" align="center"></el-table-column>
                <el-table-column prop="username" align="center" label="发奖用户名" :formatter="formatter">
                </el-table-column>
                <el-table-column prop="nickname" align="center" label="收奖用户昵称">
                </el-table-column>
                <el-table-column prop="tel" align="center" label="收奖用户手机号">
                </el-table-column>
                <el-table-column prop="prize" align="center" label="奖品名称">
                </el-table-column>
                <el-table-column prop="datetime"  align="center" label="发奖时间" sortable width="155">
                </el-table-column>
                <el-table-column label="操作" width="150" align="center">
                    <template slot-scope="scope">
                        <!--<el-button type="text" icon="el-icon-edit" @click="handleEdit(scope.$index, scope.row, 2)">编辑</el-button>-->
                        <!--<el-button type="text" icon="el-icon-delete" class="red" @click="handleDelete(scope.$index, scope.row)">删除</el-button>-->
                    </template>
                </el-table-column>
            </el-table>
            <div class="pagination">
                <el-pagination background @current-change="handleCurrentChange" layout="prev, pager, next" :total="sumPage">
                </el-pagination>
            </div>
        </div>

        <!-- 编辑弹出框 -->
        <el-dialog title="编辑" v-loading="loading" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="form" label-width="100px">
                <el-form-item label="收奖用户">
                    <el-select v-model="selectedUser" placeholder="请选择收奖用户">
                        <el-option
                                v-for="item in userList"
                                :key="item.id"
                                :label="item.nickname"
                                :value="item.id">{{item.nickname}}
                        </el-option>
                        <!--<el-option key="bbk" label="步步高" value="bbk"></el-option>-->
                        <!--<el-option key="xtc" label="小天才" value="xtc"></el-option>-->
                        <!--<el-option key="imoo" label="imoo" value="imoo"></el-option>-->
                    </el-select>
                </el-form-item>
                <el-form-item label="奖品">
                    <el-select v-model="selectedPrize" placeholder="请选择奖品">
                        <el-option
                                v-for="item in prizeList"
                                :key="item.id"
                                :label="item.prize"
                                :value="item.id">{{item.prize}}
                        </el-option>
                        <!--<el-option key="bbk" label="步步高" value="bbk"></el-option>-->
                        <!--<el-option key="xtc" label="小天才" value="xtc"></el-option>-->
                        <!--<el-option key="imoo" label="imoo" value="imoo"></el-option>-->
                    </el-select>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="hideEditVisible">取 消</el-button>
                <el-button type="primary" @click="saveEdit('form')">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 删除提示框 -->
        <el-dialog title="提示" :visible.sync="delVisible" width="300px" center>
            <div class="del-dialog-cnt">删除不可恢复，是否确定删除？</div>
            <span slot="footer" class="dialog-footer">
                <el-button @click="delVisible = false">取 消</el-button>
                <el-button type="primary" @click="deleteRow">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: 'basetable',
        data() {
            return {
                url: './vuetable.json',
                tableData: [],
                cur_page: 1,  //当前页
                number: 10,  //每页显示条数
                sumPage:10,  //总页数
                multipleSelection: [],
                select_cate: '',
                select_word: '',
                del_list: [],
                is_search: false,
                editVisible: false,
                delVisible: false,
                form: {
                    id: "",
                    wechatuserid: "",
                    prizeid: "",
                    datetime: "",
                    username: "",
                    nickname: "",
                    tel: "",
                    prize: ""
                },
                idx: -1,
                dialogVisible: false,
                AddOrSave:'',  //1表示添加，2表示更新
                loading:false,
                userList:[],  //微信用户列表
                prizeList:[],  //奖品列表
                selectedUser:'',  //选中用户
                selectedPrize:'',  //选中奖品
            }
        },
        created() {
            //获取初始数据
            this.getData();
        },
        computed: {
            data() {
                return this.tableData.filter((d) => {
                    let is_del = false;
                    for (let i = 0; i < this.del_list.length; i++) {
                        if (d.username === this.del_list[i].username) {
                            is_del = true;
                            break;
                        }
                    }
                    return d;
                    // if (!is_del) {
                    //     if (d.b_datetime.indexOf(this.select_cate) > -1 &&
                    //         (d.b_titile.indexOf(this.select_word) > -1 ||
                    //             d.b_datetime.indexOf(this.select_word) > -1)
                    //     ) {
                    //         return d;
                    //     }
                    // }
                })
            }
        },
        methods: {
            // 分页导航
            handleCurrentChange(val) {
                this.cur_page = val;
                this.getData();
            },
            // 获取 easy-mock 的模拟数据
            getData() {
                // 开发环境使用 easy-mock 数据，正式环境使用 json 文件
                // if (process.env.NODE_ENV === 'development') {
                //     this.url = '/ms/table/list';
                // };
                //不传参数要写null
                var params=this.$qs.stringify({
                    select_word: this.select_word,
                    pageIndex: this.cur_page,
                    number: this.number
                });
                // console.log(params);
                this.$api.post('GrantPrize/getGrantPrizeList', params, res => {
                    this.tableData = res.data.list;
                    this.sumPage = res.data.sumPage*10;
                    this.cur_page = res.data.currentPage;
                    console.log(this.tableData);
                }, err => {
                    this.tableData = [];
                    this.$message.error(err.msg);
                });
                // this.$axios.post(this.url, {
                //     page: this.cur_page
                // }).then((res) => {
                //     this.tableData = res.data.list;
                //     console.log(this.tableData);
                // })
            },
            search() {
                this.is_search = true;
                this.getData();
            },
            formatter(row, column) {
                return row.username;
            },
            filterTag(value, row) {
                return row.tag === value;
            },
            handleEdit(index, row, status) {
                this.getPrizeList();  //获取奖品列表
                this.getwechatUserList();  //获取微信用户列表
                console.log(row);
                this.AddOrSave=status;
                //如果是添加则把form清空
                if(status==1){
                    this.form = {
                        id: "",
                        wechatuserid: "",
                        prizeid: "",
                        datetime: "",
                        username: "",
                        nickname: "",
                        tel: "",
                        prize: ""
                    };
                    this.selectedUser='';
                    this.selectedPrize='';
                }
                if(index!=undefined && row!=undefined){
                    this.idx = index;
                    const item = this.tableData[index];
                    this.form = {
                        id: item.id,
                        wechatuserid: item.wechatuserid,
                        prizeid: item.prizeid,
                        datetime: item.datetime,
                        username: item.username,
                        nickname: item.nickname,
                        tel: item.tel,
                        prize: item.prize
                    };
                }
                this.editVisible = true;
            },
            handleDelete(index, row) {
                this.idx = index;
                this.form = row;
                this.delVisible = true;
            },
            delAll() {
                const length = this.multipleSelection.length;
                let str = '';
                this.del_list = this.del_list.concat(this.multipleSelection);
                for (let i = 0; i < length; i++) {
                    str += this.multipleSelection[i].b_title + ' ';
                }
                console.log(this.del_list);
                // this.$message.error('删除了' + str);
                // this.multipleSelection = [];
            },
            handleSelectionChange(val) {
                this.multipleSelection = val;
            },
            // 保存编辑
            saveEdit(formName) {
                this.editVisible = false;
                var adminuserid=JSON.parse(localStorage.getItem('userInfo')).id;
                var params=null;
                //1表示添加，2表示更新
                if(this.AddOrSave==1){
                    params=this.$qs.stringify({
                        adminuserid: adminuserid,
                        wechatuserid: this.selectedUser,
                        prizeid: this.selectedPrize
                    });
                }else{
                    params=this.$qs.stringify({
                        id: this.form.id,
                        status: this.form.statusMsg,
                    });
                }
                // console.log(params);
                // return;
                this.$api.post('GrantPrize/saveGrantPrize', params, res => {
                    this.getData();
                    this.$message.success(res.msg);
                }, err => {
                    this.$message.error(err.msg);
                });




                // this.$message.success(`修改第 ${this.idx+1} 行成功`);
            },
            // 确定删除
            deleteRow(){
                // console.log(this.form);
                // return;
                var params=this.$qs.stringify({
                    id: this.form.id
                });
                this.$api.post('InCode/deleteInCode', params, res => {
                    this.getData();
                    this.$message.success(res.msg+res.data+"条数据");
                }, err => {
                    this.$message.error(err.msg);
                });
                this.delVisible = false;
            },
            //获取微信用户列表
            getwechatUserList(){
                this.$api.post('GrantPrize/wechatUserList', null, res => {
                    console.log(res);
                    this.userList=res.data;
                }, err => {
                    this.$message.error(err.msg);
                });
            },
            //获取奖品列表
            getPrizeList(){
                this.$api.post('GrantPrize/getPrizeList', null, res => {
                    console.log(res);
                    this.prizeList=res.data;
                }, err => {
                    this.$message.error(err.msg);
                });
            },
            //隐藏增加弹框
            hideEditVisible(){
                this.editVisible=false;
            }
        }
    }

</script>

<style scoped>
    .handle-box {
        margin-bottom: 20px;
    }

    .handle-select {
        width: 120px;
    }

    .handle-input {
        width: 300px;
        display: inline-block;
    }
    .del-dialog-cnt{
        font-size: 16px;
        text-align: center
    }
    .table{
        width: 100%;
        font-size: 14px;
    }
    .red{
        color: #ff0000;
    }
    .mr10{
        margin-right: 10px;
    }
    .avatar-uploader .el-upload {
        border: 1px dashed #d9d9d9;
        border-radius: 6px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
    }
    .avatar-uploader .el-upload:hover {
        border-color: #409EFF;
    }
    .avatar-uploader-icon {
        font-size: 28px;
        color: #8c939d;
        width: 178px;
        height: 178px;
        line-height: 178px;
        text-align: center;
    }
    .avatar {
        width: 100%;
        /*height: 100%;*/
        display: block;
    }
</style>
