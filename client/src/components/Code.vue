<template>
  <div>
    loginurl：<input type="text" v-model="config.loginurl" style="width: 200px"></input>
    <button @click="fakelogin">假登录</button>

    <p>配置服务器信息：</p>
    文件上传url设置：<input type="text" v-model="config.url"></input>
    wss 连接设置：host:<input type="text" v-model="config.host"></input>port:<input type="text" v-model="config.port"></input>username:<input type="text" v-model="config.username"></input>password:<input type="text" v-model="config.password"></input>
    <br/><br/>
    <button @click="connect_websocket">保存设置，并连接服务器</button>

    <md-card>
      <md-card-actions>
        <div class="md-subhead">
          <span>mode: {{ cmOption.mode }}</span>
          <span>&nbsp;&nbsp;&nbsp;</span>
          <span>theme: {{ cmOption.theme }}</span>
        </div>
      </md-card-actions>
      <md-card-media>
        <div class="codemirror">
          <!-- codemirror -->
          <codemirror v-model="code" :options="cmOption"></codemirror>
        </div>
      </md-card-media>
    </md-card>
    <br>
    <button @click="get_code">执行代码</button>

    <br>
    <br>
    <div class="console" id="terminal"></div>
  </div>

</template>

<script>
  // language
  import 'codemirror/mode/python/python.js'
  // theme css
  import 'codemirror/theme/night.css'

  // require active-line.js
  import 'codemirror/addon/selection/active-line.js'
  // closebrackets
  import 'codemirror/addon/edit/closebrackets.js'
  // keyMap
  import 'codemirror/mode/clike/clike.js'
  import 'codemirror/addon/edit/matchbrackets.js'
  import 'codemirror/addon/comment/comment.js'
  import 'codemirror/addon/dialog/dialog.js'
  import 'codemirror/addon/dialog/dialog.css'
  import 'codemirror/addon/search/searchcursor.js'
  import 'codemirror/addon/search/search.js'
  import 'codemirror/keymap/emacs.js'

  import Terminal from './Xterm'

  export default {
    name: "Code",
    props: {
      terminal: {
        type: Object,
        default: function () {
          return {}
        }
      }
    },
    data() {
      const code =
        `print("hello world")`
      return {
        code,
        ssh_cmd: null,
        cmOption: {
          autoCloseBrackets: true,
          tabSize: 4,
          styleActiveLine: true,
          lineNumbers: true,
          line: true,
          mode: 'text/x-python',
          theme: 'night',
          keyMap: "emacs"
        },
        config: {
          url: null,
          host: null,
          port: null,
          username: null,
          password: null,
          loginurl: null,
        },
        term: null,
        terminalSocket: null,
      }
    },
    methods: {
      get_code() {
        console.log(this.code)
        let that = this
        this.$axios.post(that.config.url, {
          "content_id": "c209ac5888d34e7a8ef5453a8dbe87dd",
          "code": this.code
        }).then(function (response) {
          that.ssh_cmd = response.data.data.ssh_command
        }).then(function () {
            that.terminalSocket.send(that.ssh_cmd + "\n")
          }
        )
      },
      fakelogin() {
        this.$axios.get(this.config.loginurl).then(
          alert("假登录成功")
        )
      },
      connect_websocket() {
        console.log('pid : ' + this.terminal.pid + ' is on ready')
        let terminalContainer = document.getElementById('terminal')
        this.term = new Terminal()
        this.term.open(terminalContainer)
        // open websocket
        this.terminalSocket = new WebSocket(`wss://wx.piaoshiyun.com/ws/shell?host=${this.config.host}&port=${this.config.port}&username=${this.config.username}&password=${this.config.password}`)
        this.terminalSocket.onopen = this.runRealTerminal
        this.terminalSocket.onclose = this.closeRealTerminal
        this.terminalSocket.onerror = this.errorRealTerminal
        this.term.attach(this.terminalSocket)
        this.term._initialized = true
        console.log('mounted is going on')
      },
      runRealTerminal() {
        console.log('webSocket is finished')
      },
      errorRealTerminal() {
        console.log('error')
      },
      closeRealTerminal() {
        console.log('close')
      }
    },
    beforeDestroy() {
      this.terminalSocket.close()
      this.term.destroy()
    }
  }
</script>
