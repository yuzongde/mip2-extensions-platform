<template>
  <div class="wrapper">
    <mip-inservice-login
      id="log"
      :config="config"
      on="login:example.customLogin logout:example.customLogout"/>
    <div class="mybg">
      <img
        :src="userInfo.iconUrl?userInfo.iconUrl:'http://www.daoway.cn/images/myicon.png'"
        class="myicon">
      <div
        v-if="userId"
        class="txt">
        <div class="username">{{ userInfo.nick }}
          <img
            v-if="userInfo"
            :src="userInfo.levelIcon"
            class="starimg">
        </div>
      </div>
      <div
        v-if="!userId"
        class="txt2"
        on="tap:log.login">请点击登录</div><!--@click="goLoginPage"-->
    </div>
    <div
      v-if="userInfo.couponCount"
      class="mylist"
      style="margin-top:0; border:0"
      @click="goVouchersPage" >
      <div class="item">
        <img
          src="http://www.daoway.cn/images/quan.png"
          class="cartimg"
          style="width:20px; height:20px">
        <div class="cart">代金券</div>
        <div class="carttel">{{ userInfo.couponCount }}张<img
          class="m-more"
          src="https://www.daoway.cn/h5/image/go_06.png"></div>
      </div>
    </div>
    <div class="mylist">
      <div class="item" >
        <img
          src="http://www.daoway.cn/images/tel.png"
          class="cartimg"
          style="width:20px; height:20px">
        <div class="cart">联系客服</div>
        <div class="carttel"><a
          href="tel:400-0908-608"
          rel="external">400-0908-608</a><img
            class="m-more"
            src="https://www.daoway.cn/h5/image/go_06.png"></div>
      </div>
    </div>
    <div class="mylist">
      <div
        class="item"
        @click="toabout">
        <img
          src="http://www.daoway.cn/images/about.png"
          class="cartimg"
          style="width:20px; height:20px">
        <div class="cart">关于</div>
        <div class="carttel"><img
          class="m-more"
          src="https://www.daoway.cn/h5/image/go_06.png"></div>
      </div>
    </div>
    <div
      class="exit"
      on="tap:log.logout"
      @click="exit">退出登录</div><!-- @click="loginOut"-->

    <!--提示-->
    <div
      v-show="warn.show"
      class="layer">
      <div class="layer-content zoomIn">
        <p
          class="layer-text"
          v-text="warn.texts"/>
        <p
          class="layer-sure active-layer"
          @click="closeLayer">知道了</p>
      </div>
    </div>
    <mip-fixed type="bottom">
      <div class="bottomnav">
        <a
          data-type="mip"
          data-title="到位上门服务"
          @click="toindex"
        ><img src="http://www.daoway.cn/mip/common/images/home2.png">首页</a>
        <a
          data-type="mip"
          data-title="订单"
          @click="toorder"><img src="http://www.daoway.cn/mip/common/images/order2.png">订单</a>
        <a
          class="regclolr"
          data-type="mip"
          data-title="我的"
          @click="tomy"><img src="http://www.daoway.cn/mip/common/images/my.png">我的</a>
      </div>
    </mip-fixed>

  </div>
</template>
<script>
import base from '../../common/utils/base'
import '../../common/utils/base.less'

export default {
  props: {
    info: {
      type: Object,
      required: true
    },
    config: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      phoneNumber: '400-0908-608',
      iconUrl: 'http://www.daoway.cn/mip/common/images/myicon.png',
      couponCount: 0,
      userInfo: {},
      warn: {
        // 弹窗
        show: false,
        texts: ''
      },
      redirect_uri: base.htmlhref.my,
      client_id: 'vnQZ7pPB0gsWHZZF4n6h0WDOl8KOr7Lq',
      ClientSecret: 'kM6rbBN43zhAEOFxeQ9Wnj2MzVzkROA0',
      userId: '',
      token: ''
    }
  },
  mounted () {
    let userId = localStorage.getItem('mipUserId')
    let token = localStorage.getItem('mipToken')
    if (userId && token) {
      this.userId = userId
      this.token = token
      this.getmyhtml()
    } else {
      this.$element.customElement.addEventAction('customLogin', event => {
        this.info = event.userInfo
        this.userId = event.userInfo.userId
        this.token = event.userInfo.token
        this.getmyhtml()
        localStorage.setItem('mipUserId', event.userInfo.userId)
        localStorage.setItem('mipToken', event.userInfo.token)
        localStorage.setItem('nick', event.userInfo.nick)
      })
    }
  },
  methods: {
    exit () {
      this.userId = ''
      this.token = ''
      this.userInfo.couponCount = false
      localStorage.removeItem('mipUserId')
      localStorage.removeItem('mipToken')
      localStorage.removeItem('nick')
      MIP.viewer.open(base.htmlhref.my, {isMipLink: false})
    },
    showloading () {
      this.loading = true
    },
    closeLayer () {
      this.warn.show = false
    },
    getmyhtml () {
      let that = this
      let url = '/daoway/rest/user/' + that.userId + '?isowner=1'
      fetch(url, {
        method: 'get',
        credentials: 'include',
        headers: {
          'content-type': 'application/x-www-form-urlencoded'
        }
      }).then(function (res) {
        return res.json()
      }).then(function (text) {
        if (text.status === 'ok') {
          let data = text.data
          let userInfo = {}
          if (data.iconUrl) {
            userInfo.iconUrl = data.iconUrl ? data.iconUrl : 'http://www.daoway.cn/images/myicon.png'
          }
          userInfo.nick = data.nick
          userInfo.couponCount = data.couponCount
          let level = data.level
          if (level === '100') { // 青铜会员
            userInfo.levelIcon = 'http://www.daoway.cn/mip/common/images/q.png'
            userInfo.levelhtml = '青铜会员'
          } else if (level === '200') { // 白银会员
            userInfo.levelhtml = '白银会员'
            userInfo.levelIcon = 'http://www.daoway.cn/mip/common/images/b.png'
          } else if (level === '300') { // 黄金会员
            userInfo.levelhtml = '黄金会员'
            userInfo.levelIcon = 'http://www.daoway.cn/mip/common/images/h.png'
          } else if (level === '400') { // 钻石会员
            userInfo.levelhtml = '钻石会员'
            userInfo.levelIcon = 'http://www.daoway.cn/mip/common/imagesz.png'
          } else {
            userInfo.levelIcon = 'http://www.daoway.cn/mip/common/images/p.png'
          }
          that.userInfo = userInfo
        } else {
          that.warn.show = true
          that.warn.texts = text.msg
        }
      }).catch(function (error) {
        console.log(error)
      })
    },
    goVouchersPage: function () {
      let miniUserId = this.userId
      if (miniUserId) {
        MIP.viewer.open(base.htmlhref.vouchers + '?from=my', { isMipLink: true })
      } else {
        this.userId = ''
      }
    },
    toabout () {
      MIP.viewer.open(base.htmlhref.about, { isMipLink: true })
    },
    toindex () {
      MIP.viewer.open(base.htmlhref.index, {isMipLink: false})
    },
    toorder () {
      MIP.viewer.open(base.htmlhref.order, {isMipLink: false})
    },
    tomy () {
      MIP.viewer.open(base.htmlhref.my, {isMipLink: false})
    }
  }
}
</script>
<style scoped>
    * {
        padding: 0;
        margin: 0;
        box-sizing: border-box;
    }

    i {
        font-style: normal
    }

    li, ol {
        list-style: none
    }
    .regclolr{
        color:#f64e4e ;
    }

    .bottomnav{
        width: 100%;
        background: #fff;
        border-top: 1px solid #ededed;
    }
    .bottomnav a{
        line-height: 23px;
        display: inline-block;
        width: 32%;
        text-align: center;
        font-size: 12px;
        margin-top: 5px;
    }
    .bottomnav a img{
        width: 25px;
        height: auto;
        display: block;
        text-align: center;
        margin: 0 auto;
    }
    .regclolr{
        color: red;
    }
    .layer-text{
        text-align: center;
    }

    .mybg {
        width: 100%;
        height: 134px;
        position: relative;
        padding: 35px 15px;
        color: #fff;
        background: -webkit-linear-gradient(#fd5f46, #fd322b);
        background: linear-gradient(#fd5f46, #fd322b);

    }

    .username, .txt, .starimg, .myicon, .set, .rightarrow {
        display: inline-block;
    }
    .username{
        font-size: 18px;
    }

    .myicon {
        width: 64px;
        height: 64px;
        border-radius: 100%;
        border: 3px solid #fff;
        margin-left: 10px;
    }

    .txt {
        width: 60%;
        line-height: 30px;
        vertical-align: top;
        margin-top: 15px;
        margin-left: 10px;
    }

    .txt2 {
        width: 20%;
        line-height: 30px;
        vertical-align: top;
        margin-top: 15px;
        margin-left: 10px;
        display: inline-block;
    }

    .starimg {
        width: 17px;
        height: 17px;
        position: relative;
        top: 2px;
        margin-right: 3px;
    }

    .mylist {
        width: 100%;
        background: #fff;
        margin-top: 10px;
    }

    .cart {
        display: inline-block;
        color: #4c4c4c;
        font-size: 16px
    }

    .cartimg {
        position: absolute;
        top: 12px;
        left: 1.5%;
    }

    .item {
        margin-left: 1.5%;
        width: 97%;
        height: 40px;
        line-height: 40px;
        position: relative;
    }

    .cart {
        margin-left: 10%;
    }

    .carttel {
        float: right;
        margin-right: 3%;
        color: #898989;
        font-size: 14px
    }

    .exit {
        text-align: center;
        width: 94%;
        margin: 0 3%;;
        border: none;
        border-radius: 5px;
        height: 40px;
        line-height: 40px;
        background: #fff;
        margin-top: 20px;
        color: #4c4c4c;
        font-size: 16px
    }
    .m-more{
        width: 7px;
        height: auto;
        margin-left: 5px;
    }

</style>
