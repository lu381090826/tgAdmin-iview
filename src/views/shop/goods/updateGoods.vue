<template>
    <div>
        <Card>
            <Form ref="modalForm" :model="data" :rules="ruls"
                  :label-width="80">
                <FormItem label="商品名称" prop="sku_name">
                    <Input v-model.trim="data.skuName"></Input>
                </FormItem>
                <FormItem label="商品简单名称" prop="simple_name">
                    <Input v-model.trim="data.simpleName"></Input>
                </FormItem>
                <FormItem label="商品编号" prop="sku_no">
                    <Input v-model.trim="data.skuNo"></Input>
                </FormItem>
                <FormItem label="价格" prop="price">
                    <InputNumber :min="0" :step="1" v-model.trim="data.price" style="width:100%"/>
                </FormItem>
                <FormItem label="原价格" prop="originPrice">
                    <InputNumber :min="0" :step="1" v-model.trim="data.originPrice" style="width:100%"/>
                </FormItem>
                <FormItem label="原人民币价格" prop="originPriceCn">
                    <InputNumber :min="0" :step="1" v-model.trim="data.originPriceCn" style="width:100%"/>
                </FormItem>
                <FormItem label="库存" prop="price">
                    <InputNumber :min="0" :step="1" v-model.trim="data.stock" style="width:100%"/>
                </FormItem>
                <FormItem label="单位" prop="units">
                    <Input v-model.trim="data.units"/>
                </FormItem>
                <FormItem label="商品类型" prop="goodsType">
                    <RadioGroup v-model="data.goodsType">
                        <Radio v-for="(item,index) in goodsType" :label="item.desc" :key="item.code">{{item.desc}}
                        </Radio>
                    </RadioGroup>
                </FormItem>
                <FormItem label="主图片" prop="img">
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
                <FormItem label="商品详情">
                    <div>
                        <div class="demo-upload-list" v-for="item in uploadList">
                            <template v-if="item.status === 'finished'">
                                <img :src="item.url">
                                <div class="demo-upload-list-cover">
                                    <Icon type="ios-eye-outline" @click.native="handleView(item.name)"></Icon>
                                    <Icon type="ios-trash-outline" @click.native="handleRemove(item)"></Icon>
                                </div>
                            </template>
                            <template v-else>
                                <Progress v-if="item.showProgress" :percent="item.percentage" hide-info></Progress>
                            </template>
                        </div>
                        <Upload
                                ref="upload"
                                :show-upload-list="false"
                                :on-success="handleSuccess"
                                :format="['jpg','jpeg','png']"
                                :max-size="2048"
                                :on-format-error="handleFormatError"
                                :on-exceeded-size="handleMaxSize"
                                :before-upload="handleBeforeUpload"
                                multiple
                                type="drag"
                                action="//localhost:9000/FileUpload/FileUp"
                                style="display: inline-block;width:58px;"
                                class="uploadImage"
                        >
                            <div style="display: none;">
                                <Icon type="ios-camera" size="20"></Icon>
                            </div>
                        </Upload>
                        <Modal title="View Image" v-model="visible">
                            <img :src="'http://img.localhost/'+imgName" v-if="visible"
                                 style="width: 100%">
                        </Modal>
                    </div>

                    <div class="edit_container">
                        <quill-editor
                                v-model="data.skuDetail"
                                ref="myQuillEditor"
                                :options="editorOption"
                                @blur="onEditorBlur($event)" @focus="onEditorFocus($event)"
                                @change="onEditorChange($event)">
                        </quill-editor>
                    </div>
                </FormItem>
            </Form>
            <div style="text-align: center">
                <Button type="primary" size="large" @click="ok">提交</Button>
            </div>
        </Card>
    </div>
</template>
<script>
    import {post} from '@/libs/axios-cfg'
    import goodsType from '@/conts/GoodsType'
    import {quillEditor} from "vue-quill-editor"; //调用编辑器
    import 'quill/dist/quill.core.css';
    import 'quill/dist/quill.snow.css';
    import 'quill/dist/quill.bubble.css';

    // 工具栏配置
    const toolbarOptions = [
        ['bold', 'italic', 'underline', 'strike'],        // toggled buttons
        ['blockquote', 'code-block'],

        [{'header': 1}, {'header': 2}],               // custom button values
        [{'list': 'ordered'}, {'list': 'bullet'}],
        [{'script': 'sub'}, {'script': 'super'}],      // superscript/subscript
        [{'indent': '-1'}, {'indent': '+1'}],          // outdent/indent
        [{'direction': 'rtl'}],                         // text direction

        [{'size': ['small', false, 'large', 'huge']}],  // custom dropdown
        [{'header': [1, 2, 3, 4, 5, 6, false]}],

        [{'color': []}, {'background': []}],          // dropdown with defaults from theme
        [{'font': []}],
        [{'align': []}],
        ['image', 'video'],
        // ['clean']                                         // remove formatting button
    ];

    export default {
        data() {
            return {
                show: true,
                loading: false,
                upload: null,
                editorOption: {
                    modules: {
                        toolbar: {
                            container: toolbarOptions,  // 工具栏
                            handlers: {
                                'image': function (value) {
                                    if (value) {
                                        document.querySelector('.uploadImage input').click();
                                    } else {
                                        this.quill.format('image', false);
                                    }
                                },
                            }
                        }
                    }
                },
                imgName: '',
                visible: false,
                uploadList: [],
                uploadFileList: [],
                goodsType: null,
                data: {
                    skuName: "",
                    skuNo: "",
                    price: 0,
                    originPrice: 0,
                    stock: 0,
                    units: "元",
                    status: 1,
                    goodsType: null,
                    skuDetail: "",
                    originPriceCn: 0,
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
        computed: {
            editor() {
                return this.$refs.myQuillEditor.quill;
            },
        },
        mounted() {
            this.uploadList = this.$refs.upload.fileList;
        },
        components: {
            quillEditor
        },
        created() {
            this.goodsType = goodsType;
        },
        methods: {
            onEditorReady(editor) { // 准备编辑器
            },
            onEditorBlur() {
            }, // 失去焦点事件
            onEditorFocus() {
            }, // 获得焦点事件
            onEditorChange() {
            }, // 内容改变事件
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

                file.url = 'http://img.localhost/' + res.data;
                file.name = res.data;

                let quill = this.$refs.myQuillEditor.quill;
                let length = quill.getSelection().index;

                quill.insertEmbed(length, 'image', file.url);
                quill.setSelection(length + 1);


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
            handleBeforeUpload() {
                const check = this.uploadList.length < 5;
                if (!check) {
                    this.$Notice.warning({
                        title: '最多只能上传 5 张图片。'
                    });
                }
                return check;
            },
            handleView(name) {
                this.imgName = name;
                this.visible = true;
            },
            handleRemove(file) {
                const fileList = this.$refs.upload.fileList;
                this.$refs.upload.fileList.splice(fileList.indexOf(file), 1);
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
                        // console.log(data)
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
                    let res = await post('/shop/sku/add', data);
                    this.$Message.success(data.skuName + " 添加成功");
                    this.cancel(true)
                } catch (error) {
                    this.$throw(error)
                }
                this.loading = false;
            }
        }
    };
</script>
<style>
    .uploadFile {
        width: 0;
        height: 0;
        display: none;
    }

    .demo-upload-list {
        display: inline-block;
        width: 60px;
        height: 60px;
        text-align: center;
        line-height: 60px;
        border: 1px solid transparent;
        border-radius: 4px;
        overflow: hidden;
        background: #fff;
        position: relative;
        box-shadow: 0 1px 1px rgba(0, 0, 0, .2);
        margin-right: 4px;
    }

    .demo-upload-list img {
        width: 100%;
        height: 100%;
    }

    .demo-upload-list-cover {
        display: none;
        position: absolute;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;
        background: rgba(0, 0, 0, .6);
    }

    .demo-upload-list:hover .demo-upload-list-cover {
        display: block;
    }

    .demo-upload-list-cover i {
        color: #fff;
        font-size: 20px;
        cursor: pointer;
        margin: 0 2px;
    }
</style>
