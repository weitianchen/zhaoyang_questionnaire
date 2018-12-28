<template>
    <div class="search-drug">
        <mt-popup
        v-model="popupVisible"
        position="right">
            <div>
                <div class="info-header">
                    <span class="go-back" @click="chooseDrug()"></span>
                    <h2>选择药品</h2>
                    <p></p>
                </div>
                <div class="search-box">
                    <span>药品</span>
                    <div class="search-b">
                        <i class="search-icon"></i>
                        <i class="clear-icon" @click="clearKeyWord"></i>
                        <input type="text" placeholder="请输入药品名称" @input="search" v-model="keyword">
                    </div>
                </div>
                <scroller :height="height" v-on:down="loadMore" :show="load">
                    <div>
                        <div class="drug-item" v-for="(item,index) in list" :key="index">
                            <h2>{{item.drug_name}}</h2>
                            <div class="drug-list">
                                <span class="title">选择规格</span>
                                <ul class="ul">
                                    <li v-for="(spec,$k) in item.spec_img" :key="$k" @click="showDrugList(item,$k)">
                                        <div class="clearfix drug-info">
                                            <div class="lf">
                                                <span :class="{active:(drugName == item.drug_name && drugSpec == spec.spec) ||(drugName == item.drug_name && drugSpec=='其他规格'&&spec.spec == '不清楚规格') || (drugName == item.drug_name && drugSpec=='其他规格(不清楚规格)'&&spec.spec == '不清楚规格')}"></span>
                                            </div>
                                            <div class="rt">
                                               <img v-if="spec.img && spec.img == 'dimness'" src="../../../assets/images/dimness.png" alt="">
                                               <img v-if="spec.img && spec.img != 'dimness'" :src="spec.img" alt="">
                                               <img v-if="!spec.img" src="../../../assets/images/no-drug-img.png" alt=""> 
                                            </div>
                                        </div>
                                        <p>{{spec.spec}}</p>
                                    </li>
                                </ul>
                            </div>
                        </div>
                        <div v-if="list.length >4 &&over" style="text-align:center;line-height:40px;font-size:15px;">
                            已经到底了
                        </div>
                        <div  v-if="over" class="clearfix end-title">
                            <div class="lf title">
                                如果没有显示您想选的药品，可继续填写完整药名，并点击保存
                                <i class="pointer"></i>
                            </div>
                            <div class="lf button">
                                <button @click="chooseDrug('save')">保存</button>
                            </div>
                        </div>
                        
                    </div>
                </scroller>
                <div class="record-swiper" v-if="drugList.drug_name">
                    <swiper :options='drugOption'>
                        <div class="swiper-slide" v-for="(drug,index) in drugList.spec_img" :key="index">
                            <div class="zy-card">
                                <h3>是否选取该药品(规格)</h3>
                                <div class="card-body">
                                   <div class="img-box">
                                       <div>
                                            <img v-if="drug.img && drug.img == 'dimness'" src="../../../assets/images/dimness.png" alt="">
                                            <img v-if="drug.img && drug.img != 'dimness'" :src="drug.img" alt="">
                                            <img v-if="!drug.img" src="../../../assets/images/no-drug-img.png" alt=""> 
                                       </div>
                                   </div>
                                   <h3>{{drugList.drug_name}}</h3>
                                   <p>{{drug.spec}}</p>
                                   <button @click="chooseDrug(drugList,drug)">选取</button>
                                </div>
                                <div class="card-footer">
                                    <i class="close" @click="closeDrugOption">&times;</i>
                                </div>
                                
                            </div>
                        </div>
                    </swiper>
                </div>
            </div>
            <div class="drug-course" v-if="drugCourse">
                <img v-if="course_1" src="../../../assets/images/course/drug-course-1.jpg" alt="">
                <img v-if="course_2" src="../../../assets/images/course/drug-course-2.jpg" alt="">
                <img v-if="course_2" class="course-3" src="../../../assets/images/course/drug-course-3.jpg" alt="">
                <span class="next" @click="next()"></span>
            </div> 
        </mt-popup>
    </div>
</template>
<style lang="sass" scoped>
    .info-header{
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
    .search-box{
        height: 55px;
        padding: 10px;
        text-align: left;
        border-bottom: 2px solid #e2e2e2;
        .search-b{
            display: inline-block;
            vertical-align: middle;
            position: relative;
            width: calc(100% - 40px);
            width: -webkit-calc(100% - 40px);
            input{
                height: 35px;
                background-color: #f1f1f1;
                border: 1px  solid #c1cacf;
                border-radius: 5px;
                width: 100%;
                text-indent: 28px;
                color: #000;
                &:focus{
                    outline: none;
                }
            }
            .search-icon{
                width: 20px;
                height: 20px;
                display: inline-block;
                position: absolute;
                left: 5px;
                top: 8px;
                background: url("../../../assets/images/sprite3.png") no-repeat;
                background-size: 384px 384px;
                background-position: -71px -1px;
            }
            .clear-icon{
                width: 22px;
                height: 22px;
                display: inline-block;
                position: absolute;
                right: 5px;
                top: 8px;
                background: url("../../../assets/images/sprite3.png") no-repeat;
                background-size: 384px 384px;
                background-position: -1px -1px;
            }
        }
    }
    .drug-item{
        padding: 0.2rem;
        border-bottom: 1px solid #ededed;
        h2{
            font-size: 16px;
            font-weight: 500;
        }
        .drug-list{
            width: 5.8rem;
            position: relative;
            &:after{
                content: '';
                display: block;
                position: absolute;
                top: 0;
                right: 0;
                width: 30px;
                height: 100%;
                background: -webkit-linear-gradient(left,hsla(0,0%,71%,0),hsla(0,0%,93%,.9));
            }
            .title{
                width: 15px;
                display: inline-block;
                line-height: 16px;
                font-size: 14px;
                vertical-align: top;
            }
            .ul{
                display: inline-block;
                margin-left: 5px;
                overflow-x: auto;
                white-space: nowrap;
                width: calc(100% - 25px);
                width: -weblit-calc(100% - 25px);
                p{
                    text-align: center;
                    line-height: 25px;
                }
                li{
                    display: inline-block;
                    margin-left: 0.1rem;
                }
                .drug-info{
                    background-color:#eff3f5;
                    padding: 0.04rem;
                    height: 1rem;
                    .lf{
                        width: 20px;
                        height: 100%;
                        position: relative;
                        span{
                            width:15px;
                            height: 15px;
                            border: 2px solid #d1d2d2;
                            display: inline-block;
                            position: absolute;
                            top: 50%;
                            left: 2.5px;
                            margin-top: -8px;
                            &.active{
                                &:after{
                                    content:'';
                                    display: block;
                                    width: 17px;
                                    height: 14px;
                                    position: absolute;
                                    left: -2px;
                                    top: -2px;
                                    background:url("../../../assets/images/wq.png") no-repeat;
                                    background-size: 137px 33px;
                                    background-position: -72px -6px;
                                }
                            }
                        }
                    }
                    .rt{
                        width: 1.6rem;
                        height: 0.92rem;
                        background-color: #fff;
                        padding: 0.05rem;
                        img{
                            width: 100%;
                            height: 100%;
                        }
                    }
                }
            }
        }
    }
    .end-title{
        color: #000;
        padding: 0.25rem 0.3rem;
        .title{
            width: 4.6rem;
            line-height: 25px;
            font-size: 15px;
            font-weight: 500;
        }
        .pointer{
            width: 26px;
            height: 21px;
            display: inline-block;
            vertical-align: middle;
            background: url("../../../assets/images/sprite3.png") no-repeat;
            background-size: 384px 384px;
            background-position: -32px -1px;
        }
        .button{
            width: 1.2rem;
            button{
                height: 35px;
                width: 1rem;
                margin: 10px auto 0;
                display: block;
            }
        }
    }
    .record-swiper{
		position: fixed;
		top: 0;
		bottom: 0;
		left: 0;
		right: 0;
		background: rgba(0,0,0,0.5);
		z-index: 999;
		.swiper-container {
			height:100%;
			margin: 0 auto;
			position: relative;
			overflow: hidden;
			z-index: 1;
		}
		.swiper-wrapper{
			position: relative;
			width: 100%;
			height: 100%;
			z-index: 1;
			display: -webkit-box;
			display: -moz-box;
			display: -ms-flexbox;
			display: -webkit-flex;
			display: flex;
			-webkit-transition-property: -webkit-transform;
			-moz-transition-property: -moz-transform;
			-o-transition-property: -o-transform;
			-ms-transition-property: -ms-transform;
			transition-property: transform;
			-webkit-box-sizing: content-box;
			-moz-box-sizing: content-box;
			box-sizing: content-box;
			-webkit-transform: translate3d(0,0,0);
			-moz-transform: translate3d(0,0,0);
			-o-transform: translate(0,0);
			-ms-transform: translate3d(0,0,0);
			transform: translate3d(0,0,0);
		}
		.swiper-slide{
			display: -webkit-box;
			display: -ms-flexbox;
			display: flex;
			display: -webkit-flex;
			-webkit-box-align: center;
			-webkit-align-items: center;
			-ms-flex-align: center;
			align-items: center;
			-webkit-box-pack: center;
			-webkit-justify-content: center;
			-ms-flex-pack: center;
			justify-content: center;
		}
		
		.swiper-slide.swiper-slide-active .zy-card {
			-webkit-transform: scale(1);
		}
		.swiper-slide .zy-card {
			-webkit-transform: scale(.8);
		}
		.zy-card{
            position: relative;
			font-size: 14px;
			width: 110%;
			border-radius: 5px;
			overflow: hidden;
			text-align:left;
			-webkit-box-orient: vertical;
            >h3{
                font-size: 14px;
                color: #fff;
                text-align: center;
                line-height: 35px;
            }
            .img-box{
                border: 1px solid #ededed;
                border-radius: 2px;
                padding: 5px;
                height: 3rem;
                >div{
                    overflow: hidden;
                    height: calc(3rem - 12px);
                    height: -webkit-calc(3rem - 12px);
                    background-color: #f5f5f5;
                }
                img{
                    width: 100%;
                }
            }
            .card-body {
                -webkit-box-flex: 1;
                padding:5px;
                background-color: #fff;
                border-radius: 5px;
                text-align: center;
                h3{
                    font-size: 16px;
                    font-weight: 500;
                    color: #555;
                    line-height: 25px;
                    padding-top: 5px;
                }
                p{
                    font-size: 14px;
                    color: #777;
                    line-height: 18px;
                }
                button{
                    display: block;
                    width: 85px;
                    height: 25px;
                    box-shadow: none;
                    margin: 5px auto;
                }
            }
            .card-footer {
                height: 100px;
                position: relative;
                .close {
                    display: block;
                    position: absolute;
                    right: 50%;
                    top: 50%;
                    border: 2px solid #FFF;
                    border-radius: 50%;
                    line-height: 25px;
                    text-align: center;
                    text-indent: -2px;
                    width: 28px;
                    height: 28px;
                    color: #FFF;
                    font-size:25px;
                    margin-top: -14px;
                    margin-right: -14px;
                }
            }
        }
	}
    .search-drug{
        position: relative;
        .drug-course{
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            bottom: 0;
            overflow: hidden;
            background-color:#fff;
            img{
                width: 6.4rem;
                &.course-3{
                    position: absolute;
                    bottom: 0;
                    left: 0;
                }
            }
            .next{
                width: 1.8rem;
                height: 0.8rem;
                position: absolute;
                top:5.25rem;
                left: 2.25rem;
            }
        }
    }
</style>
<style lang="sass">
    .search-drug{
        text-align: left;
        .mint-popup-right{
            left: 0;
            bottom: 0;
            height: 100%;
        }  
    }
    .mint-indicator-wrapper{
        z-index: 3000;
    }
</style>
<script>
    import Cookies from 'js-cookie'
    import 'swiper/dist/css/swiper.css'
	import { swiper, swiperSlide } from 'vue-awesome-swiper'
    import scroller from '../scroll.vue'
    import Vue from 'vue'
    import { Popup,Indicator } from 'mint-ui'
    import {Ajax, Tips, topPopup, preTreated} from '../../../lib/common'
    Vue.component(Popup.name, Popup);
    export default{
        data(){
            return{
                keyword:'',
                popupVisible: true,
                height: document.body.clientHeight - 105 +'px',
                load: false,
                page: 1,
                last_page:0,
                over: false,
                list:[],
                timer: null,
                drugOption:{
					slidesPerView: 1.3,
                    centeredSlides: true,
                    slideToClickedSlide: true,
                    observer: true,
                    initialSlide:1,
                    onSlideChangeEnd: swiper =>{

                    }
                },
                drugList: {},
                no_spec_img: 'dimness',
                drugCourse: false, // 选择药品教程
                course_1: true,
                course_2: false
            }
        },
        props:{
            drugName:'',
            drugSpec:''
        },
        mounted() {
            let self = this
            self.drugName ? self.keyword = self.drugName : ''
            setTimeout(() => {
                if(!Cookies.get('drigCourse')){
                    Cookies.set('drigCourse','true',COPT)
                    self.drugCourse = true
                }else{
                    self.search_drug()
                }
            },500)
        },
        methods:{
            // 教程下一步
            next() {
                let self = this
                if(self.course_1){
                    self.course_1 = false
                    self.course_2 = true
                }else{
                    self.drugCourse = false
                    self.search_drug()
                }
            },
            // 清除文案
            clearKeyWord(){
                this.keyword = ''
                this.page = 1
                this.search_drug('clear')
            },
            // 选取药品
            chooseDrug(ev,item){
                let self = this
                let data = {}
                if(ev&&item){
                    data = {
                        name: ev.drug_name,
                        spec: item.spec
                    }
                }else if(ev){
                    if(self.keyword){
                        data = {
                            name: self.keyword,
                            spec: ''
                        }
                    }
                }
                self.$emit('selectDrug',data)
                
            },
            // 隐藏药品轮播图
            closeDrugOption(){
                let self = this
                self.drugOption.initialSlide = 1
                self.drugList = {}
            },
            // 显示药品大图
            showDrugList(item,index){
                let self=this
                self.drugOption.initialSlide = index
                self.drugList = item
            },
            // 加载药品
            loadMore(){
                if(this.page < this.last_page || this.page == this.last_page ){
                    this.load = true
                    this.search_drug();
                }else{
                    this.over = true
                }
            },
            search_drug(clear) {
                let self =this
                Indicator.open()
                Ajax('get','list/search-drug',{keyword:self.keyword,page: self.page}).then(data => {
                    Indicator.close()
                    preTreated(data, res => {
                        clear ? self.list = [] : ''
                        var data = res.data.data;
                        let obj ={
                            img: self.no_spec_img,
                            spec: '不清楚规格'
                        }
                        data.forEach(function(val,index){
                            val.spec_img.push(obj)
                            self.list.push(val)
                        })
                        self.list.length == res.data.total ? self.over = true : self.over = false
                        self.load = false
                        self.page ++;
                        self.last_page = res.data.last_page;
                    })
                }).catch(err => {
                    console.log(err)
                })
            },
            search() {
                let self = this
                clearTimeout(self.timer)
                self.timer = setTimeout(() =>{
                    self.page =1
                    self.search_drug('clear')
                },500)
            },
        },
        components:{
            scroller,
            swiper,
			swiperSlide,
        }
    }
</script>
