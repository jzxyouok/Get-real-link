<template>
  <div class="wrapper">
    <h2 class="platform">选择平台</h2>
    <div class="classify">
      <ul>
        <li v-for="(item, index) in list" :key="index" :class="item.active ? 'active' : ''" @click="setSource(item.title, index, item.url)">{{ item.title }}<img v-if="!item.url" src="../../../static/images/ok.svg"></li>
      </ul>
    </div>
    <h2>{{ title }}</h2>
    <div class="get">
      <input :disabled="readonly" type="text" v-model="input" placeholder="输入短视频链接…">
      <div class="btn" @click="getUrl">立即还原</div>
    </div>
    <h2>{{ reTitle }}</h2>
    <div class="res">
      <div v-for="(item, index) in result" :key="index" class="result" @click="copy(item)">{{ item }}</div>
    </div>
    <h2>如何使用？</h2>
    <div class="tips">
      1、通过分享视频复制链接，选择对应的平台后粘贴到输入框中点击还原。
      <br>
      2、点击复制链接在浏览器中打开，然后下载、分享或保存到相册。
      <br>
      3、下载：iOS 可复制链接配合 快捷指令 下载，相关快捷指令关注公众号：Slothful 回复“指令”获取。
      <p>关注公众号：Slothful 了解更多</p>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      readonly: false,
      title: '抖音',
      input: '',
      url: 'https://fiume.cn/tools/api/video/douyin/',
      result: ['The result will be displayed here.'],
      list: [
        {title: '抖音', url: 'https://fiume.cn/tools/api/video/douyin/', active: true},
        {title: '快手', url: 'https://fiume.cn/tools/api/video/pipixia/', active: false},
        {title: '皮皮虾', url: 'https://fiume.cn/tools/api/video/pipixia/', active: false},
        {title: '西瓜', url: 'https://fiume.cn/tools/api/video/pipixia/', active: false},
        {title: '火山', url: 'https://fiume.cn/tools/api/video/pipixia/', active: false},
        {title: '秒拍', url: 'https://fiume.cn/tools/api/video/pipixia/', active: false},
        {title: '全民', url: 'https://fiume.cn/tools/api/video/pipixia/', active: false},
        {title: '其它', url: 'https://fiume.cn/tools/api/video/pipixia/', active: false}
      ]
    }
  },
  computed: {
    reTitle () {
      return `${this.result.length} 条结果`
    }
  },
  watch: {
    input () {
      let temp = this.input.match(/(http)(.*)(?= )/g)
      if (temp) {
        this.input = temp[0]
        wx.showToast({
          title: '已格式化链接',
          icon: 'success',
          duration: 1200
        })
      }
    }
  },
  methods: {
    setSource (title, index, url) {
      this.list.map((ele) => {
        ele.active = false
      })
      this.list[index].active = true
      this.title = title
      this.input = ''
      this.readonly = false
      this.url = url
      if (this.url === false) {
        this.input = `暂不支持 ${this.title}`
        this.readonly = true
      }
    },
    getUrl () {
      if (this.input.indexOf('http') === -1) {
        wx.showModal({
          title: '错误',
          content: '请检查你的链接后继续',
          showCancel: false
        })
        return
      }
      wx.showLoading({
        title: '正在获取…',
        mask: true
      })
      this.getAjax()
    },
    getAjax () {
      wx.request({
        url: this.url,
        data: {
          url: this.input
        },
        header: {
          'content-type': 'application/json'
        },
        success: (res) => {
          this.opResult(res.data)
        },
        fail () {
          wx.hideLoading({
            success: () => {
              wx.showModal({
                title: '错误',
                content: '未知错误，请尝试重试',
                showCancel: false
              })
            }
          })
        }
      })
    },
    opResult (data) {
      switch (data.status) {
        case true:
          this.result = data.urls
          this.hideLoad()
          break
        case 0:
          let {realDownloadURL, videoURL} = data.data
          this.result = [realDownloadURL, videoURL]
          this.hideLoad()
          break
        default:
          wx.hideLoading({
            success: () => {
              wx.showModal({
                title: '错误',
                content: '未知错误，请尝试重试',
                showCancel: false
              })
            }
          })
      }
    },
    hideLoad () {
      wx.hideLoading({
        success: () => {
          wx.showToast({
            title: this.reTitle,
            icon: 'success',
            duration: 1000
          })
        }
      })
    },
    copy (url) {
      wx.setClipboardData({
        data: url,
        success (res) {
          wx.showToast({
            title: '已复制',
            icon: 'success',
            duration: 2200
          })
        }
      })
    }
  }
}
</script>

<style scoped>
  .wrapper{
    min-height: 100vh;
    padding: 0 16px 20px;
    background: #f3f3f8;
    overflow: hidden;
  }
  h2{
    font-size: 20px;
    margin: 20px 0 6px;
    font-weight: bold;
    color: #010101;
  }
  div,li{
    font-size: 16px;
  }
  .classify{
    border-radius: 12px;
    background: #fff;
  }
  .classify ul{
    display: flex;
    flex-wrap: wrap;
  }
  .classify ul li{
    position: relative;
    width: 25%;
    height: 50px;
    line-height: 50px;
    text-align: center;
    color: #8a8a8e;
  }
  .classify ul li.active{
    font-weight: bold;
    color: #ff6834;
  }
  .classify ul li img{
    position: absolute;
    top: 30%;
    left: 50%;
    transform: translateX(160%);
    width: 12px;
    height: 12px;
  }
  .get input[type='text']{
    height: 36px;
    border-radius: 8px;
    padding-left: 8px;
    background: #e4e3e9;
    color: #8a8a8e;
  }
  .get div.btn{
    height: 40px;
    line-height: 40px;
    margin-top: 8px;
    text-align: center;
    border-radius: 8px;
    background: #fff;
    color: #1a85fe;
  }
  .get div.btn:active{
    background: rgb(253, 253, 253);
    color: #4a9fff;
  }
  .result{
    width: 100%;
    height: auto;
    padding: 6px 8px;
    background: #fff;
    color: #8a8a8e;
    border-radius: 10px;
    font-size: 12px;
    border: 2px solid transparent;
    box-sizing: border-box;
    word-break: break-all;
    word-wrap: break-word;
    transition: all .3s;
  }
  .res .result:not(:last-child){
    position: relative;
    z-index: 2;
    margin-bottom: 8px;
  }
  .res .result:not(:last-child)::after{
    content: '';
    display: block;
    width: 4px;
    height: 8px;
    position: absolute;
    top: calc(100% + 2px);
    left: 50%;
    z-index: 1;
    transform: translateX(-50%);
    background: #e4e3e9;
  }
  .result:active{
    border-color: #ffa485;
  }
  .tips{
    font-size: 14px;
    border-radius: 12px;
    padding: 10px 14px;
    color: #8a8a8e;
    background: #fff;
  }
  .tips p{
    text-align: right;
    font-size: 12px;
  }
</style>
