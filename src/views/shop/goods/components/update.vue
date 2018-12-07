<template>
    <div>
        <Modal v-model="show" title="更新用户"
               :mask-closable="false" :closable="false">
            <Form ref="modalForm" :model="data" :rules="ruls"
                  :label-width="80">
                <FormItem label="商品名称" prop="sku_name">
                    <Input v-model.trim="data.skuName"></Input>
                </FormItem>
                <FormItem label="商品编号" prop="sku_no">
                    <Input v-model.trim="data.skuNo"></Input>
                </FormItem>
                <FormItem label="价格" prop="price">
                    <InputNumber :min="0" :step="1" v-model.trim="data.price" style="width:100%"/>
                </FormItem>
                <FormItem label="库存" prop="price">
                    <InputNumber :min="0" :step="1" v-model.trim="data.stock" style="width:100%"/>
                </FormItem>
                <FormItem label="单位" prop="units">
                    <Input v-model.trim="data.units"/>
                </FormItem>
                <FormItem label="状态" prop="status">
                    <Select v-model.trim="data.status" style="width:100%">
                        <Option v-for="item in [{label:'正常',value:1},{label:'关闭',value:0}]"
                                :value="item.value" :key="item.value">{{ item.label }}
                        </Option>
                    </Select>
                </FormItem>
                <FormItem label="商品类型" prop="goodsType">
                    <RadioGroup v-model="data.goodsType">
                        <Radio v-for="(item,index) in goodsType" :label="item.code" :key="index">
                            {{item.desc}}
                        </Radio>
                    </RadioGroup>
                </FormItem>
                <FormItem label="图片" prop="img">
                    <Upload v-model="data.img" action="//localhost:9000/FileUpload/FileUp"
                            :format="['jpg','jpeg','png']"
                            :max-size="2048"
                            :on-success="handleSuccess"
                            :on-error="handleError"
                            :on-format-error="handleFormatError"
                            :on-exceeded-size="handleMaxSize"
                    >
                        <Button icon="ios-cloud-upload-outline">上传</Button>
                    </Upload>
                </FormItem>
            </Form>
            <div slot="footer">
                <Button type="default" :disabled="loading" @click="cancel(false)">取消</Button>
                <Button type="primary" :loading="loading" @click="ok">确定</Button>
            </div>
        </Modal>
    </div>
</template>
<script>
    import {post} from '@/libs/axios-cfg'

    export default {
        data() {
            return {
                show: true,
                loading: false,
                data: {
                    skuName: "",
                    skuNo: "",
                    price: 0,
                    stock: 0,
                    units: "元",
                    status: 1,
                    goodsType: "",
                },
                ruls: {
                    skuName: [
                        {required: true, message: "商品名不能为空"},
                        {pattern: /^.{4,16}$/, message: '商品名应为4-16位字符'}
                    ],
                    skuNo: [
                        {required: true, message: "商品名不能为空"},
                        {pattern: /^.{4,16}$/, message: '商品名应为4-16位字符'}
                    ],
                    price: [{required: true, message: "价格不能为空"}],
                    units: [{required: true, message: "单位不能为空"}],
                    stock: [{required: true, message: "库存不能为空"}],
                    status: [{required: true, message: "用户状态不能为空"}],
                }
            };
        },
        props: {
            goodsType: {
                type: Array,
                default: []
            },
            id: {
                type: String,
                default: {}
            }
        },
        created() {
            this.getUserInfo();
        },
        methods: {
            handleSuccess(res, file) {
                if (res.status !== 1) {
                    this.$Notice.warning({
                        title: '上传失败',
                        desc: res.msg + ":" + res.data
                    });
                    return false;
                }
                this.data.img = res.data;
                this.data.smallImg = res.data;
            },
            handleError(res, file) {
                this.$Notice.warning({
                    title: '上传失败',
                    desc: res
                });
            },
            handleFormatError(file) {
                this.$Notice.warning({
                    title: '上传格式有误',
                    desc: '请上传jpg,jpeg,png格式的图片'
                });
            },
            handleMaxSize(file) {
                this.$Notice.warning({
                    title: '文件太大了',
                    desc: '请上传不超过2M的图片'
                });
            },
            /**
             * @description 关闭Modal
             * @param reload 是否重新加载数据
             */
            cancel(reload = false) {
                this.$emit("cancel", "update", reload);
            },
            /**
             * @description 获取用户信息
             */
            async getUserInfo() {
                try {
                    let res = await post('/shop/sku/get/id/{id}', null, {
                        id: this.id
                    });
                    this.data = res.data;
                    this.data.goodsType = Number(this.data.goodsType);
                    console.log(this.data.goodsType);
                    console.log(this.goodsType);
                } catch (error) {
                    this.$throw(error)
                }
            },
            /**
             * @description 确定按钮单击回调
             */
            ok() {
                this.$refs.modalForm.validate(valid => {
                    if (valid) {
                        let data = JSON.parse(JSON.stringify(this.data));
                        this.update(data)
                    }
                });
            },
            /**
             * @description 更新用户数据请求
             */
            async update(data) {
                this.loading = true;
                try {
                    let res = await post('/shop/sku/update/{id}', data, {
                        id: this.data.id
                    });
                    this.$Message.success("更新成功");
                    this.cancel(true)
                } catch (error) {
                    this.$throw(error)
                }
                this.loading = false;
            }
        }
    };
</script>

