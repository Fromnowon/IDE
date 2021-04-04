<template>
  <div class="main">
    <div class="btn" @click="openDebug">
      <i v-if="openDebugFlag" class="el-icon-close"></i>
      <i v-else class="el-icon-position"></i>
    </div>
    <el-row :gutter="24">
      <el-col :span="span_editor"
        ><codemirror v-model="code" :options="options" ref="myEditor" />
      </el-col>
      <el-col :span="24 - span_editor" class="debug_div">
        <div class="info">
          <span style="font-size: 10pt">Develop by Zihang : </span>
          <el-button type="text" @click="go()">Github</el-button>
        </div>
        <div v-show="span_editor != 24" class="debug">
          <h4>设置</h4>
          <el-switch
            style="margin-bottom: 20px"
            v-model="mode"
            active-text="深色"
            inactive-text="浅色"
          >
          </el-switch>
          <h4>运行</h4>
          <el-button
            type="primary"
            style="width: 100%; margin-bottom: 20px"
            @click="debug"
            :loading="ondebug"
            >运 行</el-button
          >
          <el-input
            type="textarea"
            ref="stdin"
            :autosize="{ minRows: 5, maxRows: 10 }"
            placeholder="stdin"
            v-model="stdin"
            style="width: 100%; margin-bottom: 20px"
          >
          </el-input>
          <el-input
            type="textarea"
            placeholder="stdout"
            v-model="stdout"
            :autosize="{ minRows: 8, maxRows: 20 }"
            style="width: 100%; margin-bottom: 20px"
          >
          </el-input></div
      ></el-col>
    </el-row>
  </div>
</template>

<script>
import axios from "axios";
import { codemirror } from "vue-codemirror-lite";

// theme
import "codemirror/theme/idea.css";
import "codemirror/theme/darcula.css";

// mode
import "codemirror/mode/clike/clike.js";

// active-line.js
import "codemirror/addon/selection/active-line.js";

// foldGutter
import "codemirror/addon/fold/foldgutter.css";
import "codemirror/addon/fold/foldgutter.js";
import "codemirror/addon/fold/brace-fold.js";
import "codemirror/addon/fold/indent-fold.js";

import "codemirror/addon/edit/matchBrackets";
import "codemirror/addon/edit/closebrackets";
export default {
  name: "Main",
  components: {
    codemirror,
  },
  data() {
    return {
      code: "",
      span_editor: 24,
      stdin: "",
      stdout: "",
      token: "",
      ondebug: false,
      mode: false,
      openDebugFlag: false,
      options: {
        // codemirror options
        tabSize: 4,
        indentUnit: 4,
        indentWithTabs: true,
        styleActiveLine: true,
        matchBrackets: true,
        autoCloseBrackets: true,
        mode: "text/x-c++src",
        lineNumbers: true,
        theme: "idea",
        line: true,
        // 代码折叠
        foldGutter: true,
        gutters: ["CodeMirror-linenumbers", "CodeMirror-foldgutter"],
        // 选中文本自动高亮，及高亮方式
        styleSelectedText: true,
        lineWrapping: true,
        highlightSelectionMatches: { showToken: /\w/, annotateScrollbar: true },
      },
    };
  },
  mounted() {
    let _this = this;
    let Base64 = require("js-base64").Base64;

    // 读取code
    let code = localStorage.getItem("codemirror_code");
    if (code) _this.code = Base64.decode(code);

    this.editor.focus();
    this.bindKey();

    this.resize();
    window.onresize = () => {
      this.resize(); // 动态调整高度
    };

    setInterval(() => {
      if (_this.code.length > 0) {
        let code = Base64.encode(_this.code);
        localStorage.setItem("codemirror_code", code);
        console.log("code saved");
      }
    }, 30000);
  },
  methods: {
    go() {
      window.open("https://github.com/Fromnowon/IDE");
    },
    openDebug() {
      this.span_editor = 40 - this.span_editor;
    },
    debug() {
      if (this.code.length == 0) {
        this.$message.error("代码不能为空");
        return;
      }
      const _this = this;
      _this.ondebug = true;
      _this.stdout = "";
      let Base64 = require("js-base64").Base64;
      axios
        .post(
          "http://47.107.131.235:2358/submissions/?base64_encoded=true&wait=false",
          {
            source_code: Base64.encode(_this.code),
            language_id: 54,
            stdin: Base64.encode(_this.stdin),
          }
        )
        .then(function (response) {
          _this.token = response.data.token;
          _this.getResult();
        });
    },
    getResult() {
      const _this = this;
      let Base64 = require("js-base64").Base64;
      let loop = setInterval(() => {
        axios
          .get(
            "http://47.107.131.235:2358/submissions/" +
              _this.token +
              "?base64_encoded=true"
          )
          .then(function (response) {
            let e = response.data;
            if (e.status.id != 1 && e.status.id != 2) {
              clearInterval(loop);
              _this.ondebug = false;
              if (e.status.id == 3) {
                _this.stdout = Base64.decode(e.stdout || "5peg");
              } else {
                _this.$message({
                  message: "调试结果异常，请查看输出信息",
                  type: "warning",
                });
                if (e.status.id == 6)
                  _this.stdout = Base64.decode(e.compile_output);
                else _this.stdout = e.status.description;
              }
            }
          });
      }, 1000);
    },
    resize() {
      this.editor.setSize("100%", document.documentElement.clientHeight + "px");
    },
    bindKey() {
      const _this = this;
      let Base64 = require("js-base64").Base64;
      window.addEventListener(
        "keydown",
        function (e) {
          if ((e.key == "r" || e.key == "R") && e.altKey) {
            e.preventDefault();
            _this.span_editor = 40 - _this.span_editor;
          }
          if ((e.key == "s" || e.key == "S") && e.altKey) {
            e.preventDefault();
            if (_this.code.length > 0) {
              let code = Base64.encode(_this.code);
              localStorage.setItem("codemirror_code", code);
              console.log("code saved");
            }
          }
        },
        false
      );
    },
  },
  computed: {
    editor() {
      // get current editor object
      return this.$refs.myEditor.editor;
    },
  },
  watch: {
    mode(v) {
      if (v) this.editor.setOption("theme", "darcula");
      else this.editor.setOption("theme", "idea");
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
  },
};
</script>

<style>
.main {
  position: relative;
  width: 100%;
  height: 100%;
}
.CodeMirror {
  padding-right: 3px;
  border-right: 0.1pt solid rgba(0, 0, 0, 0.3);

  font-size: 12pt !important;
}
.CodeMirror-scroll {
  padding-top: 10px;
}
.debug {
  padding: 20px;
}
.btn {
  z-index: 9;
  position: absolute;
  top: 20px;
  right: 20px;
  color: rgba(0, 0, 0, 0.3);
  font-size: 20pt;
}
.btn:hover {
  cursor: pointer;
  color: #409eff;
  transition: color 0.5s;
}
.debug_div {
  transition: all 1s !important;
}
.info {
  z-index: 9;
  position: absolute;
  bottom: 30px;
  right: 30px;
}
</style>
