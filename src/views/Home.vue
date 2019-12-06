<template>
  <div class="home">
    1111111111
  </div>
</template>
<script>
// import SDK from "./sdk/NIM_Web_SDK_v7.0.3.js";
const SDK = require("./../sdk/NIM_Web_SDK_v7.0.3.js");

export default {
  data() {
    return {
      nim: null,
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
      type: "chatroom"
    };
  },
  async created() {
    await this.initSDK();
  },
  watch: {
    screenWidth(val) {
      if (val) {
        if (val < 1100) {
          this.isRightShow = false;
        } else {
          this.isRightShow = true;
        }
      }
    }
  },
  props: { isLeader: Boolean, params: Object },
  components: {

  },
  mounted() {
    window.onresize = () => {
      this.screenResize();
    };
  },
  methods: {
    initSDK() {
      this.nim = SDK.NIM.getInstance({
        // debug: true,
        appKey: "fe416640c8e8a72734219e1847ad2547",
        // account: '',
        // token: ''
        // privateConf: {}, // 私有化部署方案所需的配置
        done: function sendMsgDone(error, msg) {
          console.log(error);
          console.log(msg);
          // ...
        },
        onconnect: this.onConnect, // 链接已经建立(登录成功)
        onwillreconnect: this.onWillReconnect,
        ondisconnect: this.onDisconnect,
        onerror: this.onError,
        // 用户名片
        onmyinfo: this.onMyInfo,
        onupdatemyinfo: this.onUpdateMyInfo,
        onusers: this.onUsers,
        onupdateuser: this.onUpdateUser,
        // // 会话
        syncSessionUnread: true,
        onsessions: this.onSessions,
        onupdatesession: this.onUpdateSession,
        // 同步完成
        onsyncdone: this.onSyncDone
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
      console.log(event);
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
      console.log(88888, this.users);
    },
    onUpdateUser(user) {
      console.log("用户名片更新了", user);
      this.users = this.nim.mergeUsers(this.users, user);
    },
    // 获取用户信息
    getUserInfo() {
      console.log(3333);
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
          console.log(888);
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
      console.log("发送消息", this.msgsUser.to);
      this.nim.sendText({
        scene: "p2p",
        to: this.msgsUser.to,
        text: rs,
        custom: JSON.stringify({
          chatType: this.chatType,
          sysUserId: this.sysUserId,
          transfer: transferInfo,
          customerAccid: this.msgsUser.to,
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
    clickCommonly() {
      this.chooseChatData.isCommonly = !this.chooseChatData.isCommonly;
    },
    changeCenter(data, className) {
      this.$refs["communication-chat"].scrollDom();
      console.log(data);
      this.rightType = data.type;
      this.chooseChatData = data;
      this.centerVisible = true;
      if (className == "commonly-used") {
        this.chooseChatData.isCommonly = true;
      }
    },
    orgClickFun() {
      this.centerVisible = false;
    },
    treeNodeClick(data) {
      console.log(data);
      this.rightType = "empBriefProduce";
    },
    handleBusiness(from) {
      this.fromType = from;
      this.rightType = "handleBusiness";
    },
    handleBusinessBack() {
      this.rightType = this.fromType;
    }
  }
};
</script>
<style lang="scss">

</style>
