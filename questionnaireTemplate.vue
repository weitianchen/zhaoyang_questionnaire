<template>
    <div class="questionnaire-box">
        <div class="header">
            <div class="info-header">
                <a class="go-back" @click="goback"></a>
                <h2>{{templateName}}</h2>
                <p @click="save" v-if="orderStatus">保存</p>
                <p v-else></p>
            </div>
        </div>
        <questionnaireT ref="saveQuestion" :list='list' :mapList="mapList" :initList='initList' @saveReturn='acceptItem' @contrast="contrastData" :orderStatus="orderStatus"></questionnaireT>
    </div>
</template>
<style lang="sass" scoped>
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
            z-index: 10;
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
    import {Ajax, Tips, topPopup, preTreated} from '../../lib/common'
    import questionnaireT from '../../components/public/questionnaire/questionModule'
    export default{
        data(){
            return{
                id: this.$route.query.id || '',
                templateName: this.$route.query.name || '',
                orderStatus: true,
                // id: 4457,
                scale_list: [], // 量表
                template_list: [], // 模板
                list: [], // 页面渲染数组
                mapList: {}, //以question_id 为key值方便关联题目查询
                initList: [], //初始化时的原数组
                //  保存问卷时的参数
                saveQuestionnaireArgument: {
                    new_question_order_list:[],
                    new_refill_item_list: [],
                    question_order_list:[],
                    questionnaire_id: 0,
                    refill_item_list: [],
                    submit: true
                },
                iscontrast: false , // 用户是否有操作过
                strList: '',
                strNewList: '',
                refill_item_list:[],// 初始化完成后保存结果，用来比较按返回键
                contrastList:[]
            } 
        },
        mounted() {
            if(!this.id){
                topPopup('订单id不存在')
                return
            }
           this.getTemplate()
        },
        methods:{
            
            goback() {
                let self = this
                self.iscontrast = false
                self.$refs.saveQuestion.goBackContrast()
                setTimeout(()=> {
                    self.iscontrast? self.contrastHint() : window.history.back()
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
            contrastData(str) {
                let self = this
                JSON.stringify(self.contrastList) != JSON.stringify(str) ? self.iscontrast = true : ''
            },
            save(){
                let self = this
                self.$refs.saveQuestion.save()
            },
            // 接收模板
            acceptItem(val){
                let self = this
                self.saveQuestionnaireArgument.question_order_list = val.question_order_list
                self.saveQuestionnaireArgument.refill_item_list = val.refill_item_list
                self.saveTemplate()

            },
            // 保存模板
            saveTemplate(){   
                let self = this
                Tips("正在保存...",'show')
                Ajax('post','java/doctor_template/questionnaire/save',self.saveQuestionnaireArgument).then(data => {
                    Tips("正在保存...",'hide')
                    preTreated(data,res => {
                        if(res.status){
                            topPopup('保存成功')
                            setTimeout(() => {
                                window.history.back()
                            },2000)
                        }
                        console.log(res)
                    })
                }).catch(err => {
                    console.log(err)
                })
            },
            // 获取问卷
            getTemplate(){
                let self = this
                Tips('','show')
                Ajax('get','java/doctor_template/questionnaire',{id: self.id}).then(data =>{
                    Tips('','hide')
                    preTreated(data,res => {
                        let data = res.data
                        let arr = []
                        // 保存问卷id
                        self.saveQuestionnaireArgument.questionnaire_id = res.data.questionnaire_id
                       
                        // 保存原始问卷
                        self.initList = self.addField(data.questionnaire_item_list)
                      
                        // 构建map结构,用于填写过问卷重组
                        self.mapList = self.creaetLookMap(self.initList)
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
                            self.list = self.resetQuestion(data.question_order_list,self.mapList)
                        }
                        self.strList = JSON.stringify(self.list)
                        self.contrastList = self.saveContrast(self.list)
                        self.$route.query.orderStatus == 'false' || self.$route.query.orderStatus == false ? self.orderStatus = false : self.orderStatus =true
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
                            item.option_list[0].option_content ? item.selectTitle = item.option_list[0].option_content : item.selectTitle = '请点击选择'
                        }else{
                            item.selectTitle = ''
                        } 
                    }
                    // 图片上传
                    if(item.question_type == 'PICTURE_UPLOAD'){
                        item.hideAdd = true
                    }
                    arr.push(item)
                }
                return arr
            },
            // 提取模板信息
            resetQuestion(arr,map){
                let resetArr = []
                // 循环问卷
                for(let item of arr){
                    // 获取问卷对应的原始数据
                    let tag = map[item.questionnaire_item_id]
                    // 处方题 
                    if(tag.question_type =="PRESCRIPTION"){
                        // 如果存在历史药单且大于原始的
                        if(item.answer_list.length > tag.option_list.length){
                            for(let i = 0;i<item.answer_list.length-tag.option_list.length;i++){
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
                                    console.log()
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
                        item.answer_list.forEach((val,index) =>{ 
                            tag.option_list.forEach((k,i) =>{
                                if(val.answer_id == k.id) {
                                    tag.isChecked = true
                                    val.answer_content ? k.second_option_content = val.answer_content : k.second_option_content = ''
                                }
                            })
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
                return resetArr
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
            questionnaireT
        }
    }
</script>