<template>
    <div class="writeRecipe">
        <div class="info-header-rr">
            <a class="go-back"  @click="saveRecipeD('back')"></a>
            <h2>添加/编辑用药</h2>
            <p @click="saveRecipeD()">完成</p>
        </div>
        <div class="recipe-list" :style='{height:recipeHeight}'>
            <div class="item">
                <span><i>*</i>药名：</span>
                <input style="width: 4rem" readonly="readonly" onfocus="this.blur()" type="text" placeholder="点击填写（必填）" v-model="editData.drug_name" @click="showSearchDrug">
            </div>

            <div class="item" v-if="drugDetail.norm && editData.spec !='' && editData.spec">
                <span><i>*</i>规格：</span>
                <span class="down-icon">
                    <input class="select" style="width: 4rem" v-model="editData.spec" type="text" readonly="readonly" onfocus="this.blur()" @click="showSpecPicker()">
                </span>
                <!-- <select style="width: 4rem" @change="choseSpec()" v-model="editData.spec">
                    <option v-for="(val,$index) in drugDetail.norm" :key="$index" :value="val.spec">{{val.spec}}</option>
                </select> -->
            </div>

            <div class="item">
                <span><i>*</i>用法：</span>
                <span class="down-icon">
                    <input class="select" readonly="readonly" onfocus="this.blur()" style="width: 2.5rem" v-model="editData.usage" type="text" @click="showUsagePicker()">
                </span>
            </div>

            <div class="item br clearfix">
                <div class="lf" style="position:relative">
                    <span><i>*</i>单位：</span>
                    <em style="font-style: normal;position: absolute;font-size: 12px;color: red;bottom: -15px;left: 0.2rem;" v-if="(!editData.drug_unit && !loseFocus) || editData.drug_unit =='请选择单位'">请选择单位</em>
                    <span class="down-icon">
                        <input class="select" :class="{loseFocus: (!editData.drug_unit && !loseFocus) || editData.drug_unit =='请选择单位'}" readonly="readonly" onfocus="this.blur()" style="width: 1.5rem" type="text" v-model="editData.drug_unit" @click="showUnitsPicker()">
                    </span>
                    <!-- <select :class="{loseFocus: (!editData.drug_unit && !loseFocus) || editData.drug_unit =='请选择单位'}" style="width: 1.5rem" name="" id="" v-model="editData.drug_unit">
                        <option>请选择单位</option>
                        <option v-for="(val,$index) in defaluteUnit" :key="$index" :value="val">{{val}}</option>
                    </select> -->
                </div>
                <div class="lf">
                    <span><i>*</i>间隔：</span>
                    <span class="down-icon">
                        <input class="select" readonly="readonly" onfocus="this.blur()" style="width: 1.5rem" v-model="editData.frequency" type="text" @click="showFrequencyPicker()">
                    </span>
                </div>
            </div>
            <div class="item" style="padding: 0.05rem 0.2rem;" v-if="editData.spec">
                已选规格 {{editData.spec}}
            </div>
            <div class="item" style="padding: 0.05rem 0.2rem;background-color: #ededed;border-color:#c1cacf;border-top: 1px solid #c1cacf;margin-top: -1px;">
                <span><i>*</i>用法用量</span>
            </div>
            <div class="item br clearfix">
                <div class="lf">
                    <span>早：</span>
                    <img src="../../../assets/images/pen.png" alt="">
                    <input class="dosage" v-model="editData.morning">
                    ({{editData.drug_unit && editData.drug_unit !='请选择单位'?editData.drug_unit:'单位'}})
                </div>
                <div class="lf" style="padding-left: 0.2rem;">
                    <span>午：</span>
                    <img src="../../../assets/images/pen.png" alt="">
                    <input class="dosage" v-model="editData.noon">
                    ({{editData.drug_unit && editData.drug_unit !='请选择单位'?editData.drug_unit:'单位'}})
                </div>
            </div>

            <div class="item br clearfix">
                <div class="lf">
                    <span>晚：</span>
                    <img src="../../../assets/images/pen.png" alt="">
                    <input class="dosage" v-model="editData.night">
                    ({{editData.drug_unit && editData.drug_unit !='请选择单位'?editData.drug_unit:'单位'}})
                </div>
                <div class="lf" style="padding-left: 0.2rem;">
                    <span>睡前：</span>
                    <img src="../../../assets/images/pen.png" alt="">
                    <input style="width: -webkit-calc(100% - 95px - 0.2rem);width: calc(100% - 95px - 0.2rem);" class="dosage" v-model="editData.before_sleep">
                    ({{editData.drug_unit && editData.drug_unit !='请选择单位'?editData.drug_unit:'单位'}})
                </div>
            </div>

            <div class="remark item">
                <p>备注信息</p>
                <textarea name="" id=""  placeholder="点击填写" v-model="editData.remark"></textarea>
                <img src="../../../assets/images/pen.png" alt="">
            </div>
        </div>
        <searchDrug v-if="showSearch" @selectDrug='chooseDrug' :drugName='editData.drug_name' :drugSpec="editData.spec"></searchDrug>
        <mt-popup
            v-model="showPicker"
            position="bottom"
            :modal='hideModel'
            popup-transition="popup-fade" class="picker-box">
            <mt-picker v-if="showUsage" :showToolbar="true" ref="drug_usage" :slots="defaluteUsage">
                <div class="clearfix" style="background-color: #339de1;">
                    <span @click="handleCancel()" class="btn lf">取消</span>
                    <span @click="handleConfirmUsage()" class="btn rt">确认</span>
                </div>
            </mt-picker>
            <mt-picker v-if="showFrequency" :showToolbar="true" ref="drug_frequency" :slots="defaluteFrequency">
                <div class="clearfix" style="background-color: #339de1;">
                    <span @click="handleCancel()" class="btn lf">取消</span>
                    <span @click="handleConfirmFrequency()" class="btn rt">确认</span>
                </div>
            </mt-picker>
            <mt-picker v-if="showUnits" :showToolbar="true" ref="drug_units" :slots="defaluteUnit">
                <div class="clearfix" style="background-color: #339de1;">
                    <span @click="handleCancel()" class="btn lf">取消</span>
                    <span @click="handleConfirmUnits()" class="btn rt">确认</span>
                </div>
            </mt-picker>
            <mt-picker v-if="showSpec" :showToolbar="true" ref="drug_spec" :slots="norm">
                <div class="clearfix" style="background-color: #339de1;">
                    <span @click="handleCancel()" class="btn lf">取消</span>
                    <span @click="handleConfirmSpec()" class="btn rt">确认</span>
                </div>
            </mt-picker>
        </mt-popup>
        <div class="dialog" v-if="showPicker"></div>
    </div>
</template>
<style lang="sass">
    .picker-box{
        .picker-slot{
            width: 100%;
        }
    }
</style>

<style lang="sass" scoped>
    .dialog{
        position: fixed;
        width: 100%;
        height: 100%;
        left: 0;
        bottom: 0;
        background:rgba(0,0,0,0.4);
    }
    .picker-box{
        width: 100%;
        .btn{
            height: 40px;
            line-height: 40px;
            width: 60px;
            text-align: center;
            color: #fff;
        }
    }
    .info-header-rr{
        height:50px;
        line-height:50px;
        background-color:#339de1;
        color:#fff;
        display:-webkit-flex;
        display:flex;
        justify-content: space-between;
        padding:0 15px;
        p{
            font-size:14px;
        }
		h2{
			font-size:16px;
		}
    }
    .recipe-list{
        overflow: auto;
        text-align: left;
        input{
            border: none;

        }
        .item{
            padding: 0.3rem 0.2rem;
            border-bottom: 1px solid #ededed;
            position: relative;
            &.br{
                text-align: left;
                padding: 0.3rem 0;
                &:before{
                    content: '';
                    height: 100%;
                    width: 1px;
                    background-color: #ededed;
                    position: absolute;
                    left: 50%;
                    top: 0;
                }
                >.lf{
                    width: 50%;
                    padding: 0 0.2rem;
                }
            }
            i{
                color: red;
                vertical-align: middle;
                margin-right: 2px;
                font-size: 16px;
            }
            .down-icon{
                position: relative;
                display: inline-block;
                &:after{
                    content: '';
                    display:block;
                    border: 5px solid transparent;
                    border-top: 5px solid #c1cacf;
                    position: absolute;
                    top: 13px;
                    right: 7px;
                }
                &:before{
                    content: '';
                    display:block;
                    border-right: 1px solid #c1cacf;
                    height: 100%;
                    position: absolute;
                    top: 0;
                    right: 23px;
                }
            }
            input.select{
                border: 1px solid #c1cacf;
                background-color: #f1f1f1;
                height: 30px;
                line-height: 30px;
                border-radius: 3px;
                text-indent: 5px;
                &.loseFocus{
                    border-color: red;
                }
            }
            input,select{
                color: #000;
                &:focus,&:hover{
                    outline: none;
                }
            }
            .dosage{
                width: calc(100% - 90px - 0.2rem);
                width: -webkit-calc(100% - 90px - 0.2rem);
                border-bottom: 1px solid #c1cacf;
            }
            img{
                width: 0.19rem;
                height: 0.28rem;
                vertical-align: middle;
            }
        }
        .remark{
            border-bottom: 0;
            position: relative;
            p{
                color: #333;
                font-size: 15px;
                line-height: 0.4rem;
            }
            textarea{
                border: 1px solid #c1cacf;
                background-color: #f1f1f1;
                width: 100%;
                min-height: 100px;
                border-radius: 0.1rem;
                padding: 0.2rem 0.3rem;
                resize:none;
                margin-top: 10px;
                font-size: 15px;
            }
            img{
                width: 0.19rem;
                height: 0.28rem;
                vertical-align: middle;
                position: absolute;
                left: 0.3rem;
                top: 1.1rem;
            }
        }
    }
</style>
<script>
    import searchDrug from './searchDrug'
    import {Ajax, Tips, topPopup, preTreated} from '../../../lib/common'
    import $ from 'jquery'
    import { Picker,Popup } from 'mint-ui';
    import Vue from 'vue'
    Vue.component(Popup.name, Popup);
    Vue.component(Picker.name, Picker);
    export default{
        data(){
            return{
                recipeHeight: document.body.clientHeight - 50 + 'px',
                editData:{
                    drug_name: '', //药名
                    spec: '', //规格
                    usage: '', // 用法
                    drug_unit: '', // 用药单位
                    frequency: '每天', // 间隔
                    morning: '', // 早上
                    noon: '', // 中午
                    night: '', // 晚上
                    before_sleep: '', // 睡前
                    remark: '', // 备注
                    specification:'',
                    scientific_name:'',
                    take_medicine_days:'',
                    is_first_unit: 0 // 是否是一级单位
                },
                showSearch: false,
                drugDetail:{},
                norm:[{values:[]}],
                defaluteUsage: [{values:['口服','含服','外用','口腔吸入','肌肉注射','静脉注射'],}],
                defaluteUnit:[{values:['请选择单位','毫克','克','粒','袋','支','毫升']}],
                defaluteFrequency:[{values:['每天','必要时','隔天','每周','每月','隔两天','隔三天']}],
                loseFocus: true,
                isEdit: false,
                hideModel: false,
                showPicker: false,

                showUsage: false,
                showUnits: false,
                showFrequency: false,
                showSpec: false,
                // 阻止默认事件
                handler: function(e) {e.preventDefault();},

            }
        },
        props:{
            drug_info:{
                type: Object,
                default: function(){
                    return {}
                }
            }
        },
        mounted() {
           
        },
        methods:{
             // 解决iPhone页面滑动穿透问题
            closeTouch(){
                let self = this
                document.getElementsByTagName('body')[0].addEventListener('touchmove',self.handler,{passive: false})
            },
            openTouch(){
                let self = this
                document.getElementsByTagName('body')[0].removeEventListener('touchmove',self.handler,{passive: false})
            },
            // 打开用法
            showUsagePicker(){
                let self = this
                self.showUsage = true
                self.showPicker = true
                setTimeout(() => {
                    self.$refs.drug_usage.setSlotValue(0, self.editData.usage)
                },100) 
            },
            // 打开间隔
            showFrequencyPicker(){
                let self = this
                self.showFrequency = true
                self.showPicker = true
                setTimeout(() => {
                    self.$refs.drug_frequency.setSlotValue(0, self.editData.frequency)
                },100)  
            },
            // 打开单位
            showUnitsPicker() {
                let self = this
                self.showUnits = true
                self.showPicker = true
                setTimeout(() => {
                    self.$refs.drug_units.setSlotValue(0, self.editData.drug_unit ? self.editData.drug_unit : '请选择单位')
                },100)
            },
            // 打开规格
            showSpecPicker(){
                let self = this
                self.showSpec = true
                self.showPicker = true
                setTimeout(() => {
                    self.$refs.drug_spec.setSlotValue(0, self.editData.spec ? self.editData.spec : '')
                },100)
            },
            // 选择用法
            handleConfirmUsage(){
                let self = this
                self.editData.usage = self.$refs.drug_usage.getValues()[0]
                self.handleCancel()
            },
            // 选择间隔
            handleConfirmFrequency(){
                let self = this
                self.editData.frequency = self.$refs.drug_frequency.getValues()[0]
                self.handleCancel()
            },
            // 选择单位
            handleConfirmUnits() {
                let self = this
                self.$refs.drug_units.getValues()[0] == '请选择单位' ? self.editData.drug_unit = '' : self.editData.drug_unit = self.$refs.drug_units.getValues()[0]
                self.handleCancel()
            },
            // 选择规格
            handleConfirmSpec(){
                let self = this
                self.editData.spec = self.$refs.drug_spec.getValues()[0]
                self.handleCancel()
                self.choseSpec()
            },
            // 取消picker弹窗
            handleCancel(){
                let self = this
                self.showPicker=false
                self.showUsage=false
                self.showUnits=false
                self.showFrequency=false
                self.showSpec=false
            },
            clearNoNum(obj){
                obj = obj.replace(/[^\d.]/g,""); //清除"数字"和"."以外的字符
                obj = obj.replace(/^\./g,""); //验证第一个字符是数字
                obj = obj.replace(/\.{2,}/g,"."); //只保留第一个, 清除多余的
                obj = obj.replace(".","$#$").replace(/\./g,"").replace("$#$","."); ////保证.只出现一次，而不能出现两次以上
                return obj
            },
            // 监控规格改变
            choseSpec(){
                let self = this
                let name = self.editData.spec
                if(!self.drugDetail.norm) return
                for(let val of self.drugDetail.norm){
                    if(val.spec == name){
                        self.editData.specification = val.specification.toString()
                        self.editData.units = val.units
                        val.id != undefined || val.id != null ? self.editData.id = val.id : ''
                    }
                }
            },
            // 重新初始化药品
            initDrugInfo(){
                this.editData={
                    drug_name: '', //药名
                    spec: '', //规格
                    usage: '', // 用法
                    drug_unit: '', // 用药单位
                    frequency: '每天', // 间隔
                    morning: '', // 早上
                    noon: '', // 中午
                    night: '', // 晚上
                    before_sleep: '', // 睡前
                    remark: '', // 备注
                    specification:'',
                    units: '',
                    scientific_name:'',
                    take_medicine_days:''
                }
            },
            amendDrug(){
                let self = this
                console.log(JSON.parse(JSON.stringify(this.drug_info)))
                self.editData = JSON.parse(JSON.stringify(this.drug_info)) 
                self.editData.spec == '-1' ? self.editData.spec = '其他规格(不清楚规格)' : ''
                self.isEdit = true
                self.getDrugInfo('edit')
            },
            // 保存处方
            saveRecipeD(back) {
                let self = this
                let data = self.editData
                if(back){
                    self.$emit('saveRecipe','')
                    return
                }
                if(data.morning ==0){data.morning= ''}
                if(data.noon ==0){data.noon= ''}
                if(data.night ==0){data.night= ''}
                if(data.before_sleep ==0){data.before_sleep= ''}
                if(!data.drug_name){
                    topPopup('请填写药名') 
                    return
                }
                if(!data.drug_unit || data.drug_unit =='请选择单位'){
                    topPopup('请选择用药单位') 
                    return
                }
                if(!data.morning && !data.noon && !data.night && !data.before_sleep){
                    topPopup('请填写用法用量') 
                    return
                }
                data.spec== '其他规格(不清楚规格)'  ? data.spec = '-1' : ''
                data.id == undefined || data.id == null ? data.id = 0 : ''
                self.isEdit ? data.isEdit = true : ''
                // 判断是否是一级单位
                if(!self.drugDetail || (self.drugDetail.secondary_units&&self.drugDetail.secondary_units.indexOf('ml')>0)) {
                   data.is_first_unit = 0 
                }else {
                    data.is_first_unit = 1
                }
                self.$emit('saveRecipe',data)
            },
            // 获取返回的药品名字和规格
            chooseDrug(val){
                let self = this
                if(val.name){
                    self.initDrugInfo()
                    self.editData.drug_name = val.name
                    val.spec == '不清楚规格' ? self.editData.spec = '其他规格(不清楚规格)' : self.editData.spec = val.spec
                    self.getDrugInfo()
                    self.loseFocus = false
                }
                self.showSearchDrug()
            },
            // 获取药品信息
            getDrugInfo(edit){
                let self = this
                Tips('','show')
                Ajax('get','drug/drug-level-spec',{drug_name:self.editData.drug_name}).then(data => {
                    Tips('','hide')
                    preTreated(data,res => {
                        if(res.data){
                            let obj={
                                spec: '其他规格(不清楚规格)',
                                specification: '',
                                units: '',
                                id: 0,
                            }
                            res.data.norm.push(obj)
                            let arr = []
                            for(let k of res.data.norm){
                                arr.push(k.spec)
                            }
                            self.norm[0].values = arr
                            self.drugDetail = res.data
                            self.choseSpec()
                            if(res.data.units&&res.data.units.length){
                                res.data.units.unshift('请选择单位')
                                self.defaluteUnit[0].values = res.data.units
                            }else {
                                self.defaluteUnit=[{values:['请选择单位','毫克','克','粒','袋','支','毫升']}]
                            }
                            // 当用法为空且不是修改药品
                            if(self.editData.usage == '' && !self.isEdit) {
                                res.data.usage == '' || !res.data.usage ? self.editData.usage = '口服' : self.editData.usage = res.data.usage
                            }
                            self.editData.frequency == ''|| self.editData.frequency == null || self.editData.frequency == undefined ? self.editData.frequency == '每天' : ''
                        }else {
                            // 当库里没有这个药时
                            // 当用法为空且不是修改药品
                            if(self.editData.usage == '' && !self.isEdit) {
                                self.editData.usage = '口服'
                            }
                            
                        }
                    })
                }).catch(err => {
                    console.log(err)
                })
            },
            showSearchDrug(){
                this.showSearch = !this.showSearch
            }
        },
        watch:{
            'editData.morning'(){
                let self = this
                let val = self.clearNoNum(self.editData.morning)
                self.editData.morning = val
            },
            'editData.noon'(){
                let self = this
                let val = self.clearNoNum(self.editData.noon)
                self.editData.noon = val
            },
            'editData.night'(){
                let self = this
                let val = self.clearNoNum(self.editData.night)
                self.editData.night = val
            },
            'editData.before_sleep'(){
                let self = this
                let val = self.clearNoNum(self.editData.before_sleep)
                self.editData.before_sleep = val
            },
            showPicker(){
                let self = this
                if(self.showPicker){
                    self.closeTouch()
                }else{
                    self.openTouch()
                }
            }
        },
        components:{
           searchDrug 
        }
    }
</script>

