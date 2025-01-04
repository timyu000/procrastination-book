# 电子书转换和 GitHub Pages 发布 SOP

本文档描述了如何将 EPUB 电子书转换为 HTML 并发布到 GitHub Pages 的标准操作程序。

## 前提条件

1. 安装 Calibre 电子书管理软件
2. 安装 Git
3. 拥有 GitHub 账号

## 步骤

### 1. 转换电子书为 HTML

```bash
# 1.1 将 EPUB 转换为 HTML 目录结构
ebook-convert "book.epub" "html_output" --pretty-print

# 1.2 如果找不到 ebook-convert 命令，添加 Calibre 命令行工具到 PATH
echo 'export PATH="/Applications/calibre.app/Contents/MacOS:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### 2. 创建 GitHub 仓库结构

```bash
# 2.1 创建新目录并复制文件
mkdir book-repo
cp -r html_output/* book-repo/

# 2.2 创建索引页面
cd book-repo
cat > index.html << 'EOL'
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>电子书标题</title>
    <link rel="stylesheet" href="stylesheet.css">
</head>
<body>
    <h1>电子书标题</h1>
    <h2>目录</h2>
    <div id="toc"></div>
    <script>
        fetch("toc.ncx").then(r => r.text()).then(text => {
            const parser = new DOMParser();
            const doc = parser.parseFromString(text, "text/xml");
            const items = doc.getElementsByTagName("navPoint");
            let html = "<ul>";
            for (let item of items) {
                const label = item.getElementsByTagName("text")[0].textContent;
                const content = item.getElementsByTagName("content")[0].getAttribute("src");
                html += `<li><a href="${content}">${label}</a></li>`;
            }
            html += "</ul>";
            document.getElementById("toc").innerHTML = html;
        });
    </script>
</body>
</html>
EOL
```

### 3. 初始化 Git 仓库

```bash
# 3.1 初始化仓库
git init
git add .
git commit -m "Initial commit: Add book content"
```

### 4. 创建 GitHub 仓库

1. 访问 GitHub 网站
2. 点击 "New repository"
3. 设置仓库名称（例如：book-name）
4. 选择仓库类型（公开/私有）
   - 注意：使用免费账户时，只有公开仓库可以使用 GitHub Pages

### 5. 推送到 GitHub

```bash
# 5.1 添加远程仓库（替换 USERNAME 和 REPO）
git remote add origin git@github.com:USERNAME/REPO.git

# 5.2 推送代码
git branch -M main
git push -u origin main
```

### 6. 配置 GitHub Pages

1. 在仓库页面点击 "Settings"
2. 在左侧菜单找到 "Pages"
3. 在 "Build and deployment" 部分：
   - Source: 选择 "Deploy from a branch"
   - Branch: 选择 "main" 分支和 "/ (root)" 目录
   - 点击 "Save"

### 7. 访问网站

- 等待几分钟后，可以通过 `https://USERNAME.github.io/REPO` 访问电子书
- 例如：`https://username.github.io/book-name`

## 注意事项

1. 确保有足够的磁盘空间
2. 检查文件权限
3. 私有仓库需要付费账户才能使用 GitHub Pages
4. 即使仓库是私有的，发布的网站仍然是公开的

## 故障排除

1. 如果 `ebook-convert` 命令不可用：
   - 检查 Calibre 是否正确安装
   - 确认 PATH 环境变量是否正确设置

2. 如果 GitHub Pages 不工作：
   - 检查仓库是否为公开
   - 确认是否正确选择了分支和目录
   - 查看 GitHub Actions 中的构建日志 