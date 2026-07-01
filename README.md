# 岗位测评平台 · 部署包

这个文件夹就是要挂到网上的全部内容：

| 文件 | 用途 | 上线后网址 |
|---|---|---|
| `index.html` | **答题端**（发给各组组长） | `https://你的用户名.github.io/仓库名/` |
| `admin.html` | **管理后台**（你自己用） | `https://你的用户名.github.io/仓库名/admin.html` |

> 用英文文件名 + 根目录 `index.html`，是为了让分享到飞书/微信的链接干净、不出乱码。

---

## ⚠️ 上线前必做：更新一下数据库

打开 **Supabase → SQL Editor → New query**，把**完整的 `~/supabase_setup.sql`** 内容全部粘进去 → Run。
脚本是幂等的（重复运行不报错），这一次会一并完成：

1. 补上缺失的 `roster_empty()` 函数（不补的话答题端打不开）；
2. 新增 `shuffle` / `timelimit` 两列（题目乱序、限时用）；
3. 更新 `submit_quiz`，把「成绩显示方式」改到服务器端判定（防止有人直接调接口扒答案）；
4. 新增题干/选项配图 `images` 列 + 连线题判分 + 建图片存储桶 `quiz-images`（做「图片题、连线题」必须）。

> 如果你只想先救急、暂不要新功能，也可以只跑 `~/修复_roster_empty.sql`——但乱序/限时/防捞答案要等跑完整脚本才生效。

---

## 方式一：命令行发布（最快，需先登录 GitHub）

```bash
# 1. 登录 GitHub（浏览器授权，一次即可）
gh auth login

# 2. 进入部署包目录并初始化
cd ~/ceping-deploy
git init -b main
git add .
git commit -m "岗位测评平台上线"

# 3. 一条命令创建仓库并推送（公开仓库）
gh repo create job-assessment --public --source=. --push

# 4. 开启 GitHub Pages
gh api -X POST repos/{owner}/job-assessment/pages -f 'source[branch]=main' -f 'source[path]=/' 2>/dev/null || \
  echo "若上面报错，就到仓库网页 Settings → Pages → Branch 选 main → Save"
```

几分钟后访问：
- 答题端：`https://你的用户名.github.io/job-assessment/`
- 管理后台：`https://你的用户名.github.io/job-assessment/admin.html`

## 方式二：网页上传（不想用命令行）

1. 打开 https://github.com/new 建一个仓库（如 `job-assessment`，选 Public）
2. 在仓库页点 **Add file → Upload files**，把本文件夹里的 `index.html` 和 `admin.html` 拖进去 → Commit
3. **Settings → Pages → Branch 选 `main` → Save**
4. 等几分钟，页面上会显示两个网址（同上）

---

## 题目批量导入（Excel 模板）

后台「题库管理」顶部有 **📊 Excel 模板导入题目**：
1. 点 **⬇ 下载导入模板** → 得到 `题目导入模板.xlsx`
2. 按「一行一题」填：分组 / 题型（单选·多选·判断·排序·填空）/ 题干 / 选项A-F / 正确答案
   - 单选填一个字母(B)，多选填多个(ABD)，排序按先后填字母(CADB)，判断填 对/错，填空多答案用 / 隔开
   - 分组名要和「设置」里的一致；判断题、填空题选项列留空
3. 上传 → 预览（每行显示可导入/问题原因）→ 确认导入

> 起点文件 `~/题目导入_起点.xlsx` 已从旧试卷自动提取了 22 道文字题（单选/多选/判断），可直接上传；旧试卷里的**排序、连线、图片实景题**需手动补录（平台不支持图片/连线题）。

## 上线后自检清单

1. 打开 `admin.html` → 用你在 Supabase 建的管理员邮箱/密码登录
2. 「设置」里确认分组、及格线；「题库管理」加几道题；（可选）「人员名单」传 Excel
3. 打开 `index.html`（答题端）→ 填姓名 → 能进去答题、提交后能出成绩
4. 回到后台「成绩记录」→ 能看到刚才的提交

> 管理员账号还没建？Supabase → Authentication → Users → Add user，填邮箱密码并勾选 Auto Confirm。
