<template>
  <div class="main">
    <el-container>
      <el-header
        :class="themeClass"
        :height="barHeight + 'px'"
        style="border-bottom: 1px solid rgba(144, 147, 153, 0.3)"
      >
        <div
          class="bar"
          :style="{ height: barHeight + 'px', lineHeight: barHeight + 'px' }"
        >
          <el-tooltip
            class="item"
            effect="dark"
            content="设置"
            placement="bottom"
          >
            <div
              class="btn_settings btn"
              @click="settingsDialogVisible = !settingsDialogVisible"
            >
              <i class="el-icon-setting"></i>
            </div>
          </el-tooltip>
          <el-tooltip
            class="item"
            effect="dark"
            content="帮助信息"
            placement="bottom"
          >
            <div
              class="btn_settings btn"
              @click="helpDialogVisible = !helpDialogVisible"
            >
              <i class="el-icon-warning-outline"></i>
            </div>
          </el-tooltip>
          <el-tooltip
            class="item"
            effect="dark"
            content="保存文文件到本地"
            placement="bottom"
          >
            <div class="btn_settings btn" @click="saveToLocal">
              <i class="el-icon-document-checked"></i>
            </div>
          </el-tooltip>
          <el-tooltip
            class="item"
            effect="dark"
            content="打开本地文件"
            placement="bottom"
          >
            <div class="btn_settings btn" @click="read">
              <i class="el-icon-folder-opened"></i>
            </div>
          </el-tooltip>
          <el-tooltip
            class="item"
            effect="dark"
            content="运行代码"
            placement="bottom"
          >
            <div class="btn_settings btn run" @click="debug">
              <i v-if="ondebug" class="el-icon-loading"></i>
              <i v-else class="el-icon-caret-right"></i>
            </div>
          </el-tooltip>

          <div style="float: left">
            {{ "COMPLIER : " }}
            <span style="font-weight: bold">{{
              mode === "c_cpp" ? "C++ (GCC 9.2.0)" : "Python (3.8.1)"
            }}</span>
            <el-divider direction="vertical"></el-divider>
            {{ "AUTOCOMPLETE : " }}
            <span style="font-weight: bold">{{ autoCmp ? "ON" : "OFF" }}</span>
          </div>
        </div>
      </el-header>
      <el-container>
        <el-main>
          <div class="editor_div">
            <VueAceEditor
              width="100%"
              :height="height"
              ref="ace_editor"
              :content="code_"
              :options="options"
              :fontSize="fontsize"
              :lang="mode"
              :theme="theme"
              @init="ace_editorInit"
              @onChange="codeChange"
            >
            </VueAceEditor>
          </div>
        </el-main>
        <el-aside
          :width="debug_width + 'px'"
          :class="themeClass"
          :style="{ width: debug_width + 'px' }"
        >
          <el-container>
            <div class="area_tip">
              Stdin
              <i
                v-show="stdin"
                style="margin-left: 10px; cursor: pointer; color: #e6a23c"
                @click="clearStdin"
                title="清空"
                class="el-icon-delete"
              />
            </div>
            <el-header
              style="border-bottom: 1px solid rgba(144, 147, 153, 0.3)"
              :height="height - barHeight - debug_output_height + 'px'"
            >
              <VueAceEditor
                width="100%"
                height="100%"
                ref="stdin"
                :options="{
                  showLineNumbers: false,
                  showGutter: false,
                  printMargin: false,
                  tabSize: 4,
                }"
                :fontSize="14"
                :lang="mode"
                :theme="theme"
                @onChange="stdinChange"
              >
              </VueAceEditor>
            </el-header>
            <div class="area_tip">
              Stdout<i
                v-show="stdout"
                style="margin-left: 10px; cursor: pointer; color: #e6a23c"
                @click="stdout = ''"
                title="清空"
                class="el-icon-delete"
              />
            </div>
            <el-main :style="{ height: debug_output_height - 20 + 'px' }">
              <VueStaticHighlight
                v-if="stdout"
                :theme="theme"
                :content="stdout"
                :lang="mode"
                :height="debug_output_height + 'px' - 5"
                style="overflow: auto"
              />
            </el-main>
          </el-container>
        </el-aside>
      </el-container>
    </el-container>
    <el-dialog
      title="设置"
      :visible.sync="settingsDialogVisible"
      custom-class="settingsDialog"
      width="40%"
    >
      <div style="padding: 0 10px">
        <el-row :span="24">
          <el-col :span="12"
            ><p>语言</p>
            <el-select v-model="mode" size="small" style="max-width: 120pt">
              <el-option :key="0" :label="'C++ (GCC 9.2.0)'" :value="'c_cpp'">
              </el-option>
              <el-option :key="1" :label="'Python (3.8.1)'" :value="'python'">
              </el-option> </el-select
          ></el-col>
          <el-col :span="12" style="padding-left: 10px">
            <p>
              服务器IP<i
                style="margin-left: 10px; cursor: pointer; color: #e6a23c"
                @click="server = default_server"
                title="重置"
                class="el-icon-refresh-left"
              />
            </p>
            <el-input
              ref="server"
              placeholder="请输入内容"
              v-model="server"
              size="small"
            >
            </el-input>
          </el-col>
        </el-row>
        <el-row :span="24">
          <el-col :span="12"
            ><p>主题</p>
            <el-select v-model="theme" size="small" style="max-width: 120pt">
              <el-option
                v-for="(item, index) in themeList"
                :key="index"
                :label="item"
                :value="item"
              >
              </el-option> </el-select
          ></el-col>
          <el-col :span="12" style="padding-left: 10px"
            ><p>提示</p>
            <el-switch
              v-model="autoCmp"
              active-color="#13ce66"
              inactive-color="#C0C4CC"
            >
            </el-switch
          ></el-col>
        </el-row>
        <p>字号</p>
        <el-slider v-model="fontsize" :min="10" :max="30"></el-slider>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="settingsDialogVisible = false">关 闭</el-button>
      </span>
    </el-dialog>
    <el-dialog title="提示" :visible.sync="helpDialogVisible" width="40%">
      <h4>关于</h4>
      <p>编辑器目前支持C++、Python，后续会添加更多语言</p>
      <p>
        开源：<el-link type="primary" @click="go"
          >https://github.com/Fromnowon/IDE</el-link
        >
      </p>
      <h4>快捷键</h4>
      <p><span style="color: #409eff">F9</span> - 运行代码</p>
      <p><span style="color: #409eff">Ctrl + F</span> - 查找，替换</p>
      <p><span style="color: #409eff">Ctrl + Enter</span> - 下方插入一行</p>
      <p>
        <span style="color: #409eff">Ctrl + Alt + Enter</span> - 上方插入一行
      </p>
      <p>
        <span style="color: #409eff">Shift + Alt + F</span> - 格式化代码<span
          style="color: red"
          >（测试功能，慎用）</span
        >
      </p>
    </el-dialog>
    <input
      ref="readInput"
      type="file"
      accept=".cpp, .py"
      @change="loadFile"
      style="display: none"
    />
    <div
      class="resize_debug"
      :style="{
        top: height - barHeight - debug_output_height + 2 * 24 + 'px',
        left: width - debug_width - 21 + 'px',
      }"
    >
      <i v-if="debug_width" class="el-icon-d-arrow-right"></i>
      <i v-else class="el-icon-d-arrow-left"></i>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import moment, { fn } from "moment";
import { VueAceEditor, VueStaticHighlight } from "vue2x-ace-editor";
import { js_beautify } from "js-beautify";

export default {
  name: "Main",
  components: {
    VueAceEditor,
    VueStaticHighlight,
  },
  data() {
    return {
      editor: "",
      width: null,
      height: null,
      fontsize: 16,
      theme: "textmate",
      themeList: [],
      code: "",
      code_: "",
      stdin: "",
      stdout: null,
      token: "",
      mode: "c_cpp",
      server: "",
      default_server: "106.52.130.81:2358",
      ondebug: false,
      autoCmp: false,
      tip: null,
      themeClass: "textmate",
      barHeight: 32,
      settingsDialogVisible: false,
      helpDialogVisible: false,
      debug_width: 400,
      debug_output_height: 600,
      options: {
        enableBasicAutocompletion: false, //启用基本自动完成功能
        enableLiveAutocompletion: true, //启用实时自动完成功能 （比如：智能代码提示）
        enableSnippets: false, //启用代码段
        printMargin: false,
        vScrollBarAlwaysVisible: true,
        scrollPastEnd: 0.2,
        tabSize: 4,
      },
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      if (/windows phone|iphone|android/gi.test(window.navigator.userAgent)) {
        //h5
        alert("IDE布局未适配移动端，请谨慎使用");
      }
      this.bindKey();
      this.dragLoadFileInit();
      this.resize();
      window.onresize = () => {
        this.resize(); // 动态调整高度
      };
      window.onbeforeunload = () => {
        this.save();
      };
      this.getKeywords();
      this.initDrag();
    },
    ace_editorInit(editor) {
      require("brace/ext/language_tools");
      require("brace/ext/searchbox");
      require("brace/mode/c_cpp");
      require("brace/mode/python");

      // 导入主题
      const requireAll = require.context("brace/theme", false, /\.js$/);
      requireAll.keys().forEach((item) => {
        let theme = item.replace(/\.\//, "").replace(/\.js$/, "");
        this.themeList.push(theme);
        require("brace/theme/" + theme + ".js");
      });
      // 初始化
      this.loadConf();
      this.editor = editor;
      editor.focus();
    },
    initDrag() {
      const _this = this;
      const obj = document.querySelector(".resize_debug");
      obj.onmousedown = function (e) {
        e.preventDefault();
        var e = e || window.event; // 兼容 IE
        // 鼠标点击物体那一刻相对于物体左侧边框的距离=点击时的位置相对于浏览器
        // 最左边的距离-物体左边框相对于浏览器最左边的距离，纵向同理
        var divX = e.clientX - this.offsetLeft;
        var divY = e.clientY - this.offsetTop;

        if (obj.setCapture) {
          obj.setCapture(); // 修复低版本 IE bug
        }
        document.onmousemove = function (e) {
          var e = e || window.event;

          var disX = e.clientX - divX;
          var disY = e.clientY - divY;

          // 控制拖拽物体的范围只能在浏览器视窗内，不允许出现滚动条或拖出可视区域
          if (disX < 0) {
            disX = 0;
          } else if (
            disX >
            document.documentElement.clientWidth - obj.offsetWidth
          ) {
            disX = document.documentElement.clientWidth - obj.offsetWidth;
          }

          if (disY < 0) {
            disY = 0;
          } else if (
            disY >
            document.documentElement.clientHeight - obj.offsetHeight
          ) {
            disY = document.documentElement.clientHeight - obj.offsetHeight;
          }

          // 移动时重新得到物体的距离，解决拖动时出现晃动现象
          // obj.style.top = disY + "px";
          //obj.style.left = disX + "px";
          _this.debug_width =
            _this.width - disX - divX - 15 > 1000
              ? 1000
              : _this.width - disX - divX - 15;
          let n_height = _this.height - disY - divY + _this.barHeight - 5;
          _this.debug_output_height =
            n_height > _this.height * 0.8
              ? _this.height * 0.8
              : n_height < _this.height * 0.2
              ? _this.height * 0.2
              : n_height;

          _this.editor.resize(); // 调整尺寸
          _this.$refs.stdin.resize(); // 调整尺寸
          document.onmouseup = function () {
            if (_this.debug_width <= 100) _this.debug_width = 0;
            // 鼠标抬起时不再移动
            // 预防鼠标弹起来后还会循环（即预防鼠标放上去的时候还会移动）
            document.onmousedown = document.onmousemove = null;
            if (obj.releaseCapture) {
              obj.releaseCapture(); // 修复低版本 IE bug
            }
          };
        };
      };
    },
    clearStdin() {
      this.$refs.stdin.setValue("");
    },
    codeChange(editor) {
      this.code = editor.getValue();
    },
    stdinChange(editor) {
      this.stdin = editor.getValue();
    },
    getKeywords() {
      // 补全，文件从远程拉取
      axios
        .get("http://106.52.130.81/lib/keywords.json?" + Date.now())
        .then((res) => {
          let keywords = res.data,
            key_arr = [];
          keywords.forEach((item) => {
            key_arr.push(item);
          });
          this.$refs.ace_editor.setCompleteData(key_arr);
        });
    },
    loadConf() {
      // 读取信息
      let Base64 = require("js-base64").Base64;
      let code = localStorage.getItem("ace_code");
      if (code) this.code_ = Base64.decode(code);
      this.theme = localStorage.getItem("ace_theme") || "textmate";
      this.fontsize = parseInt(localStorage.getItem("ace_fontsize")) || 18;
      this.autoCmp = localStorage.getItem("ace_auto") == "true" ? true : false;
      this.mode = localStorage.getItem("ace_lang") || "c_cpp";
      this.server = localStorage.getItem("ace_server") || this.default_server;
    },
    saveToLocal() {
      //定义文件内容，类型必须为Blob 否则createObjectURL会报错
      let content = new Blob([this.code]);
      //生成url对象
      let urlObject = window.URL || window.webkitURL || window;
      let url = urlObject.createObjectURL(content);
      //生成<a></a>DOM元素
      let el = document.createElement("a");
      //链接赋值
      el.href = url;
      let fname =
        "code-" +
        moment(new Date()).format("YYYY-MM-DD HH:mm:ss") +
        (this.mode == "c_cpp" ? ".cpp" : ".py");
      el.download = fname;
      //必须点击否则不会下载
      el.click();
      //移除链接释放资源
      urlObject.revokeObjectURL(url);
    },
    read() {
      this.$refs.readInput.click();
    },
    loadFile(event) {
      var input = event.target;
      var reader = new FileReader();
      reader.onload = () => {
        if (reader.result) {
          //显示文件内容
          this.editor.setValue(reader.result);
          event.target.value = "";
        }
      };
      reader.readAsText(input.files[0]);
    },
    dragLoadFileInit() {
      // 获取目标元素
      var box = document.querySelector(".editor_div");
      // (1)需要解决一旦拖拽外部文件就覆盖掉当前页面的问题
      //  解决：给document绑定drop事件
      //  drop事件默认触发不了，需要在dragover事件里面阻止默认事件
      document.ondrop = (e) => {
        e.preventDefault();
      };
      // 这个阻止默认事件是为了让drop事件得以触发
      document.ondragover = (e) => {
        e.preventDefault();
      };
      let _this = this;
      box.ondrop = (e) => {
        // 得到拖拽过来的文件
        var dataFile = e.dataTransfer.files[0];
        // FileReader实例化
        if (dataFile == undefined) return;
        var fr = new FileReader();
        // 读取完毕之后执行
        fr.onload = () => {
          // 获取得到的结果
          this.editor.setValue(fr.result);
          e.target.value = "";
        };
        // 异步读取文件
        fr.readAsText(dataFile);
      };
    },
    saveCode() {
      let Base64 = require("js-base64").Base64;
      let code = Base64.encode(this.code);
      localStorage.setItem("ace_code", code); // 保存代码
    },
    save() {
      this.saveCode();
      localStorage.setItem("ace_theme", this.theme); // 保存主题
      localStorage.setItem("ace_fontsize", this.fontsize); // 保存字号
      localStorage.setItem("ace_auto", this.autoCmp); // 保存补全
      localStorage.setItem("ace_lang", this.mode); // 保存语言
      localStorage.setItem("ace_server", this.server); // 保存语言
      console.log("saved");
    },
    go() {
      window.open("https://github.com/Fromnowon/IDE");
    },
    debug() {
      if (this.code.trim().length == 0) {
        this.$message.error("代码不能为空");
        return;
      }
      if (this.ondebug) return;
      if (!this.debug_width) this.debug_width = 400;

      this.tip = this.$message({
        dangerouslyUseHTMLString: true,
        message: "<strong>处理中，请稍等...</strong>",
        duration: 0,
        center: true,
      });

      let Base64 = require("js-base64").Base64;
      let code = Base64.encode(this.code);

      const _this = this;
      _this.ondebug = true;
      _this.stdout = null;
      _this.stdin = _this.stdin;

      axios
        .post(
          "http://" +
            _this.server +
            "/submissions/?base64_encoded=true&wait=false",
          {
            source_code: code,
            language_id: _this.mode == "c_cpp" ? 54 : 71,
            stdin: Base64.encode(_this.stdin),
          },
          { timeout: 5000 }
        )
        .then((response) => {
          _this.token = response.data.token;
          _this.getResult();
        })
        .catch((error) => {
          console.log(error);
          _this.ondebug = false;
          _this.tip.close();
          this.$alert("请求异常，请检查服务器地址", "错误", {
            confirmButtonText: "确定",
            type: "error",
          });
        });
    },
    getResult() {
      const _this = this;
      let Base64 = require("js-base64").Base64;
      let loop = setInterval(() => {
        axios
          .get(
            "http://" +
              _this.server +
              "/submissions/" +
              _this.token +
              "?base64_encoded=true"
          )
          .then((response) => {
            if (_this.stdout) return; // 若已有结果，不再接受返回
            let e = response.data;
            if (e.status.id != 1 && e.status.id != 2) {
              clearInterval(loop);
              _this.tip.close();
              _this.ondebug = false;
              if (e.status.id == 3) {
                _this.stdout = Base64.decode(e.stdout || "5peg");
              } else {
                console.log(e);
                _this.$message({
                  message: "调试结果异常，请查看输出信息",
                  type: "warning",
                });
                if (e.status.id == 6)
                  _this.stdout = Base64.decode(e.compile_output);
                else if (e.status.id == 11 || e.status.id == 5)
                  // runtime error
                  _this.stdout = e.status.description;
                else _this.stdout = Base64.decode(e.stderr);
              }
            }
          });
      }, 1000);
    },
    resize() {
      this.height = document.documentElement.clientHeight - this.barHeight;
      this.width = document.documentElement.clientWidth;
    },
    bindKey() {
      const _this = this;
      window.addEventListener(
        "keydown",
        function (e) {
          if (e.keyCode == 120) {
            //调试
            e.preventDefault();
            _this.debug();
          }
          if ((e.key == "s" || e.key == "S") && e.altKey) {
            // alt + s 保存
            e.preventDefault();
            if (_this.editor.getValue().length > 0) _this.save();
          }
          if (e.keyCode == 70 && e.altKey && e.shiftKey) {
            // 格式化代码
            e.preventDefault();
            let pos = _this.editor.selection.getCursor();
            _this.editor.setValue(
              js_beautify(_this.editor.getValue(), {
                indent_size: 4,
                brace_style: "preserve-inline",
                space_before_conditional: true,
              }),
              1
            );
          }
          if (e.keyCode == 13 && e.altKey && e.ctrlKey) {
            // atl + ctrl + enter上方新增一行
            e.preventDefault();
            let nowRow = _this.editor.selection.getCursor().row; // 获取当前行号，从0开始
            let nowRowContent = _this.editor.session.getLine(nowRow); // 获取当前行内容
            let spaceCounter = 0;
            while (nowRowContent[spaceCounter] == " ") spaceCounter++; // 获取空格数
            _this.editor.gotoLine(nowRow); // 定位到指定行，从1开始
            _this.editor.session.insert({ row: nowRow - 1, col: 0 }, "\n");
            _this.editor.gotoLine(nowRow + 1);
            _this.editor.session.insert(
              { row: nowRow, col: 0 },
              nowRowContent.slice(0, spaceCounter)
            ); // 同步插入行与当前行的缩进
          }
          if (e.keyCode == 13 && e.ctrlKey && !e.altKey) {
            // ctrl + enter 下方新增一行
            e.preventDefault();
            let nowRow = _this.editor.selection.getCursor().row;
            let nowRowContent = _this.editor.session.getLine(nowRow); // 获取当前行内容
            let spaceCounter = 0;
            while (nowRowContent[spaceCounter] == " ") spaceCounter++; // 获取空格数
            _this.editor.session.insert({ row: nowRow, col: 0 }, "\n");
            _this.editor.gotoLine(nowRow + 2);
            _this.editor.session.insert(
              { row: nowRow + 1, col: 0 },
              nowRowContent.slice(0, spaceCounter)
            ); // 同步插入行与当前行的缩进
          }
        },
        false
      );
    },
  },
  watch: {
    theme(v) {
      this.themeClass = "ace-" + v.split("_").join("-");
    },
    autoCmp(v) {
      this.editor.setOptions({
        enableLiveAutocompletion: v,
      });
    },
  },
};
</script>

<style>
.main {
  position: relative;
  width: 100%;
  height: 100%;
}
.editor_div {
  overflow: hidden;
}
.debug {
  padding: 20px;
}
.bar {
  padding: 0 0 0 10px;
  font-size: 10px;
}
.ace_static_highlight {
  font-size: 14px !important;
}
.btn {
  font-size: 18px;
  cursor: pointer;
  border-left: 1px solid rgba(0, 0, 0, 0.1);
  transition: all 0.5s;
}
.btn:hover {
  background: rgba(0, 0, 0, 0.205);
}
.btn_settings {
  height: 32px;
  line-height: 32px;
  width: 60px;
  float: right;
  text-align: center;
}
.run {
  background: #67c23a;
  width: 80px;
  color: white;
  border: none;
}
.run:hover {
  background: #489223;
}
.settingsDialog {
  box-shadow: 0 3px 15px rgb(0 0 0 / 20%);
}
.el-header {
  padding: 0 !important;
}
.el-main {
  padding: 0 !important;
}
.resize_debug {
  cursor: move;
  position: absolute;
  color: rgba(155, 155, 155, 0.5);
  z-index: 999;
}
.area_tip {
  font-size: 14px;
  line-height: 24px;
  padding-left: 5px;
  border-bottom: 1px solid rgba(144, 147, 153, 0.3);
}
.ace-tm {
  background-color: unset !important;
  color: unset !important;
}
/* 滚动条样式 */
::-webkit-scrollbar {
  width: 3px;
  height: 6px;
}
/* 滑轨 */
::-webkit-scrollbar-track {
  -webkit-box-shadow: inset 0 0 3px rgba(155, 155, 155, 0.3);
  -webkit-border-radius: 3px;
  border-radius: 3px;
}
/* Handle */
::-webkit-scrollbar-thumb {
  -webkit-border-radius: 3px;
  border-radius: 3px;
  background: rgba(144, 147, 153, 0.3);
}
::-webkit-scrollbar-thumb:window-inactive {
  background: rgba(144, 147, 153, 0.5);
}

::-webkit-scrollbar-thumb:hover {
  background-color: #777;
}
</style>
