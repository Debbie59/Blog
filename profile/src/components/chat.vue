<template>
  <div id="chat" :class="{ 'has-prompt': hasPrompt }">
    <div id="chat-body">
      <div id="chat-body-bg"></div>
      <div id="chat-body-content">
        <div id="mock-msg-row" class="msg-row">
          <div id="mock-msg" class="msg" v-html="latestMsgContent"></div>
        </div>
        <div class="msg-row"
             v-for="(msg, index) in messages"
             :key="index"
             :class="msg.author === 'erxiao' ? 'msg-erxiao' : 'msg-me'">
          <img v-if="msg.author != 'erxiao'" class="avater avatar-me" src="../../static/img/custom.jpg"/>
          <img v-if="msg.author === 'erxiao'" class="avater avatar-erxiao" src="../../static/img/xinyi.jpg"/>
          <div class="msg"
               :class="'msg-bounce-in-' + (msg.author === 'erxiao' ? 'left': 'right')"
               :style="msg.width && msg.height && {width: msg.width + 5 + 'px', height: msg.height + 3 + 'px'}"
               v-html="msg.content"></div>
        </div>
      </div>
    </div>
    <div id="chat-foot">
      <div id="prompt">
        <div id="prompt-head">
          <div class="say-something">说点什么……</div>
          <div class="close-btn" v-on:click="togglePrompt(false)">×</div>
        </div>
        <div id="prompt-body">
          <ul class="responses" v-if="lastDialog">
            <li v-for="res in lastDialog.responses">
              <a href="javascript:;" v-on:click="respond(res)">{{ res.content }}</a>
            </li>
          </ul>
          <div class="next-topic"
               v-if="!lastDialog || !lastDialog.responses">
            <ul class="topics">
              <li v-for="topic in nextTopics">
                <a href="javascript:;" v-on:click="ask(topic)">{{ topic.brief }}</a>
              </li>
            </ul>
          </div>
        </div>
      </div>
      <div id="input-hint" class="say-something"
           v-on:click="togglePrompt(true)"
           :class="{'clickable': !isXinyiTyping }">
        <div v-if="!isXinyiTyping" class="input-div">说点什么……</div>
        <div v-if="isXinyiTyping" class="input-div">小玴正在输入中……</div>
        <div class="input-btn">+</div>
      </div>
    </div>
    <div id="prompt-bg" v-on:click="togglePrompt(false)"></div>
  </div>
</template>

<script>
  import $ from 'jquery'
  import MD5 from 'js-md5';

  const AUTHOR = {
    ERXIAO: 'erxiao',
    ME: 'me'
  }

  const TYPING_MSG_CONTENT = `
  <div style="display:inline-block;width:10px;height:10px;background:#b5e2ec;-webkit-border-radius:50%;border-radius:50%;-webkit-transform-origin:50% 50%;-ms-transform-origin:50% 50%;transform-origin:50% 50%;-webkit-animation:dotZoomIn 1.4s infinite;animation:dotZoomIn 1.4s infinite;-webkit-animation-delay:-0.32s;animation-delay:-0.32s"></div>
  <div style="display:inline-block;width:10px;height:10px;background:#b5e2ec;-webkit-border-radius:50%;border-radius:50%;-webkit-transform-origin:50% 50%;-ms-transform-origin:50% 50%;transform-origin:50% 50%;-webkit-animation:dotZoomIn 1.4s infinite;animation:dotZoomIn 1.4s infinite;-webkit-animation-delay:-0.16s;animation-delay:-0.16s"></div>
  <div style="display:inline-block;width:10px;height:10px;background:#b5e2ec;-webkit-border-radius:50%;border-radius:50%;-webkit-transform-origin:50% 50%;-ms-transform-origin:50% 50%;transform-origin:50% 50%;-webkit-animation:dotZoomIn 1.4s infinite;animation:dotZoomIn 1.4s infinite"></div>
`

  export default {
    data() {
      return {
        messages: [],
        lastDialog: null,
        msgChain: Promise.resolve(),
        dialogs: null,
        isErXiaoTyping: false,
        // topics that user can ask
        nextTopics: [],
        hasPrompt: false,
        latestMsgContent: null,
        xinyi: null,
        isXinyiTyping: false
      }
    },
    mounted() {
      this.$axios.get('./static/dialog.json').then(data => {
        this.dialogs = data;
        this.nextTopics = this.dialogs.data.fromUser;
        this.appendDialogs('0000');
      },function (response) {
        console.log(response)
      })
    },
    methods: {
      appendDialogs(id) {
        //me
        if (typeof id === 'object' && id.length > 0) {
          id.forEach(id => this.appendDialogs(id));
          return;
        } else if (id == null) {
          this.lastDialog.responses = null;
          return;
        }
        //xinyi
        this.isXinyiTyping = true;
        const dialog = this.getDialog(id)
        getRandomMsg(dialog.details).forEach(content => {
          this.msgChain = this.msgChain
            .then(() => delay(700))
            .then(() => this.sendMsg(content, AUTHOR.ERXIAO))
        });
        return dialog.nextErxiao ? this.appendDialogs(dialog.nextErxiao) : this.msgChain.then(() => {
            this.lastDialog = dialog
            this.isXinyiTyping = false
          }
        )

      },
      // 获取一组
      getDialog(id) {
        const dialogs = this.dialogs.data.fromXinyi.filter(dialog => dialog.id === id)
        return dialogs ? dialogs[0] : null;
      },
      // 谁发送？
      sendMsg(msg, author) {
        switch (author) {
          case 'me':
            return this.sendMeMsg(msg);
          default :
            return this.sendXinyiMsg(msg, author)
        }
      },
      // me发送内容设置
      sendMeMsg(msg) {
        this.messages.push({
          author: AUTHOR.ME,
          content: msg
        })
        onMessageSending()
        return Promise.resolve()
      },
      //  xinyi发送内容设置
      sendXinyiMsg(message, author) {
        const content = getRandomMsg(message);
        const length = content.replace(/<[^>]+>/g, '').length;
        const isImg = /<img[^>]+>/.test(content);
        const isTyping = length > 5 || isImg;
        const msg = {
          author: author,
          content: isTyping ? TYPING_MSG_CONTENT : content,
          isImg: isImg
        }
        this.messages.push(msg)
        if (isTyping) {
          this.markMsgSize(msg)
          setTimeout(updateScroll)

          return delay(Math.min(100 * length, 2000))
            .then(() => {
              return this.markMsgSize(msg, content)
            })
            .then(() => delay(150))
            .then(() => {
              msg.content = content
              onMessageSending()
            })
        }
        onMessageSending()
        return Promise.resolve()
      },
      // 计算图片/内容宽度
      markMsgSize(msg, content = null) {
        this.latestMsgContent = content || msg.content;
        return delay(0)
          .then(() => {
            msg.isImg && onImageLoad($("#mock-msg img"))
              .then(() => {
                Object.assign(msg, getMockMsgSize())
                this.messages = [...this.messages];
              })
          })
      },
      getDialogFromUser(id) {
        // only one dialog should be matched by id
        const dialogs = this.dialogs.fromUser
          .filter(dialog => dialog.id === id)
        return dialogs ? dialogs[0] : null
      },
      togglePrompt(toShow) {
        if (this.isXinyiTyping) {
          return
        }
        this.hasPrompt = toShow
      },
      respond(response) {
        return this.say(response.content, response.nextErxiao)
      },
      // 答
      say(content, dialogId) {
        // close prompt
        this.hasPrompt = false

        return delay(200)
        // send user msg
          .then(() => this.sendMsg(content, AUTHOR.ME))
          .then(() => delay(300))
          // add ErXiao's next dialogs
          .then(() => this.appendDialogs(dialogId))
      },
      // 列表随便问
      ask(fromUser) {
        const content = getRandomMsg(fromUser.details)
        return this.say(content, fromUser.nextErxiao)
      }
    }
  }

  // 计算内容宽度
  function getMockMsgSize() {
    const $mockMsg = $('#mock-msg')
    return {
      width: $mockMsg.width(),
      height: $mockMsg.height()
    }
  }

  // 发送信息
  // 滚动高度
  // 如果有a标签加_blank
  // 加载图片
  //  滚动
  function onMessageSending() {
    setTimeout(() => {
      updateScroll();
      const $latestMsg = $('#chat-body-content .msg-row:last-child .msg');
      // add target="_blank" for links
      $latestMsg.find('a').attr('target', '_blank')

      // update scroll position when images are loaded
      onImageLoad($latestMsg).then(updateScroll)
    })
  }

  // 图片加载成功
  function onImageLoad($img) {
    return new Promise(resolve => {
      $img.on("load", resolve).each((index, target) => {
        target.complete && $(target)
      })
    })
  }

  /**
   * get a random message from message array
   */
  function getRandomMsg(messages) {
    // single item
    if (typeof messages === 'string' || !messages.length) {
      return messages
    }

    const id = Math.floor(Math.random() * messages.length)
    return messages[id]
  }

  //延迟加载
  function delay(amount = 0) {
    return new Promise(resolve => {
      setTimeout(resolve, amount)
    })
  }

  //  滚动=滚动高度+padding；
  function updateScroll() {
    const $chatbox = $('#chat-body-content')
    const distance = $chatbox[0].scrollHeight - $chatbox.height() - $chatbox.scrollTop()
    const duration = 250
    const startTime = Date.now()

    requestAnimationFrame(function step() {
      const p = Math.min(1, (Date.now() - startTime) / duration)
      $chatbox.scrollTop($chatbox.scrollTop() + distance * p)
      p < 1 && requestAnimationFrame(step)
    })
  }


</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }

  .content {
    position: absolute;
    top: 40px;
    left: 0;
    background: linear-gradient(180deg, #6ca6c3, #b5e2ec);
    right: 0;
    bottom: 0;
  }
</style>
