## 序言

![avatar](d/图片/screen.jpg)

最近有挺多东西想记录下来，于是抽空改了一下我的 `win95风格` 静态博客网站，也就是现在用的这个，它放在 Github Pages里，用 `vue` 开发的，我叫它 `w95` ，它是开源的，你可以在我的仓库中找到它，但是源码太乱了我暂时不打算pin到首页。其实很早之前我用 `Gatsby.js` 做过一个正常网页风格的，部署在我自己的服务器上，后来续不起服务器费用加上没什么内容就关掉了。

## Github Actions

换到 `Github Pages` 后，我只需要用 markdown 写好内容，然后编译成静态网页，最后push上去就行了。但我想更方便一些，所以就用到了 `Github Actions` 。现在我只需要提交内容，剩下的工作就交给它，网站就会自动更新，省掉了每次push前编译一次静态文件的步骤。


关于 `Github Actions` 网上有很多介绍，其中包含很多专业名词，这些在我平时接触的工作中是完全用不上的，所以我也没办法解释它是什么。我只知道它可以像写代码一样通过编写一个文件（workflow）实现订阅一些事件（push、pull...），在事件触发时执行指定的操作（Action）。



## # 01

为了保持博客内容仓库干净，我决定把程序源代码和内容分成两个仓库，静态文件和内容文件分成两个分支，这样不会混在一团。

在内容仓库存放内容、静态文件以及 `workflow` 文件。

在 `w95` ，内容文件是放在源码目录中的 `src\disk` 下的，所以要把 `disk` 文件夹拿出来放到这个仓库中，我把它放到 `master` 分支。

随后要创建一个静态文件分支，用来存放编译后的 `vue` 网页文件，我放到了 `ghpage` 分支，最后到仓库设置中将 `Pages` `Source` 切换到 `ghpage`。

## # 02

创建 Actions workflow

在仓库中可以看到 Actions 选项，点进去能看到这个仓库所有的 workflow，以及执行记录等等。

点击 `New workflow` > `set up a workflow yourself` 开始创建一个新的workflow

main.yml
```yml
# 从w95仓库拉取源码然后构建静态网页

name: Build & Deploy from w95 repo

on:
  # 只有在 master 分支修改时执行
  push:
    branches: [ "master" ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

jobs:
  # 拉取两个仓库文件然后构建
  build:
    # The type of runner that the job will run on
    runs-on: windows-2019

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # 获取静态数据
      - name: checkout data 💽
        uses: actions/checkout@v3
        with:
          path: data

      # 获取源码
      - name: clone w95 📀
        uses: actions/checkout@v3
        with:
          repository: noberumotto/w95
          path: w95

      # 合并
      - name: copy files 💾
        run: |
          Xcopy /Y /E /I data w95\source\src
          cd w95\source\src
          dir
      # 构建静态文件
      - name: build 🛠
        working-directory: w95\source
        run: |
          npm install
          npm run w95file
          npm run build
      # 上传
      - name: upload 📤
        uses: actions/upload-artifact@v1
        with:
          name: site
          path: w95\source\dist
  # 发布静态文件
  deploy:
    concurrency: ci-${{ github.ref }}
    needs: [build]
    runs-on: ubuntu-latest
    steps:
      - name: checkout data
        uses: actions/checkout@v3
      # 下载
      - name: download 📥
        uses: actions/download-artifact@v1
        with:
          name: site
      # 发布
      - name: deploy 🎉
        uses: JamesIves/github-pages-deploy-action@v4.3.3
        with:
          # 发布到 ghpage 分支上
          branch: ghpage
          folder: 'site'
```

保存并提交后会立即触发事件执行这个 workflow 文件




## # 03

就这样，所有的步骤完成了。

但还有个问题待解决，为了更像`windows95`，每个`文件`都有创建时间和修改时间两个属性，现在的做法是在构建时获取文件的真实属性，当我在本地手动构建时工作正常，但是放到 `Github Actions`中就不行了，每次构建的时候所有文件属性的时间都会更新成构建时间。因为每一次`Github Actions`操作都要从仓库中签出、复制这些文件，这无法避免的导致时间被覆盖。