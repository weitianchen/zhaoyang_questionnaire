<!--
    注： 此模板通用 正常问卷、新增问卷、量表、模板
-->
<template>
    <div  class="question-box">
        <div class="question">
            <div v-for="(item,index) in list" :key="item.id">
                <div v-if="item.question_type == 'PRESCRIPTION'" class="item history-drug">
                    <h2>
                        <i :class="{active:!isNewQuestion,pitchOn:item.isChecked,remind: item.remind}">{{index +1}}</i>
                        <span>{{item.question_content}}</span>
                    </h2>
                    <div v-if="orderStatus" v-for="(recipe,$k) in item.option_list" :key="$k">
                        <div v-if="recipe.option_content.drug_name" class="take-medicine-history" @click="editDrug(recipe.option_content,index,$k)">
                            <span>服药记录{{$k+1}}</span>
                            <div class="detail">
                                <h3>{{recipe.option_content.drug_name}}{{recipe.option_content.spec&&recipe.option_content.spec != '-1'&&recipe.option_content.spec !='其他规格' ? '(' + recipe.option_content.spec + ')' : ''}}</h3>
                                <p v-html="instruction(recipe)"></p>
                                <p v-if="recipe.option_content.remark"><span>备注：</span>{{recipe.option_content.remark}}</p>
                            </div>
                            <i class="del-history" @click.stop="delHistory($k,index)">删除</i>
                        </div>
                        <input v-else type="text" onfocus="this.blur()" readonly='readonly' @click="showWriteRecipe(index)" :placeholder="'点击输入第'+ ($k+1) +'种历史服药情况'">
                    </div>
                    <!--订单已结束-->
                    <div  v-if="!orderStatus">
                        <div v-for="(recipe,$k) in item.option_list"  :key="$k" :class="{hideborder:!recipe.option_content.drug_name}"  class="finish-drug">
                            <div v-if="recipe.option_content.drug_name">
                                <p><i>药名：</i>{{recipe.option_content.drug_name}}{{recipe.option_content.spec&&recipe.option_content.spec != '-1'&&recipe.option_content.spec !='其他规格' ? '(' + recipe.option_content.spec + ')' : ''}}</p>
                                <p v-if="recipe.option_content.frequency !='每天'"><i>间隔：</i>{{recipe.option_content.frequency}}</p>
                                <p v-if="recipe.option_content.usage !='口服'"><i>用法：</i>{{recipe.option_content.usage}}</p>
                                <p><i>用法用量：</i><span v-html="instruction(recipe,true)"></span></p>
                                <p v-if="recipe.option_content.remark"><i>备注：</i>{{recipe.option_content.remark}}</p>
                            </div>
                        </div>
                        <div v-if="!item.isChecked" style="color:#999;width: 5.3rem;margin-left: 0.65rem;">
                            未填写
                        </div>
                    </div>

                    <a v-if="orderStatus" class="add" @click="showWriteRecipe(index)"><i class="icon"></i>添加更多服药记录</a>
                    <a v-if="orderStatus" class="to-lead" @click="lastRecipe(index)"><i class="icon"></i>导入医生上次用药</a>
                </div>
                <!--循环-->
                <div v-else class="item choice-question">
                    <h2>
                        <b v-if="isNewQuestion" :class="{remind: item.remind}">增</b><i :class="{active:!isNewQuestion,pitchOn:item.isChecked,remind: item.remind}">{{index +1}}</i>
                        <span :class="{active:isNewQuestion}">{{item.question_content}}</span>
                    </h2>
                    <ul class="answer-list">
                        <!--li 循环-->
                        <!--选择题  NORMAL_CHOICE  MULTIPLE_NORMAL_CHOICE-->
                        <li v-if="(item.question_type == 'NORMAL_CHOICE' || item.question_type == 'MULTIPLE_NORMAL_CHOICE') && orderStatus" v-for="(row,$index) in item.option_list" @click="choiceAnswer(item,index,row,$index)" :key="row.id">
                            <div class="chooser clearfix" :class="{active:row.isactive}">
                                <!-- 如果是选择题则出现大写字母 -->
                                <!--NORMAL_CHOICE,MASSIVE_CHOICE,MULTIPLE_NORMAL_CHOICE-->
                                <div class="lf charcode">
                                    {{row.option_code}}、
                                </div>

                                <div class="lf choice" v-if="row.option_type == 'TEXT'">
                                    {{row.first_option_content}}
                                </div>

                                <div class="lf choice" v-if="row.option_type == 'TEXT_INPUT'">
                                    {{row.first_option_content}}
                                    <input @click.stop="stopBubble(item,index,row,$index)" v-if="row.only_number" v-model="row.inputTitle" type="number" :placeholder="row.placeholder ? row.placeholder : ''" :maxlength="row.size ? row.size : 100">
                                    <input @click.stop="stopBubble(item,index,row,$index)" v-if="!row.only_number" v-model="row.inputTitle" type="text"  :placeholder="row.placeholder ? row.placeholder : ''" :maxlength="row.size ? row.size : 100">
                                </div>
                                
                                <div class="lf choice" v-if="row.option_type == 'INPUT_TEXT'">
                                    <input @click.stop="stopBubble(item,index,row,$index)" v-if="row.only_number" v-model="row.inputTitle" type="number" :placeholder="row.placeholder ? row.placeholder : ''" :maxlength="row.size ? row.size : 100">
                                    <input @click.stop="stopBubble(item,index,row,$index)" v-if="!row.only_number" v-model="row.inputTitle" type="text"  :placeholder="row.placeholder ? row.placeholder : ''" :maxlength="row.size ? row.size : 100"> {{row.first_option_content}}
                                </div>

                                <div class="lf choice" v-if="row.option_type == 'INPUT'">
                                    <input @click.stop="stopBubble(item,index,row,$index)" v-if="row.only_number" v-model="row.inputTitle" type="number" :placeholder="row.placeholder ? row.placeholder : ''" :maxlength="row.size ? row.size : 100" style="width: 5rem">
                                    <input @click.stop="stopBubble(item,index,row,$index)" v-if="!row.only_number" v-model="row.inputTitle" type="text"  :placeholder="row.placeholder ? row.placeholder : ''" :maxlength="row.size ? row.size : 100" style="width: 5rem">
                                </div>
                                
                                <div class="lf choice" v-if="row.option_type == 'TEXT_INPUT_TEXT'">
                                    {{row.first_option_content}}
                                    <input @click.stop="stopBubble(item,index,row,$index)" v-if="row.only_number" v-model="row.inputTitle" type="number" :placeholder="row.placeholder ? row.placeholder : ''" :maxlength="row.size ? row.size : 100">
                                    <input @click.stop="stopBubble(item,index,row,$index)" v-if="!row.only_number" v-model="row.inputTitle" type="text"  :placeholder="row.placeholder ? row.placeholder : ''" :maxlength="row.size ? row.size : 100">
                                    {{row.second_option_content}}
                                </div> 
                            </div>
                        </li> 

                        <!--订单已结束-->
                        <div v-if="(item.question_type == 'NORMAL_CHOICE' || item.question_type == 'MULTIPLE_NORMAL_CHOICE') && !orderStatus">

                            <li class="finish" v-if='item.isChecked' v-for="row in item.option_list" :key="row.id">
                                <div v-if="row.isactive" class="chooser clearfix" :class="{active:row.isactive}">
                                    <!-- 如果是选择题则出现大写字母 -->
                                    <!--NORMAL_CHOICE,MASSIVE_CHOICE,MULTIPLE_NORMAL_CHOICE-->
                                    <div class="lf charcode">
                                        {{row.option_code}}、
                                    </div>

                                    <div class="lf choice" v-if="row.option_type == 'TEXT'">
                                        {{row.first_option_content}}
                                    </div>

                                    <div class="lf choice" v-if="row.option_type == 'TEXT_INPUT'">
                                        {{row.first_option_content}} {{row.inputTitle}}
                                    </div>
                                    
                                    <div class="lf choice" v-if="row.option_type == 'INPUT_TEXT'">
                                        {{row.inputTitle}} {{row.first_option_content}}
                                    </div>

                                    <div class="lf choice" v-if="row.option_type == 'INPUT'">
                                        {{row.inputTitle}}
                                    </div>
                                    
                                    <div class="lf choice" v-if="row.option_type == 'TEXT_INPUT_TEXT'">
                                        {{row.first_option_content}}
                                        {{row.inputTitle}}
                                        {{row.second_option_content}}
                                    </div> 
                                </div>
                            </li>
                            <li v-if='!item.isChecked'>
                                <p>未填写</p>
                            </li>
                        </div>

                        <!--块状选择 MASSIVE_CHOICE  MULTIPLE_MASSIVE_CHOICE-->
                        <li v-if="(item.question_type == 'MASSIVE_CHOICE' || item.question_type == 'MULTIPLE_MASSIVE_CHOICE') && orderStatus">
                            <div class="massive clearfix">
                                <div v-for="(row,$index) in item.option_list" :key="row.id"  @click="choiceAnswer(item,index,row,$index)" :class="{active:row.isactive}">
                                    <div class="lf" v-if="row.option_type=='TEXT'">
                                        <span>{{row.first_option_content}}</span>
                                    </div>

                                    <div class="lf" v-if="row.option_type=='TEXT_INPUT'">
                                        <span>{{row.first_option_content}}</span>
                                        <input @click.stop="stopBubble(item,index,row,$index)" v-if="row.only_number" v-model="row.inputTitle" type="number" :placeholder="row.placeholder ? row.placeholder : ''" :maxlength="row.size ? row.size : 100" style="width: 4rem">
                                        <input @click.stop="stopBubble(item,index,row,$index)" v-if="!row.only_number" v-model="row.inputTitle" type="text"  :placeholder="row.placeholder ? row.placeholder : ''"  :maxlength="row.size ? row.size : 100" style="width: 4rem">
                                    </div>

                                    <div class="lf" v-if="row.option_type=='INPUT_TEXT'">
                                        <input @click.stop="stopBubble(item,index,row,$index)" v-if="row.only_number" v-model="row.inputTitle" type="number" :placeholder="row.placeholder ? row.placeholder : ''"  :maxlength="row.size ? row.size : 100" style="width: 4rem">
                                        <input @click.stop="stopBubble(item,index,row,$index)" v-if="!row.only_number" v-model="row.inputTitle" type="text"  :placeholder="row.placeholder ? row.placeholder : ''"  :maxlength="row.size ? row.size : 100" style="width: 4rem">
                                        <span>{{row.first_option_content}}</span>
                                    </div>

                                    <div class="lf" v-if="row.option_type=='INPUT'">
                                        <input @click.stop="stopBubble(item,index,row,$index)" v-if="row.only_number" v-model="row.inputTitle" type="number" :placeholder="row.placeholder ? row.placeholder : ''"  :maxlength="row.size ? row.size : 100" style="width: 5rem">
                                        <input @click.stop="stopBubble(item,index,row,$index)" v-if="!row.only_number" v-model="row.inputTitle" type="text"  :placeholder="row.placeholder ? row.placeholder : ''"  :maxlength="row.size ? row.size : 100" style="width: 5rem">
                                    </div>

                                    <div class="lf" v-if="row.option_type=='TEXT_INPUT_TEXT'">
                                        <span>{{row.first_option_content}}</span>
                                        <input @click.stop="stopBubble(item,index,row,$index)" v-if="row.only_number" v-model="row.inputTitle" type="number" :placeholder="row.placeholder ? row.placeholder : ''"  :maxlength="row.size ? row.size : 100" style="width: 3rem">
                                        <input @click.stop="stopBubble(item,index,row,$index)" v-if="!row.only_number" v-model="row.inputTitle" type="text"  :placeholder="row.placeholder ? row.placeholder : ''"  :maxlength="row.size ? row.size : 100" style="width: 3rem">
                                        <span>{{row.second_option_content}}</span>
                                    </div>
                                </div>
                            </div>
                        </li>

                        <div v-if="(item.question_type == 'MASSIVE_CHOICE' || item.question_type == 'MULTIPLE_MASSIVE_CHOICE') && !orderStatus">
                            <li class="finish" v-if='item.isChecked'>
                                <div class="massive clearfix" style="color:#339de1;line-height: 25px;">
                                    <div v-for="(row,$index) in item.option_list" :key="row.id" v-if="row.isactive" :class="{active:row.isactive}">
                                        <div class="lf" v-if="row.option_type=='TEXT'" style="margin-left: 3px;">
                                            {{row.first_option_content}} {{$index != item.option_list.length -1 ? "| " : ''}} 
                                        </div>

                                        <div class="lf" v-if="row.option_type=='TEXT_INPUT'" style="margin-left: 3px;">
                                            {{row.first_option_content}} {{row.inputTitle}} {{$index != item.option_list.length -1 ? "|" : ''}} 
                                            
                                        </div>

                                        <div class="lf" v-if="row.option_type=='INPUT_TEXT'" style="margin-left: 3px;">
                                           {{row.inputTitle}} {{row.first_option_content}} {{$index != item.option_list.length -1 ? "| " : ''}} 
                                        </div>

                                        <div class="lf" v-if="row.option_type=='INPUT'" style="margin-left: 3px;">
                                            {{row.inputTitle}} {{$index != item.option_list.length -1 ? "| " : ''}} 
                                        </div>

                                        <div class="lf" v-if="row.option_type=='TEXT_INPUT_TEXT'" style="margin-left: 3px;">
                                            {{row.first_option_content}} {{row.inputTitle}} {{row.second_option_content}} {{$index != item.option_list.length -1 ? "| " : ''}} 
                                        </div>
                                    </div>
                                </div>
                            </li>
                            <li v-if='!item.isChecked'>
                                <p>未填写</p>
                            </li>
                        </div>

                        <!-- 填空题 FILL-->
                        <li v-if="item.question_type == 'FILL' && orderStatus"  v-for="row in item.option_list" :key="row.id">
                            <textarea  v-if="row.only_number" v-model="row.first_option_content" class="fill" :maxlength="row.size ? row.size : 500" style="width: 5.2rem;height:1.5rem;"  :placeholder="row.placeholder ? row.placeholder : ''" @input="fillVal($event,item)" onkeyup="this.value=this.value.replace(/[^\r\n0-9]/g,'');"></textarea>

                            <textarea v-if="!row.only_number" v-model="row.first_option_content" class="fill" :maxlength="row.size ? row.size : 500" style="width: 5.2rem;height:1.5rem;"  :placeholder="row.placeholder ? row.placeholder : ''" @input="fillVal($event,item)"></textarea>
                            <span style="float:right">{{row.first_option_content.length}}/{{row.size ? row.size : 500}}字</span>
                        </li>
                        <div v-if="item.question_type == 'FILL' && !orderStatus">
                            <li class="finish" v-if="item.isChecked">
                                <div v-for="row in item.option_list" :key="row.id" style="color:#339de1;line-height: 25px;">{{row.first_option_content}}</div>
                            </li>
                            <li v-if="!item.isChecked">
                                <p>未填写</p>
                            </li>
                        </div>

                        <!-- 图片选择 PICTURE_UPLOAD-->
                        <li v-if="item.question_type == 'PICTURE_UPLOAD' && orderStatus">
                            <div class="imglink" v-for="(row,$j) in item.option_list" :key="$j" v-if="row.second_option_content">
                                <img @click="showFullImg(row.second_option_content)" :src="row.second_option_content" alt="">
                                <i class="closeImg" @click="delImg(item,index,row,$j)"></i>
                            </div>
                            <label :for="dynamic? 'compressfile'+dynamic : 'compressfile'" v-if="item.hideAdd">
                                <span class="photo" @click="addPhoto(item,index)"></span>
                            </label>
                        </li>
                        <div v-if="item.question_type == 'PICTURE_UPLOAD' && !orderStatus">
                            <li class="finish" v-if="item.isChecked">
                                <div class="imglink" v-for="(row,$j) in item.option_list" :key="$j" >
                                    <img @click="showFullImg(row.second_option_content)" :src="row.second_option_content" alt="">
                                </div>
                            </li>
                            <li v-if="!item.isChecked">
                                <p>未填写</p>
                            </li>
                        </div>
                        
                        <!-- 选择器选择 SELECTOR_CHOICE-->
                        <li v-if="item.question_type == 'SELECTOR_CHOICE' && orderStatus">
                            <select class="select-box" v-model="item.selectTitle" @change="selectVal($event,item)">
                                <option>请点击选择</option>
                                <option v-for="(row,$index) in item.option_list" :key="$index" :value="row.option_content" >{{row.option_content}}</option>
                            </select>
                        </li>
                        <div v-if="item.question_type == 'SELECTOR_CHOICE' && !orderStatus">
                            <div v-if="item.isChecked" style="color:#339de1;line-height: 25px;">{{item.selectTitle}}</div>
                            <div v-if="!item.isChecked">
                                <p>未填写</p>
                            </div>
                        </div>

                        <!-- 搜索填空题 SEARCH_FILL-->
                        <li v-if="item.question_type == 'SEARCH_FILL' && orderStatus">
                            <input  v-if="row.only_number" v-for="row in item.option_list" v-model="row.first_option_content" class="fill" :maxlength="row.size ? row.size : 100" style="width: 5.2rem" type="number" :placeholder="row.placeholder ? row.placeholder : ''" :key="row.id" onfocus="this.blur()" readonly='readonly' @click="showSearchBox(row,item)">
                            <input v-if="!row.only_number" v-for="row in item.option_list" v-model="row.first_option_content" class="fill" :maxlength="row.size ? row.size : 100" style="width: 5.2rem" type="text"  :placeholder="row.placeholder ? row.placeholder : ''" :key="row.id" onfocus="this.blur()" readonly='readonly' @click="showSearchBox(row,item)">
                        </li>
                        <div v-if="item.question_type == 'SEARCH_FILL' && !orderStatus">
                            <li v-if="item.isChecked">
                                <div v-for="row in item.option_list" :key="row.id" style="color:#339de1;line-height: 25px;">{{row.first_option_content}}</div>
                            </li>
                            <li v-if="!item.isChecked">
                                <div>未填写</div>
                            </li>
                        </div>

                        <!--时间选择器 TIME_SELECT-->
                        <li class="time-select" v-if="item.question_type == 'TIME_SELECT' && orderStatus" v-for="(row,$index) in item.option_list" :key="row.id">
                            <!--年月日-->
                            <div class="nyr" v-if="row.option_type == 'DATE'" @click="openPickerDate(row,item)">
                                <input v-model="row.first_option_content" readonly="readonly" onfocus="this.blur()" class="fill" type="text" placeholder="年-月-日">
                            </div>
                            <!--年月日时分-->
                            <div class="sf" v-if="row.option_type == 'DATETIME'">
                                <input v-model="row.first_option_content" readonly="readonly" onfocus="this.blur()" class="fill" type="text" placeholder="年-月-日 时:分" @click="openPickerDateTime(row,item)">
                            </div>
                            <!--年月日-年月日-->
                            <div class="nyr-nyr" v-if="row.option_type == 'DATE_RANAGE'" @click="openPickerDate(row,item)">
                                <input v-model="row.first_option_content" readonly="readonly" onfocus="this.blur()" class="fill" type="text" placeholder="年-月-日">
                                {{item.option_list.length > 1 && $index ==0 ? "至" : ''}}
                            </div>
                            <!--年月日时分-年月日时分-->
                            <div class="sf-sf" v-if="row.option_type == 'DATETIME_RANAGE'">
                                <input v-model="row.first_option_content" readonly="readonly" onfocus="this.blur()" class="fill" type="text" placeholder="年-月-日 时:分" @click="openPickerDateTime(row,item)">
                                {{item.option_list.length > 1 && $index ==0 ? "至" : ''}}
                            </div>
                        </li>
                        <div v-if="item.question_type == 'TIME_SELECT' && !orderStatus">
                            <li v-if="item.isChecked" class="time-select" v-for="(row,$index) in item.option_list" :key="row.id" style="color:#339de1;line-height: 25px;">
                                <!--年月日-->
                                <div class="nyr" v-if="row.option_type == 'DATE'">
                                    {{row.first_option_content}}
                                </div>
                                <!--年月日时分-->
                                <div class="sf" v-if="row.option_type == 'DATETIME'">
                                    {{row.first_option_content}}
                                </div>
                                <!--年月日-年月日-->
                                <div class="nyr-nyr" v-if="row.option_type == 'DATE_RANAGE'">
                                    {{row.first_option_content}}
                                    {{item.option_list.length > 1 && $index ==0 ? "至" : ''}}
                                </div>
                                <!--年月日时分-年月日时分-->
                                <div class="sf-sf" v-if="row.option_type == 'DATETIME_RANAGE'">
                                    {{row.first_option_content}}
                                    {{item.option_list.length > 1 && $index ==0 ? "至" : ''}}
                                </div>
                            </li>
                            <li v-if="!item.isChecked">
                                <p>未填写</p>
                            </li>
                        </div>
                    </ul>
                </div>
            </div>
        </div>
        <!-- 搜索填空 -->
        <mt-popup
        v-model="searchVisible"
        :closeOnClickModal="falseModle"
        position="bottom" class="search-h" id="search-h">
            <div class="searchlist-box" v-if="searchVisible">  
                <div class="searchlist-list">
                    <div style="background-color: #ededed;">
                        <span class="close" @click="closeDialog">&times;</span>
                        <div class="title" >{{searchTitle.title}}</div>
                        <div class='search-box'>
                            <input ref="search" type="text" v-model="searchTitle.name" @input="searchValue('clear')" placeholder="输入关键词">
                            <button @click="choseSearchItem()">确定</button>
                            <span class="ui-icon-search"></span>
                        </div>
                    </div>
                    <scroller :height="searchTitle.height" v-on:down="loadMore" :show="searchTitle.load">
                        <ul class="searchlist" style="overflow: auto;">
                            <li v-for="item in searchlist" :key="item.id" @click="choseSearchItem(item.value)">{{item.value}}</li>
                            <li v-if="searchlist.length == 0">
                                暂无数据
                            </li>
                            <div v-if="searchlist.length >0&&searchTitle.over" style="text-align:center;line-height:40px;font-size:15px;">
                                已经到底了
                            </div>
                        </ul>
                    </scroller>
                </div>
            </div>
        </mt-popup>
        
        <!-- 时间选择 -->
        <mt-datetime-picker
            ref="pickerDate"
            type="date"
            year-format="{value} 年"
            month-format="{value} 月"
            date-format="{value} 日"
            @confirm="handleChangeDate"
            @cancel='cancelChangeDate'
            v-model="pickerDateValue"
            :startDate="startDate"
            :endDate="endDate"
            >
        </mt-datetime-picker>
        <div class="datetime">
            <mt-datetime-picker
                ref="pickerDateTime"
                type="datetime"
                year-format="{value} 年"
                month-format="{value} 月"
                date-format="{value} 日"
                @confirm="handleChangeDateTime"
                @cancel='cancelChangeDateTime'
                v-model="pickerDateTimeValue"
                :startDate="startDate"
                :endDate="endDate"
                >
            </mt-datetime-picker>
        </div>
        <!-- 上传压缩图片 -->
        <compressImg v-if="showcompressImg" @unloadurl='unloadurl' :dynamic='dynamic'></compressImg>
        <!-- 导入药品 -->
        <mt-popup
        v-model="showRecipe"
        closeOnClickModal="false"
        position="bottom"
        class="recipe-popup"
        popup-transition="popup-fade"
        lockScroll="true"
        >
        <writeRecipe v-if="showRecipe" ref="wRecipe" @saveRecipe="saveRecipe" :drug_info="drug_info"></writeRecipe>
        </mt-popup>
        <!-- 导入医生上次用药 -->
        <mt-popup
            v-model="addLastRecipe"
            popup-transition="popup-fade"
            class="last-popup"
        >
            <div class="last-box" v-if="lastList.length !=0">
                <h2>请选择需要使用的处方</h2>
                <ul>
                    <li v-for="(item,$k) in lastList" :key="$k" @click="saveDoctorRecipe(item)">
                        <h3>
                            {{item.created_at.split(' ')[0]}}
                            <span>【开药医生:{{item.name}}】</span>
                        </h3>
                    </li>
                </ul>
            </div>
            <div v-else class="last-box" style="text-align: center;">
                暂无上次用药记录
            </div>
        </mt-popup>

        <div class="chat-fullscreen" :class="{active:showImg}" @click.stop="hideImg()">
			<div class="chat-mask"></div>
			<div class="chat-image">
				<img :src="fullImg" alt="">
			</div>
		</div>
    </div>
</template>
<style lang="sass" scoped>
    .finish-drug{
        position:relative;
        margin-bottom: 10px;
        &:last-child{
            margin-bottom: 0;  
            &:after{
                content: '';
                display: block;
                width: 0;
            }
        }
        &.hideborder{
            &:after{
                content: '';
                display: block;
                width: 0;
                height: 0;
            }
        }
        &:after{
            content: '';
            display: block;
            width: calc(100% + 0.6rem);
            width: -webkit-calc(100% + 0.6rem);
            height: 1px;
            background-color: #ededed;
            position: absolute;
            left: -0.3rem;
            bottom: -5px;
        }
    }
    input::-webkit-input-placeholder{
        font-size: 13px;
    }
    input:-moz-input-placeholder{
        font-size: 13px;
    }
    input::-moz-input-placeholder{
        font-size: 13px;
    }
    .chat-fullscreen, .chat-image img {
		top: 50%;
		left: 50%;
		transform: translate(-50%,-50%);
	}
	.chat-fullscreen {
		position: fixed;
		height: 100%;
		width: 100%;
		overflow: hidden;
		visibility: hidden;
		z-index: 1000;
		&.active {
		    visibility: visible;
            .chat-mask {
                opacity: 0.7;
            }
            .chat-image {
                transform: scale3d(1, 1, 1);
            }
		}
		.chat-mask, .chat-image {
            position: absolute;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            transition: all 0.5s;
		}
		.chat-mask {
            left: 0;
            top: 0;
            background-color: #000;
            opacity: 0;
		}
		.chat-image {
		    transform: scale3d(0, 0, 0);
            img {
                display: block;
                position: absolute;
                width: 100%;
                height: auto;
		    }
		}
  	}
</style>

<script>
    import Vue from 'vue'
    import utils from '../../../lib/utils'
    import scroller from '../scroll'
    import compressImg from '../compressImg'
    import writeRecipe from './writeRecipe'
    import moment from 'moment'
    import { Popup, DatetimePicker,Picker } from 'mint-ui';
    import $ from 'jquery'
    Vue.component(DatetimePicker.name, DatetimePicker);
    Vue.component(Popup.name, Popup);
    import {Ajax, Tips, topPopup, preTreated} from '../../../lib/common'
    export default{
        data(){
            return{
                id: this.$route.query.id || '',
                refill_item_list:[], // 记录题目是有重填
                // 搜索列表数据
                searchVisible: false,
                searchlist: [],
                searchTitle: {
                    title: '',
                    name: '',
                    row:{},
                    search_key_id:0,
                    page: 1,
                    size: 20,
                    timer: null,
                    empty:false,
                    load:false,//滑动加载
                    last_page:0,
                    height: (document.body.clientHeight*0.8) - 85 + 'px',
                    over: false,
                    item:{}
                },
                // 时间选择器
                // 年月日
                pickerDateValue: '',
                // 年月日时分
                pickerDateTimeValue: '',
                falseModle: false,
                startDate: new Date('2000-01-01'),
                endDate: new Date(),
                dateValue:{
                    row:{},
                    item:{}
                },
                dateTimeValue:{
                    row:{},
                    item:{}
                },
                imgItem:{
                    item:{},
                    row:{},
                    index: 0
                },
                // 控制编辑用药
                showRecipe: true,
                // 保存处方药出现的位置
                recipeIndex: 0,
                editRecipeIndex: 0, //保存修改药品的位置
                // 是否继续添加导入药品输入框
                addRecipe: false,
                // 导入医生上次用药
                addLastRecipe: false,
                // 上次用药列表
                lastList:[],
                showcompressImg: false,
                drug_info: {}, // 修改用药
                offsetHeight: 0, // 填写搜索填空时记录滚动条高度
                delRelatedProblem: [], //保存已经关联的问题
                //放大图片
                showImg: false,
                fullImg: '',
                // 阻止默认事件
                handler: function(e) {e.preventDefault();},
                falsehandler: false
            }
        },
        props:{
            isRecipe:{
                type: Number,
                default: function() {
                    return 0
                }
            },
            list:{
                type: Array,
                default:function (){
                    return []
                }
            },
            mapList:{
                type: Object,
                default:function (){
                    return {}
                }
            },
            initList:{
                type: Array,
                default: function (){
                    return []
                }
            },
            isNewQuestion: false,
            dynamic: '',
            orderStatus: {
                type: Boolean,
                default: function() {
                    return true
                }
            },
        },
        mounted() {
            let self = this
            // 监控药品弹窗，移除阻止默认事件
            let btn = document.getElementById("question-box")
            if(btn){
                btn.addEventListener('click',function(e){
                    if(e.target.className == 'v-modal' && self.falsehandler){
                    self.openTouch()
                    }
                })
            }
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
            // 放大
            showFullImg(val) {
                let self = this
                self.fullImg = val
                self.showImg = true
            },
            //隐藏大图片
			hideImg(){
				this.showImg=false
			},
            //  返回时对比数据
            goBackContrast(){
                let self= this
                let val= self.save('contrast')
                self.$emit('contrast',val)
            },
            save(contrast){
                let self = this
                let question_order_list= []
                let arr = self.list
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
                if(contrast){
                    return val
                } else {
                    self.$emit('saveReturn',val)
                }
            },
            // 删除图片
            delImg(item,index,row,$j){
                let self = this
                item.option_list.splice($j,1)
                self.$set(item,index,row)
                if(item.option_list.length == 0){
                    let obj ={
                            second_option_content: ''
                        }
                    item.option_list.push(obj)
                }
                item.imgSize == item.option_list.length ? self.$set(tiem,'hideAdd',false) : self.$set(item,'hideAdd',true)
                let checkedArr = []
                item.option_list.forEach( (tag,k) =>{
                    tag.second_option_content ? checkedArr.push(k) : ''
                })
                checkedArr.length ==0 ? self.$set(item,'isChecked',false) : self.$set(item,'isChecked',true)
                // 记录用户是否有重填
                if(item.remind){
                    self.refill_item_list.push(item.id)
                    item.remind = false
                }
            },
            // 保存点击位置
            addPhoto(item,index){
                let self = this
                self.imgItem={
                    index: 0
                }
                self.showcompressImg = true
                self.imgItem.index= index
                // 记录用户是否有重填
                if(item.remind){
                    self.refill_item_list.push(item.id)
                    item.remind = false
                }
            },
            // 图片上传
            unloadurl(ev){
                let self = this
                let tag = self.list[self.imgItem.index]
                tag.isChecked = true
                if(tag.imgSize >= tag.option_list.length){ 
                    if(tag.option_list&&tag.option_list[0].second_option_content) {
                        let obj ={
                            second_option_content: ev
                        }
                        tag.option_list.push(obj)
                    }else{
                        self.$set(tag.option_list[0],'second_option_content',ev)
                    }
                    tag.imgSize == tag.option_list.length ? self.$set(tag,'hideAdd',false) : self.$set(tag,'hideAdd',true)
                }
            },
            // 时间选择 年月日时分
            openPickerDateTime(e,item) {
                let self = this
                self.dateTimeValue.row = e
                self.dateTimeValue.item = item
                self.pickerDateTimeValue = e.first_option_content ? e.first_option_content : new Date()
                self.$refs.pickerDateTime.open();
                self.closeTouch()
                self.falsehandler = true
                // 记录用户是否有重填
                if(item.remind){
                    self.refill_item_list.push(item.id)
                    item.remind = false
                }
            },
            handleChangeDateTime(date){
                let self = this
                self.openTouch()
                self.falsehandler = false
                self.dateTimeValue.row.first_option_content = moment(date).format('YYYY-MM-DD hh:mm')
                self.dateTimeValue.item.isChecked = true
            },
            // 取消选择日期
            cancelChangeDateTime(data) {
                let self = this
                self.openTouch()
                self.falsehandler = false
                self.dateTimeValue.row.first_option_content = ''
                let item = self.dateTimeValue.item
                let checkedArr = []
                item.option_list.forEach( (tag,k) =>{
                    tag.first_option_content ? checkedArr.push(k) : ''
                })
                checkedArr.length ==0 ? self.$set(item,'isChecked',false) : self.$set(item,'isChecked',true)
            },
            // 时间选择 年月日
            openPickerDate(e,item) {
                let self = this
                self.dateValue.row = e
                self.dateValue.item = item
                self.pickerDateValue = e.first_option_content ? e.first_option_content : new Date()
                self.$refs.pickerDate.open();
                self.closeTouch()
                self.falsehandler = true
                // 记录用户是否有重填
                if(item.remind){
                    self.refill_item_list.push(item.id)
                    item.remind = false
                }
            },
            // 选择日期
            handleChangeDate(date){
                let self = this
                self.openTouch()
                self.falsehandler = false
                self.dateValue.row.first_option_content = moment(date).format('YYYY-MM-DD')
                self.dateValue.item.isChecked = true
            },
            // 取消选择日期
            cancelChangeDate(data) {
                let self = this
                self.openTouch()
                self.falsehandler = false
                self.dateValue.row.first_option_content = ''
                let item = self.dateValue.item
                let checkedArr = []
                item.option_list.forEach( (tag,k) =>{
                    tag.first_option_content ? checkedArr.push(k) : ''
                })
                checkedArr.length ==0 ? self.$set(item,'isChecked',false) : self.$set(item,'isChecked',true)
            },
            // 加载更多
            loadMore (){
                let self = this
                let d = self.searchTitle
                if(d.page < d.last_page || d.page == d.last_page ){
                    d.load = true
                    self.searchValue();
                }else{
                    d.over = true
                }
            },
            // 选中 列表
            choseSearchItem(val){
                let self = this
                val ? self.searchTitle.row.first_option_content = val : self.searchTitle.row.first_option_content =   self.searchTitle.name
                self.closeDialog()
            },
            // 显示搜索列表，并保存参数
            showSearchBox(e,item){
                let self = this
                self.searchTitle.title = e.placeholder
                self.searchTitle.row = e
                self.searchTitle.item = item
                self.searchTitle.search_key_id = e.search_key_id
                self.searchTitle.name = e.first_option_content
                self.closeDialog()
                self.searchValue()
            },
            // 模糊搜索列表
            searchValue(clear) {
                let self = this
                let d = self.searchTitle
                clear ? d.page = 1 : ''
                let search = {
                    search_key_id: d.search_key_id,
                    search_value: d.name,
                    page: d.page,
                    size: d.size
                }
                d.timer ? clearTimeout(d.timer) : ''
                d.timer = setTimeout(() => {
                    Ajax('get','java/question_bank/search/search_value',search).then(data => {
                        preTreated(data, res =>{
                            clear ? self.searchlist = [] : ''
                            d.page++
                            d.last_page = res.data.last_page
                            var data = res.data.list
                            data.forEach(function(val,index){
                                self.searchlist.push(val)
                            })
                            d.load = false;
                            })
                    }).catch(err => {
                        console.log(err)
                    })
                },300)
                
            },
            forSearchVal() {
                let self = this
                let item = self.searchTitle.item
                let checkedArr = []
                item.option_list.forEach( (tag,k) =>{
                    tag.first_option_content ? checkedArr.push(k) : ''
                })
                checkedArr.length ==0 ? self.$set(item,'isChecked',false) : self.$set(item,'isChecked',true)
            },
            // 关闭搜索列表弹窗
            closeDialog(){
                let self = this
                self.searchVisible = !self.searchVisible
                if(!self.searchVisible) {
                    self.searchlist = []
                    self.forSearchVal()
                    setTimeout(()=>{
                        self.searchTitle= {
                            title: '',
                            name: '',
                            row:{},
                            search_key_id:0,
                            page: 1,
                            size: 20,
                            timer: null,
                            empty:false,
                            load:false,//滑动加载
                            last_page:0,
                            height: (document.body.clientHeight*0.8) - 85 + 'px',
                            over: false,
                            item:{}
                        }
                    },200)
                    
                }
            },
            // 选择题输入框点击阻止冒泡
            stopBubble(item,index,row,$index) {
                let self = this
                if(row.isactive) {
                    return
                }
                // 当前选项如果没选中
                self.choiceAnswer(item,index,row,$index)
            },
            choiceAnswer(item,index,row,$index) {
                let self = this
                let pointTo = self.list[index].option_list
                // 当前答案是否有选中
                if(row.isactive) {
                    // 查询是否有关联，有取消
                    row.next_question_list&&row.next_question_list.length > 0 ? self.handleTopic(index,false,row.next_question_list,row.id) :  ''
                    // 取消选中
                    self.$set(pointTo[$index],'isactive',false) 
                    self.$set(pointTo, $index, pointTo[$index])
                }else{
                    // 记录已选多少个
                    let checkedNum = []
                    // 判断该答案是否互斥 注：单选题全部互斥 多选题有可能互斥
                    if(row.mutex) {
                        // 如果互斥取消所有
                        item.option_list.forEach( (tag,k) =>{
                            // 查找不是点击当前的答案，且有选中的，则取消选中状态，
                            if(k != $index) {
                                // 如果有关联题目
                                tag.isactive&&tag.next_question_list&&tag.next_question_list.length > 0 ? self.handleTopic(index,false,tag.next_question_list,row.id) : ''
                                self.$set(pointTo[k],'isactive',false)
                                self.$set(pointTo, k, pointTo[k])
                            }
                        })
                    }else{
                        // 如果没有互斥 循环所有答案查看问题是否有选中互斥的答案，如果有则取消
                         item.option_list.forEach( (tag,k) =>{
                            if(tag.isactive && tag.mutex) {
                                self.$set(pointTo[k],'isactive',false)
                                self.$set(pointTo, k, pointTo[k])
                                tag.next_question_list&&tag.next_question_list.length > 0 ? self.handleTopic(index,false,tag.next_question_list) : ''
                            }
                            tag.isactive ? checkedNum.push(k) : ''
                        })
                    }
                    // 检查多项选择是否是不定项选择
                    if(checkedNum.length < item.answer_size) {
                        // 查询是否有关联，有取消
                            row.next_question_list&&row.next_question_list.length > 0 ? self.handleTopic(index,true,row.next_question_list,row.id) :  ''
                        // 最后选中点击当前的答案
                        self.$set(pointTo[$index],'isactive',true) 
                        self.$set(pointTo, $index, pointTo[$index])
                    }else{
                        topPopup('此问题最多可选'+item.answer_size+ '个')
                    }
                    
                }  

                // 重新遍历是否有选中
                let checkedArr = []
                item.option_list.forEach( (tag,k) =>{
                    tag.isactive ? checkedArr.push(k) : ''
                })
                checkedArr.length ==0 ? self.$set(item,'isChecked',false) : self.$set(item,'isChecked',true)
                // 记录用户是否有重填
                if(item.remind){
                    self.refill_item_list.push(item.id)
                    item.remind = false
                }
            },
            // 插入或移除关联题目
            handleTopic(index,status,qList,id) {
                let self = this
                if(!status){
                    // 构建map
                    let map = self.creaetListMap()
                    // 移除关联题目
                    self.ruleRemoveQuestions(qList,map)
                }else{
                    self.ruleAddQuestions(qList,index,id)
                }
            },
            // 遍历添加问题
            ruleAddQuestions(qList,index,id) {
                let self = this
                let super_list = []
                for(let q of qList){
                    // 插入
                    for(let val of self.initList) {
                        if(q == val.question_id && val.order == -1){
                            let super_list = []
                            // 保存关联此题目的答案id
                            super_list.push(id)
                            val.super_list = super_list
                            self.list.splice(index+1,0,val)
                            let i = index+1
                            for(let k of val.option_list){
                                if(k.isactive&&k.next_question_list&&k.next_question_list.length>0){
                                    self.ruleAddQuestions(k.next_question_list,i,k.id)
                                }
                            }

                        } 
                    }
                }
            },
            // 遍历移除问题
            ruleRemoveQuestions(qList,map) {
                let self = this
                qList.forEach((q,i) =>{
                    let tag = {}
                    self.list.forEach((name,index) =>{
                        if(q == name.question_id && name.order == -1){
                            if(name.super_list &&name.super_list.length > 1){
                                name.super_list.forEach((val,i) =>{
                                    if(i == q){
                                        name.super_list.splice(i,1)
                                    }
                                })
                            }else{
                                self.list.splice(index,1)
                                self.delRelatedProblem.push(name)
                                name.super_list=[]
                            }
                            tag = name
                        }
                    })
                    if(tag.option_list){
                        for(let k of tag.option_list){
                            if(k.isactive&&k.next_question_list&&k.next_question_list.length>0){
                                self.ruleRemoveQuestions(k.next_question_list,map)
                            }
                        }
                    }
                })
            },

            // 移除问题时构建渲染页面的 list 的map结构 查询数组
            creaetListMap(){
                let self = this
                let strMap = {}
                for(let k of self.list) {
                    strMap[k.question_id] = k
                }
                return strMap
            },
            // 监控填空题
            fillVal(ev,item){
                let self = this
                ev.target.value == '' ? item.isChecked = false : item.isChecked = true
                // 记录用户是否有重填
                if(item.remind){
                    self.refill_item_list.push(item.id)
                    item.remind = false
                }
            },
            // 监控选择器选择
            selectVal(ev,item){
                let self = this
                ev.target.value == '请点击选择' ? item.isChecked = false : item.isChecked = true
                // 记录用户是否有重填
                if(item.remind){
                    self.refill_item_list.push(item.id)
                    item.remind = false
                }
            },
            // 监控搜索填空
            searchFillVal(ev,item){
                let self = this
                let checkedArr = []
                item.option_list.forEach( (tag,k) =>{
                    tag.first_option_content ? checkedArr.push(k) : ''
                })
                checkedArr.length ==0 ? self.$set(item,'isChecked',false) : self.$set(item,'isChecked',true)
                // 记录用户是否有重填
                if(item.remind){
                    self.refill_item_list.push(item.id)
                    item.remind = false
                }
            },
            // 过滤用法用量q
            instruction(res,hide){
                let html = ''
                if(res.option_content){
                    let ev = res.option_content
                    if(!hide){
                        ev.frequency&& ev.frequency !='每天' ? html+=ev.frequency + '：' : ''
                    }
                    ev.morning ? html+= '早'+ ev.morning + ev.drug_unit +',': ''
                    ev.noon ? html+= '午' + ev.noon + ev.drug_unit +',' : ''
                    ev.night ? html+= '晚' + ev.night + ev.drug_unit +',' : ''
                    ev.before_sleep ? html+= '睡前' + ev.before_sleep + ev.drug_unit : ''
                    if(!hide){
                        ev.usage && ev.usage!='口服' ? html+= ' / '+ ev.usage : '' 
                    }
                }
                return html
            },

            // 修改填写好的药品
            editDrug(val,index,$k) {
                let self = this
                index ? self.recipeIndex = index : ''
                $k ? self.editRecipeIndex = $k : ''
                self.showRecipe = !self.showRecipe
                self.drug_info = val
                setTimeout(() =>{
                    self.$refs.wRecipe.amendDrug()
                },300)
            },
            // 保存处方
            saveRecipe(val){
                let self = this
                if(val.drug_name){
                    self.saveRecipeCommon(val)
                }
                self.showWriteRecipe()
            },
            saveRecipeCommon(val){
                let self = this
                // 修改药品
                console.log(val)
                if(val.isEdit){
                    let data = self.list[self.recipeIndex]
                    data.option_list[self.editRecipeIndex].option_content = val
                }else{
                    // 当填写超过两个时增加多一个
                    if(self.addRecipe || self.isRecipe >1 ){
                        let obj ={
                            option_content:{}
                        }
                        self.list[self.recipeIndex].option_list.push(JSON.parse(JSON.stringify(obj)))
                    }
                    // 标记已选中
                    self.list[self.recipeIndex].isChecked = true
                    let data = self.list[self.recipeIndex]
                    
                    for(var i=0;i<data.option_list.length;i++){
                        if(!data.option_list[i].option_content.drug_name){
                            data.option_list[i].option_content = val
                            if(i>0){
                                self.addRecipe = true
                            }
                            break
                        }
                    }
                }
                self.$set(self.list[self.recipeIndex],self.recipeIndex,self.list[self.recipeIndex].option_list)
                // 记录用户是否有重填
                if(self.list[self.recipeIndex].remind){
                    self.refill_item_list.push(self.list[self.recipeIndex].id)
                    self.list[self.recipeIndex].remind = false
                }
            },
            // 显示编辑处方弹窗
            showWriteRecipe(index){
                let self = this
                index ? self.recipeIndex = index : ''
                self.showRecipe = !self.showRecipe
            },
            // 删除编辑好的药品
            delHistory(index,$index){
                let self = this
                self.list[$index].option_list.splice(index,1)
                if(self.list[$index].option_list.length ==1){
                    let obj ={
                        option_content:{}
                    }
                    self.list[$index].option_list.push(JSON.parse(JSON.stringify(obj)))
                    self.addRecipe = false
                }
                let checkedArr = []
                for(let item of self.list[$index].option_list){
                   item.option_content.drug_name ? checkedArr.push('item') : ''
                }
                checkedArr.length !=0 ? self.list[$index].isChecked = true : self.list[$index].isChecked = false
                // 记录用户是否有重填
                if(self.list[$index].remind){
                    self.refill_item_list.push(self.list[$index].id)
                    self.list[$index].remind = false
                }
            },
            // 导入医生上次用药
            lastRecipe(index){
                let self = this
                index ? self.recipeIndex = index : ''
                Tips('','show')
                Ajax('get','smartDrug/record-last',{appointId: self.id}).then(data => {
                    Tips('','hide')
                    preTreated(data, res => {
                        // 有两个医生额药单时，让患者选择
                        if(res.data.length >1){
                            self.lastList = res.data
                            self.showLastPopup()
                        // 只有一个时默认填入
                        } else if(res.data.length == 1){
                            self.saveDoctorRecipe(res.data[0])
                        } else{
                            self.showLastPopup()
                            setTimeout(() => {
                                self.addLastRecipe = false
                            },2000)
                        }
                    })
                }).catch(err =>{
                    console.log(err)
                })
            },
            // 显示上次用药弹窗
            showLastPopup() {
                let self = this
                self.addLastRecipe = !self.addLastRecipe
            },
            //  点击保存上次用药
            saveDoctorRecipe(item){
                let self = this
                for(let val of item.drug_data){
                    self.saveRecipeCommon(val)
                }
                self.addLastRecipe = false
            }
        },
        components:{
            scroller,
            compressImg,
            writeRecipe
        },
        watch:{
            showRecipe(val){
                let self = this
                if(self.showRecipe){
                    self.$emit('hideNav',false)
                }else{
                   self.$emit('hideNav',true) 
                }
            }
        }
    }
</script>