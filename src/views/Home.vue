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
import { mapState } from "vuex";
import util from '../utils'

export default {
  data() {
    return {
      
    };
  },

  watch: {
    
  },
  components: {

  },
  computed: {
    sessionlist () {
      let sessionlist = this.$store.state.sessionlist.filter(item => {
        item.name = ''
        item.avatar = ''
        if (item.scene === 'p2p') {
          let userInfo = null
          if (item.to !== this.myPhoneId) {
            userInfo = this.userInfos[item.to]
          } else {
            // userInfo = this.myInfo
            // userInfo.alias = '我的手机'
            // userInfo.avatar = `${config.myPhoneIcon}`
            return false
          }
          if (userInfo) {
            item.name = util.getFriendAlias(userInfo)
            item.avatar = userInfo.avatar
          }
        } else if (item.scene === 'team') {
          let teamInfo = null
          teamInfo = this.$store.state.teamlist.find(team => {
            return team.teamId === item.to
          })
          if (teamInfo) {
            item.name = teamInfo.name
            item.avatar = teamInfo.avatar || (teamInfo.type === 'normal' ? this.myGroupIcon : this.myAdvancedIcon)
          } else {
            item.name = `群${item.to}`
            item.avatar = item.avatar || this.myGroupIcon
          }
        }
        let lastMsg = item.lastMsg || {}
        if (lastMsg.type === 'text') {
          item.lastMsgShow = lastMsg.text || ''
        } else if (lastMsg.type === 'custom') {
          item.lastMsgShow = util.parseCustomMsg(lastMsg)
        } else if (lastMsg.scene === 'team' && lastMsg.type === 'notification') {
          item.lastMsgShow = util.generateTeamSysmMsg(lastMsg)
        } else if (util.mapMsgType(lastMsg)) {
          item.lastMsgShow = `[${util.mapMsgType(lastMsg)}]`
        } else {
          item.lastMsgShow = ''
        }
        if (item.updateTime) {
          item.updateTimeShow = util.formatDate(item.updateTime, true)
        }
        return item
      })
      return sessionlist
    }
  },
  mounted() {
   
  },
  methods: {
    
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
