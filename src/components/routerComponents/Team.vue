<template>
  <el-container class="main-container">
      <el-header style="position: relative;">
          <div style="position: absolute;width:100%">
              <i class="el-icon-back" @click="handleBack" style="font-size: 40px;vertical-align: middle;"
                  v-if="!label"></i>
              <el-input placeholder="请输入群名称进行搜索，可以直接回车搜索..." v-model="searchInput" class="searchClass" clearable
                  @clear="search" @keydown.enter.native="search" style="width:87%;float:left"></el-input>
              <el-button slot="append" icon="el-icon-search" @click="search"  style="width:10%;float:left"></el-button>
          </div>
      </el-header>
      <el-row style="height: 100%;" :gutter="20">
          <el-col style="height: 100%;" :span="16">
              <el-table :data="tableData" style="height: 100%;border-radius: 7px;" :max-height="tableHeight" highlight-current-row
                  @row-click="handleCurrentChange">
                  <el-table-column v-if="label" label="群组列表" min-width="20%">
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
                              <p>姓名: {{ scope.row.teamName }}</p>
                              <p>简介: {{ scope.row.description }}</p>

                              <div slot="reference" class="name-wrapper">
                                  <div>{{ scope.row.teamName }}</div>
                                  <!-- <el-tag size="medium" color="white">{{ scope.row.name }}</el-tag> -->
                                  <el-tag>{{ scope.row.teamName }}</el-tag>
                              </div>
                          </el-popover>
                      </template>
                  </el-table-column>

                  <el-table-column label="" min-width="17%">
                      <template slot-scope="scope">


                          <el-button v-if="label" size="mini"
                              @click="handleVideoChat(scope.$index, scope.row)">视频聊天</el-button>
                          <el-button v-else size="mini" @click="handleOpenDialog">添加好友</el-button>
                      </template>
                  </el-table-column>

                  <el-table-column label="" min-width="13%">
                      <template slot-scope="scope">
                          <div v-if="messageCountList[scope.$index]">
                              <el-badge v-if="messageCountList[scope.$index].count !== 1"
                                  :value="messageCountList[scope.$index].count" class="badge" style="width: 100%;">
                              </el-badge>
                          </div>
                      </template>
                  </el-table-column>


              </el-table>
          </el-col>

          <el-col style="height: 100%;" :span="8">
              <el-table :data="applicationList" style="height: 100%;border-radius: 7px;" :max-height="tableHeight" highlight-current-row
                  @row-click="handleApplicationChange">
                  <el-table-column v-if="label" label="群组邀请" min-width="30%">
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
                      <template slot-scope="scope">
                          <div>
                              <el-button size="mini" @click="handleOpenDetail(scope.row)">查看信息</el-button>
                          </div>
                      </template>
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

                    <el-main style="position: absolute;height: 75%;width: 100%;top: 10%;" ref="chat">
                        <infinite-loading :identifier="customIdentifier" direction="top"
                            @infinite="infiniteHandler"></infinite-loading>
                        <el-row v-for="(item, $index) in chatList" :key="$index" style="margin-top: 10px;">
                            <div v-if="item.fromID !== user.userID">
                                <el-col :span="2" :offset="2">
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
                                        <el-button size="mini" @click="listenAudio(item.attach)">语音消息</el-button>
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
                                    <!-- <div v-else style="border-style: solid;solid: #000;
                                background-color: #add6fa;
                                border-width: 1px;
                                border-radius: 7px;display: inline-block;float: right">
                                        <el-button size="mini" @click="listenAudio(item.attach)">语音消息</el-button>
                                    </div> -->
                                </el-col>
                                <el-col :span="2" style="float: right">
                                    <el-avatar icon="el-icon-user-solid" :size=33></el-avatar>
                                </el-col>
                            </div>
                        </el-row>
                    </el-main>

                    <el-footer style="position:absolute;top: 85%;left:0;width:100%;height: 15%;">
                        <div>
                            <el-input type="textarea" v-model="chatInfo" autosize @clear="sendInfo"
                                @keyup.enter.native="sendInfo" style="width: 80%"></el-input>
                            <el-button @click="sendInfo" type="primary" size="mini"
                                style="width: 20%;height: 32px">发送</el-button>
                        </div>
                        <el-upload class="upload-file" :action="uploadURL" :show-file-list="false" :auto-upload="false"
                            :on-change="handleChange" ref="uploadFile" :name="chatInfo">
                            <el-button type="primary" size="mini" class="el-icon-folder-opened"></el-button>
                        </el-upload>
                        <!-- <div class="play-audio">
                            <button @click="startRecording" v-show="!recording">开始录音</button>
                            <button @click="stopRecording" v-show="recording">停止录音</button>
                            <button @click="sendAudioData" v-show="sendShow">发送</button>
                        </div> -->
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
          console.log(JSON.parse(event.data))
          this.chatList.push(JSON.parse(event.data))
          console.log(this.chatList)
      }
  },
  methods: {
      connect() {
          Vue.use(VueNativeSock, this.NET.BASE_URL.ws + 'websocket/' + this.$cookies.get('userID'), {
              format: 'json',
              reconnection: true, // 自动重连
              reconnectionAttempts: 5, // 重连尝试次数
              reconnectionDelay: 3000, // 重连延迟（毫秒）
          })
          this.$axios({
              method: 'GET',
              url: this.NET.BASE_URL.http + 'getTeamUnReadMessages',
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
              console.log(this.searchInput.trim())
              this.$axios({
                  method: 'GET',
                  url: this.NET.BASE_URL.http + 'searchTeam',
                  params: {
                    teamName: this.searchInput.trim(),
                  }
              }).then(response => {
                  console.log(response.data)
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
                  this.$refs.uploadFile.submit()
                  this.uploadFile = false
              }
              this.$socket.sendObj({
                  message: this.chatInfo.trim(),
                  from: this.$cookies.get('userID'),
                  to: this.currentRow.userID,
                  kind: 0,
              });
              this.chatInfo = ''
          }
      },
      handleCurrentChange(val) {
          this.drawer = this.listbtn;
          if (this.currentRow !== val && this.drawer) {
              this.chatList = []
              this.page = 0
              this.currentRow = val;
              this.customIdentifier = true
              this.uploadURL = this.NET.BASE_URL.http + 'upload/'
                  + this.$cookies.get('userID') + '/' + this.currentRow.userID + '/0'
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
              console.log(response.data.msg)
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
              url: this.NET.BASE_URL.http + 'getTeamHistoryMessage/' + this.currentRow.teamID,
              params: {
                  page: this.page,
              }
          }).then(response => {
              if (response.data.data.length) {
                  this.page += 1
                  for (var i = 0; i < response.data.data.length; i++) {
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
      getTableMaxHeight() {
          this.$nextTick(function () {
              let box = document.querySelector(".main-container").attributes
              let box_clientHeight = box[0].ownerElement.clientHeight;
              this.tableHeight = box_clientHeight - 60;
          })
      },
      handleScrollBottom() {
          this.$nextTick(() => {
              let scrollElem = this.$refs.chat;
              scrollElem.scrollTo(0, scrollElem.scrollHeight)
              // scrollElem.scrollTo({ top: scrollElem.scrollHeight, behavior: 'smooth' });
          });
      },
      handleBack() {
          this.label = true
          this.searchInput = ''
          this.handleFriend()
      },
      handleFriend() {
          this.$axios({
              method: 'POST',
              url: this.NET.BASE_URL.http + 'showTeamList',
              data: {
                  userID: this.$cookies.get('userID'),
                  password: this.$cookies.get('username')
              }
          }).then(response => {
              this.tableData = response.data.data
          }, error => {
              console.log('错误', error.message)
          })
      },
      handleMessageCount() {
          this.$axios({
              method: 'GET',
              url: this.NET.BASE_URL.http + 'getTeamUnReadMessages',
              params: {
                  userID: this.user.userID
              }
          }).then(response => {
              for (var i = 0; i < this.tableData.length; i++) {

                  var item = response.data.data[0][0].filter(item => item.toID === this.tableData[i].teamID)[0]
                  var count = response.data.data[0][1][response.data.data[0][0].findIndex(item => {
                      if (item.toID === this.tableData[i].teamID) {
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
                  console.log('信息查找成功')
                  console.log(response.data)
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
      handleChange(file) {
          if (file) {
              this.uploadFile = true
              this.chatInfo = file.name
          }
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
</style>