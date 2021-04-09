<template>
  <div class="main">
    <div class="btn" @click="openDebug">
      <i v-if="openDebugFlag" class="el-icon-close"></i>
      <i v-else class="el-icon-position"></i>
    </div>
    <el-row :span="24">
      <el-col :span="span_editor" class="editor_div">
        <VueAceEditor
          width="100%"
          :height="height"
          ref="ace_editor"
          :content="code"
          :options="options"
          :fontSize="fontsize"
          :lang="mode"
          :theme="theme"
          @init="ace_editorInit"
        >
          <div
            :class="barClass + ' bar'"
            :style="{ height: barHeight + 'px', lineHeight: barHeight + 'px' }"
          >
            {{ "COMPLIER : " }}
            <span style="font-weight: bold">{{
              mode === "c_cpp" ? "C++ (GCC 9.2.0)" : "Python (3.8.1)"
            }}</span>
            <el-divider direction="vertical"></el-divider>
            {{ "AUTOCOMPLETE : " }}
            <span style="font-weight: bold">{{ autoCmp ? "ON" : "OFF" }}</span>
          </div>
        </VueAceEditor>
      </el-col>
      <el-col
        :span="24 - span_editor"
        class="debug_div"
        :style="{ height: height + 'px' }"
      >
        <div v-show="span_editor != 24" class="debug">
          <el-collapse v-model="activeNames" style="margin-top: 50px">
            <el-collapse-item name="1">
              <template slot="title">
                <div style="font-size: 16px; font-weight: bold">
                  <span>设置 </span>
                </div>
              </template>
              <h4>文件</h4>
              <div style="padding: 0 10px">
                <el-button
                  type="primary"
                  plain
                  size="small"
                  @click="read"
                  style="margin-right: 10px"
                  ><i class="el-icon-folder-opened" /> 打 开</el-button
                >
                <input
                  ref="readInput"
                  type="file"
                  accept=".cpp, .py"
                  @change="loadFile"
                  style="display: none"
                />
                <el-button
                  type="success"
                  plain
                  size="small"
                  @click="saveToLocal()"
                  ><i class="el-icon-document-checked"></i> 另存为</el-button
                >
              </div>
              <h4>选项</h4>
              <div style="padding: 0 10px">
                <el-row :span="24">
                  <el-col :span="12"
                    ><p>语言</p>
                    <el-select
                      v-model="mode"
                      size="small"
                      style="max-width: 120pt"
                    >
                      <el-option
                        :key="0"
                        :label="'C++ (GCC 9.2.0)'"
                        :value="'c_cpp'"
                      >
                      </el-option>
                      <el-option
                        :key="1"
                        :label="'Python (3.8.1)'"
                        :value="'python'"
                      >
                      </el-option> </el-select
                  ></el-col>
                  <el-col :span="12" style="padding-left: 10px">
                    <p>
                      服务器IP<i
                        style="
                          margin-left: 10px;
                          cursor: pointer;
                          color: #e6a23c;
                        "
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
                    <el-select
                      v-model="theme"
                      size="small"
                      style="max-width: 120pt"
                    >
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
            </el-collapse-item>
          </el-collapse>

          <h4>调试</h4>
          <div style="padding: 0 10px">
            <el-button
              type="primary"
              style="width: 100%; margin-bottom: 20px"
              @click="debug"
              :loading="ondebug"
              >运 行</el-button
            >
            <p>输入</p>
            <el-input
              :disabled="ondebug"
              type="textarea"
              ref="stdin"
              :autosize="{ minRows: 3, maxRows: 10 }"
              placeholder="stdin"
              v-model="stdin"
              style="width: 100%; margin-bottom: 20px"
            >
            </el-input>
            <div v-if="stdout">
              <p>输出</p>
              <VueStaticHighlight
                theme="textmate"
                :content="stdout"
                :lang="mode"
                height="320px"
                style="overflow: auto"
                :gutter="false"
                width="100%"
              />
            </div>
            <p v-else style="color: gray; font-size: 14px">
              Tip : 运行代码以获取输出
            </p>
            <div style="text-align: right">
              <span style="font-size: 10pt">Develop by Zihang : </span>
              <el-button type="text" @click="go()">Github</el-button>
            </div>
          </div>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from "axios";
import moment, { fn } from "moment";
import { VueAceEditor, VueStaticHighlight } from "vue2x-ace-editor";

export default {
  name: "Main",
  components: {
    VueAceEditor,
    VueStaticHighlight,
  },
  data() {
    return {
      editor: "",
      height: 500,
      fontsize: 16,
      theme: "textmate",
      themeList: [],
      code: "",
      span_editor: 24,
      stdin: "",
      stdout: null,
      token: "",
      mode: "c_cpp",
      server: "",
      default_server: "106.52.130.81:2358",
      ondebug: false,
      openDebugFlag: false,
      autoCmp: false,
      tip: null,
      activeNames: ["2"],
      barClass: "textmate",
      barHeight: 32,
      options: {
        enableLiveAutocompletion: false,
        printMargin: false,
        tabSize: 4,
      },
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
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
    },
    ace_editorInit(editor) {
      require("brace/ext/language_tools");
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
      if (code) this.code = Base64.decode(code);
      this.theme = localStorage.getItem("ace_theme") || "textmate";
      this.fontsize = parseInt(localStorage.getItem("ace_fontsize")) || 18;
      this.autoCmp = localStorage.getItem("ace_auto") == "true" ? true : false;
      this.mode = localStorage.getItem("ace_lang") || "c_cpp";
      this.server = localStorage.getItem("ace_server") || this.default_server;
    },
    saveToLocal() {
      //定义文件内容，类型必须为Blob 否则createObjectURL会报错
      let content = new Blob([this.editor.getValue()]);
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
      let code = Base64.encode(this.editor.getValue());
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
    openDebug() {
      this.span_editor = 40 - this.span_editor;
    },
    debug() {
      if (this.editor.getValue().trim().length == 0) {
        this.$message.error("代码不能为空");
        return;
      }

      this.tip = this.$message({
        dangerouslyUseHTMLString: true,
        message: "<strong>处理中，请稍等...</strong>",
        duration: 0,
        center: true,
      });

      let Base64 = require("js-base64").Base64;
      let code = Base64.encode(this.editor.getValue());

      const _this = this;
      _this.ondebug = true;
      _this.stdout = null;

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
    },
    bindKey() {
      const _this = this;
      window.addEventListener(
        "keydown",
        function (e) {
          if (e.keyCode == 13 && e.altKey) {
            e.preventDefault();
            if (_this.openDebugFlag) {
              if (!_this.ondebug) _this.debug();
            } else _this.span_editor = 40 - _this.span_editor;
          }
          if (e.keyCode == 27) {
            e.preventDefault();
            if (_this.openDebugFlag) _this.span_editor = 40 - _this.span_editor;
          }
          if ((e.key == "s" || e.key == "S") && e.altKey) {
            e.preventDefault();
            if (_this.editor.getValue().length > 0) _this.save();
          }
        },
        false
      );
    },
  },
  watch: {
    theme(v) {
      this.barClass = "ace-" + v.split("_").join("-");
    },
    span_editor(v) {
      this.openDebugFlag = !this.openDebugFlag;
      if (v == 24) {
        this.editor.focus();
      } else {
        this.$nextTick(function () {
          this.$refs.stdin.focus();
        });
      }
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
.debug {
  padding: 20px;
}
.btn {
  z-index: 9;
  position: absolute;
  top: 30px;
  right: 20px;
  color: rgba(0, 0, 0, 0.3);
  font-size: 20pt;
}
.btn:hover {
  cursor: pointer;
  color: #409eff;
  transition: color 0.5s;
}
.ace_editor {
  box-shadow: 5px 0 10px -5px lightgrey;
}
.debug_div {
  overflow: auto;
  padding-left: 10px;
}
.bar {
  padding-right: 10px;
  font-size: 10px;
  text-align: right;
  border-bottom: 1px solid lightgray;
  box-shadow: 5px 0 10px -5px lightgrey;
}
.ace_static_highlight {
  font-size: 14px !important;
}
</style>
