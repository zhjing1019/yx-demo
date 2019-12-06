<template>
  <div class="home">
    <div class="home-list">
        <ul>
          <li>我的手机</li>
        </ul>
    </div>
    <div class="chat-main">
      对话部分
    </div>
  </div>
</template>
<script>
// import util from '../utils'
// import {config} from './../configs/index.js'
const SDK = require("../sdk/NIM_Web_SDK_v7.0.3.js");


export default {
  data() {
    return {
      appkey: '45c6af3c98409b18a84451215d0bdd6e',
      dnim: null,
      sessions: [],
      accountList: null,
      users: [],
      myInfo: {},
      chatInfo: [],
      msgsUser: {},
      chatType: "3", // 标识客服
      // sysUserId: getUserId(), // 存储用户id
      transfer: false, // 是否转客服
      centerVisible: true,
      chooseChatData: {
        type: "",
        name: "李芳芳",
        dutyName: "产品",
        isOnline: "",
        isCommonly: null,
        establishTime: null
      },
      chooseOrgData: {
        id: "000"
      },
      queryText: "",
      rightType: "",
      fromType: "",
      isRightShow: true,
      screenWidth: document.documentElement.clientWidth,
      screenHeight: document.documentElement.clientHeight,
      type: "chatroom",

    };
  },
  async created() {
    await this.initSDK();
  },
  watch: {
    // sessionlist(val) {
    //   console.log(val);
    // }
    
  },
  components: {

  },
  computed: {
    
  },
  mounted() {
   
  },
  methods: {
    initSDK() {
      let self = this;
      self.nim = SDK.NIM.getInstance({
        // debug: true,
        appKey: '45c6af3c98409b18a84451215d0bdd6e',
        // token: localStorage.getItem("Authorization"),
        // account: self.params.id,
        token: localStorage.getItem("sdktoken"),
        account: localStorage.getItem("uid"),
        transports: ["websocket"],
        onconnect: self.onConnect, // 链接已经建立(登录成功)
        onwillreconnect: self.onWillReconnect,
        ondisconnect: self.onDisconnect,
        onerror: self.onError,
        // 用户名片
        onmyinfo: self.onMyInfo,
        onupdatemyinfo: self.onUpdateMyInfo,
        onusers: self.onUsers,
        onupdateuser: self.onUpdateUser,
        // // 会话
        syncSessionUnread: true,
        onsessions: self.onSessions,
        onupdatesession: self.onUpdateSession,
        // 同步完成
        onsyncdone: self.onSyncDone
      });
    },
    onConnect(event) {
      console.log("已经连接", event);
      // if (loginInfo) {
      //   // 连接上以后更新uid
      //   commit('updateUserUID', loginInfo)
      // }
    },
    onWillReconnect(event) {
      // 此时说明 SDK 已经断开连接, 请开发者在界面上提示用户连接已断开, 而且正在 重新建立连接
      console.log("已经断开连接", event);
    },
    onDisconnect(error) {
      // 此时说明 SDK 处于断开状态, 开发者此时应该根据错误码提示相应的错误信息, 并且跳转到登录页面
      switch (error.code) {
        // 账号或者密码错误, 请跳转到登录页面并提示错误
        case 302:
          console.log("帐号或密码错误", "login");
          break;
        // 被踢, 请提示错误后跳转到登录页面
        case "kicked":
          // const map = {
          //   PC: '电脑版',
          //   Web: '网页版',
          //   Android: '手机版',
          //   iOS: '手机版',
          //   WindowsPhone: '手机版'
          // }
          // // // let str = error.from
          // const errorMsg = `你的帐号于${map.PC} || '其他端')}踢出下线，请确定帐号信息安全!`
          console.log(error, "login");
          break;
        default:
          break;
      }
    },
    onError(event) {
      console.log(event);
      // alert(JSON.stringify(event))
      alert("网络连接状态异常");
      // location.href = config.loginUrl
    },
    onSyncDone() {
      console.log("数据同步完成");
      // 数据同步完成
      // dispatch('hideLoading')
      // // 说明在聊天列表页
      // if (store.state.currSessionId) {
      //   dispatch('setCurrSession', store.state.currSessionId)
      // }
    },
    onSessions(sessions) {
      // 这里使用sdk自带的方法合并会话列表重新赋值
      this.sessions = this.nim.mergeSessions(this.sessions, sessions);
      // this.usDateSessionsUI() // 更新UI
      this.sessions.forEach(item => {
        const {
          lastMsg: { custom }
        } = item;
        if (custom) {
          const customList = (item.lastMsg.custom = typeof custom === "object" ? custom : JSON.parse(custom));
          if (customList.sysUserId && customList.sysUserId === this.sysUserId) {
            item.hiddleChat = customList.transfer;
          } else {
            item.hiddleChat = false;
          }
        } else {
          item.hiddleChat = false;
        }
      });
      console.log(11111111, this.sessions);
    },
    // 更新会话
    onUpdateSession(session) {
      this.sessions = this.nim.mergeSessions(this.sessions, session);
      this.sessions.forEach(item => {
        const {
          lastMsg: { custom }
        } = item;
        if (custom) {
          const customList = (item.lastMsg.custom = typeof custom === "object" ? custom : JSON.parse(custom));
          if (customList.sysUserId && customList.sysUserId === this.sysUserId) {
            item.hiddleChat = customList.transfer;
          } else {
            item.hiddleChat = false;
          }
        } else {
          item.hiddleChat = false;
        }
      });
      console.log(2222222, this.sessions);
      // this.usDateSessionsUI()// 更新UI
    },
    // 获取历史记录
    getHistoryMsgs(e) {
      this.nim.getHistoryMsgs({
        scene: "p2p",
        to: e,
        asc: true,
        done: getHistoryMsgsDone
      });
      const that = this;
      function getHistoryMsgsDone(error, obj) {
        if (!error) {
          console.log("获取历史记录", obj.msgs);
          const msgs = obj.msgs;
          that.chatInfo = msgs;
          that.$refs.common_chat.goEnd();
        }
      }
    },
    // 名片
    onMyInfo(user) {
      console.log("收到我的名片", user);
      this.myInfo = user;
    },
    onUpdateMyInfo(user) {
      console.log("我的名片更新了", user);
      this.myInfo = SDK.NIM.util.merge(this.myInfo, user);
    },
    onUsers(users) {
      console.log("收到用户名片列表", users);
      this.users = this.nim.mergeUsers(this.users, users);
    },
    onUpdateUser(user) {
      console.log("用户名片更新了", user);
      this.users = this.nim.mergeUsers(this.users, user);
    },
    // 获取用户信息
    getUserInfo() {
      console.log("获取用户信息88888");
      const accounts = this.sessions.map(item => {
        if (item.lastMsg) {
          if (item.lastMsg.flow === "in") {
            return item.lastMsg.from;
          } else {
            return item.lastMsg.to;
          }
        }
      });
      console.log(7777, accounts);
      this.nim.getUsers({
        accounts: accounts,
        done: (error, users) => {
          console.log("获取用户信息8888888");
          if (!error) {
            this.accountList = users.map((it, idx) => {
              for (let i = 0; i < this.sessions.length; i++) {
                if (this.sessions[i].lastMsg.flow === "in") {
                  if (it.account === this.sessions[i].lastMsg.from) {
                    console.log(33333, it);
                  }
                } else {
                  if (it.account === this.sessions[i].lastMsg.to) {
                    console.log(444444, it);
                  }
                }
              }
              return it;
            });
          }
        }
      });
    },
    // 选中了会话
    selectedChat(e) {
      console.log("选中了会话", e);
      this.msgsUser = e;
      this.getHistoryMsgs(e.to);
    },
    // 转移客服
    transferChat(e) {
      this.sendMsg(e, { transfer: true });
    },
    /**
     * 发送消息
     * @param {Object} rs 回调对象
     * @Chat 判断是否正在聊天中
     */
    sendMsg(rs, transfer) {
      const transferInfo = (transfer && transfer["transfer"]) || this.transfer;
      console.log("发送消息", "ls");
      this.nim.sendText({
        scene: "p2p",
        // to: this.msgsUser.to,
        to: "ls",
        text: rs,
        custom: JSON.stringify({
          chatType: this.chatType,
          sysUserId: this.sysUserId,
          transfer: transferInfo,
          // customerAccid: this.msgsUser.to,
          customerAccid: "ls",
          // customerId: this.msgsUser.lastMsg["customerId"]
          customerId: this.msgsUser.lastMsg["customerId"]
        }),
        done: sendMsgDone
      });
      const self = this;
      function sendMsgDone(error, msg) {
        if (!error) {
          self.chatInfo.push(msg);
          self.$refs.common_chat.goEnd();
        }
      }
    },
    // 预览文件
    previewFile() {
      const self = this;
      self.nim.previewFile({
        type: "image",
        fileInput: "common_chat_opr_fileUpload",
        done: function(error, file) {
          if (!error) {
            console.log(file);
          }
        }
      });
    },
    // 上传图片
    fileUpload() {
      const self = this;
      self.nim.sendFile({
        scene: "p2p",
        to: self.msgsUser.to,
        type: "image",
        custom: JSON.stringify({
          chatType: self.chatType,
          sysUserId: self.sysUserId,
          transfer: self.transfer,
          customerAccid: self.msgsUser.to,
          customerId: this.msgsUser.lastMsg["customerId"]
        }),
        fileInput: "common_chat_opr_fileUpload",
        done: sendMsgDone
      });
      function sendMsgDone(error, msg) {
        if (!error) {
          self.chatInfo.push(msg);
          self.$refs.common_chat.goEnd();
        }
      }
    },

    emotion(val) {
      console.log(val);
      this.sendMsg(val);
    },
    Unfold() {
      this.isRightShow = !this.isRightShow;
    },
    screenResize() {
      this.screenWidth = document.documentElement.clientWidth;
      this.screenHeight = document.documentElement.clientHeight;
    },
    clickGetAllPeople() {
      console.log("获取右侧群组人员");
    },
  }
};
</script>
<style lang="scss">
.home{
  display: flex;
  .home-list{
    width: 200px;
    height: 100vh;
    border-right: 1px solid #000;
    ul{
      list-style: none;
      li{
        height: 30px;
        line-height: 30px;
        border-bottom: 1px solid #000;
        cursor: pointer;
      }
    }
  }
  .chat-main{
    height: 100vh;
    width: calc(100vw - 200px);
  }
}

</style>
