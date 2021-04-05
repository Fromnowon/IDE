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
        </VueAceEditor>
      </el-col>
      <el-col :span="24 - span_editor" class="debug_div">
        <div class="info">
          <span style="font-size: 10pt">Develop by Zihang : </span>
          <el-button type="text" @click="go()">Github</el-button>
        </div>
        <div v-show="span_editor != 24" class="debug">
          <h4>设置</h4>
          <div style="padding: 0 10px">
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
              <el-col :span="12"
                ><p>提示</p>
                <el-switch
                  v-model="autoCmp"
                  active-color="#13ce66"
                  inactive-color="#C0C4CC"
                >
                </el-switch
              ></el-col>
            </el-row>
            <p>语言</p>
            <el-select v-model="mode" size="small" style="max-width: 120pt">
              <el-option :key="0" :label="'C++ (GCC 9.2.0)'" :value="'c_cpp'">
              </el-option>
              <el-option :key="1" :label="'Python (3.8.1)'" :value="'python'">
              </el-option>
            </el-select>
            <p>字号</p>
            <el-slider v-model="fontsize" :min="10" :max="30"></el-slider>
          </div>
          <h4>调试</h4>
          <div style="padding: 0 10px">
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
            </el-input>
          </div></div
      ></el-col>
    </el-row>
  </div>
</template>

<script>
import axios from "axios";
import { VueAceEditor } from "vue2x-ace-editor";

export default {
  name: "Main",
  components: {
    VueAceEditor,
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
      stdout: "",
      token: "",
      mode: "c_cpp",
      ondebug: false,
      openDebugFlag: false,
      autoCmp: false,
      options: {
        enableSnippets: true,
        enableBasicAutocompletion: true,
        enableLiveAutocompletion: false,
        tabSize: 4,
      },
    };
  },
  mounted() {
    let _this = this;

    this.bindKey();

    this.resize();
    window.onresize = () => {
      this.resize(); // 动态调整高度
    };

    window.onbeforeunload = () => {
      _this.save();
    };
  },
  methods: {
    ace_editorInit(editor) {
      require("brace/ext/language_tools");
      require(`brace/snippets/c_cpp`);
      require("brace/mode/c_cpp");
      require(`brace/snippets/python`);
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
      // 补全
      this.$refs.ace_editor.setCompleteData([
        { meta: "custom", caption: "include", value: "include", score: 1000 },
        { meta: "custom", caption: "iostream", value: "iostream", score: 1000 },
        {
          meta: "custom",
          caption: "algorithm",
          value: "algorithm",
          score: 1000,
        },
        { meta: "custom", caption: "cstring", value: "cstring", score: 1000 },
      ]);
      this.editor = editor;
      editor.focus();
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
    },
    saveCode() {
      let Base64 = require("js-base64").Base64;
      let code = Base64.encode(this.editor.getValue());
      localStorage.setItem("ace_code", code); // 保存代码
    },
    save() {
      let Base64 = require("js-base64").Base64;
      this.saveCode();
      localStorage.setItem("ace_theme", this.theme); // 保存主题
      localStorage.setItem("ace_fontsize", this.fontsize); // 保存字号
      localStorage.setItem("ace_auto", this.autoCmp); // 保存补全
      localStorage.setItem("ace_lang", this.mode); // 保存语言
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
      let Base64 = require("js-base64").Base64;
      let code = Base64.encode(this.editor.getValue());

      const _this = this;
      _this.ondebug = true;
      _this.stdout = "";

      axios
        .post(
          "http://47.107.131.235:2358/submissions/?base64_encoded=true&wait=false",
          {
            source_code: code,
            language_id: _this.mode == "c_cpp" ? 54 : 71,
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
                console.log(e);
                _this.$message({
                  message: "调试结果异常，请查看输出信息",
                  type: "warning",
                });
                if (e.status.id == 6)
                  _this.stdout = Base64.decode(e.compile_output);
                else if (e.status.id == 11)
                  // runtime error
                  _this.stdout =
                    e.status.description + "\n" + Base64.decode(e.stderr);
                else _this.stdout = Base64.decode(e.stderr);
              }
            }
          });
      }, 1000);
    },
    resize() {
      this.height = document.documentElement.clientHeight;
    },
    bindKey() {
      const _this = this;
      window.addEventListener(
        "keydown",
        function (e) {
          if ((e.key == "r" || e.key == "R") && e.altKey) {
            e.preventDefault();
            _this.span_editor = 40 - _this.span_editor;
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
  top: 10px;
  right: 30px;
  color: rgba(0, 0, 0, 0.3);
  font-size: 20pt;
}
.btn:hover {
  cursor: pointer;
  color: #409eff;
  transition: color 0.5s;
}
.editor_div {
  box-shadow: 0px 0px 10px lightgrey;
}
.debug_div {
  padding-left: 10px;
}
.info {
  z-index: 9;
  position: absolute;
  bottom: 30px;
  right: 30px;
}
</style>
