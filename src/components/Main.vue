<template>
  <div :class="'main ' + (theme != 'vs' ? 'dark' : 'vs')">
    <el-container>
      <el-header
        :height="barHeight + 'px'"
        style="border-bottom: 1px solid rgba(144, 147, 153, 0.3)"
      >
        <div
          class="bar"
          :style="{ height: barHeight + 'px', lineHeight: barHeight + 'px' }"
        >
          <div
            :style="{
              height: barHeight + 'px',
              lineHeight: barHeight + 'px',
              float: 'left',
            }"
            class="items format_tip_div"
          >
            <div class="format_tip" v-if="formated">
              <span>代码已格式化</span>
              <i
                class="el-icon-check"
                title="确认"
                style="margin-left: 10px; color: #67c23a; cursor: pointer"
                @click="(formated = false), clearTimeout_(formatTip)"
              ></i>
              <i
                class="el-icon-refresh-left"
                title="撤销"
                style="margin-left: 10px; color: #f56c6c; cursor: pointer"
                @click="
                  (formated = false), (code = code__), clearTimeout_(formatTip)
                "
              ></i>
            </div>
          </div>
          <div class="items" :style="{ float: 'right' }">
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
            <el-popover placement="top" width="160" trigger="click">
              <div style="text-align: center">
                <el-button type="text" @click="read">打 开</el-button>
                <el-button type="text" @click="saveToLocal">保 存</el-button>
              </div>
              <div slot="reference" class="btn_settings btn">
                <i class="el-icon-folder"></i>
              </div>
            </el-popover>
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
          </div>
        </div>
      </el-header>
      <el-container>
        <el-main>
          <div class="editor_div">
            <MonacoEditor
              v-show="!diff"
              :style="{ height: height + 'px', width: '100%' }"
              ref="manoco"
              v-model="code"
              :theme="theme"
              :language="mode"
              :options="options"
              @editorWillMount="manocoWillMountHandler"
              @editorDidMount="manocoMountHandler"
            />
            <MonacoEditor
              ref="manoco_diff"
              v-show="diff"
              :style="{ height: height + 'px', width: '100%' }"
              :language="mode"
              :options="options"
              :diffEditor="true"
              :value="code"
              :original="code_"
              @editorDidMount="DiffMountHandler"
              @change="diffChangeHandler"
            />
          </div>
        </el-main>
        <el-aside
          :width="debug_width + 'px'"
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
              <el-input
                spellcheck="false"
                :style="{
                  overflow: 'auto',
                }"
                type="textarea"
                resize="none"
                v-model="stdin"
                @focus="autoSelectStdin"
              >
              </el-input>
            </el-header>
            <div
              class="area_tip"
              style="border-top: 1px solid rgba(144, 147, 153, 0.3)"
            >
              Stdout<i
                v-show="stdout"
                style="margin-left: 10px; cursor: pointer; color: #e6a23c"
                @click="stdout = ''"
                title="清空"
                class="el-icon-delete"
              />
            </div>
            <el-main
              :style="{
                height: debug_output_height - 20 + 'px',
                overflow: 'hidden',
              }"
            >
              <MonacoEditor
                v-if="debug_width > 0"
                :style="{
                  height: debug_output_height - 20 + 'px',
                  width: debug_width,
                  padding: '8px 0',
                }"
                ref="stdout"
                v-model="stdout"
                :theme="theme"
                :language="mode"
                :options="{
                  lineNumbers: 'off',
                  glyphMargin: false,
                  folding: false,
                  lineDecorationsWidth: 14,
                  lineNumbersMinChars: 0,
                  wordWrap: true,
                  readOnly: true,
                  glyphMargin: false,
                  overviewRulerBorder: false,
                  minimap: {
                    enabled: false,
                  },
                  scrollbar: {
                    verticalScrollbarSize: 8,
                    horizontalScrollbarSize: 8,
                  },
                }"
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
              <el-option :key="0" :label="'C++ (GCC 9.2.0)'" :value="'cpp'">
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
        <el-row>
          <p>主题</p>
          <el-col>
            <el-select
              v-model="theme"
              placeholder="请选择"
              style="max-width: 120pt"
            >
              <el-option key="1" label="浅色" value="vs"> </el-option>
              <el-option key="2" label="深色" value="vs-dark"> </el-option>
            </el-select>
          </el-col>
        </el-row>
        <p>字号</p>
        <el-slider v-model="options.fontSize" :min="10" :max="30"></el-slider>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="settingsDialogVisible = false">关 闭</el-button>
      </span>
    </el-dialog>
    <el-dialog title="提示" :visible.sync="helpDialogVisible" width="40%">
      <h4>关于</h4>
      <p>编辑器支持C++、Python代码高亮及运行</p>
      <p>
        GitHub：<el-link type="primary" @click="go"
          >https://github.com/Fromnowon/IDE</el-link
        >
      </p>
      <h4>快捷键</h4>
      <p><span style="color: #409eff">F9</span> - 运行代码</p>
      <p><span style="color: #409eff">Ctrl + F</span> - 查找，替换</p>
      <p><span style="color: #409eff">Ctrl + Enter</span> - 下方插入一行</p>
      <p></p>
      <p><span style="color: #409eff">Ctrl + \</span> - 拆分编辑器</p>
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
        left: width - debug_width - 30 + 'px',
      }"
    >
      <i v-if="debug_width" class="el-icon-d-arrow-right"></i>
      <i v-else class="el-icon-d-arrow-left"></i>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import moment from "moment";
import { js_beautify } from "js-beautify";
import MonacoEditor from "vue-monaco";
const Base64 = require("js-base64").Base64;
import { Loading } from "element-ui";

export default {
  name: "Main",
  components: {
    MonacoEditor,
  },
  data() {
    return {
      editor: "",
      editor_diff: "",
      width: null,
      height: null,
      theme: "vs",
      code: "",
      code__: "",
      code_: "",
      diff: false,
      stdin: "",
      stdout: "",
      token: "",
      mode: "cpp",
      server: "",
      hints: [], // 关键词
      fun_hints_key: [], // 函数名
      fun_hints: {}, // 函数补全
      decorations: [], // 错误行标记
      ondebug: false,
      tip: null,
      barHeight: 32,
      loadingInstance: null,
      settingsDialogVisible: false,
      helpDialogVisible: false,
      debug_width: 400,
      debug_output_height: 600,
      formated: false,
      formatTip: null,
      options: {
        fontSize: 18,
        tabSize: 4,
        roundedSelection: false,
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
      this.loadingInstance = Loading.service({ fullscreen: true });
      this.resize();
      window.onresize = () => {
        this.editor.layout();
        this.resize(); // 动态调整高度
      };
      window.onbeforeunload = () => {
        this.save();
      };
      this.dragLoadFileInit();
      this.initDrag();
      // 初始化
      this.loadConf();
    },
    diffChangeHandler(v, e) {
      this.code = v;
    },
    DiffMountHandler() {
      this.code_ = this.code;
    },
    manocoWillMountHandler(monaco) {
      window.monaco = monaco;
    },
    manocoMountHandler(editor) {
      this.editor = editor;
      // 全局绑定f9
      window.addEventListener("keydown", (e) => {
        if (e.keyCode == 120) {
          //调试
          e.preventDefault();
          this.debug();
        }
        if (e.ctrlKey && e.keyCode == 220) {
          e.preventDefault();
          this.diff = !this.diff;
        }
      });

      // 菜单，快捷键
      const arr = [
        {
          id: "1", // 菜单项 id
          label: "Run", // 菜单项名称
          keybindings: [window.monaco.KeyCode.F9], // 绑定快捷键
          contextMenuGroupId: "1_modification", // 所属菜单的分组
          run: () => {
            this.debug();
          }, // 点击后执行的操作
        },
        {
          id: "2", // 菜单项 id
          label: "Format", // 菜单项名称
          keybindings: [
            window.monaco.KeyMod.Alt |
              window.monaco.KeyMod.Shift |
              window.monaco.KeyCode.KEY_F,
          ],
          contextMenuGroupId: "1_modification", // 所属菜单的分组
          run: () => {
            clearTimeout(this.formatTip); // 可能在倒计时内多次格式化
            this.code__ = this.code;
            this.formated = true;
            this.code = js_beautify(this.code, {
              indent_size: 4,
              brace_style: "collapse,preserve-inline",
              space_before_conditional: true,
              max_preserve_newlines: 2,
              wrap_attributes: "auto",
            });
            this.formatTip = setTimeout(() => {
              this.formated = false;
            }, 30000);
          },
        },
        {
          id: "3", // 菜单项 id
          label: "Clear Markers", // 菜单项名称
          contextMenuGroupId: "1_modification", // 所属菜单的分组
          run: () => {
            this.clearMarkers();
          },
        },
      ];
      for (let i = 0; i < arr.length; i++) editor.addAction(arr[i]);

      // 请求数据
      axios
        .get("http://106.52.130.81/lib/ide.json?" + Date.now())
        .then((res) => {
          this.hints = res.data.keywords;
          this.server = res.data.server;
          const functions = res.data.snippets;
          this.fun_hints = functions;
          for (let key in functions) this.fun_hints_key.push(key);
          this.initCompletion();
          this.$nextTick(() => {
            // 以服务的方式调用的 Loading 需要异步关闭
            this.loadingInstance.close();
          });
        });

      // 清除错误标记
      editor.onKeyDown((e) => {
        const startLineNumber = this.editor.getSelection().startLineNumber;
        const endLineNumber = this.editor.getSelection().endLineNumber;
        const model = this.editor.getModel();

        const arr = model.getLinesDecorations(startLineNumber, endLineNumber);
        arr.map((ds) => {
          if (this.decorations.includes(ds.id)) {
            model.deltaDecorations([ds.id], []);
          }
        });
      });
    },
    clearTimeout_(ins) {
      clearTimeout(ins);
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
          const minWidth = 100;
          _this.debug_width =
            _this.width - disX - divX - 20 > 1000
              ? 1000
              : _this.width - disX - divX - 20 <= minWidth
              ? minWidth
              : _this.width - disX - divX - 20;
          let n_height = _this.height - disY - divY + _this.barHeight + 10;
          _this.debug_output_height =
            n_height > _this.height * 0.8
              ? _this.height * 0.8
              : n_height < _this.height * 0.2
              ? _this.height * 0.2
              : n_height;
          _this.resize();
          document.onmouseup = function () {
            if (_this.debug_width <= minWidth) {
              _this.debug_width = 0;
              _this.resize();
            }
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
    initCompletion() {
      // 为该语言注册一个语言提示器--联想
      const that = this;
      let createCompleters = (textUntilPosition) => {
        const monaco = window.monaco;
        //过滤特殊字符
        let _textUntilPosition = textUntilPosition
          .replace(/[\*\[\]@\$\(\)]/g, "")
          .replace(/(\s+|\.|\#|\<|\>)/g, " "); // 以+ . # < >分割字符串
        //切割成数组
        let arr = _textUntilPosition.split(" ");
        //取当前输入值
        let activeStr = arr[arr.length - 1];
        //获得输入值的长度
        let len = activeStr.length;

        //获得编辑区域内已经存在的内容
        let rexp = new RegExp("([^\\w]|^)" + activeStr + "\\w*", "gim");
        let match = that.code.match(rexp);
        let _hints = !match
          ? []
          : match.map((ele) => {
              let rexp = new RegExp(activeStr, "gim");
              let search = ele.search(rexp);
              return ele.substr(search);
            });

        //查找匹配当前输入值的元素
        let hints = Array.from(
          new Set([...that.hints, ...that.fun_hints_key, ..._hints])
        )
          .sort()
          .filter((ele) => {
            let rexp = new RegExp(ele.substr(0, len), "gim");
            return (match && match.length === 1 && ele === activeStr) ||
              ele.length === 1
              ? false
              : activeStr.match(rexp);
          });
        //添加内容提示
        let res = hints.map((ele) => {
          if (that.hints.indexOf(ele) > -1) {
            // 匹配关键字
            return {
              label: ele,
              kind: monaco.languages.CompletionItemKind.Keyword,
              documentation: ele,
              insertText: ele,
            };
          } else if (that.fun_hints_key.indexOf(ele) > -1) {
            // 匹配函数
            return {
              label: ele,
              kind: monaco.languages.CompletionItemKind.Function,
              insertTextRules:
                monaco.languages.CompletionItemInsertTextRule.InsertAsSnippet,
              documentation: ele,
              insertText: that.fun_hints[ele],
            };
          } else {
            // 否则加入本地
            return {
              label: ele,
              kind: monaco.languages.CompletionItemKind.Text,
              documentation: ele,
              insertText: ele,
            };
          }
        });
        return res;
      };
      window.monaco.languages.registerCompletionItemProvider("cpp", {
        provideCompletionItems(model, position) {
          var textUntilPosition = model.getValueInRange({
            startLineNumber: position.lineNumber,
            startColumn: 1,
            endLineNumber: position.lineNumber,
            endColumn: position.column,
          });
          var suggestions = createCompleters(textUntilPosition);
          return {
            suggestions: suggestions,
          };
        },
      });
    },
    clearStdin() {
      this.stdin = "";
    },
    autoSelectStdin(Event) { // stdin获取焦点自动全选
      Event.target.select();
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
        (this.mode == "cpp" ? ".cpp" : ".py");
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
      let code = Base64.encode(this.code);
      localStorage.setItem("zzh_code", code); // 保存代码
    },
    save() {
      this.saveCode();
      localStorage.setItem("zzh_fontsize", this.options.fontSize); // 保存字号
      localStorage.setItem("zzh_stdin", this.stdin); // 保存字号
      localStorage.setItem("zzh_lang", this.mode); // 保存语言
      console.log("saved");
    },
    loadConf() {
      // 读取信息
      const code = localStorage.getItem("zzh_code");
      if (code) this.code = Base64.decode(code);
      this.stdin = localStorage.getItem("zzh_stdin") || "";
      this.options.fontSize =
        parseInt(localStorage.getItem("zzh_fontsize")) || 18;
      this.mode = localStorage.getItem("zzh_lang") || "cpp";
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

      this.clearMarkers();
      // 提交提示
      this.tip = this.$message({
        dangerouslyUseHTMLString: true,
        message: "<strong>处理中，请稍等...</strong>",
        duration: 0,
        center: true,
      });

      let code = Base64.encode(this.code);

      const _this = this;
      _this.ondebug = true;
      _this.stdout = "";
      let stdin = _this.stdin;
      if (stdin[stdin.length - 1] != "\n") stdin += "\n";
      axios
        .post(
          "http://" +
            _this.server +
            "/submissions/?base64_encoded=true&wait=false",
          {
            source_code: code,
            language_id: _this.mode == "cpp" ? 54 : 71, // cpp python
            stdin: Base64.encode(stdin),
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
          this.$alert("请求异常，服务器无响应", "错误", {
            confirmButtonText: "确定",
            type: "error",
          });
        });
    },
    getResult() {
      const _this = this;
      setTimeout(() => {
        axios
          .get(
            "http://" +
              _this.server +
              "/submissions/" +
              _this.token +
              "?base64_encoded=true"
          )
          .catch((error) => {
            console.log(error);
            _this.ondebug = false;
            _this.tip.close();
            this.$alert("请求异常，服务器无响应", "错误", {
              confirmButtonText: "确定",
              type: "error",
            });
          })
          .then((response) => {
            let e = response.data;
            if (e.status.id != 1 && e.status.id != 2) {
              _this.tip.close();
              _this.ondebug = false;
              if (e.status.id == 3) {
                // 正常运行
                _this.stdout = Base64.decode(e.stdout || ""); // 程序执行成功并返回结果与信息
                _this.stdout +=
                  "\n\ntime: " +
                  parseFloat(e.time) * 1000 +
                  " ms\nmemory: " +
                  e.memory / 1000 +
                  " MB";
              } else {
                console.log(e);
                _this.$message({
                  message: "调试结果异常，请查看输出信息",
                  type: "warning",
                });
                if (e.status.id == 6) {
                  // 编译错误
                  const info = Base64.decode(e.compile_output);
                  _this.stdout = info;
                  if (_this.mode == "cpp") _this.getErr(info);
                } else if (e.status.id == 11 || e.status.id == 5)
                  // 段错误或超时
                  // runtime error
                  _this.stdout = e.status.description;
                else _this.stdout = Base64.decode(e.stderr);
              }
            } else {
              setTimeout(() => {
                _this.getResult();
              }, 1000);
            }
          });
      }, 1000);
    },
    getErr(r) {
      const arr = r.split("\n");
      this.decorations = [];
      arr.forEach((element, index) => {
        // 先验证main.cpp
        if (element.substr(0, 8) == "main.cpp") {
          // 判断第10位是否数字
          let str = element.substr(9, element.length);
          if (str[0] >= "0" && str[0] <= "9") {
            // 确定错误代码长度
            const tmp = arr[index + 2];
            const s = tmp.lastIndexOf("^") + 1;
            let _s = s;
            while (tmp[_s] == "~" && tmp[_s] != "\n") _s++;
            const len = _s - s + 1;
            // 到下个冒号之间为错误行号
            let row = str.substr(0, str.indexOf(":"));
            str = str.substr(str.indexOf(":") + 1, str.length);
            // 到下个冒号之间为错误列号
            let col = str.substr(0, str.indexOf(":"));
            // 最后一个冒号到换行为错误信息
            // const info = str.substr(str.indexOf(":") + 1, str.length).trim();
            let endCol = parseInt(col) + parseInt(len);
            if (
              endCol >
              this.editor.getModel().getLineContent(parseInt(row)).length
            )
              col = 0;
            const id = this.editor.getModel().deltaDecorations(
              // 添加错误行背景标记
              [],
              [
                {
                  range: new monaco.Range(
                    parseInt(row),
                    parseInt(col),
                    parseInt(row),
                    endCol
                  ),
                  options: {
                    // isWholeLine: true,
                    className:
                      this.theme == "vs"
                        ? "errorContentClassLight"
                        : "errorContentClassDark",
                  },
                },
              ]
            );
            this.decorations.push(id[0]);
          }
        }
      });
    },
    clearMarkers() {
      // 提交时清除所有错误
      const model = this.editor.getModel();
      model.deltaDecorations(this.decorations, []);
    },
    resize() {
      this.height = document.documentElement.clientHeight - this.barHeight;
      this.width = document.documentElement.clientWidth;
      this.$nextTick(() => {
        this.editor.layout();
        this.$refs.manoco_diff.getEditor().layout();
        this.$refs.manoco_diff.getModifiedEditor().layout();
        try {
          document.querySelector(".el-textarea__inner").style.height =
            this.height - this.barHeight - this.debug_output_height + "px";
          this.$refs.stdout.getEditor().layout();
        } catch (error) {}
      });
    },
  },
  watch: {
    theme(v) {
      this.clearMarkers();
      let stdin_el = document.querySelector(".el-textarea__inner");
      if (v != "vs") {
        stdin_el.style.background = "#1e1e1e";
        stdin_el.style.color = "white";
      } else {
        stdin_el.style.background = "#fffffe";
        stdin_el.style.color = "black";
      }
    },
    diff(v) {
      if (v) this.debug_width = 0;
      this.$nextTick(() => {
        this.editor.layout();
        this.$refs.manoco_diff.getEditor().layout();
        this.$refs.manoco_diff.getModifiedEditor().layout();
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
.dark {
  background: #1e1e1e;
  color: white;
}
.editor_div {
  overflow: hidden;
  border-right: 0.5px solid rgba(144, 147, 153, 0.3);
}
.debug {
  padding: 20px;
}
.el-textarea__inner {
  border: none !important;
  border-radius: 0 !important;
}
.bar {
  padding: 0 0 0 10px;
  font-size: 10px;
}
.btn {
  font-size: 14px;
  cursor: pointer;
  transition: all 0.5s;
  border-radius: 3px;
}
.btn:hover {
  background: rgba(0, 0, 0, 0.205);
}
.items {
  width: 250px;
  position: relative;
  text-align: right;
}
.btn_settings {
  height: 28px;
  line-height: 28px;
  width: 50px;
  text-align: center;
  display: inline-block;
}
.format_tip_div {
  display: flex;
  align-items: center;
  width: 150px;
}
.format_tip {
  font-size: 12px;
  padding: 0 8px;
  background-color: #fdf6ec;
  border-radius: 20px;
  line-height: 28px;
  height: 28px;
  width: 200px;
  text-align: center;
}
.run {
  background: #67c23a;
  width: 60px;
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
.stdout {
  font-size: 14px;
  padding: 10px;
  white-space: pre-wrap;
}
.area_tip {
  font-size: 14px;
  line-height: 24px;
  padding-left: 5px;
  border-bottom: 1px solid rgba(144, 147, 153, 0.2);
}

.errorContentClassLight {
  background: rgba(255, 0, 0, 0.35);
}
.errorContentClassDark {
  background: #8a202085;
}
/* 滚动条样式 */
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}
/* 滑轨 */
::-webkit-scrollbar-track {
  -webkit-box-shadow: inset 0 0 1px rgba(99, 99, 99, 0.2);
}
/* Handle */
::-webkit-scrollbar-thumb {
  background: rgba(206, 206, 206, 0.3);
}
::-webkit-scrollbar-thumb:window-inactive {
  background: rgba(144, 147, 153, 0.3);
}
::-webkit-scrollbar-thumb:hover {
  background-color: rgba(119, 119, 119, 0.274);
}
</style>
