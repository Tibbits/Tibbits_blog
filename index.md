## 简介
> 这是一份文档博客搭建工具指南，同时也是一个docsify博客网站最完整的架子，建议直接Clone到本地自己修改定制。

## 如何使用

- clone本仓库到本地[github地址](https://github.com/pycrab/docsifyGuide)，打开docs文件夹
  - 修改index.html中的仓库为你的github仓库
  - 修改/docs/cover.md封面
  - 修改/docs/index.md主页
  - 修改/docs/sidebar.md侧边栏导航
  - 修改/docs/navbar.md顶部菜单（如果需要）
  
- 全局安装docsify脚手架

  `npm i docsify-cli -g`，首先保证已安装npm包管理工具

- 本地预览
  - docs文件夹下执行`docsify serve docs`
  - 或者执行全局命令`npm run doc`，该配置在package.json的scripts中