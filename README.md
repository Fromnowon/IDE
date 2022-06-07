# IDE

在线IDE，上课自用，支持C++和Python在线运行并返回结果，编辑器为 [Monaco Editor](https://github.com/microsoft/monaco-editor)，后端采用 [Judge0](https://github.com/judge0/judge0)。

编辑器初始化时会从远程请求相关数据：[https://lib.zzh.today/ide.json](https://lib.zzh.today/ide.json)

demo: https://ide.zzh.today/

**更新记录：**
<details>
<summary>2022.6.7新增功能：File IO 模式</summary>
某些输入输出规模很大，无法正常复制粘贴到页面上，此时可以使用文件读取的方式。注意：
  <code>1、nginx需调整POST传输数据上限；2、由于输入数据需要进行Base64编码，输入数据很大时可能会导致页面短暂卡顿。
  </code>
  </details>

介绍：

1、界面

![image](https://user-images.githubusercontent.com/2792725/115114458-1afa1180-9fc2-11eb-9cd6-e042dde16290.png)

2、文件IO模式
![p](https://user-images.githubusercontent.com/2792725/172306525-04fe9261-8ab2-4751-b078-63acd48bb85c.png)

3、黑色主题，错误标记（通过解析后端返回错误信息实现，仅供参考）

![image](https://user-images.githubusercontent.com/2792725/115114499-5a286280-9fc2-11eb-8077-f2ce248f42e7.png)

4、代码对比

![image](https://user-images.githubusercontent.com/2792725/115114417-e1c1a180-9fc1-11eb-92ea-b320e721c1e6.png)
