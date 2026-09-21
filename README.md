# ”2026江苏省大学新生安全知识教育“一键完成脚本
源作者项目：
jiangsu-safety-platform-skip

**🤔 食用指导**

1.安装 Python3 ，并且确保安装了这个额外的库： requests 。

2.程序只有登陆版一个版本，对应仓库内的 main.py ，登陆版需要输入学校名称、账号和密码。

3.针对于平台 2026 年 08 月 28 日的策略进行了略微的调整，采用 session 对全局进行管理，并移除了 userid 版本，回归 main 分支，并将发布版本回退到 v1.0.6 。

4.请尊重 Apache2.0 协议与开源精神，二次开发保留原作者信息。

⚙ **基本原理**

通过数据包重放的方式完成课程学习，通过将考题对应答案写入 database.db 中来实现答案获取和处理。

✒️ **进阶**

欢迎提交 Issue 来交换您的看法和对脚本的更多建议！

作者：南京晓庄学院 Scwizard

修复：南京信息职业技术学院  一屿

感谢：ECXiaobai | Leeyus | Mr_Zhen_cn (排名不分先后) 对本项目的贡献




# 2026-09-21修复
## 修复 jiangsu-safety-platform-skip 脚本（main/jiangsu-safety-platform-skip-main）

### 问题根因（3 个）
1. **课程提交被学习行为校验拦截**：平台新增 B3 埋点，答题前必须 POST `/wap/markArticleViewed`（articleId, userId）上报"课件已学完"，否则 unitTest 返回"请先完成本课程的学习后再作答"。发现方式：抓 `/wap/article` 页面内联 JS（`createExamSession()`）。
2. **考试最短答题时长上调到 300 秒**（原来 5 秒），不足返回 1006。课程单元测试最短 10 秒即可。
3. **creatExam 失败时 data 是字符串**，`res["data"]["logId"]` 抛 `string indices must be integers`（实际是课程未完成的连锁反应）。

### 修改的文件
- `main.py`（v1.0.9）：finish_course 增加 markArticleViewed 调用；单元测试 1006 重试；EXAM_WAIT_SECONDS=300；creatExam 返回值校验；WAIT_SECONDS=10
- `utils.py`：新增 `markArticleViewed()`；`end()` 加默认参数；`getUserSchool()` 递归改循环（原来失败时返回 None）

### 逆向方法论（可复用）
- 平台 500 = 路径存在但参数/前置条件不满足，404 = 路径不存在
- layui 前端：页面 JS 在 `/guns-vip-main/assets/modular/wap/<页面名>.js`，无需鉴权可直接读，从中找 ajax 端点
- 课程页流转：compulsory.js → `/wap/directory` → directory.js → `/wap/article` → 内联 JS 有全部提交逻辑