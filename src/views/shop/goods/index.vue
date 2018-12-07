<template>
    <div>
        <Card>
            <p slot="title" class="card-title">
                <Icon type="home"></Icon>
                <span>商品管理</span>
            </p>
            <div>
                <template>
                    <Row>
                        <Col span="15">
                            <Button type="info" @click="openAddModal(null)">
                                <Icon type="plus"></Icon>&nbsp;添加商品
                            </Button>
                            <Button :disabled="setting.loading" type="success" @click="getData">
                                <Icon type="refresh"></Icon>&nbsp;刷新数据
                            </Button>
                            <Button type="primary" @click="exportData(1)">
                                <Icon type="ios-download-outline"></Icon>&nbsp;导出表格
                            </Button>
                        </Col>
                        <Col span="9">
                            <Input v-model="search.value" placeholder="请输入您想要搜索的内容..." class="margin-bottom-10">
                            <Button slot="append" icon="ios-search"></Button>
                            </Input>
                        </Col>
                    </Row>
                    <Table ref="table" class="margin-bottom-10"
                           :columns="columns" :loading="setting.loading" :border="setting.showBorder"
                           :data="data.records"></Table>
                    <Page :total="data.total" class="tr" @on-change="pageChange" :current.sync="data.current"
                          :page-size="data.size"
                          @on-page-size-change="pageSizeChange" show-elevator show-sizer></Page>
                </template>
            </div>
        </Card>

        <AddDonation v-if="addModal" :goodsType="goodsType" @cancel="onModalCancel"/>
        <UpdateDonation v-if="updateModal" :goodsType="goodsType" :id="updateId"
                        @cancel="onModalCancel"/>
    </div>
</template>
<script>
    import dayjs from 'dayjs'
    import {post} from '@/libs/axios-cfg'
    import AddDonation from './components/add.vue'
    import UpdateDonation from './components/update.vue'

    export default {
        data() {
            return {
                addModal: false,
                updateModal: false,
                resetPasswordModal: false,
                updateId: null,
                resetPasswordUser: null,
                selections: [],
                setting: {
                    loading: true,
                    showBorder: true
                },
                search: {
                    type: 'name',
                    value: ''
                },
                columns: [
                    {title: 'ID', key: 'id', sortable: true},
                    {title: 'SkuId', key: 'skuNo', sortable: true},
                    {title: '商品名', key: 'skuName', sortable: true},
                    {title: '库存', key: 'stock', sortable: true},
                    {
                        title: '状态',
                        key: 'status',
                        render: (h, params) => {
                            return h('span',
                                {
                                    style: {
                                        color: params.row.status == 1 ? 'green' : 'red'
                                    }
                                }, params.row.status == 1 ? '正常' : '关闭')
                        },
                        sortable: true
                    },
                    {
                        title: '创建日期',
                        key: 'createTime',
                        render: (h, params) => {
                            return h('span', dayjs(params.row.createTime).format('YYYY年MM月DD日 HH:mm:ss'))
                        },
                        sortable: true
                    },
                    {
                        title: '操作',
                        key: 'action',
                        width: 260,
                        align: 'center',
                        render: (h, params) => {
                            return h('div', [
                                h('Button', {
                                    props: {type: params.row.status == 1 ? 'success' : 'warning', size: 'small'},
                                    style: {marginRight: '5px'},
                                    on: {
                                        click: () => {
                                            this.lock(params.row)
                                        }
                                    }
                                }, params.row.status == 1 ? '关闭' : '恢复'),
                                h('Button', {
                                    props: {type: 'primary', size: 'small'},
                                    style: {marginRight: '5px'},
                                    on: {
                                        click: () => {
                                            this.openAddModal(params.row.id)
                                        }
                                    }
                                }, '修改'),
                            ]);
                        }
                    }
                ],
                data: {},
                dataFilter: {
                    page: 1,
                    pageSize: 10
                },
                goodsType: null
            }
        },
        components: {
            AddDonation, UpdateDonation
        },
        created() {
            this.getData();
        },
        methods: {
            /**
             * @description 分页更换事件回调
             */
            pageChange(p) {
                this.dataFilter.page = p;
                this.getData();
            },
            /**
             * @description 分页每页显示数量改变事件回调
             */
            pageSizeChange(p) {
                this.dataFilter.pageSize = p;
                this.getData();
            },
            /**
             * @description 关闭/开启公益项目
             */
            async lock(obj) {
                this.setting.loading = true;
                let status = obj.status;
                let req_url = status == 1 ? 'lock' : 'unlock';
                let req_rep = status == 1 ? 0 : 1;
                let req_msg = status == 1 ? '已锁定' : '已解锁';
                try {
                    let res = await post('/shop/sku/{method}/{id}', null, {
                        id: obj.id,
                        method: req_url
                    })
                    this.$Message.destroy();
                    this.$Message.success(req_msg);
                    obj.status = req_rep;
                } catch (error) {
                    this.$throw(error)
                }
                this.setting.loading = false;
            },
            /**
             * @description 获取用户列表
             */
            async getData() {
                this.setting.loading = true;
                try {
                    let res = await post('/shop/sku/list', {
                        page: this.dataFilter.page,
                        pageSize: this.dataFilter.pageSize
                    });
                    this.data = res.data;
                } catch (error) {
                    this.$throw(error)
                }
                this.setting.loading = false;
            },
            getGoodsType() {
                try {
                    let res = {data: [{code: Number(1000001), desc: '书'}]};
                    this.goodsType = res.data;
                } catch (error) {
                    this.$throw(error)
                }
            },
            /**
             * @description 打开模态窗口
             * @param donationId 公益id
             * @param type 打开类型
             */
            openAddModal(id, type = 'update') {
                if (id == null || type === 'update') {
                    if (this.goodsType == null) {
                        this.getGoodsType();
                    }
                }
                if (id == null) {
                    this.addModal = true;
                } else if (type === 'update') {
                    this.updateId = id;
                    this.updateModal = true;
                }
            },
            /**
             * @description 关闭模态窗口
             * @param type 窗口类型
             * @param reload 是否重新加载数据
             */
            onModalCancel(type, reload = false) {
                switch (type) {
                    case 'add': {
                        this.addModal = false;
                    }
                        break;
                    case 'update': {
                        this.updateModal = false;
                    }
                        break;
                }
                if (reload) {
                    this.getData();
                }
            },
            /**
             * @description 导出表格CSV
             */
            exportData(type) {
                if (type === 1) {
                    this.$refs.table.exportCsv({
                        filename: new Date().getTime(),
                        columns: this.columns.filter((col, index) => index > 1 && index < this.columns.length - 1),
                        data: this.data
                    });
                }
            }
        }
    }
</script>