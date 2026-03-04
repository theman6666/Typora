# Git命令使用笔记与个人经验

## 1. 个人经验

### 1.1 云端的代码不再master分支，而是在其他分支：

首先我们需要进行项目的克隆

```bash
git clone 你的链接
```

然后要进行远程的文件追踪：

```bash
# 如果是刚刚clone，本地还没有其他的分支（假设是develop）
git checkout -t origin/develop

# 如果本地已经有了develop分支，但是没有追踪远程
git branch --set-upstream-to = origin/develop

# 如果本地是脏数据，云端才是正确的
git fetch origin
git reset --hard origin/develop
```

追踪完之后，如何查看当前的跟踪状态？

```bash
# 查看远程有哪些分支
git branch -r

# 查看当前追踪关系
git branch -vv
```

### 1.2 本地已开发完成，但需要新建分支提交

#### 场景

当前在 `develop`（或其他分支）上已经写了一些代码，但这些代码**应该提交到一个新的分支**

#### 情况一：当前的代码还没有commit（最常见）

```bash
# 1.基于当前状态，直接创建一个新的分支
git checkout -b feature/your-feature-name

# 2.确认代码仍然存在
git status

# 3.提交代码
git add .
git commit -m "你自己的描述"

# 4.推送到远端并建立追踪
git push -u origin feature/your-feature-name
```

说明：

- `checkout -b` **不会丢失任何未提交的改动**

- 本地改动会自动带到新分支中

#### 情况二：不小心将commit提交在了错误的分支

```bash
# 1. 创建新分支，指向当前 commit
git branch feature/your-feature-name

# 2. 切换到新分支
git checkout feature/your-feature-name

# 3. 回退原分支（比如 develop，如果你不想在develop上撤销这次的更改，你就不用运行下面的命令））
git checkout develop
git reset --hard HEAD~1
```

 

---

## 2. 一些常见命令的解析：

### 2.1 `git reset --hard HEAD~1 `

> **`git reset --hard HEAD~1` 的作用是：  
> 把“当前分支”强制回退到上一个 commit，并丢弃这次提交之后的所有本地修改。**

关键词：

- **当前分支**

- **强制**

- **回退 1 个 commit**

- **不可恢复（除非 reflog）**

#### **（1）逐词拆解**

 **`git reset`**

含义：

> **移动当前分支指针（branch pointer）**

本质：

- 不创建新 commit

- 只是“改指向”

**`--hard`**

`reset` 有三种常见模式：

| 模式            | 影响分支指针 | 影响暂存区 | 影响工作区 |
| ------------- | ------ | ----- | ----- |
| `--soft`      | ✅      | ❌     | ❌     |
| `--mixed`（默认） | ✅      | ✅     | ❌     |
| `--hard`      | ✅      | ✅     | ✅     |

👉 **`--hard` 是最彻底、最危险的**

它会：

- 移动分支指针

- 重置 index（stage）

- 重置 working tree（你本地的代码）

**`HEAD`**

- `HEAD`：**当前分支的最新 commit**

- 通常等价于：
  
  ```bash
  git rev-parse HEAD
  ```

**`HEAD~1`**

这是 Git 的**相对引用语法**：

- `HEAD~1`：上一个 commit

- `HEAD~2`：上上个 commit

例子：

```text
A --- B --- C   ← HEAD
      ↑
   HEAD~2
```

#### **（2）把整条命令连起来读**

```bash
git reset --hard HEAD~1
```

完整含义是：

> **把当前分支指针移动到“上一个 commit”，  
> 并让暂存区和工作区的内容，强制和该 commit 保持一致。**

#### （3）执行前后发生了什么（图解）

**执行前**

```text
A --- B
        ↑
     develop (HEAD)
```

**执行后**

```text
A --- B
↑
develop (HEAD)
```

- commit `B` 不再属于 `develop`

- 如果没有其他分支指向 `B`，它会变成“悬空 commit”

#### （4） 典型使用场景

  ✅ 场景 1：**刚刚 commit 写错了**

```bash
git reset --hard HEAD~1
```

重新改代码 → 重新 commit。

✅ 场景 2：**误把功能提交到错误分支**

```bash
# 先把 commit 保存在新分支
git branch feature/xxx

# 再让当前分支回退
git reset --hard HEAD~1
```

❌ 不适合的场景

- commit 已经 `push` 到远端

- 团队公共分支（develop / main）

#### （5）风险与恢复（一定要写）

 ⚠️ 风险点

- `--hard` 会 **直接丢弃未提交的修改**

- 误用可能导致代码“消失”

🛟 补救方式（极重要）

```bash
git reflog
```

你可以找到刚才的 commit hash：

```bash
git reset --hard <commit-id>
```

👉 **这就是你的“后悔药”**

#### （6）和其他 reset 的对比（简记版）

| 命令                        | 常用场景             |
| ------------------------- | ---------------- |
| `git reset --soft HEAD~1` | 改 commit message |
| `git reset HEAD~1`        | 取消 add，不丢代码      |
| `git reset --hard HEAD~1` | 彻底回退，重来          |

#### （7）总结：

> **`git reset --hard HEAD~1`**  
> 用于将当前分支强制回退到上一个 commit。  
> 该操作会同时重置分支指针、暂存区和工作区，  
> 丢弃所有未提交和已提交但被回退的修改。  
> ⚠️ 已 push 的提交慎用，可通过 `git reflog` 恢复。

---

### 2.2 echo "your_folder_name/" >> .git/info/exclude

这个命令是用来如果你有一些文件是不用上传上去的，但是你又不想写在.gitignore里面时需要用上的

#### 只需要使用 `echo` 追加一行规则

```bash
echo "your_folder_name/" >> .git/info/exclude
```

- `>>` 表示追加内容到文件末尾（如果文件不存在，则会创建）。

- 规则语法与 `.gitignore` 完全一致，例如忽略 `temp` 文件夹：`temp/`

**注意**：如果 `.git/info/exclude` 文件不存在，这条命令会自动创建它（前提是 `.git/info` 目录已存在）。通常 Git 仓库初始化后该目录是存在的。

---

## 3.git本地名字和邮箱的更改

#### 一、修改本地 Git 用户名（影响提交记录里的作者名）

##### 查看当前用户名

```
git config user.name
```

如果显示你现在改好的名字，说明已经改好了。

##### 修改当前项目的用户名（只影响这个仓库）

```
git config user.name "你想要的名字"
```

例如：

```
git config user.name "DeepDreamChaser"
```

##### 修改全局用户名（影响本机所有 Git 仓库）

```
git config --global user.name "你想要的名字"
```

### 二、修改本地 Git 邮箱

#### 查看当前的邮箱

```bash
git config user.email
```

#### 修改当前仓库的邮箱

```bash
git config user.email "你的邮箱"
```

#### 修改全局邮箱（影响本机所有 Git 仓库）

```bash
git config --global user.email "你的邮箱"
```
