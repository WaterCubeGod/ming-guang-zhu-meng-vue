<template>
    <el-container class="main-container">
        <el-header style="position: relative;">
            <div style="position: absolute;width:100%">
                <el-button class="el-icon-back" @click="handleBack" size="middle"
                    v-if="!label"></el-button>
                <el-input placeholder="请输入昵称进行搜索，可以直接回车搜索..." v-model="searchInput" class="searchClass" clearable
                    @clear="search" @keydown.enter.native="search" style="width:87%;float:left"></el-input>
                <el-button slot="append" icon="el-icon-search" @click="search" style="width:10%;float:left"></el-button>
            </div>
        </el-header>
        <el-row style="height: 100%;" :gutter="20">
            <el-col style="height: 100%;" :span="16">
                <el-table :data="tableData" style="height: 100%;border-radius: 7px;" :max-height="tableHeight" highlight-current-row 
                    @row-click="handleCurrentChange" >
                    <el-table-column v-if="label" label="好友列表" min-width="20%" >
                        <template>
                            <el-avatar icon="el-icon-user-solid"></el-avatar>
                        </template>
                    </el-table-column>

                    <el-table-column v-else label="查找结果" min-width="20%">
                        <template>
                            <el-avatar icon="el-icon-user-solid"></el-avatar>
                        </template>
                    </el-table-column>

                    <el-table-column label="" min-width="50%">
                        <template slot-scope="scope">
                            <el-popover trigger="hover" placement="top">
                                <p>姓名: {{ scope.row.username }}</p>
                                <p>住址: {{ scope.row.address }}</p>

                                <div slot="reference" class="name-wrapper">
                                    <div>{{ scope.row.username }}</div>
                                    <!-- <el-tag size="medium" color="white">{{ scope.row.name }}</el-tag> -->
                                    <el-tag>{{ scope.row.address }}</el-tag>
                                </div>
                            </el-popover>
                        </template>
                    </el-table-column>

                    <el-table-column label="" min-width="17%">
                        <template slot-scope="scope">
                            <div v-if="label || friendData.some(item => item.userID === scope.row.userID)">
                                <router-link
                                    :to="{ name: 'videoPlayer', params: { toID: scope.row.userID, type: 'video' } }">
                                    <el-button size="mini">视频聊天</el-button>
                                </router-link>
                                <router-link
                                    :to="{ name: 'videoPlayer', params: { toID: scope.row.userID, type: 'audio' } }">
                                    <el-button size="mini">语音聊天</el-button>
                                </router-link>
                            </div>
                            <div v-else>
                                <el-button @click="handleOpenDialog" size="mini">添加好友</el-button>
                            </div>
                        </template>
                    </el-table-column>

                    <el-table-column label="" min-width="13%">
                        <template slot-scope="scope">
                            <div v-if="messageCountList[scope.$index]">
                                <el-badge v-if="messageCountList[scope.$index].count !== 0"
                                    :value="messageCountList[scope.$index].count" class="badge" style="width: 100%;">
                                </el-badge>
                            </div>
                        </template>
                    </el-table-column>


                </el-table>
            </el-col>

            <el-col style="height: 100%;" :span="8">
                <el-table :data="applicationList" style="height: 100%; border-radius: 7px;" :max-height="tableHeight" highlight-current-row
                    @row-click="handleApplicationChange">
                    <el-table-column v-if="label" label="好友申请" min-width="30%">
                        <template>
                            <el-avatar icon="el-icon-user-solid"></el-avatar>
                        </template>
                    </el-table-column>
                    <!-- <el-table-column v-else label="申请记录" min-width="30%">
                        <template>
                            <el-avatar icon="el-icon-user-solid"></el-avatar>
                        </template>
                    </el-table-column> -->
                    <el-table-column label="" min-width="30%">
                        <template slot-scope="scope">
                            <el-popover trigger="hover" placement="top">
                                <div slot="reference" class="name-wrapper">
                                    <div>{{ scope.row.fromID }}</div>
                                    <!-- <el-tag size="medium" color="white">{{ scope.row.name }}</el-tag> -->
                                    <el-tag>{{ scope.row.additionalMessage }}</el-tag>
                                </div>
                            </el-popover>
                        </template>
                    </el-table-column>
                    <!-- <el-table-column label="申请记录" min-width="30%">

                    </el-table-column> -->
                </el-table>
            </el-col>
        </el-row>

        <el-drawer :visible.sync="drawer" :with-header="false" size="50%">
            <div style="position:absolute;top:0;left:0;width:100%;height:100%;">
                <el-container style="height: 100%;">
                    <el-header style="width: 100%;height: 10%;background: #add6fa;">
                        {{ this.currentRow.username }}
                    </el-header>

                    <el-main style="position: absolute;height: 75%;width: 100%;top: 10%;" class="chat-main">
                        <infinite-loading :identifier="customIdentifier" direction="top"
                            @infinite="infiniteHandler"></infinite-loading>
                        <el-row v-for="(item, $index) in checkShowRule(chatList, 'time')" :key="$index" style="margin-top: 10px;">
                            <div v-if="item.is_show_time" style="font-size: small;color: rgba(0, 0, 0, 0.5);">{{ toggleTime(item.time).time }}</div>
                            <div v-if="item.fromID !== user.userID">
                                <el-col :span="2">
                                    <el-avatar icon="el-icon-user-solid" :size=33></el-avatar>
                                </el-col>
                                <el-col :span="20">
                                    <div v-if="item.attach === 0 && item.kind === 0" style="border-style: solid;solid: #000;
                                background-color: #add6fa;
                                border-width: 1px;
                                border-radius: 7px;display: inline-block;float: left;text-align: left;">
                                        {{ item.content }}
                                    </div>
                                    <div v-else-if="item.attach !== 0 && item.kind === 0" style="border-style: solid;solid: #000;
                                background-color: #add6fa;
                                border-width: 1px;
                                border-radius: 7px;height: 31px;display: inline-block;float: left;text-align: left;">
                                        <el-link type="success" :href="NET.BASE_URL.http + 'download/' + item.attach">{{
                                            item.content }}</el-link>
                                    </div>
                                    <div v-else style="border-style: solid;solid: #000;
                                background-color: #add6fa;
                                border-width: 1px;
                                border-radius: 7px;display: inline-block;float: left">
                                        <el-button size="mini" @click="listenAudio(item.attach)">语音消息<i class="el-icon-video-play" v-show="item.flag"></i><i class="el-icon-video-pause" v-show="!item.flag"></i></el-button>
                                    </div>
                                </el-col>
                            </div>
                            <div v-else>
                                <el-col :span="20" :offset="2">
                                    <div v-if="item.attach === 0 && item.kind === 0" style="border-style: solid;solid: #000;
                                background-color: #add6fa;
                                border-width: 1px;
                                border-radius: 7px;display: inline-block;float: right;text-align: left;">
                                        {{ item.content }}
                                    </div>
                                    <div v-else-if="item.attach !== 0 && item.kind === 0" style="border-style: solid;solid: #000;
                                background-color: #add6fa;
                                border-width: 1px;
                                border-radius: 7px;display: inline-block;float: right;;text-align: left;">
                                        <el-link type="success" :href="NET.BASE_URL.http + 'download/' + item.attach">
                                            {{ item.content }}
                                        </el-link>
                                    </div>
                                    <div v-else style="border-style: solid;solid: #000;
                                background-color: #add6fa;
                                border-width: 1px;
                                border-radius: 7px;display: inline-block;float: right">
                                        <el-button type="primary" size="mini" @click="listenAudio(item.attach)" style="background-color: #add6fa; border-width: 1px;
                                border-radius: 7px;height: 35px;color: #000;">语音消息<i class="el-icon-video-play" v-show="item.flag"></i><i class="el-icon-video-pause" v-show="!item.flag"></i></el-button>
                                    </div>
                                </el-col>
                                <el-col :span="2" style="float: right">
                                    <el-avatar icon="el-icon-user-solid" :size=33></el-avatar>
                                </el-col>
                            </div>
                        </el-row>
                    </el-main>

                    <el-footer style="position:absolute;top: 85%;left:0;width:100%;height: 10%;border-top: 1px solid rgb(167, 165, 165);">
                        <div>
                            <el-button type="primary" size="mini" @click="startRecording" 
                                style="width: 10%;height: 32px"><i class="el-icon-microphone"></i></el-button>
                            <el-input type="textarea" v-model="chatInfo" autosize @clear="sendInfo"
                                @keyup.enter.native="sendInfo" style="width: 65%"></el-input>
                            <el-button @click="sendInfo" type="primary" size="mini"
                                style="width: 15%;height: 32px">发送</el-button>
                                <el-upload class="upload-file" :action="uploadURL" :show-file-list="false" :auto-upload="false"
                                    :on-change="handleChange" ref="uploadFile" :on-success="success" style="width: 10%;height: 32px;display:inline-block;">
                                    <el-button type="primary" size="mini" class="el-icon-folder-opened" style="height: 32px;"></el-button>
                                </el-upload>
                        </div>
                        
                        <div class="play-audio" v-show="audioDivVisable">
                            
                            <el-button type="primary" size="mini" @click="stopRecording" v-show="stopRecordingAudio" style="margin: auto;margin-top: 10px;">停止录音</el-button>
                            <el-button type="primary" size="mini" @click="sendAudioData" v-show="sendShow" style=" float: left;margin-left: 10px;margin-top: 10px;">发送</el-button>
                            <el-button type="primary" size="mini" @click="quitSend" v-show="sendShow" style=" float:right;margin-right: 10px;margin-top: 10px;">取消</el-button>
                        </div>
                    </el-footer>
                </el-container>
            </div>
        </el-drawer>

        <el-dialog title="添加好友" :visible.sync="dialogInfo.systemMessageVisible">
            <el-form :model="currentRow">
                <el-form-item label="昵称" :label-width="dialogInfo.formLabelWidth">
                    <div>{{ currentRow.username }}</div>
                </el-form-item>
                <el-form-item label="地址" :label-width="dialogInfo.formLabelWidth">
                    <div>{{ currentRow.address }}</div>
                </el-form-item>
                <el-form-item label="留言" :label-width="dialogInfo.formLabelWidth">
                    <el-input v-model="addtionalMessage" autocomplete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialogInfo.systemMessageVisible = false">取 消</el-button>
                <el-button type="primary" @click="sendSystemMessage">确 定</el-button>
            </div>
        </el-dialog>

        <el-dialog title="用户详情" :visible.sync="detailDialogVisable">
            <el-form :model="userInfo">
                <el-form-item label="昵称" :label-width="dialogInfo.formLabelWidth">
                    <div>{{ userInfo.username }}</div>
                </el-form-item>
                <el-form-item label="地址" :label-width="dialogInfo.formLabelWidth">
                    <div>{{ userInfo.address }}</div>
                </el-form-item>
                <el-form-item label="邮箱" :label-width="dialogInfo.formLabelWidth">
                    <div>{{ userInfo.email }}</div>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="acceptOrReject('reject')">拒绝</el-button>
                <el-button type="primary" @click="acceptOrReject('accept')">接受</el-button>
            </div>
        </el-dialog>
        <el-dialog title="提示" :visible.sync="messageDialog.messageDialogVisable" width="30%">
            <span>{{ messageDialog.message }}</span>
            <span slot="footer" class="dialog-footer">
                <el-button type="primary" @click="closeMessageDialog">确 定</el-button>
            </span>
        </el-dialog>

    </el-container>
</template>

<script>
import Vue from 'vue'
import VueNativeSock from 'vue-native-websocket'
import InfiniteLoading from 'vue-infinite-loading'
import dayjs from 'dayjs'

export default {
    name: 'MyFriend',
    data() {
        return {
            customIdentifier: true,
            page: 0,
            tableHeight: 400,
            searchInput: '',
            chatInfo: '',
            drawer: false,
            receivedData: {},
            currentRow: {},
            label: true,
            listbtn: true,
            friendData: [],
            tableData: [],
            chatList: [],
            user: {
                userID: parseInt(this.$cookies.get('userID'))
            },
            messageCountList: [],
            dialogInfo: {
                systemMessageVisible: false,
                formLabelWidth: '120px',
            },
            messageDialog: {
                messageDialogVisable: false,
                message: ''
            },
            detailDialogVisable: false,
            addtionalMessage: '',
            applicationList: [],
            applicationRow: {},
            userInfo: {},
            uploadFile: false,
            uploadURL: '',
            mediaRecorder: null,
            audioChunks: [],
            recording: false,
            stopRecordingAudio:false,
            audioDivVisable:false,
            sendShow:false,
            broadCasting:false,
            clickAttach:0,
            attachNum: 0,

        };
    },
    mounted() {
        //获取容器当前高度，重设表格的最大高度
        this.getTableMaxHeight();
        let _this = this;
        window.onresize = function () {//用于使表格高度自适应的方法
            _this.getTableMaxHeight();//获取容器当前高度，重设表格的最大高度
        }
        this.connect()
        
        this.$socket.onmessage = (event) => {
            var homeThis=this
            if (event.data instanceof Blob) {
                console.log('我是音频');
                // this.broadCasting=true;
                const audioContext = new AudioContext();
                audioContext.onstatechange=()=>{
                    console.log(audioContext.state)
                    if(audioContext.state==='running'){
                        console.log(111)
                        console.log(homeThis.chatList)
                        homeThis.chatList.filter(item => item.attach === this.clickAttach)[0].flag = false
                    }
                }
                const audioData = event.data;
                const fileReader = new FileReader();

                fileReader.onload = function () {
                    const arrayBuffer = fileReader.result; // 从 FileReader 中获取 ArrayBuffer
                    audioContext.decodeAudioData(arrayBuffer, (buffer) => {
                        const source = audioContext.createBufferSource();
                        source.buffer = buffer;
                        source.connect(audioContext.destination);
                        source.start();
                        source.onended=()=>{
                            console.log(222)
                            console.log(homeThis.chatList)
                            for(let i=0;i<homeThis.chatList.length;i++){
                                if(homeThis.chatList[i].attach ===  homeThis.clickAttach){
                                    homeThis.chatList[i].flag=true
                                    break
                                }
                            }
                        }
                    });
                };

                fileReader.readAsArrayBuffer(audioData); // 读取 Blob 数据并转换为 ArrayBuffer
            } else {
                if (!isNaN(Number(JSON.parse(event.data)))) {

                    homeThis.$socket.sendObj({
                        from: this.$cookies.get('userID'),
                        to: this.currentRow.userID,
                        kind: 2,
                        attach: Number(JSON.parse(event.data))
                    })
                } else {
                    let message = JSON.parse(event.data)
                    if(message.kind===2) {
                        message['flag'] = true
                    }
                    this.chatList.push(message)
                    if ((message.fromID !== this.currentRow.userID || !this.drawer) && message.fromID !== this.user.userID) {
                        this.messageCountList[this.messageCountList.findIndex(item => {
                            if (item.message.fromID === message.fromID) {
                                return true;
                            }
                        })].count++
                    }
                    if (this.currentRow.userID === message.fromID && this.drawer) {
                        this.$axios({
                            method: 'GET',
                            url: this.NET.BASE_URL.http + 'changeMessageStatusInTime',
                            params: {
                                fromID: message.fromID,
                                toID: message.toID,
                                kind: 0
                            }
                        }).then(error => {
                            console.log('错误', error.message)
                        })
                    }
                }

            }
            this.chatInfo = ''
            this.handleScrollBottom()
        }
    },
    methods: {
        success(response) {
            console.log(response,111111)
            this.attachNum = response.data
            console.log(this.attachNum)
            this.$socket.sendObj({
                message:this.chatInfo.trim(),
                    from: Number(this.$cookies.get('userID')),
                    to: Number(this.currentRow.userID),
                    attach:response.data ,
                    kind:0
            })
            this.uploadFile = false
            this.attach = 0
                this.chatInfo = ''
                this.handleScrollBottom()
                console.log(this.chatInfo,1231)
        },
        connect() {
            Vue.use(VueNativeSock, this.NET.BASE_URL.ws + 'websocket/' + this.$cookies.get('userID'), {
                format: 'json',
                reconnection: true, // 自动重连
                reconnectionAttempts: 5, // 重连尝试次数
                reconnectionDelay: 3000, // 重连延迟（毫秒）
            })
            this.$axios({
                method: 'GET',
                url: this.NET.BASE_URL.http + 'getP2PUnReadMessages',
                params: {
                    userID: parseInt(this.$cookies.get('userID')),
                }
            }).then(response => {
                this.messageCountList.message = response.data.data
            }, error => {
                console.log('错误', error.message)
            })
        },
        search() {
            if (this.searchInput !== '') {
                this.label = false
                this.$axios({
                    method: 'POST',
                    url: this.NET.BASE_URL.http + 'searchUser',
                    data: {
                        username: this.searchInput.trim(),
                    }
                }).then(response => {
                    this.tableData = response.data.data
                    this.label = false
                }, error => {
                    console.log('错误', error.message)
                })
            } else {
                this.$message({
                    message: '您还未输入任何有效字符',
                    type: 'warning',
                    duration: 1000
                });
            }
        },
        sendInfo() {
            if (this.chatInfo.trim() !== '') {
                if (this.uploadFile) {
                    console.log(222222)
                    this.$refs.uploadFile.submit()
                    this.uploadFile=false
                }else{
                    var abaaba={
                    message:this.chatInfo.trim(),
                    from: Number(this.$cookies.get('userID')),
                    to: Number(this.currentRow.userID),
                    attach: 0,
                    kind:0
                }
                this.$socket.sendObj(abaaba);
                this.attach = 0
                this.chatInfo = ''
                this.handleScrollBottom()
                }
            }
        },
        handleCurrentChange(val) {
            this.currentRow = val;
            if (this.friendData.some(item => item.userID === val.userID)) {
                this.drawer = this.listbtn;
                if (this.drawer) {
                    this.chatList = []
                    this.page = 0
                    this.customIdentifier = true
                    this.uploadURL = this.NET.BASE_URL.http + 'upload/'
                        + this.$cookies.get('userID') + '/' + this.currentRow.userID + '/0'
                }
                if (this.drawer) {
                    this.messageCountList[this.messageCountList.findIndex(item => {
                        if (item.message.fromID === this.currentRow.userID) {
                            return true;
                        }
                    })].count = 0
                }
            } else {
                this.$message({
                    showClose: true,
                    message: '您和他还不是好友',
                    type: 'warning',
                });
            }

            this.listbtn = true
        },
        handleApplicationChange(val) {
            if (this.applicationRow !== val) {
                this.applicationRow = val
            }
        },
        acceptOrReject(status) {
            // this.detailDialogVisable=false
            var homeThis = this
            this.$axios({
                method: 'POST',
                url: this.NET.BASE_URL.http + 'solveApplication',
                data: {
                    time: this.applicationRow.time,
                    fromID: this.applicationRow.fromID,
                    toID: this.applicationRow.toID,
                    additionalMessage: this.applicationRow.additionalMessage,
                    kind: this.applicationRow.kind,
                    status: status
                }
            }).then(response => {
                homeThis.detailDialogVisable = false
                homeThis.messageDialog.messageDialogVisable = true
                if (response.data.code === 1) {
                    if (status === 'accept') {
                        homeThis.messageDialog.message = "接受成功"
                    } else if (status === 'reject') {
                        homeThis.messageDialog.message = "拒绝成功"

                    }
                } else {
                    homeThis.messageDialog.message = "操作失败，卡咯"
                }
            }, error => {
                console.log('错误', error.message)
            })
        },
        infiniteHandler($state) {
            this.$axios({
                method: 'GET',
                url: this.NET.BASE_URL.http + 'getHistoryMessage/' + this.$cookies.get('userID') + '/' + this.currentRow.userID,
                params: {
                    page: this.page
                }
            }).then(response => {
                if (response.data.data.length) {
                    this.page += 1
                    for (var i = 0; i < response.data.data.length; i++) {
                        if(response.data.data[i].kind===2) {
                            response.data.data[i]['flag'] = true
                        }
                        this.chatList.unshift(response.data.data[i])
                    }
                    $state.loaded();
                } else {
                    $state.complete();
                    this.customIdentifier = false
                }
            }, error => {
                console.log('错误', error.message)
            })
        },
        getNowDate() {
            var myDate = new Date;
            var year = myDate.getFullYear(); //获取当前年
            var mon = myDate.getMonth() + 1; //获取当前月
            var date = myDate.getDate(); //获取当前日
            var hours = myDate.getHours(); //获取当前小时
            var minutes = myDate.getMinutes(); //获取当前分钟
            var seconds = myDate.getSeconds(); //获取当前秒
            var now = year + "-" + mon + "-" + date + " " + hours + ":" + minutes + ":" + seconds;
            return now;
        },
        getTableMaxHeight() {
            this.$nextTick(function () {
                let box = document.querySelector(".main-container").attributes
                let box_clientHeight = box[0].ownerElement.clientHeight;
                this.tableHeight = box_clientHeight - 60;
            })
        },
        handleScrollBottom() {
            this.$nextTick(() => {
                let scrollElem = document.querySelector('.chat-main')
                scrollElem.scrollTop = scrollElem.scrollHeight
            });
        },
        handleBack() {
            this.label = true
            this.searchInput = ''
            this.handleFriend()
        },
        handleChange(file) {
            if (file) {
                this.uploadFile = true
                this.chatInfo = file.name
            }
        },
        handleFriend() {
            this.$axios({
                method: 'POST',
                url: this.NET.BASE_URL.http + 'showFriendList',
                data: {
                    userID: this.$cookies.get('userID'),
                    password: this.$cookies.get('username')
                }
            }).then(response => {
                this.tableData = response.data.data
                this.friendData = this.tableData
            }, error => {
                console.log('错误', error.message)
            })
        },
        handleMessageCount() {
            this.$axios({
                method: 'GET',
                url: this.NET.BASE_URL.http + 'getP2PUnReadMessages',
                params: {
                    userID: this.user.userID
                }
            }).then(response => {
                for (var i = 0; i < this.tableData.length; i++) {

                    var item = response.data.data[0][0].filter(item => item.fromID === this.tableData[i].userID)[0]
                    var count = response.data.data[0][1][response.data.data[0][0].findIndex(item => {
                        if (item.fromID === this.tableData[i].userID) {
                            return true;
                        }
                    })]
                    this.messageCountList.push({
                        message: item,
                        count: count,
                    })
                }
            }, error => {
                console.log('错误', error.message)
            })
        },
        handleApplication() {
            this.$axios({
                method: 'GET',
                url: this.NET.BASE_URL.http + 'getUnSolvedApplication',
                params: {
                    userID: this.user.userID
                }
            }).then(response => {
                this.applicationList = response.data.data
            }, error => {
                console.log('错误', error.message)
            })
        },
        handleOpenDialog() {
            this.listbtn = false
            this.dialogInfo.systemMessageVisible = true
        },
        handleOpenDetail(row) {
            this.detailDialogVisable = true
            this.applicationRow = row
            // console.log(this.applicationRow)
            this.$axios({
                method: 'GET',
                url: this.NET.BASE_URL.http + 'getUserInfo',
                params: {
                    userID: this.applicationRow.fromID
                }
            }).then(response => {
                if (response.data.code === 1) {
                    this.userInfo = response.data.data
                } else {
                    console.log('查找失败')
                }
            }, error => {
                console.log('错误', error.message)
            })
        },
        sendSystemMessage() {
            var homeThis = this
            this.dialogInfo.systemMessageVisible = false
            this.$axios({
                method: 'POST',
                url: this.NET.BASE_URL.http + 'sendSystemMessage',
                data: {
                    fromID: this.$cookies.get('userID'),
                    toID: this.currentRow.userID,
                    addtionalMessage: this.addtionalMessage,
                    kind: 'friendApplication',
                    teamID: 0
                }
            }).then(response => {
                if (response.data.code === 1) {
                    homeThis.messageDialog.messageDialogVisable = true
                    homeThis.messageDialog.message = '申请发送成功！'
                }
                else {
                    homeThis.messageDialog.messageDialogVisable = true
                    homeThis.messageDialog.message = '申请发送失败，已经向对方发送过申请'
                }
            }, error => {
                console.log('错误', error.message)
            })
        },
        handleVideoChat() {
            // 使用 Vue Router 跳转到指定页面并传递参数
            this.$router.push({
                path: '/videoPlayer',
                params: {
                    toID: this.currentRow.userID
                }
            });
        },
        closeMessageDialog() {
            this.messageDialog.messageDialogVisable = false
            this.reload()
        },
        reload() {
            this.$router.go(0)
        },
        startRecording() {
        if (!this.recording) {
          navigator.mediaDevices
            .getUserMedia({ audio: true })
            .then((stream) => {
              this.audioChunks = [];
              this.mediaRecorder = new MediaRecorder(stream);
              this.mediaRecorder.ondataavailable = (event) => {
                if (event.data.size > 0) {
                  this.audioChunks.push(event.data);
                  this.sendShow=true
                }
                
              };
              this.mediaRecorder.start();
              this.recording = true;
              this.stopRecordingAudio=true;
              this.audioDivVisable=true
            })
            .catch((error) => {
              console.error('获取麦克风访问权限失败:', error);
            });
        }
      },
      stopRecording() {
        if (this.recording) {
          this.mediaRecorder.stop();
        //   this.recording = false;
        this.stopRecordingAudio=false
        }
      },
      getAudioData(){
            const audioBlob = new Blob(this.audioChunks, { type: 'audio/wav' });
          console.log(audioBlob)
      },
      sendAudioData(){
        this.sendShow=false
        this.recording=false
        this.stopRecordingAudio=false
        this.audioDivVisable=false
            const audioBlob = new Blob(this.audioChunks, { type: 'audio/wav' });
          console.log(audioBlob)
          console.log('发送')
          this.$socket.send(audioBlob)
        //   this.$socket.sendObj({
        //     message:audioBlob,
        //     from: this.$cookies.get('userID'),
        //     to: this.currentRow.userID,
        //     kind: 2,
        //     attach:0
        //   }); // 发送二进制语音数据
      },
      quitSend(){
        this.sendShow=false
        this.recording=false
        this.audioDivVisable=false
      },
      listenAudio(attach){
        this.clickAttach=attach
        this.$socket.sendObj({
            message:'999',
            from: this.$cookies.get('userID'),
            to: this.currentRow.userID,
            kind: 999,
            attach:attach
          }); // 发送二进制语音数据
    },
    checkShowRule(arr, key) {
            var newArr = arr.map((item, index, array) => {
                var obj = this.toggleTime(item[key]);
                item['show_time_type'] = obj.type;
                item['show_time'] = obj.time;
                if (index > 0) {
                    item['is_show_time'] = this.compareTimeInterval(array[index - 1][key], array[index][key]);
                } else {
                    item['is_show_time'] = true;
                }
                return item;
            });
            return newArr;
        },
        //根据不同时间的消息，输出不同的时间格式
        toggleTime(date) {
            var time;
            var type = this.getDateDiff(date);
            //1：新消息，2：当天消息,3：昨天消息，4：今年消息，5：其他消息
            if (type == 1) {
                time = "以下为最新消息";//新消息，不显示时间，但是要显示"以下为最新消息"
            } else if (type == 2) {
                time = dayjs(date).format("H:mm");//当天消息，显示：10:22
            } else if (type == 3) {
                time = dayjs(date).format("昨天 H:mm");//昨天消息，显示：昨天 20:41
            } else if (type == 4) {
                time = dayjs(date).format("M月D日 AH:mm").replace("AM", "上午").replace("PM", "下午");//今年消息，上午下午，显示：3月17日 下午16:45
            } else if (type == 5) {
                time = dayjs(date).format("YYYY年M月D日 AH:mm").replace("AM", "上午").replace("PM", "下午");//其他消息，上午下午，显示：2020年11月2日 下午15:17
            }
            return {
                time: time,
                type: type
            };
        },
        //判断消息类型
        getDateDiff(date) {
            var nowDate = dayjs(new Date());//当前时间
            var oldDate = dayjs(new Date(date));//参数时间
            var result;
            if (nowDate.year() - oldDate.year() >= 1) {
                result = 5;
            } else if (nowDate.month() - oldDate.month() >= 1 || nowDate.date() - oldDate.date() >= 2) {
                result = 4;
            } else if (nowDate.date() - oldDate.date() >= 1) {
                result = 3;
            } else if (nowDate.hour() - oldDate.hour() >= 1 || nowDate.minute() - oldDate.minute() >= 5) {
                result = 2;
            } else {
                result = 1;
            }
            return result;
        },

        //判断两个时间差是否大于5分钟
        compareTimeInterval(t1, t2) {
            // console.log(t1,t2,dayjs(t2)-dayjs(t1));
            return dayjs(t2) - dayjs(t1) >= 300000 ? true : false;
        },
    broadCastOrStop(){
        
    }
        
    },

    async created() {
        this.handleFriend()
        this.handleApplication()
        this.handleMessageCount()
    },
    components: {
        InfiniteLoading,
    }

}
</script>

<style scoped>
.searchClass {
    border: 1px solid #c5c5c5;
    border-radius: 20px;
    background: #f4f4f4;
    line-height: 0px;
    width: 50%;
}

.searchClass .el-input-group__prepend {
    border: none;
    background-color: transparent;
    padding: 0 10px 0 30px;
}

.searchClass .el-input-group__append {
    border: none;
    background-color: transparent;
}

.searchClass .el-input__inner {
    height: 36px;
    line-height: 36px;
    border: none;
    background-color: transparent;
}

.searchClass .el-icon-search {
    font-size: 16px;
}

.searchClass .centerClass {
    height: 100%;
    line-height: 100%;
    display: inline-block;
    vertical-align: middle;
    text-align: right;
}

.searchClass .line {
    width: 1px;
    height: 26px;
    background-color: #c5c5c5;
    margin-left: 14px;
}

.searchClass:hover {
    border: 1px solid #D5E3E8;
    background: #fff;
}

.searchClass:hover .line {
    background-color: #D5E3E8;
}

.searchClass:hover .el-icon-search {
    color: #409eff;
    font-size: 16px;
}

.main-container {
    overflow: hidden;
    position: relative;
    height: calc(100% - 3rem);
}

.el-drawer {
    line-height: 10px;
}

:deep(.el-main) {
    height: 800px;
    overflow: auto;
    scrollbar-width: thin;
    /* 定义滚动条的宽度 */
    scrollbar-color: transparent transparent;
    /* 定义滚动条的颜色 */
}

/* 隐藏滚动条 */
:deep(.el-main::-webkit-scrollbar) {
    width: 0px;
    /* 定义滚动条宽度 */
    height: 8px;
    background-color: transparent;
    /* 滚动条背景颜色 */
}

.badge {
    margin-top: 10px;
    margin-right: 40px;
}
.play-audio{
    
    width: 200px;
    height: 50px;
    background-color: rgba(0, 0, 0, 0.1);
    margin: auto;
    margin-top: 20px;

}

</style>