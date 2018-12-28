<template>
    <div class="doctor-advice">
        <p style="background-color: #ddd;font-size:14px;line-height:25px;text-align:center;" v-if="systemFinish">系统自动结束</p>

        <div v-if="systemFinish">
            <div class="title-box clearfix">
                <div class="lf serial-number"><span>1</span></div>
                <h2 class="l'f">
                    嘱咐：
                </h2>
            </div>
            <div class="content-box">
                坚持治疗，定期复诊
            </div>
        </div>
        <div v-else :class="{addPadding:showKefu}">
            <div v-if="!waitingAdvise && list.length" v-for="(item,$k) in list" :key="$k">
                <div class="title-box clearfix">
                    <div class="lf serial-number"><span>{{$k+1}}</span></div>
                    <h2 class="lf">
                        {{item.question_content}}：
                    </h2>
                </div>
                <div v-if="(item.question_type == 'DOCTOR_PRESCRIPTION' || item.question_type == 'DOCTOR_FOLLOW_UP') && item.option_list.length" class="take-medicine clearfix" style="margin-left: 0.7rem;">
                    <div class="lf medicine-info bt" v-for="(val,$j) in item.option_list" :key="$j">
                        <ul>
                            <li><span>药名：</span>{{val.option_content.drug_name}}{{val.option_content.spec&&val.option_content.spec != '-1'&&val.option_content.spec !='其他规格' ? '(' + val.option_content.spec + ')' : ''}}</li>
                            <li v-if="val.option_content.frequency && val.option_content.frequency !='每天'"><span>间隔：</span>{{val.option_content.frequency}}</li>
                            <li v-if="val.option_content.usage && val.option_content.usage!='口服'"><span>用法：</span>{{val.option_content.usage}}</li>
                            <li><span>用法用量：</span><i style="font-style: normal" v-html="instruction(val)"></i></li>
                            <li><span>用药天数：</span>{{val.option_content.take_medicine_days}}天</li>
                            <li v-if="val.option_content.remark"><span>备注：</span>{{val.option_content.remark}}</li>
                        </ul>
                    </div>
                </div>
                <!-- "嘱咐"-->
                <div v-if="item.question_type == 'DOCTOR_ADVICE' && item.option_list.length" class="content-box" v-for="(val,$j) in item.option_list" :key="$j" style="color: #339de1">
                    {{val.option_content}}
                </div>
                <!-- 建议下次预约 -->
                <div v-if="item.question_type == 'RETURN_VISIT' && item.option_list.length" class="take-medicine clearfix" style="margin-left: 0.7rem;">
                    <div class="lf serial-number"></div>
                    <div class="lf medicine-info" style="line-height:25px;color: #339de1" v-for="(val,$j) in item.option_list" :key="$j">
                        医生建议您预约{{val.option_content}}的{{val.first_option_content}}
                    </div>
                </div>

                <!-- 转诊 -->
                <div v-if="item.question_type == 'REFERRAL' && doctor.name" class="take-medicine clearfix">
                    <div class="lf serial-number"></div>
                    <div class="lf medicine-info">
                        <div class="doctor-detail" v-if="doctor.name">
                            <div class="doctor-info">
                                <router-link tag="a" :to="'/doctorDetail?id=' + doctor.id">
                                    <div class="img">
                                        <img v-if="doctor.avatar" :src="doctor.avatar" alt="">
                                        <img v-if="!doctor.avatar" src="../../../assets/images/male_doctor.png" alt="">
                                    </div>
                                    <div class="right">
                                        <h4>
                                            {{doctor.name}}
                                            <span :class="{'low-level':doctor.level_number == '1'}">
                                                {{doctor.level}}
                                            </span>
                                            
                                        </h4>
                                        <p class="star">
                                            <span>
                                                <i v-for='(i,$k) in Math.round(Number(doctor.point))' class="light" :key="$k"></i><i  v-for='(i,$j) in (5 - Math.round(Number(doctor.point)))' class="dark" :key="$j"></i>
                                            </span>
                                        </p>
                                        <div class="doctor-company">
                                            {{doctor.hospital_name}}  {{doctor.specialist}}  {{doctor.title}}
                                        </div>
                                    </div>
                                </router-link>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- 其他提醒事项 -->
                <div v-if="item.question_type == 'REMIND' && item.option_list.length">
                    <div class="take-medicine clearfix" style="margin-left: 0.7rem;">
                        <div class="lf serial-number"></div>
                        <div class="lf medicine-info">
                            <ul>
                                <li style="color: #339de1" v-for="(val,$j) in item.option_list" :key="$j">{{$j+1}}、请于{{val.option_content.split(':')[1]}}{{val.option_content.split(':')[0]}}</li>
                            </ul>
                        </div>
                    </div>
                    <p style="color:#999;margin: 20px 0;text-align: center">*提醒事项在提醒日期当天8:00以短信发给患者</p>
                </div>
            </div>
            <div v-if="waitingAdvise || (!waitingAdvise && !list.length)">
                <img style="width: 5rem;display: block;margin: 0.5rem auto 0;" src="../../../assets/images/advise.png" alt="">
            </div>
            <div class="kefu" v-if="showKefu">
                <img src="../../../assets/images/drug_icon.png" alt="">
                <span>
                    如需寄药，请点击联系
                    <i class='point'></i>
                </span>
                <router-link tag="a" to="/chatMedical">小助手</router-link>
            </div>
        </div>
    </div>
</template>
<script>
    import {Ajax, Tips, topPopup, preTreated} from '../../../lib/common'
    export default{
        data(){
            return{
                id: this.$route.query.id || '',
                questionnaire_item_list: [], // 医生建议题目
                question_order_list: [] , // 医生建议答案
                list: [],
                doctor: {}
            }
        },
        props:{
            systemFinish:{
                type: Boolean,
                default: function() {
                    return false
                }
            },
            waitingAdvise:{
                type: Boolean,
                default: function(){
                    return true
                }
            },
            showKefu:{
                type: Boolean,
                default: function() {
                    return false
                }
            }
        },
        mounted() {
            
        },
        methods:{
            getAdvice(){
                let self = this
                Ajax('get','java/doctor_record/get_questionnaire',{appointment_id: self.id,answerer_type: 'DOCTOR'}).then(data =>{
                    preTreated(data, res => {
                        let data = res.data
                        if(data.questionnaire_item_list.length && data.question_order_list.length){
                            self.questionnaire_item_list = self.creaetLookMap(data.questionnaire_item_list) 
                            self.list = self.resetAdvice(data.question_order_list,self.questionnaire_item_list)
                        }
                        console.log(data)
                    })
                }).catch(err => {
                    console.log(err)
                })
            },
            resetAdvice(arr,map){
                let self = this
                let resetArr = []
                // 循环建议
                for(let item of arr){
                    let tag = map[item.questionnaire_item_id]
                    // 处方建议
                    if(tag.question_type == 'DOCTOR_PRESCRIPTION' || tag.question_type == "DOCTOR_FOLLOW_UP") {
                        if(item.answer_list.length > tag.option_list.length){
                            for(let i = 0;i<item.answer_list.length;i++){
                                let obj ={
                                    option_content:{}
                                }
                                tag.option_list.push(obj)
                            }
                        }
                        item.answer_list.forEach((val,index) =>{
                            tag.option_list[index].option_content = JSON.parse(val.answer_content)
                        }) 
                    }
                    // "嘱咐"
                    if(tag.question_type == 'DOCTOR_ADVICE') {
                        item.answer_list.forEach((val,index) =>{
                            tag.option_list[index].option_content = val.answer_content
                        })
                    }

                    // "建议下次咨询类型"
                    if(tag.question_type == 'RETURN_VISIT') {
                        let a = []
                        item.answer_list.forEach((val,index) =>{
                            tag.option_list.forEach((k,i) =>{
                                if(val.answer_id == k.id){
                                    k.option_content = val.answer_content
                                    a.push(JSON.parse(JSON.stringify(k)))
                                }
                            })
                        })
                        tag.option_list = a
                    }

                    // "是否需要转诊"
                    if(tag.question_type == 'REFERRAL' && item.answer_list[0] && item.answer_list[0].answer_content) {
                        let id = item.answer_list[0].answer_content
                        self.getDoctor(id)
                    }

                    // "提醒事项"
                    if(tag.question_type == 'REMIND') {
                        if(item.answer_list.length > 0){
                            for(let i = 0;i<item.answer_list.length;i++){
                                let obj ={
                                    option_content:{}
                                }
                                tag.option_list.push(obj)
                            }
                        }
                        item.answer_list.forEach((val,index) =>{
                            tag.option_list[index].option_content = val.answer_content
                        })
                    }
                    resetArr.push(tag)
                }
                return resetArr
            },
            getDoctor(id) {
                Ajax('get','tool/doctorInfo/'+id,{}).then(data => {
                    preTreated(data,res => {
                        this.doctor = res.data
                    })
                }).catch(err => {
                    console.log(err)
                })
            },
            // 构建 map 
            creaetLookMap(arr){
                let self = this
                let strMap = {}
                for(let k of arr) {
                    strMap[k.id] = k
                }
                return strMap
            },
            // 过滤用法用量q
            instruction(res){
                let html = ''
                if(res.option_content){
                    let ev = res.option_content
                    if(res.option_content.titration.length >0) {
                        let items = res.option_content.titration
                        items.forEach(function(val,index){
                            if(index == items.length-1){
                                html += '</br>' + val.take_medicine_days + '～' + (Number(ev.take_medicine_days)) + '天:'
                                html += val.morning?'早' + val.morning + ev.drug_unit + ',':''
                                html += val.noon?'午' + val.noon + ev.drug_unit + ',':''
                                html += val.night?'晚' + val.night + ev.drug_unit + ',':''
                                html += val.before_sleep?'睡前' + val.before_sleep + ev.drug_unit :''
                                // ev.spec&&ev.spec != '-1'&&ev.spec !='其他规格' ? html+= '(' + ev.spec + ')' : ''
                                // ev.usage && ev.usage!='口服' ? html+= ' / '+ ev.usage : ''
                            }else{
                                html += '</br>' + val.take_medicine_days + '～' + (Number(items[index+1].take_medicine_days)-1) + '天'
                                html += val.morning?'早' + val.morning + ev.drug_unit + ',':''
                                html += val.noon?'午' + val.noon + ev.drug_unit + ',':''
                                html += val.night?'晚' + val.night + ev.drug_unit + ',':''
                                html += val.before_sleep?'睡前' + val.before_sleep + ev.drug_unit :''
                                // ev.spec&&ev.spec != '-1'&&ev.spec !='其他规格' ? html+= '(' + ev.spec + ')' : ''
                                // ev.usage && ev.usage!='口服' ? html+= ' / '+ ev.usage : ''
                            }
                        })
                    }else{
                        ev.morning ? html+= '早'+ ev.morning + ev.drug_unit +',': ''
                        ev.noon ? html+= '午' + ev.noon + ev.drug_unit +',' : ''
                        ev.night ? html+= '晚' + ev.night + ev.drug_unit +',' : ''
                        ev.before_sleep ? html+= '睡前' + ev.before_sleep + ev.drug_unit : ''
                        // ev.spec&&ev.spec != '-1'&&ev.spec !='其他规格' ? html+= '(' + ev.spec + ')' : ''
                        // ev.usage && ev.usage!='口服' ? html+= ' / '+ ev.usage : ''
                    }
                }
                return html
            }
        }
    }
</script>
<style lang="sass" scoped>
    .doctor-advice{
        text-align: left;
        .serial-number{
            width: 0.65rem;
            height: 100%;
            span{
                width: 0.48rem;
                height: 0.48rem;
                border-radius: 50%;
                line-height: 0.48rem;
                text-align: center;
                display: block;
                margin: 0 auto;
                background-color: #339de1;
                color: #fff;
            }
        }
        .medicine-info{
            width: 5.5rem;
            position: relative;
            padding: 0.1rem 0;
        }
        .bt:after{
            content: '';
            display: block;
            position: absolute;
            width: 6.4rem;
            height: 1px;
            background-color: #ededed;
            left: -0.7rem;
            bottom: 0;
        }
        h2{
            font-size: 15px;
            font-weight: 500;
        }
        .title-box{
            background-color: #f0f2f5;
            line-height: 0.75rem;
            .serial-number{
                margin: 0.12rem 0.1rem 0 0;
            }
        }
        .take-medicine{
            li{
                font-size: 14px;
                span{
                    color: #868686;
                }
            }
        }
        .content-box{
            width: 5.7rem;
            margin-left: 0.7rem;
            padding: 0.2rem;
            padding-left: 0;
        }
        .kefu{
            background-color: #43b7ff;
            position: fixed;
            bottom: 0;
            left: 0;
            height: 50px;
            width: 100%;
            padding-left: 15px;
            line-height: 50px;
            color: #fff;
            font-size: 16px;
            z-index: 1000;
            img{
                width: 30px;
                vertical-align: middle;
            }
            a{
                color: #fff;
                font-size: 15px;
                padding: 5px 15px;
                border: 1px solid #fff;
                box-shadow: 0 0 1px #fff;
                background-color: #43b7ff;
                border-radius: 3px;
            }
            .point {
				display: inline-block;
				vertical-align: middle;
				width: 19px;
				height: 14px;
				background: url('../../../assets/images/sprite2.png') no-repeat -295px -45px;
				background-size: 360px 1000px;
                margin-right: 3px;
			}
        }
        .addPadding{
            padding-bottom: 50px;
        }
    }
    /**医生信息 start**/ 
    .doctor-detail{
        padding: 0.15rem 0;
        .doctor-info{
            padding: 0.15rem;
            a,div.flex{
                display: -webkit-flex;
                display: flex;
            }
            .img{
                img{
                    width: 1.4rem;
                    height: 1.4rem;
                    border-radius: 50%;
                    display: block;
                    margin: 0.05rem 0 0 0.2rem;

                }
                
            }
            .right{
                padding-left: 9px;
                font-size: 14px;
                color: #000;
                font-weight: normal;
                -webkit-flex: 1;
                flex: 1;
                h4{
                    font-size: 15px;
                    margin-top: 0.1rem;
                    span{
                        display: inline-block;
                        border-radius: 0.12rem;
                        background: #f74c31;
                        color: #fff;
                        font-size: 0.24rem;
                        padding: 0 0.1rem;
                        margin-top: 0.1rem;
                        &.low-level{
                            background-color: #88cb5a;
                        }
                    }
                    
                }
                .star{
                    span{
                        i{
                            width: 12px;
                            height: 12px;
                            display: inline-block;
                            margin-top: -5px;
                        }
                        .light{
                            background: url('../../../assets/images/icons/star.png') no-repeat;
                            background-size: 100% 100%;
                            vertical-align: middle;
                        }
                        .dark{
                            background: url('../../../assets/images/icons/unstar.png') no-repeat;
                            background-size: 100% 100%;
                            vertical-align: middle;
                        }
                    }
                }
                .clinic-status{
                    color: #d7d7d7;
                    font-size: 14px;
                    margin: 0.05rem 0;
                    span{
                        background-color: #d7d7d7;
                        color: #fff;
                        padding: 1px 0.1rem;
                        border-radius: 0.1rem;
                        font-size: 14px;
                    }
                    &.have-opened{
                        color: #88cb5a;
                        span{
                            background-color: #88cb5a;
                        }
                    }
                }
                .doctor-company{
                    color: #868686;
                }
            }
            
        }
    }
    /**医生信息 end**/ 
</style>