# mesh tutorial 部署

## 1. mesh tutorial 环境

mesh tutorial基于hexo博客框架进行部署，所以要部署mesh tutorial，需要先安装hexo工具，参考 https://hexo.io/zh-cn/docs/

## 2. mesh tutorial 本地部署

`git clone git@github.com:Tencent-Cloud-Mesh/mesh-tutorial.git`: clone tutorial源码到本地，源码放于github hexo分支上

`npm install`: 安装依赖包

`hexo s`: 访问 http://localhost:4000 查看效果


## 3. mesh tutorial 修改

修改文章： 访问`./source/_post` 文件夹，修改对应的md文件

修改侧边导航栏：访问`./source/menu/menu.md` 文件, 增加对应的路径

修改图片： 访问`./source/images`文件夹，并增加其中的文件


## 4. 网站样式修改

访问`./themes/book`文件夹，按照需求进行修改

## 5. 部署github pages

`hexo g`:  生成public文件夹

`hexo d`:  将网站部署到github/gitee上，网站相关代码（public）放于master分支上

### ！！！注意事项

部署到github/gitee上之前需要先配置ssh key，仓库地址在./_config.yml deploy选项中定义

## 6. 访问网站

github pages: https://tencent-cloud-mesh.github.io/MeshDemoTutorial.github.io/

gitee pages: https://tencent-cloud-mesh.github.io/meshdemotutorial.github.io/overview/abstract/

