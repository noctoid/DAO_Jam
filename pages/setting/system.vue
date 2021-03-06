<template>
  <userLayout>
    <template slot="main">
      <h2 class="tag-title">
        {{ $t('user.systemSetting') }}
      </h2>
      <div class="list">
        <span class="title">{{ $t('user.transfer') }}</span>
        <el-switch
          v-model="isTransfer"
          @change="changeTransfer"
          active-color="#542DE0"
        />
      </div>
      <div class="list">
        <el-button @click="clearCache" type="danger" icon="el-icon-delete">
          一键清除缓存
        </el-button>
      </div>

      <div class="list">
        <a class="href" target="_blank" href="https://www.yuque.com/matataki">
          帮助和支持
        </a>
      </div>

    </template>
    <template slot="nav">
      <myAccountNav />
    </template>
  </userLayout>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'
import userLayout from '@/components/user/user_layout.vue'
import myAccountNav from '@/components/my_account/my_account_nav.vue'
import store from '@/utils/store.js'
import { removeCookie, clearAllCookie } from '@/utils/cookie'

export default {
  components: {
    userLayout,
    myAccountNav
  },
  data() {
    return {
      isTransfer: true
    }
  },
  mounted() {
    this.getMyUserData()
  },
  methods: {
    ...mapActions(['resetAllStore']),
    // 获取用户信息 - 转让状态
    async getMyUserData() {
      const res = await this.$API.getMyUserData().then(res => {
        if (res.code === 0) {
          this.isTransfer = !!res.data.accept
        } else console.log('获取用户信息失败')
      }).catch(err => {
        console.log(`获取用户信息失败${err}`)
      })
    },
    // 改变转让状态
    async changeTransfer(status) {
      try {
        this.articleTransfer = status
        const accept = status ? 1 : 0
        const res = await this.$backendAPI.setProfile({ accept })
        if (res.status === 200 && res.data.code === 0) {
          this.$message({
            message: this.$t('success.success'),
            type: 'success'
          })
        } else {
          this.$message.error(this.$t('error.fail'))
          this.articleTransfer = !status
        }
      } catch (error) {
        this.articleTransfer = !status
        console.log(`转让状态错误${error}`)
        this.$message.error(this.$t('error.fail'))
      }
    },
    clearCache() {

      // 出错后弹出框提示
      const alertDialog = () => {
        this.$alert('很抱歉，退出登录失败，点击确定刷新', '温馨提示', {
          showClose: false,
          type: 'success',
          callback: action => {
            window.location.reload()
          }
        })
      }

      // 清除
      const clear = () => {
        // 重置all store
        this.resetAllStore()
          .then(res => {
            clearAllCookie()
            // 防止没有清除干净
            removeCookie('ACCESS_TOKEN')
            removeCookie('idProvider')
            removeCookie('referral')
            store.clearAll()
            sessionStorage.clear()
            this.$router.replace({
              name: 'article'
            })

            // 通知刷新其他页面
            setTimeout(() => {
              this.$userMsgChannel.postMessage('logout')
            }, 2000)

          }).catch(err => {
            console.log(err)
            alertDialog()
          })
      }

      this.$confirm('清除浏览器缓存, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        clear()
      }).catch(() => { })
    }
  }
}
</script>

<style lang="less" scoped>

.title {
  font-size:16px;
  font-weight:400;
  color:#333;
  line-height:28px;
  margin-right: 20px;
}
.list {
  margin: 20px 0 0 10px;
  a {
    color: #333;
    text-decoration: underline;
  }
}
.tag-title {
  font-weight: bold;
  font-size: 20px;
  padding-left: 10px;
  margin: 0;
}
</style>
