<template>
    <div>
        <Modal v-model="show" title="添加一对一捐助"
               :mask-closable="false" :closable="false">
            <Form ref="modalForm" :model="data" :rules="ruls"
                  :label-width="80">
                <FormItem label="一对一捐助类型" prop="one2oneType">
                    <RadioGroup v-model="data.one2oneType">
                        <Radio v-for="(item,index) in one2oneType" :label="item.code" :key="index" >{{item.desc}}
                        </Radio>
                    </RadioGroup>
                </FormItem>
                <FormItem label="姓名" prop="name">
                    <Input v-model.trim="data.name"></Input>
                </FormItem>
                <FormItem label="性别" prop="sex">
                    <RadioGroup v-model="data.sex">
                        <Radio label="男">男</Radio>
                        <Radio label="女">女</Radio>
                    </RadioGroup>
                </FormItem>
                <FormItem label="项目简介" prop="donationBrief">
                    <Input v-model="data.donationBrief"></Input>
                </FormItem>
                <FormItem label="项目详情" prop="donationInfo">
                    <Input v-model="data.donationInfo"></Input>
                </FormItem>
                <FormItem label="区块地址" prop="blAddress">
                    <Input v-model="data.blAddress"></Input>
                </FormItem>
                <FormItem label="省市" prop="donationBrief">
                    <Cascader :data="citysData" trigger="hover" @on-change="citysHandler"></Cascader>
                </FormItem>
                <FormItem label="住址" prop="donationBrief">
                    <Input v-model="data.address"></Input>
                </FormItem>
                <div v-if="data.one2oneType==='1'">
                    <FormItem label="年龄" prop="age">
                        <InputNumber :min="0" :step="1" v-model.trim="data.age" style="width:100%"/>
                    </FormItem>
                    <FormItem label="家庭状况" prop="donationBrief">
                        <Input v-model="data.homeInfo"></Input>
                    </FormItem>
                    <FormItem label="梦想" prop="donationBrief">
                        <Input v-model="data.dream"></Input>
                    </FormItem>
                    <FormItem label="兴趣" prop="donationBrief">
                        <Input v-model="data.interest"></Input>
                    </FormItem>
                </div>


                <FormItem label="状态" prop="status">
                    <Select v-model.trim="data.status" style="width:100%">
                        <Option v-for="item in [{label:'正常',value:1},{label:'关闭',value:0}]"
                                :value="item.value" :key="item.value">{{ item.label }}
                        </Option>
                    </Select>
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
    import citys from '@/libs/citys'

    export default {
        data() {
            return {
                show: true,
                loading: false,
                citysData: citys,
                citySelect: [],
                data: {
                    name: "",
                    age: 0,
                    status: 1,
                    img: "",
                    smallImg: "",
                    donationBrief: "",
                    donationInfo: "",
                    blAddress: "",
                    address: "",
                    homeInfo: "",
                    dream: "",
                    interest: "",
                    city: "",
                    province: "",
                    sex: "",
                },
                ruls: {
                    name: [
                        {required: true, message: "姓名不能为空"},
                        {pattern: /^(\w)|[\u4e00-\u9fa5]{2,10}$/, message: '姓名应该最少2个字'}
                    ],
                    one2oneType: [{required: true, message: "一对一捐助类型不能为空"}],
                    status: [{required: true, message: "状态不能为空"}],
                }
            };
        },
        props: {
            one2oneType: {
                type: Array,
                default: []
            }
        },
        methods: {
            citysHandler(value, selectedData) {
                this.data.city = selectedData[0].label;
                this.data.province = selectedData[1].label;
            },
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
                this.$emit("cancel", "add", reload);
            },
            /**
             * @description 确定按钮单击回调
             */
            ok() {
                this.$refs.modalForm.validate(valid => {
                    if (valid) {
                        let data = JSON.parse(JSON.stringify(this.data));
                        this.add(data)
                    }
                });
            },
            /**
             * @description 添加用户数据请求
             */
            async add(data) {
                this.loading = true;
                try {
                    let res = await post('/charitable/one2one/add', data);
                    this.$Message.success("募捐人 " + data.name + " 添加成功");
                    this.cancel(true)
                } catch (error) {
                    this.$throw(error)
                }
                this.loading = false;
            }
        }
    };
</script>

