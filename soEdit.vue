<!-- 
    Created by Leon on 2017/12/11.
 -->

<template>
    <container>
        <div id="app" slot="content" class="contract-container">
            <div class="operate-wrap">
                <main>
                    <tabs v-model="activeName" :saNo="soData.saNo"></tabs>

                    <el-form :model="soData" label-width="100px" ref="soData">
                        <spconc :soData="soData"></spconc>

                        <should-pay :soData="soData" :step="step"></should-pay>

                        <input-ticket :soData="soData" :step="step"></input-ticket>

                        <el-row type="flex" justify="center"  style="margin-top:40px;">
                            <el-button v-if="step === 'edit'" class="submit-btn" type="info" @click="save(saveStep.save, 'soData')" :loading="isSubmitting">保存</el-button>
                            <el-button v-if="step === 'edit'" class="submit-btn" type="info" @click="save(saveStep.submit, 'soData')" :loading="isSubmitting">提交</el-button>
                            <el-button v-if="step === 'edit'" class="submit-btn" type="danger" @click="abandon()" :loading="isSubmitting">作废</el-button>

                            <el-button v-if="step === 'audit'" class="submit-btn" type="info" @click="audit(auditStep.pass)" :loading="isSubmitting">通过</el-button>
                            <el-button v-if="step === 'audit'" class="submit-btn" type="danger" @click="audit(auditStep.reject)" :loading="isSubmitting">驳回</el-button>

                            <!-- 当前对账单处于 生效中&&审核通过 则显示回退-->
                            <!--<el-button v-if="soData.saStatus == 3 && soData.auditStatus == 4" class="submit-btn" type="danger" @click="rollback()" :loading="isSubmitting">回退</el-button>-->
                            <el-button v-if="step === 'back'" class="submit-btn" type="danger" @click="rollback()" :loading="isSubmitting">回退</el-button>

                            <el-button class="submit-btn" type="" @click="returnList()" :loading="isSubmitting">返回</el-button>
                        </el-row>
                    </el-form>

                    <div style="width: 80%;margin-top: 50px;">
                        <operationLog :logList="operationLog"></operationLog>
                    </div>
                </main>
            </div>

            <!--<reasonPop-->
            <!--:popshow="popshow"-->
            <!--:poptitle="poptitle"-->
            <!--:step="step"-->
            <!--:reason="reason"-->
            <!--&gt;</reasonPop>-->
            <Popup
                    :visible="popshow"
                    :title="poptitle"
                    @close="popshow=false;reason=''"
            >
                <div class="add-contact" slot="pop-content">
                    <el-input
                            type="textarea"
                            :autosize="{ minRows: 10, maxRows: 10}"
                            placeholder="请输入备注"
                            :maxlength="200"
                            v-model="reason"
                    ></el-input>
                    <div class="btn">
                        <el-button v-if="step === 'audit'" type="info" icon="circle-check" @click="submitAudit()">确认</el-button> <!--popshow = false;  :disabled="!reason.trim()" -->
                        <el-button v-if="step === 'edit'" type="info" icon="circle-check" @click="submitForm()">确认</el-button> <!-- popshow = false;  :disabled="!reason.trim()" -->
                        <el-button type="" icon="circle-cross" @click="popshow = false; reason=''">取消</el-button>
                        <span class="reason-content" v-if="reason.trim() == '' && reasonHint">备注不能为空!!</span>
                    </div>
                </div>
            </Popup>
        </div>
    </container>
</template>

<script>
    import HTTP from '@lib/http2';
    import Popup from '@components/Popup';
    import Container from '@components/Container';
    import Tabs from './privateComponent/Tabs.vue'
    import spconc from './privateComponent/spconc.vue';
    import shouldPay from './privateComponent/shouldPay.vue';
    import inputTicket from './privateComponent/inputTicket.vue';
    import operationLog from './privateComponent/operationLog.vue';
    import reasonPop from './privateComponent/reasonPop.vue';

    export default {
        components: {
            'Popup': Popup,
            'container': Container,
            'spconc': spconc,
            'should-pay': shouldPay,
            'input-ticket': inputTicket,
            'operationLog': operationLog,
            'reasonPop': reasonPop,
            'Tabs': Tabs
        },
        data() {
            const step = window.INIT_APP_DATA.step || 'view';
            const soData = window.INIT_APP_DATA.saData || {}; //对账单详情
            const operationLog = window.INIT_APP_DATA.operationLog || []; //日志

            return {
                activeName: '1', //默认选择第一个页签
                poptitle: '', //弹出框标题
                popshow: false, //默认不显示弹出框
                reason: '', //保存或提交理由 通过或驳回理由
                reasonHint: false, //是否显示 '备注不能为空提示'
                isSubmitting: false, //提交中
                operationLog: operationLog,
                soData: soData,
                step: step, //当前对账单属于哪一步骤 edit audit view back

                saveStepValue: 1, //保存提交默认值
                saveStep: {
                    save: 1,
                    submit: 2,
                },

                auditStepValue: 1, //审核默认值
                auditStep: {
                    pass: 1, //通过
                    reject: 2, //驳回
                },

                backUrl: '/sa/list/info', //回退的url
            }
        },
        mounted() {

        },
        methods: {
            //回退
            rollback() {
                const saNo = this.soData.saNo;
                if (!saNo) {
                    this.$alert('saNo异常:' + saNo, '提示', {confirmButtonText: '确定', type: 'info'});
                    return;
                }

                this.$confirm('确认回退此对账单？', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.isSubmitting = true;
                    HTTP.post({
                        url: `/sa/rollback/${saNo}`,
                        success: () => {
                            this.isSubmitting = false;
                            this.$alert('回退成功!', '提示', {
                                confirmButtonText: '确定',
                                type: 'success',
                                callback: () => {
                                    window.location.href = this.backUrl;
                                }
                            });
                        },
                        error: (err) => {
                            this.isSubmitting = false;

                            this.$alert(err.msg || '服务器错误', '提示', {confirmButtonText: '确定', type: 'info'});
                        },
                    });
                }).catch(() => {
                    console.log('取消了回退...');
                })
            },

            //检查reason合法性
            checkReasonVaild() {
                if (!this.reason.trim()) {
                    this.reasonHint = true;
                    return false;
                } else {
                    this.reasonHint = false;
                    this.popshow = false;
                    return true;
                }
            },

            //审核弹窗出现
            audit(value) {
                this.auditStepValue = value;
                this.poptitle = value == this.auditStep.pass ? '请输入通过意见' : '请输入驳回意见';
                this.popshow = true;
            },

            //确认通过或驳回
            submitAudit() {
                if (this.auditStepValue !== this.auditStep.pass && this.auditStepValue !== this.auditStep.reject) {
                    return;
                }

                if (!this.checkReasonVaild()) {
                    return;
                }

                this.isSubmitting = true;
                HTTP.post({
                    url: '/sa/auditResult',
                    data: {
                        saNo: this.soData.saNo,
                        status: this.auditStepValue,
                        memo: this.reason,
                    },
                    success: () => {
                        this.isSubmitting = false;

                        this.$alert('操作成功！', '提示', {
                            confirmButtonText: '确定',
                            type: 'success',
                            callback: () => {
                                window.location.href = this.backUrl;
                            }
                        });
                    },
                    error: (err) => {
                        this.isSubmitting = false;

                        this.$alert(err.msg || '服务器错误', '提示', {confirmButtonText: '确定', type: 'info'});
                    },
                })
            },

            //编辑时 保存或提交
            save(value, soData) {
                this.$refs[soData].validate((valid) => {
                    if (valid) {
                        this.saveStepValue = value;
                        this.poptitle = value == this.saveStep.save ? '请输入保存备注' : '请输入提交备注';
                        this.popshow = true;
                    } else {
                        this.$alert('有必填项或必传附件遗漏!', '提示', {confirmButtonText: '确定', type: 'info'});
                    }
                })
            },

            //确认保存提交
            submitForm() {
                if (this.saveStepValue !== this.saveStep.save && this.saveStepValue !== this.saveStep.submit) {
                    return;
                }

                if (!this.checkReasonVaild()) {
                    return;
                }

                this.isSubmitting = true;
                HTTP.post({
                    url: '/sa/save',
                    data: {
                        type: this.saveStepValue,
                        saData: this.soData,
                        memo: this.reason,
                    },
                    success: () => {
                        this.isSubmitting = false;

                        this.$alert('操作成功！', '提示', {
                            confirmButtonText: '确定',
                            type: 'success',
                            callback: () => {
                                window.location.href = this.backUrl;
                            }
                        });
                    },
                    error: (err) => {
                        this.isSubmitting = false;

                        this.$alert(err.msg || '服务器错误', '提示', {confirmButtonText: '确定', type: 'info'});
                    },
                })
            },

            abandon() {
                const soNo = this.soData.saNo;
                if (soNo != undefined) {
                    this.$confirm('确认作废此对账单？', '提示', {
                        confirmButtonText: '确定',
                        cancelButtonText: '取消',
                        type: 'warning'
                    }).then(() => {
                        this.isSubmitting = true;
                        HTTP.get({
                            url: `/sa/cancel/${soNo}`,
                            success: () => {
                                this.isSubmitting = false;
                                this.$alert('作废成功!', '提示', {
                                    confirmButtonText: '确定',
                                    type: 'success',
                                    callback: () => {
                                        window.location.href = this.backUrl;
                                    }
                                });
                            },
                            error: (error) => {
                                this.isSubmitting = false;
                                this.$alert(error.msg || '服务器错误!', '提示', {confirmButtonText: '确定', type: 'error'});
                            },
                        });
                    }).catch(() => {
                        console.log('取消了作废采购单');
                    })
                } else {
                    this.$alert('当前对账单单无saNo!', '提示', {confirmButtonText: '确定', type: 'error'});
                }
            },
            returnList() {
                window.location.href = this.backUrl;
            },
        },
    }
</script>

<style lang="less">
    @import '../../less/common.less';

    .name-span {
        min-width: 90px;
        display: inline-block;
        text-align: right;
    }

    .contract-container {
        font-family: Arial, Helvetica, Sans-Serif;
        min-width: 1000px;
        overflow-x: auto;
        /*overflow-x: scroll;*/
    }

    main {
        .el-form {
            margin-top: 40px
        }
        .el-row {
            margin: 20px;
        }

        .el-form-item {
            margin-bottom: 0;
        }

        .el-button {
            width: 80px;
        }

        .title-line {
            text-align: left;
            width: 90%;
            height: 1px;
            background: #20a0ff;
            font-weight: 500;
            line-height: 1px;
            margin-top: 22px;
            margin-bottom: 22px;
            span {
                margin-left: 20px;
                color: #CCC;
                background: #fff;
            }
        }
    }

    .add-contact {
        width: 400px;
        line-height: 35px;
        max-height: 600px;
        overflow: auto;
    }
    .add-contact .btn {
        margin: 10px 0;
    }

    .reason-content {
        color: #ff4949;
        font-size: 12px;
        line-height: 1;
        padding-top: 4px;
        padding-left: 10px;
        /*position: absolute;*/
        /*top: 100%;*/
        /*left: 0;*/
    }
</style>
