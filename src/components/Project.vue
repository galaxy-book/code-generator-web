<template>
    <div style="width:1100px;padding:10px">
        <div style="">
            <Button type="info" style="float:right" @click="jumpToCreateProject()">创建项目</Button>
        </div>
        <Table border :columns="columns" :data="data" style="float:right;margin-top:10px"></Table>
        <Page :total="total" :page-size="size" @on-page-size-change="pageSizeChange" show-sizer class="page" @on-change="pageChange" :current="1"/>
    </div>
    
</template>
<script>
    import axios from 'axios'

    var app = null;
    export default {
        data () {
            return {
                columns: [
                    {
                        title: 'ID',
                        key: 'id',
                        render: (h, params) => {
                            return h('div', [
                                h('Icon', {
                                    props: {
                                        type: 'person'
                                    }
                                }),
                                h('strong', params.row.id)
                            ]);
                        }
                    },
                    {
                        title: '名称',
                        key: 'name'
                    },
                    {
                        title: '描述',
                        key: 'desc'
                    }
                    ,
                    {
                        title: '创建时间',
                        key: 'createTime'
                    },
                    {
                        title: 'Action',
                        key: 'action',
                        width: 150,
                        align: 'center',
                        render: (h, params) => {
                            return h('div', [
                                h('Button', {
                                    props: {
                                        type: 'primary',
                                        size: 'small'
                                    },
                                    style: {
                                        marginRight: '5px'
                                    },
                                    on: {
                                        click: () => {
                                            app.$router.push({name:'ProjectUpdate',query: { id: params.row.id }})//
                                        }
                                    }
                                }, '编辑'),
                                h('Button', {
                                    props: {
                                        type: 'error',
                                        size: 'small'
                                    },
                                    on: {
                                        click: () => {
                                            app.delete(params.row)
                                            this.remove(params.index)
                                        }
                                    }
                                }, '删除')
                            ]);
                        }
                    }
                ],
                data: [],
                userId:0,
                total: 0,
                size:10,
            }
        },
        methods: {
            show (index) {
                this.$Modal.info({
                    title: 'User Info',
                    content: `Name：${this.data[index].name}<br>Age：${this.data[index].age}<br>Address：${this.data[index].address}`
                })
            },
            remove (index) {
                this.data.splice(index, 1);
            },
            getProjectList(page,size){
                axios.get(app.HOST + "/projects?page=" + page + "&size=" + size,{
                    headers:{token:localStorage.getItem("token")}
                })
                .then(function (response) {
                    var resp = response.data
                    if(resp.code == 200){
                        if(resp.data.list){
                            app.data = resp.data.list
                            app.total = resp.data.total
                        }else{
                            app.data = []
                        }
                    }else{
                        app.$Message.error(resp.msg);
                    }
                })
                .catch(function (error) {
                    app.$Message.error('err: ' + error);
                });
            },
            jumpToCreateProject(){
                app.$router.push({name:'ProjectCreate'})
            },
            pageChange(e){
                app.getProjectList(e, app.size)
            },
            pageSizeChange(e){
                app.size = e
                app.getProjectList(1, app.size)
            },
            delete(data){
                axios.delete(app.HOST + "/projects/" + data.id,{
                    headers:{token:localStorage.getItem("token")}
                })
                .then(function (response) {
                    var resp = response.data
                    if(resp.code == 200){
                        app.$Message.success('删除项目' + data.name + '成功');
                    }else{
                         app.$Message.error(resp.msg);
                    }
                    
                })
                .catch(function (error) {
                    app.$Message.error('err: ' + error);
                });
            }
        },
        created(){
            app = this
            app.getProjectList(1, app.size)
        }
    }
</script>
<style scoped>
.page{float:right;margin-top:20px}
</style>