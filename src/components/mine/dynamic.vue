<template>
  <div class="dynamic" style="height:100%;">
      <x-header :left-options="{backText: ''}" style="background-color:rgba(180,200,200,0.3);">
        <div style="color:#212121;font-family:'华文行楷'">我的动态</div>
        </x-header>
      <scroller lock-x scrollbar-y use-pullup use-pulldown @on-pullup-loading="loadMore" @on-pulldown-loading="refresh" @on-scroll="onScroll" ref="scroller" height="-46" v-model="status">
        <div>
          <div class="dynamic-headbg">
            <div>
              <img class="dynamic-headerpro" src="../../assets/img/my_head.png" width="66" height="66">
              <span class="dynamic-headername">施景程</span>
            </div>
          </div>
          <group>
            <div @on-click-item ="showinfo">
              <cell align-items="flex-start" @click.native="show1 = true" value-align="left" v-for="(pcircle,index) in pcircles" :key="index" :id="'pid'+index">
                <img slot="title" :src="pcircle.headimg" width="45" height="45" style="margin-right:1em;border-radius: 6px;"> <!--发表说说的用户头像 -->
                <div>
                  <span class="post-name">{{pcircle.name}} </span> <!--发表说说的用户昵称 -->
                  <div class="post-text" v-html="pcircle.pcontent"> </div> <!--说说的内容 -->
                  <div v-show="pcircle.imgurl && pcircle.imgurl.length>0"> <!-- 判断是否有图片-->
                    <img :class="'previewer-demo-img'+index" v-for="(list,imgindex) in pcircle.imgurl" :key="imgindex" :src="list.src" width="100" :style="{width: imgWidth, height: imgWidth}" @click="show(pcircle.imgurl,imgindex,index)" >
                  </div>
                  <div class="post-time-ico">
                    <div class="post-time">{{pcircle.adddate}}</div> <!--发表说说的时间 -->
                    <popover placement="left" @on-show="openpop()" @on-hide="closepop()" :gutter=10 > <!--评论点赞功能弹窗 -->
                      <div slot="content" class="popover-demo-content" v-show="openwindowshow.pop">
                        <div class="like"  @click="isgood(index)">
                          <i class="fa fa-heart-o" aria-hidden="true"></i>
                          {{ pcircle.ispr?'取消': '赞' }}
                        </div>
                        <div class="like" @click="openpopup(index)">
                          <i class="fa fa-comment-o" aria-hidden="true"></i>
                          评论
                        </div>
                      </div>
                      <i class="fa fa-flickr post-icon" ></i>
                    </popover>
                  </div>
                </div>
                <div class="post-content" v-show="pcircle.TbPcpraise.length>0 || pcircle.TbPccomment.length>0">
                  <div style="border-bottom: 1px solid #D9D9D9;" v-show="pcircle.TbPcpraise && pcircle.TbPcpraise.length>0">
                    <i class="fa fa-heart-o"> </i>
                    <span v-for="(like,index) in pcircle.TbPcpraise" :key="index"> {{like.uname}}, </span> <!--点赞人的昵称 -->
                  </div>
                  <div v-show="pcircle.TbPccomment && pcircle.TbPccomment.length>0" v-for="(rcomment,rindex) in pcircle.TbPccomment" :key="rindex">
                    <span>{{rcomment.runame}} :</span> <!--评论人姓名 -->
                    <span style="color:#000" @click="reply(rcomment,index)">{{rcomment.ccontent}} </span> <!--评论内容 -->
                  </div>
                </div>
              </cell>
            </div>
          </group>
        </div>
        <div slot="pullup" class="xs-plugin-pullup-container xs-plugin-pullup-up" style="position: absolute; width: 100%; height: 40px; bottom: -40px; text-align: center;">
          <span v-show="status.pullupStatus === 'loading'"><load-more tip="正在加载"></load-more></span>
        </div>
        <div slot="pulldown" class="xs-plugin-pulldown-container xs-plugin-pulldown-down" style="position: absolute; width: 100%; height: 60px; line-height: 60px; top: -60px; text-align: center;">
          <span v-show="status.pulldownStatus === 'loading'"><load-more tip="正在加载"></load-more></span>
        </div>
      </scroller>
      <div v-transfer-dom> <!--图片查看器 -->
        <previewer :list="imglist" ref="previewer" :options="options()"></previewer>
      </div>
      <div v-transfer-dom>
        <div style="position: absolute;bottom:0px;width:100%;background-color: #fbf9fe;"  v-show="openwindowshow.inputshow">
          <div class="dynamic-inputcell"> <!--评论输入框 -->
            <input type="text" :placeholder="inputplaceholder" v-model="mytext" ref="inputcomment" class="dynamic-input" @focus="onFocus" @keyup.enter="inputpcontent" @blur="onblur">
            <i slot="right" :class="openwindowshow.IsKeyorEmo?'fa fa-smile-o':'fa fa-keyboard-o'" style="font-size:34px;padding-left:5px;color: #999999;" @click="showemotion()" ></i>
          </div>
          <swiper dots-position="center" height="175px" v-show="openwindowshow.emotionshow"> <!--表情包 -->
            <swiper-item class="black" v-for="(i,index) in 5" v-bind:key="index" >
              <section class="wechatEmotion-container" >
              <ul class="emotion-list">
                <li v-for="(item,index) in list.slice(23*(i-1),23*i)" v-bind:key="index"  class="vux-center-h emotion-list-item" @click="addEmotion(item)">
                  <emotion is-gif>{{item}}</emotion>
                </li>
              </ul>
              </section>
            </swiper-item>
          </swiper>
        </div>
      </div>
      <dynamicpop :show1="show1" v-on:closepop1="show1 =$event"/>
  </div>
</template>
<script>
import dynamicpop from '../../components/mine/dynamicpop.vue'
import { XHeader, TransferDom, Cell, Group, Popover, Previewer, Scroller, LoadMore, WechatEmotion as Emotion, Swiper, SwiperItem } from 'vux'
export default {
  name: 'dynamic', // 朋友圈
  directives: {
    TransferDom
  },
  components: { XHeader, dynamicpop, Cell, Group, Popover, Previewer, Scroller, LoadMore, Emotion, Swiper, SwiperItem }, // 注册组件
  data () { // 局内数据
    return {
      show1: false,
      imgWidth: '80px', // 说说图片大小
      pid: 0, // 用于给当前说说/评论进行编号
      openwindowshow: {
        pop: false, // 点赞评论功能弹窗显示判断
        inputshow: false, // 输入框弹窗显示判断
        emotionshow: false, // 表情框显示判断
        IsKeyorEmo: true // true:表情 false：键盘
      },
      good: '赞', // 赞/取消
      imglist: [], // 存储图片
      mytext: '', // input的value
      inputplaceholder: '评论', // 评论/ **回复 **
      replycomment: '', // 要回复那个人所发的评论信息
      list: ['微笑', '撇嘴', '色', '发呆', '得意', '流泪', '害羞', '闭嘴', '睡', '大哭', '尴尬', '发怒', '调皮', '呲牙', '惊讶', '难过', '酷', '冷汗', '抓狂', '吐', '偷笑', '可爱', '白眼', '傲慢', '饥饿', '困', '惊恐', '流汗', '憨笑', '大兵', '奋斗', '咒骂', '疑问', '嘘', '晕', '折磨', '衰', '骷髅', '敲打', '再见', '擦汗', '抠鼻', '鼓掌', '糗大了', '坏笑', '左哼哼', '右哼哼', '哈欠', '鄙视', '委屈', '快哭了', '阴险', '亲亲', '吓', '可怜', '菜刀', '西瓜', '啤酒', '篮球', '乒乓', '咖啡', '饭', '猪头', '玫瑰', '凋谢', '示爱', '爱心', '心碎', '蛋糕', '闪电', '炸弹', '刀', '足球', '瓢虫', '便便', '月亮', '太阳', '礼物', '拥抱', '强', '弱', '握手', '胜利', '抱拳', '勾引', '拳头', '差劲', '爱你', 'NO', 'OK', '爱情', '飞吻', '跳跳', '发抖', '怄火', '转圈', '磕头', '回头', '跳绳', '挥手', '激动', '街舞', '献吻', '左太极', '右太极'],
      status: {
        pullupStatus: 'default', // 上拉加载状态
        pulldownStatus: 'default' // 下拉加载状态
      },
      pcircles: [
        {
          pcid: 1, // 朋友圈编号
          uid: 1, // 发布动态人id
          name: '施景程', // 发布动态人姓名
          pcontent: '值班地点: 暮云镇居委会二楼办公室联系方式: 18874083707 ', // 发布内容
          headimg: require('../../assets/img/my_head.png'), // 头像地址
          imgurl: [
            {
              id: 1,
              src: require('../../assets/img/party/affair1.jpg'),
              w: 800,
              h: 400
            }
          ],
          adddate: '2019-8-13', // 发布动态时间
          prototal: 2, // 点赞总人数
          TbPccomment: [ // 评论
            {
              coid: 1, // 评论id
              pcid: 1, // 所属动态的id
              replyid: 0, // 回复某评论的coid
              uid: 1, // 评论人id
              uname: '施景程', // 评论人姓名
              ccontent: '评论测试', // 评论内容
              cdate: '2018-11-11 17:57:53', // 评论日期
              isusercom: 1, // 当前用户是否使这条评论的主人
              runame: '施景程', // 重置回复人姓名 两种格式1.评论人姓名 2.评论人姓名 回复 被评论人姓名
              aid: 430121101001, // 用户所属地区村社区 aid
              to_uid: 0, // 被评论用户id
              to_uname: null // 被评论用户姓名
            },
            {
              coid: 2, // 评论id
              pcid: 1, // 所属动态的id
              replyid: 0,
              uid: 1,
              uname: '王李沙',
              ccontent: '评论回复测试',
              issensitive: 0,
              cdate: '2018-11-11 17:57:53',
              isusercom: 1,
              runame: '王李沙 回复 施景程',
              aid: 430121101001,
              to_uid: 0,
              to_uname: null
            }
          ],
          TbPcpraise: [
            {
              prid: null, // 点赞编号
              pcid: 2, // 对应动态编号
              uname: '施景程', // 点赞人姓名
              uid: 1, // 点赞人uid
              aid: null
            },
            {
              prid: null, // 点赞编号
              pcid: 2, // 对应动态编号
              uname: '王李沙', // 点赞人姓名
              uid: 1, // 点赞人uid
              aid: null
            }
          ],
          ismycircle: 1, // 是否是当前用户发的动态 1.是 0.不是
          ispr: 1, // 当前用户是否已点赞 1.点了 0.没有点
          prnum: 0, // 点赞用户的序列号（显示时排在第几位）
          iscom: null, // 是否已评论
          comnum: null, // 评论用户的序列号（显示时排在第几位）
          aid: 430121101001 // 动态所属地区
        },
        {
          pcid: 1, // 朋友圈编号
          uid: 1, // 发布动态人id
          name: '施景程', // 发布动态人姓名
          pcontent: '值班地点: 暮云镇居委会二楼办公室联系方式: 18874083707 ', // 发布内容
          headimg: require('../../assets/img/my_head.png'), // 头像地址
          imgurl: [
            {
              id: 1,
              src: require('../../assets/img/party/affair2.jpg'),
              w: 800,
              h: 400
            }
          ],
          adddate: '2019-8-13', // 发布动态时间
          prototal: 2, // 点赞总人数
          TbPccomment: [ // 评论
            {
              coid: 1, // 评论id
              pcid: 1, // 所属动态的id
              replyid: 0, // 回复某评论的coid
              uid: 1, // 评论人id
              uname: '施景程', // 评论人姓名
              ccontent: '评论测试', // 评论内容
              cdate: '2018-11-11 17:57:53', // 评论日期
              isusercom: 1, // 当前用户是否使这条评论的主人
              runame: '施景程', // 重置回复人姓名 两种格式1.评论人姓名 2.评论人姓名 回复 被评论人姓名
              aid: 430121101001, // 用户所属地区村社区 aid
              to_uid: 0, // 被评论用户id
              to_uname: null // 被评论用户姓名
            },
            {
              coid: 2, // 评论id
              pcid: 1, // 所属动态的id
              replyid: 0,
              uid: 1,
              uname: '王李沙',
              ccontent: '评论回复测试',
              issensitive: 0,
              cdate: '2018-11-11 17:57:53',
              isusercom: 1,
              runame: '王李沙 回复 施景程',
              aid: 430121101001,
              to_uid: 0,
              to_uname: null
            }
          ],
          TbPcpraise: [
            {
              prid: null, // 点赞编号
              pcid: 2, // 对应动态编号
              uname: '施景程', // 点赞人姓名
              uid: 1, // 点赞人uid
              aid: null
            },
            {
              prid: null, // 点赞编号
              pcid: 2, // 对应动态编号
              uname: '王李沙', // 点赞人姓名
              uid: 1, // 点赞人uid
              aid: null
            }
          ],
          ismycircle: 1, // 是否是当前用户发的动态 1.是 0.不是
          ispr: 1, // 当前用户是否已点赞 1.点了 0.没有点
          prnum: 0, // 点赞用户的序列号（显示时排在第几位）
          iscom: null, // 是否已评论
          comnum: null, // 评论用户的序列号（显示时排在第几位）
          aid: 430121101001 // 动态所属地区
        },
        {
          pcid: 1, // 朋友圈编号
          uid: 1, // 发布动态人id
          name: '施景程', // 发布动态人姓名
          pcontent: '值班地点: 暮云镇居委会二楼办公室联系方式: 18874083707 ', // 发布内容
          headimg: require('../../assets/img/my_head.png'), // 头像地址
          imgurl: [
            {
              id: 1,
              src: require('../../assets/img/party/affair2.jpg'),
              w: 800,
              h: 400
            }
          ],
          adddate: '2019-8-13', // 发布动态时间
          prototal: 2, // 点赞总人数
          TbPccomment: [ // 评论
            {
              coid: 1, // 评论id
              pcid: 1, // 所属动态的id
              replyid: 0, // 回复某评论的coid
              uid: 1, // 评论人id
              uname: '施景程', // 评论人姓名
              ccontent: '评论测试', // 评论内容
              cdate: '2018-11-11 17:57:53', // 评论日期
              isusercom: 1, // 当前用户是否使这条评论的主人
              runame: '施景程', // 重置回复人姓名 两种格式1.评论人姓名 2.评论人姓名 回复 被评论人姓名
              aid: 430121101001, // 用户所属地区村社区 aid
              to_uid: 0, // 被评论用户id
              to_uname: null // 被评论用户姓名
            },
            {
              coid: 2, // 评论id
              pcid: 1, // 所属动态的id
              replyid: 0,
              uid: 1,
              uname: '王李沙',
              ccontent: '评论回复测试',
              issensitive: 0,
              cdate: '2018-11-11 17:57:53',
              isusercom: 1,
              runame: '王李沙 回复 施景程',
              aid: 430121101001,
              to_uid: 0,
              to_uname: null
            }
          ],
          TbPcpraise: [
            {
              prid: null, // 点赞编号
              pcid: 2, // 对应动态编号
              uname: '施景程', // 点赞人姓名
              uid: 1, // 点赞人uid
              aid: null
            },
            {
              prid: null, // 点赞编号
              pcid: 2, // 对应动态编号
              uname: '王李沙', // 点赞人姓名
              uid: 1, // 点赞人uid
              aid: null
            }
          ],
          ismycircle: 1, // 是否是当前用户发的动态 1.是 0.不是
          ispr: 1, // 当前用户是否已点赞 1.点了 0.没有点
          prnum: 0, // 点赞用户的序列号（显示时排在第几位）
          iscom: null, // 是否已评论
          comnum: null, // 评论用户的序列号（显示时排在第几位）
          aid: 430121101001 // 动态所属地区
        },
        {
          pcid: 1, // 朋友圈编号
          uid: 1, // 发布动态人id
          name: '施景程', // 发布动态人姓名
          pcontent: '值班地点: 暮云镇居委会二楼办公室联系方式: 18874083707 ', // 发布内容
          headimg: require('../../assets/img/my_head.png'), // 头像地址
          imgurl: [
            {
              id: 1,
              src: require('../../assets/img/party/affair2.jpg'),
              w: 800,
              h: 400
            }
          ],
          adddate: '2019-8-13', // 发布动态时间
          prototal: 2, // 点赞总人数
          TbPccomment: [ // 评论
            {
              coid: 1, // 评论id
              pcid: 1, // 所属动态的id
              replyid: 0, // 回复某评论的coid
              uid: 1, // 评论人id
              uname: '施景程', // 评论人姓名
              ccontent: '评论测试', // 评论内容
              cdate: '2018-11-11 17:57:53', // 评论日期
              isusercom: 1, // 当前用户是否使这条评论的主人
              runame: '施景程', // 重置回复人姓名 两种格式1.评论人姓名 2.评论人姓名 回复 被评论人姓名
              aid: 430121101001, // 用户所属地区村社区 aid
              to_uid: 0, // 被评论用户id
              to_uname: null // 被评论用户姓名
            },
            {
              coid: 2, // 评论id
              pcid: 1, // 所属动态的id
              replyid: 0,
              uid: 1,
              uname: '王李沙',
              ccontent: '评论回复测试',
              issensitive: 0,
              cdate: '2018-11-11 17:57:53',
              isusercom: 1,
              runame: '王李沙 回复 施景程',
              aid: 430121101001,
              to_uid: 0,
              to_uname: null
            }
          ],
          TbPcpraise: [
            {
              prid: null, // 点赞编号
              pcid: 2, // 对应动态编号
              uname: '施景程', // 点赞人姓名
              uid: 1, // 点赞人uid
              aid: null
            },
            {
              prid: null, // 点赞编号
              pcid: 2, // 对应动态编号
              uname: '王李沙', // 点赞人姓名
              uid: 1, // 点赞人uid
              aid: null
            }
          ],
          ismycircle: 1, // 是否是当前用户发的动态 1.是 0.不是
          ispr: 1, // 当前用户是否已点赞 1.点了 0.没有点
          prnum: 0, // 点赞用户的序列号（显示时排在第几位）
          iscom: null, // 是否已评论
          comnum: null, // 评论用户的序列号（显示时排在第几位）
          aid: 430121101001 // 动态所属地区
        },
        {
          pcid: 1, // 朋友圈编号
          uid: 1, // 发布动态人id
          name: '施景程', // 发布动态人姓名
          pcontent: '值班地点: 暮云镇居委会二楼办公室联系方式: 18874083707 ', // 发布内容
          headimg: require('../../assets/img/my_head.png'), // 头像地址
          imgurl: [
            {
              id: 1,
              src: require('../../assets/img/party/affair2.jpg'),
              w: 800,
              h: 400
            }
          ],
          adddate: '2019-8-13', // 发布动态时间
          prototal: 2, // 点赞总人数
          TbPccomment: [ // 评论
            {
              coid: 1, // 评论id
              pcid: 1, // 所属动态的id
              replyid: 0, // 回复某评论的coid
              uid: 1, // 评论人id
              uname: '施景程', // 评论人姓名
              ccontent: '评论测试', // 评论内容
              cdate: '2018-11-11 17:57:53', // 评论日期
              isusercom: 1, // 当前用户是否使这条评论的主人
              runame: '施景程', // 重置回复人姓名 两种格式1.评论人姓名 2.评论人姓名 回复 被评论人姓名
              aid: 430121101001, // 用户所属地区村社区 aid
              to_uid: 0, // 被评论用户id
              to_uname: null // 被评论用户姓名
            },
            {
              coid: 2, // 评论id
              pcid: 1, // 所属动态的id
              replyid: 0,
              uid: 1,
              uname: '王李沙',
              ccontent: '评论回复测试',
              issensitive: 0,
              cdate: '2018-11-11 17:57:53',
              isusercom: 1,
              runame: '王李沙 回复 施景程',
              aid: 430121101001,
              to_uid: 0,
              to_uname: null
            }
          ],
          TbPcpraise: [
            {
              prid: null, // 点赞编号
              pcid: 2, // 对应动态编号
              uname: '施景程', // 点赞人姓名
              uid: 1, // 点赞人uid
              aid: null
            },
            {
              prid: null, // 点赞编号
              pcid: 2, // 对应动态编号
              uname: '王李沙', // 点赞人姓名
              uid: 1, // 点赞人uid
              aid: null
            }
          ],
          ismycircle: 1, // 是否是当前用户发的动态 1.是 0.不是
          ispr: 1, // 当前用户是否已点赞 1.点了 0.没有点
          prnum: 0, // 点赞用户的序列号（显示时排在第几位）
          iscom: null, // 是否已评论
          comnum: null, // 评论用户的序列号（显示时排在第几位）
          aid: 430121101001 // 动态所属地区
        }
      ],
      Inputposition: {
        position: 'fixed',
        width: '100%',
        bottom: '0px',
        'background-color': '#fbf9fe'
      }
    }
  },
  methods: { // 方法函数
    click (key) { // 点击菜单
      console.log(key)
    },
    showinfo (item) {
      this.show1 = true
      console.log(item)
    },
    openpop () { // 打开评论点赞功能窗
      this.openwindowshow.pop = true
    },
    closepop () { // 关闭评论点赞功能窗
      this.openwindowshow.pop = false
    },
    isgood (index) { // 判断更改点赞图标
      if (this.pcircles[index].ispr) {
        this.pcircles[index].ispr = 0
      } else {
        this.pcircles[index].ispr = 1
      }
    },
    show (imglist, imgindex, index) { // 图片放大显示
      this.pid = index
      this.imglist = imglist
      this.$nextTick(() => {
        this.$refs.previewer.show(imgindex)
      })
    },
    options () { // previewer配置
      let that = this
      return {
        getThumbBoundsFn (index) {
          let thumbnail = document.querySelectorAll('.previewer-demo-img' + that.pid)[index] // 查找缩略图元素
          let pageYScroll = window.pageYOffset || document.documentElement.scrollTop // 获取窗口滚动Y
          let rect = thumbnail.getBoundingClientRect() // 可选获取水平滚动，获取元素相对于视区的位置
          return { x: rect.left, y: rect.top + pageYScroll, w: rect.width } // w = width
        }
      }
    },
    loadMore () { // 上拉刷新
      // get
      setTimeout(() => {
        setTimeout(() => {
          this.$refs.scroller.donePullup() // 设置上拉刷新操作完成，在数据加载后执行
        }, 10)
      }, 2000)
    },
    refresh () { // 下拉刷新
      // get
      setTimeout(() => {
        this.$nextTick(() => {
          setTimeout(() => {
            this.$refs.scroller.donePulldown() // 设置下拉刷新操作完成，在数据加载后执行
          }, 10)
        })
      }, 2000)
    },
    onScroll (pos) { // 页面滚动触发函数
      if (this.openwindowshow.pop) {
        this.openwindowshow.pop = false
      } else if (this.openwindowshow.inputshow) {
        this.openwindowshow.inputshow = false
      }
    },
    openpopup (index) { // 显示评论输入框
      this.inputplaceholder = '评论:'
      this.pid = index // 对要评论的说说进行编号存储
      this.openwindowshow.inputshow = true // 显示输入框
      this.$nextTick(() => {
        this.$refs.inputcomment.focus()
        // document.querySelector('#pid'+index).scrollIntoView(); // 页面交互问题
      })
    },
    closepopup (val) { // popup弹窗关闭
      this.$refs.inputcomment.blur()
      this.openwindowshow.emotionshow = false
      this.openwindowshow.IsKeyorEmo = true
    },
    onFocus () { // 当表情框显示后input再焦距 时需关闭表情框
      this.openwindowshow.emotionshow = false
      this.openwindowshow.IsKeyorEmo = true
    },
    onblur () {
      // 解决苹果不回弹页面
      setTimeout(() => {
        if (document.activeElement.tagName === 'INPUT' || document.activeElement.tagName === 'TEXTAREA') {
          return
        }
        let result = 'pc'
        if (/(iPhone|iPad|iPod|iOS)/i.test(navigator.userAgent)) { // 判断iPhone|iPad|iPod|iOS
          result = 'ios'
        } else if (/(Android)/i.test(navigator.userAgent)) { // 判断Android
          result = 'android'
        }
        if (result === 'ios') {
          document.activeElement.scrollIntoViewIfNeeded(true)
        }
      }, 100)
    },
    addEmotion (emotion) { // input中添加表情
      this.mytext = this.mytext + '[' + emotion + ']'
    },
    showemotion () { // 点击表情/keybroad图标
      if (this.openwindowshow.IsKeyorEmo) {
        this.$refs.inputcomment.blur()
        this.$nextTick(() => {
          setTimeout(() => {
            this.openwindowshow.IsKeyorEmo = false // 显示键盘图标
            this.openwindowshow.emotionshow = true // 键盘->表情框
          }, 100)
        })
      } else {
        this.openwindowshow.emotionshow = false // 关闭表情框
        this.openwindowshow.IsKeyorEmo = true // 显示笑脸图标
        this.$nextTick(() => {
          this.$refs.inputcomment.focus() // 表情框->键盘
        })
      }
    },
    inputpcontent () { // 输入框点击 小键盘回车键
      let that = this
      if (this.replycomment != null || this.replycomment !== '') {
        this.pcircles[this.pid].TbPccomment.push(
          {
            runame: '施景程 回复' + this.replycomment.uname,
            ccontent: this.mytext.replace(/\[[\u4E00-\u9FA5]{1,3}\]/gi, function (word) { // 转表情图片
              let newWord = word.replace(/\[|\]/gi, '')
              let index = that.list.indexOf(newWord)
              return `<img src="https://res.wx.qq.com/mpres/htmledition/images/icon/emotion/${index}.gif" align="middle">`
            }) })
      } else {
        this.pcircles[this.pid].TbPccomment.push(
          {
            runame: '施景程',
            ccontent: this.mytext.replace(/\[[\u4E00-\u9FA5]{1,3}\]/gi, function (word) {
              let newWord = word.replace(/\[|\]/gi, '')
              let index = that.list.indexOf(newWord)
              return `<img src="https://res.wx.qq.com/mpres/htmledition/images/icon/emotion/${index}.gif" align="middle">`
            }) })
      }
      this.mytext = ''
      this.openwindowshow.inputshow = false
    },
    reply (pcomment, index) { // 回复评论打开弹窗
      this.openwindowshow.inputshow = true
      this.inputplaceholder = '回复 ' + pcomment.uname + ':'
      this.mytext = ''
      this.replycomment = pcomment
      this.pid = index
      this.$nextTick(() => {
        this.$refs.inputcomment.focus()
      })
    }
  },
  computed: { // 计算属性
  },
  watch: { // 侦听器
    'openwindowshow.inputshow' (val) {
      if (val === false) {
        this.closepopup()
      }
    }
  },
  mounted () { // 初始化函数
    // 根据实际手机屏幕 获取图片宽高
    this.imgWidth = parseInt((Number(window.screen.width) - 140) / 3) + 'px'
    let that = this
    document.addEventListener('click', function (e) {
      if (e.target.className !== 'fa fa-smile-o' && e.target.className !== 'dynamic-input' && e.target.className !== 'dynamic-inputcell' && e.target.className !== 'like' && e.target.className !== 'fa fa-keyboard-o') {
        that.openwindowshow.inputshow = false
      }
    })
  }
}
</script>
<style lang="less" scoped>
.weui-panel{
  margin-top: 0px
}
.weui-tab /deep/ .weui-tab__panel{
  padding-bottom: 0px;
}
.dynamic-headbg{
  position:relative;
  width:100%;
  background-image: url("../../assets/img/pyqtt.jpg");
  background-size:cover;
  display:block;
  height: 260px;
}
.dynamic-headerpro{
  margin-top: 210px;
  margin-right: 20px;
  float: right;
  border-radius: 6px;
}
.dynamic-headername{
  color: #fff;
  margin-top: 230px;
  margin-right: 20px;
  float: right;
}
.post-name{
  color: #1e4c97;
  font-size: 13px;
}
.post-text{
  margin-top: 3px;
  font-size: 14px;
  color:#000;
}
.post-time-ico{
  margin: 3px 0px;
  height: 25px;
  .post-time{
    float: left;
    margin-top: 5px;
    font-size: 12px;
  }
  .post-icon{
    float: right;
    line-height: 25px;
    font-size: 20px;
    color: #92abd4;
  }
}
.popover-demo-content {
  width:125px;
  height:32px;
}
.like{
  line-height: 32px;
  text-align: center;
  width: 60px;
  display: inline-block;
  font-size: 12px;
}
.post-content{
  background: #f3f3f3;
  font-size: 13px;
  color: #094dcc;
  padding: 5px;
}
.previewer-demo-img{
  margin-right: 5px;
}
.dynamic-inputcell{
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-align: center;
  -ms-flex-align: center;
  align-items: center;
  padding: 5px 15px;
  .dynamic-input{
    border: 0px;
    height: 2em;
    font-size: 17px;
    line-height: 2em;
    width: 100%;
  }
}
.weui-cell /deep/ .weui-input{
  background-color: #fff;
  height: 2.0em;
  font-size: 17px;
  line-height: 2.0em;
}

body /deep/ .vux-popover-arrow-right {
  border-top:0px;
  border-bottom:0px;
  border-left:0px;
  top:0%;
  transform:0;
  right:0px;
}
.dynamic /deep/ .weui-loadmore{
  margin: auto auto;
}
.wechatEmotion-container{
    background: rgba(238, 238, 238, 0.5);
    .emotion-list{
      width: 100%;
      padding: 10px 5px;
      display: flex;
      flex-wrap: wrap;
      height: 175px;
      overflow-y: auto;
      box-sizing: border-box;
      .emotion-list-item{
        list-style: none;
        float: left;
        height: 27%;
        width: 12.5%;
        padding: 5px 0;
        text-align: center;
      }
    }
  }

</style>
