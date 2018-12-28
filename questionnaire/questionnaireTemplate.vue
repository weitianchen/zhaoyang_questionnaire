<template>
    <div class="questionnaire-box">
        <div class="header" v-if="hideHeader">
            <div class="info-header">
                <a class="go-back" @click="goback"></a>
                <h2>填写问卷</h2>
                <p @click="save" v-if="orderStatus">保存</p>
                <p v-else></p>
            </div>
        </div>
        <div class="desc" v-if="tags.length">
            症状：<span v-for="(item,$j) in tags" :key="$j">{{item}}{{$j != tags.length ? ',' : ''}}</span>
        </div>
        <div id="question-box" class="box" :style="{height: height}">
            <div class="template-scale" v-if="scale_list.length > 0 || template_list.length > 0">
            <!-- 量表 -->
                <div class="item" v-if="scale_list.length > 0" v-for="(item,$j) in scale_list" :key="$j" @click="toscale(item)">
                    <div class="info">
                        <h3>{{item.name}}</h3>
                        <p>评定：{{item.result? item.result : '无'}}</p>
                        <p>共{{item.size}}题 / 完成{{item.finish}}题</p>
                    </div>
                    <div class="turn-right" :class="{active:item.finish == 0}"></div>
                </div>
                <div class="item" v-if="template_list.length > 0" v-for="(item,$j) in template_list" :key="$j" @click="totemplate(item)">
                    <div class="info">
                        <h3>{{item.name}}</h3>
                        <p>共{{item.size}}题 / 完成{{item.finish}}题</p>
                    </div>
                    <div class="turn-right" :class="{active:item.finish == 0}"></div>
                </div>
            </div>
            <!-- 新增题目 -->
            <survey v-if="newList.length >0" ref="saveNewQuestion" :list="newList" :mapList="mapNewList" :initList="initNewList" :isNewQuestion='isNewQuestion' @saveReturn='acceptNewItem' :dynamic='dynamic' @contrast="newContrastData" :orderStatus="orderStatus" @hideNav="hideNav"></survey>
            <!-- 问卷 -->
            <survey v-if="list.length >0" ref="saveQuestion" :list='list' :mapList="mapList" :initList='initList' @saveReturn='acceptItem' @contrast="contrastData" :orderStatus="orderStatus" :isRecipe='isRecipe' @hideNav="hideNav"></survey>
        </div>
    </div>
</template>
<style lang="sass" scoped>
    .desc{
        padding: 0.1rem 0.2rem;
        background-color: #ededed;
        line-height: 20px;
        font-size: 14px;
        width: 100%;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
    }
    .box{
        // overflow-y:scroll;
        -webkit-overflow-scrolling: touch
    }
    .header{
        height:50px;
        line-height:50px;
        .info-header{
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height:50px;
            line-height:50px;
            background-color:#339de1;
            color:#fff;
            display:-webkit-flex;
            display:flex;
            justify-content: space-between;
            padding:0 15px;
            z-index:10;
            p{
                font-size:14px;
            }
            h2{
                font-size:16px;
            }
        }
    }
</style>
<script>
    import { MessageBox } from 'mint-ui'
    import {Ajax, Tips, topPopup, preTreated} from '../../../lib/common'
    import survey from './questionModule.vue'
    export default{
        data(){
            return{
                id: this.$route.query.id || '',
                historyGoback: this.$route.query.goback || '',
                // id: 4457,
                scale_list: [], // 量表
                template_list: [], // 模板
                list: [], // 页面渲染数组
                mapList: {}, //以question_id 为key值方便关联题目查询
                initList: [], //初始化时的原数组

                newList: [], // 新增题目
                mapNewList: {}, //新增题目map结构
                initNewList: [], // 新增题目的原始排列
                //  保存问卷时的参数
                saveQuestionnaireArgument: {
                    new_question_order_list:[],
                    new_refill_item_list: [],
                    question_order_list:[],
                    questionnaire_id: 0,
                    refill_item_list: [],
                    submit: true
                },
                isNewQuestion: true, // 标记新增题
                dynamic: 'new',
                iscontrast: false , // 用户是否有操作过
                strList: '',
                strNewList: '',
                isRecipe: 0 , // 记录当前问卷是否有填写过用药
                refill_item_list:[],// 初始化完成后保存结果，用来比较按返回键
                contrastNewList:[],
                contrastList:[],
                tags: []
            } 
        },
        props:{
            height:{
                type: String,
                default: function(){
                    return ''
                }
            },
            hideHeader: true,
            orderStatus: {
                type: Boolean,
                default: function() {
                    return true
                }
            }
        },
        mounted() {
            if(!this.id){
                topPopup('订单id不存在')
                return
            }
           
        },
        methods:{
            hideNav(val){
                this.$emit('hideNav',val)
            },
            // 触发获取数据
            strike(){
                this.getQuestionnaire()
            },
            // 查看量表
            toscale(item){
                this.$router.push({
                    name: 'scale',
                    query:{
                        id: item.questionnaire_id,
                        orderStatus: this.orderStatus,
                        name: item.name,
                        desc: item.desc,
                    }
                })
            },
            // 查看模板
            totemplate(item){
                this.$router.push({
                    name: 'questionnaireTemplate',
                    query:{
                        id: item.questionnaire_id,
                        name: item.name,
                        orderStatus: this.orderStatus,
                    }
                })
            },
            goback() {
                let self = this
                self.iscontrast = false
                self.initNewList.length ? self.$refs.saveNewQuestion.goBackContrast() : ''
                self.$refs.saveQuestion.goBackContrast()
                setTimeout(()=> {
                    if(self.iscontrast) {
                        self.contrastHint()
                    }else{
                        self.historyGoback ? this.$router.push({name:'order'}) : window.history.back()
                    }
                    
                },200)
            },
            contrastHint() {
                let self = this
                MessageBox.confirm('',{
                    title: '温馨提示',
                    message:'您填写的问题尚未保存，返回前是否将改动内容保存提交？',
                    confirmButtonText:"保存提交",
                    cancelButtonText:"不保存，返回"
                }).then(action => {
                    self.save()
                }).catch(err => {
                    window.history.back()
                })
            },
            // 按返回键时对比数据看看是否有操作过
            newContrastData(str) {
                let self = this
                JSON.stringify(self.contrastNewList) != JSON.stringify(str) ? self.iscontrast = true : ''
            },
            contrastData(str) {
                let self = this
                JSON.stringify(self.contrastList) != JSON.stringify(str) ? self.iscontrast = true : ''
            },
            save(){
                let self = this
                // 如果有新增问卷 先获取新增问卷
                self.initNewList.length ? self.$refs.saveNewQuestion.save() : self.$refs.saveQuestion.save()
            },
            // 接收新增问卷
            acceptNewItem(val){
                let self = this
                self.$refs.saveQuestion.save()
                self.saveQuestionnaireArgument.new_question_order_list = val.question_order_list
                self.saveQuestionnaireArgument.new_refill_item_list = val.refill_item_list
            },
            // 接收问卷
            acceptItem(val){
                let self = this
                self.saveQuestionnaireArgument.question_order_list = val.question_order_list
                self.saveQuestionnaireArgument.refill_item_list = val.refill_item_list
                self.saveQuestionnaire()
            },
            // 保存问卷
            saveQuestionnaire(){   
                let self = this
                Tips("正在保存...",'show')
                Ajax('post','java/doctor_record/save_questionnaire_v2',self.saveQuestionnaireArgument).then(data => {
                    Tips("正在保存...",'hide')
                    preTreated(data,res => {
                        if(res.status){
                            topPopup('保存成功')
                            setTimeout(() => {
                                self.historyGoback ? self.$router.push({name:'order'}) : window.history.back()
                            },2000)
                        }
                        console.log(res)
                    })
                }).catch(err => {
                    console.log(err)
                })
            },
            // 获取问卷
            getQuestionnaire(){
                let self = this
                Tips('','show')
                Ajax('get','java/doctor_record/get_questionnaire',{appointment_id: self.id,answerer_type: 'PATIENT'}).then(data =>{
                    Tips('','hide')
                    preTreated(data,res => {
                        let data = res.data
                        let arr = []
                        self.tags = data.tags
                        // 保存问卷id
                        self.saveQuestionnaireArgument.questionnaire_id = res.data.questionnaire_id
                        // 保存量表
                        self.scale_list = res.data.scale_list
                        // 保存模板
                        self.template_list = res.data.template_list
                        // 保存原始问卷
                        self.initList = self.addField(data.questionnaire_item_list)
                        // 如果有新增题目
                        data.new_questionnaire_item_list && data.new_questionnaire_item_list.length ? self.initNewList = self.addField(data.new_questionnaire_item_list) : ''
                        
                        // 构建map结构,用于填写过问卷重组
                        self.mapList = self.creaetLookMap(self.initList)

                        data.new_questionnaire_item_list && data.new_questionnaire_item_list.length ? self.mapNewList = self.creaetLookMap(self.initNewList) : ''
                        
                        if(data.question_order_list.length == 0){
                        // 用户没填写过问卷，直接使用原始问卷
                            let itemList = []
                            for(let val of self.initList){
                                if(val.order != -1){
                                    itemList.push(val)
                                }
                            }
                            self.list = itemList
                        }else {
                        // 用户填写过问卷
                            let b =  self.resetQuestion(data.question_order_list,self.mapList)
                            self.list =b.resetArr
                            self.isRecipe = b.isRecipe
                        }
                        self.strList = JSON.stringify(self.list)
                        self.contrastList = self.saveContrast(self.list)
                        // 如果医生设置了新增问卷
                        if(data.new_question_order_list && data.new_question_order_list.length !=0){
                            let b = self.resetQuestion(data.new_question_order_list,self.mapNewList)
                            self.newList = b.resetArr
                            self.strNewList = JSON.stringify(self.newList)
                            self.contrastNewList = self.saveContrast(b.resetArr)
                        }
                    })

                }).catch(err => {
                    console.log(err)
                })
            },
            // 原始数组增加字段作为数据操作
            addField(val){
                let arr = []
                for(let item of val){
                    // 增加字段判断是否有选中
                    item.isChecked = false
                    // 如果是处方题
                    if(item.question_type =="PRESCRIPTION"){
                        let obj ={
                            option_content:{}
                        }
                        item.option_list.push(JSON.parse(JSON.stringify(obj)))
                        item.option_list.push(JSON.parse(JSON.stringify(obj)))
                    }
                    // 选择题input 增加绑定字段
                    if(item.question_type=="MULTIPLE_NORMAL_CHOICE" || item.question_type=="NORMAL_CHOICE" || item.question_type=="MASSIVE_CHOICE" || item.question_type=="MULTIPLE_MASSIVE_CHOICE"){
                        for(let k of item.option_list){
                            k.inputTitle = ''
                        }
                    }
                    // 选择器选择增加绑定字段
                    if(item.question_type == "SELECTOR_CHOICE"){
                        if(item.choice_first){
                            item.option_list[0]&&item.option_list[0].option_content ? item.selectTitle = item.option_list[0].option_content : item.selectTitle = '请点击选择'
                        }else{
                            item.selectTitle = ''
                        } 
                    }
                    // 图片上传
                    if(item.question_type == 'PICTURE_UPLOAD'){
                        item.hideAdd = true
                        item.option_list[0].size ? item.imgSize = item.option_list[0].size : ''
                    }
                    arr.push(item)
                }
                return arr
            },
            // 提取问卷信息
            resetQuestion(arr,map){
                let resetArr = []
                let isRecipe
                // 循环问卷
                for(let item of arr){
                    // 获取问卷对应的原始数据
                    let tag = map[item.questionnaire_item_id]
                    // 处方题 
                    if(tag.question_type =="PRESCRIPTION"){
                        // 如果存在历史药单且大于原始的
                        item.answer_list.length > 1 ? isRecipe = item.answer_list.length : ''
                        if(item.answer_list.length > tag.option_list.length){
                            let num = item.answer_list.length-tag.option_list.length
                            for(let i = 0;i< num;i++){
                                let obj ={
                                    option_content:{}
                                }
                                tag.option_list.push(JSON.parse(JSON.stringify(obj)))
                            }
                            
                        }
                        item.answer_list.forEach((val,index) =>{
                            tag.option_list[index].option_content = JSON.parse(val.answer_content)
                            tag.option_list[index].option_content ? tag.isChecked = true : ''
                        })
                    }
                    // 选择题
                    if(tag.question_type=="MULTIPLE_NORMAL_CHOICE" || tag.question_type=="NORMAL_CHOICE" || tag.question_type=="MASSIVE_CHOICE" || tag.question_type=="MULTIPLE_MASSIVE_CHOICE"){
                        // 循环选中的答案，对比原始答案相同则选中
                        item.answer_list.forEach((val,index) =>{ 
                            tag.option_list.forEach((k,i) =>{
                                if(val.answer_code == k.option_code) {
                                    tag.isChecked = true
                                    k.isactive = true
                                    val.answer_content ? k.inputTitle = val.answer_content : k.inputTitle = ''
                                }
                            }) 
                        })
                    }

                    // 填空题
                    if(tag.question_type == 'FILL'){
                        item.answer_list.forEach((val,index) =>{ 
                            tag.option_list.forEach((k,i) =>{
                                if(val.answer_id == k.id) {
                                   tag.isChecked = true
                                   val.answer_content ? k.first_option_content = val.answer_content : k.first_option_content = ''
                                }
                            })
                        })
                    }

                    // 图片上传
                    if(tag.question_type == 'PICTURE_UPLOAD'){
                        // 如果存在 已填的图片
                        if(item.answer_list.length > tag.option_list.length){
                            let num = item.answer_list.length-tag.option_list.length
                            for(let i = 0;i< num;i++){
                                let obj ={
                                    second_option_content: ''
                                }
                                tag.option_list.push(JSON.parse(JSON.stringify(obj)))
                            }
                        }
                        //在最外层保存最大数量 且判断是否等于最大值
                        if(tag.option_list&&tag.option_list[0].size){
                            tag.imgSize = tag.option_list[0].size
                            //  // 如果存在 已填的图片 且存在的图片和最大上限值相等隐藏添加图片按钮
                            if((item.answer_list&&item.answer_list.length) && (item.answer_list.length == tag.option_list[0].size)){
                                tag.hideAdd = false
                            }
                        } 
                        item.answer_list&&item.answer_list.length? tag.isChecked = true : ''
                        item.answer_list.forEach((val,index) =>{ 
                            if(tag.option_list[index]){
                                tag.option_list[index].second_option_content = val.answer_content
                            }else{
                                let obj ={
                                    second_option_content: ''
                                }
                                tag.option_list.push(obj)
                                tag.option_list[tag.option_list.length-1].second_option_content = val.answer_content
                            } 
                        })
                    }

                    // 选择器选择增加绑定字段
                    if(tag.question_type == "SELECTOR_CHOICE"){
                        // 如果填过的没有内容
                        if(!item.answer_list[0]){
                            if(tag.choice_first){
                                tag.isChecked = true
                                tag.option_list[0]&&tag.option_list[0].option_content ? tag.selectTitle = tag.option_list[0].option_content : tag.selectTitle =''
                            }else{
                                tag.selectTitle = '请点击选择'
                            }
                        }else{
                            tag.isChecked = true
                            if(item.answer_list[0]&&item.answer_list[0].answer_code){
                                for(let val of tag.option_list){
                                    if(val.option_code == item.answer_list[0].answer_code){
                                        tag.selectTitle = val.first_option_content
                                    }
                                }
                            }
                        }
                    }

                    // 搜索填空
                    if(tag.question_type == 'SEARCH_FILL'){
                        item.answer_list.forEach((val,index) =>{ 
                            tag.option_list.forEach((k,i) =>{
                                if(val.answer_id == k.id) {
                                    tag.isChecked = true
                                    val.answer_content ? k.first_option_content = val.answer_content : k.first_option_content = ''
                                }
                            })
                        })
                    }

                    // 时间选择
                    if(tag.question_type == 'TIME_SELECT'){
                        item.answer_list.forEach((val,index) =>{ 
                            tag.option_list.forEach((k,i) =>{
                                if(val.answer_id == k.id) {
                                    tag.isChecked = true
                                    val.answer_content ? k.first_option_content = val.answer_content : k.first_option_content = ''
                                }
                            })
                        })
                    }
                    resetArr.push(tag)
                }
                let str = {}
                str.resetArr = resetArr
                str.isRecipe = isRecipe
                return str
            },
            // 构建 map 结构方便查询
            creaetLookMap(arr){
                let self = this
                let strMap = {}
                for(let k of arr) {
                    strMap[k.id] = k
                }
                return strMap
            },
            saveContrast(arr) {
                let self = this
                let question_order_list= []
                let val ={}
                for( let [ i, item ] of new Map( arr.map( ( item, i ) => [ i, item ] ) ) ) {
                    let data = {
                        answer_list: [],
                        order: i+1,
                        questionnaire_item_id: item.id,
                        super_list:[]
                    }
                    // 处方题
                    if(item.question_type == 'PRESCRIPTION') {
                        for(let k of item.option_list){
                            let obj = {}
                            if(k.option_content.drug_name){
                                obj.answer_code = ''
                                obj.answer_id = 0
                                obj.answer_content = JSON.stringify(k.option_content)
                                data.answer_list.push(obj)
                            }
                        }
                    }
                    // 选择题
                    if(item.question_type=="MULTIPLE_NORMAL_CHOICE" || item.question_type=="NORMAL_CHOICE" || item.question_type=="MASSIVE_CHOICE" || item.question_type=="MULTIPLE_MASSIVE_CHOICE"){
                        for(let k of item.option_list){
                            let obj = {}
                            if(k.isactive){
                                obj.answer_code = k.option_code
                                obj.answer_id = k.id
                                k.inputTitle ? obj.answer_content = k.inputTitle : obj.answer_content = ''
                                data.answer_list.push(obj)
                                k.score !=null || k.score !=undefined ? obj.score = k.score : '' 
                            }
                        }
                        item.super_list ? data.super_list = item.super_list : ''
                    }
                    // 填空题
                    if(item.question_type == 'FILL'){
                       for(let k of item.option_list){
                            let obj = {}
                            if(k.first_option_content){
                                obj.answer_code = k.option_code
                                obj.answer_id = k.id
                                k.first_option_content ? obj.answer_content = k.first_option_content : obj.answer_content = ''
                                data.answer_list.push(obj)
                            }
                        } 
                    }
                    // 图片选择
                    if(item.question_type == 'PICTURE_UPLOAD'){
                        for(let k of item.option_list){
                            let obj = {}
                            if(k.second_option_content){
                                obj.answer_code = ''
                                obj.answer_id = 0
                                 obj.answer_content = k.second_option_content
                                data.answer_list.push(obj)
                            }
                        }
                    }
                    // 选择器选择
                    if(item.question_type == 'SELECTOR_CHOICE'){
                        if(item.selectTitle){
                            for(let k of item.option_list){
                                let obj = {}
                                if(item.selectTitle == k.option_content){
                                    obj.answer_code = k.option_code
                                    obj.answer_id = k.id
                                    obj.answer_content = k.option_content
                                    data.answer_list.push(obj)
                                }
                            }
                        } 
                    }
                    // 搜索填空题
                    if(item.question_type == 'SEARCH_FILL'){
                         for(let k of item.option_list){
                            let obj = {}
                            if(k.first_option_content){
                                obj.answer_code = k.option_code
                                obj.answer_id = k.id
                                obj.answer_content = k.first_option_content
                                data.answer_list.push(obj)
                            }
                        } 
                    }
                    // 时间选择
                    if(item.question_type == 'TIME_SELECT'){
                        for(let k of item.option_list){
                            if(k.first_option_content){
                                let obj = {}
                                obj.answer_code = k.option_code
                                obj.answer_id = k.id
                                obj.answer_content = k.first_option_content
                                data.answer_list.push(obj)
                            }
                        }
                    }
                    question_order_list.push(data)
                }
                val.refill_item_list = self.refill_item_list
                val.question_order_list = question_order_list 
                return val
            }
        },
        components:{
            survey
        }
    }
</script>