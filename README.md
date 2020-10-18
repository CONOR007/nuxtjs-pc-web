## 一. [购买服务器](https://blog.csdn.net/qq_42308316/article/details/109146395) && [下载nuxt js项目](https://github.com/CONOR007/realworld-nuxtJs.git)
## 二. 在服务器上安装 Node.js
- 使用 nvm 安装 Node.js
- [https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

```bash
# 查看环境变量
echo $PATH

wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
# 重新连接 ssh
nvm --version

# 查看环境变量
echo $PATH

# 安装 Node.js lts
nvm install --lts
```

- 安装 pm2进程管理工具,来管理我们的node进程
```bash
npm i pm2 -g
```

- pm2常用命令
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201018170937252.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzA4MzE2,size_16,color_FFFFFF,t_70#pic_center)

在这里插入图片描述
## 三.服务器开放端口 - 3000、80
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201018170930959.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzA4MzE2,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201018170920527.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzA4MzE2,size_16,color_FFFFFF,t_70#pic_center)

## 三. 手动部署 Nuxt.js 项目
1. 启动并打包Nuxt.js项目,然后手动压缩如下核心文件.压缩包名为`realworld.zip`.
- .nuxt
- static
- nuxt.config.js
- package.json
- package-lock.json
- pm2.config.json
2. 在服务器 root 下创建 realworld 文件夹
```bash
mkdir realworld
```
3. 本地运行如下命令,将压缩包传到服务器上
```bash
scp ./realworld.zip root@47.242.36.24:/home/realworld
```
3. 在服务器上运行
```bash
cd realworld

# 服务器上解压
unzip release.zip

# 查看隐藏目录
ls -a

# 安装依赖
npm i

# npm run start
pm2 start npm -- start

pm2 start pm2.config.json

pm2 stop xxxx
```
## 四. 自动部署 Nuxt.js 项目
#### 4.1 Github方面
1. 创建名为`realworld-nuxtJs`的远程仓库,然后把nuxt.js项目存储进来
2. 鼠标移至头像处进入settings`->`进入Developer settings`->`Personal access tokens`->`然后创建一个TOKEN并把token值记录下来,后续需要用到.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201018171842974.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzA4MzE2,size_16,color_FFFFFF,t_70#pic_center)
3. 然后进入到名为`realworld-nuxtJs`的远程仓库,点击上方的Settings然后点击Secrets进行各项指标的配置.![在这里插入图片描述](https://img-blog.csdnimg.cn/2020101817280129.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzA4MzE2,size_16,color_FFFFFF,t_70#pic_center)
**注意**:这里的KEY也可用PASSWORD服务器登录密码来代替.key的作用主要用来免密,提高安全性.具体配置可查看[购买阿里云服务器及免密登录](https://editor.csdn.net/md?not_checkout=1&articleId=109146967).
#### 4.2 项目方面
1. 由于这里的代码是交由github作为管理,所以需要在项目中写一个github运行的脚本,从而达到部署项目的目的.在项目的根目录新建.github/workflows/main.yml文件,并输入以下内容
```yml
name: Publish And Deploy Demo
on:
  push:
    tags:
      - 'v*'

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:

    # 下载源码
    - name: Checkout
      uses: actions/checkout@master

    # 打包构建
    - name: Build
      uses: actions/setup-node@master
    - run: npm install
    - run: npm run build
    - run: tar -zcvf release.tgz .nuxt static nuxt.config.js package.json package-lock.json pm2.config.json

    # 发布 Release
    - name: Create Release
      id: create_release
      uses: actions/create-release@master
      env:
        GITHUB_TOKEN: ${{ secrets.TOKEN }}
      with:
        tag_name: ${{ github.ref }}
        release_name: Release ${{ github.ref }}
        draft: false
        prerelease: false

    # 上传构建结果到 Release
    - name: Upload Release Asset
      id: upload-release-asset
      uses: actions/upload-release-asset@master
      env:
        GITHUB_TOKEN: ${{ secrets.TOKEN }}
      with:
        upload_url: ${{ steps.create_release.outputs.upload_url }}
        asset_path: ./release.tgz
        asset_name: release.tgz
        asset_content_type: application/x-tgz

    # 部署到服务器
    - name: Deploy
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.KEY }}
        port: ${{ secrets.PORT }}
        script: |
          cd /home/realworld
          wget https://github.com/CONOR007/realworld/releases/latest/download/release.tgz -O release.tgz
          tar zxvf release.tgz
          cnpm install --production
          pm2 reload pm2.config.json

```
#### 4.3 运行自动化构建
1. 接下来可以运行部署了,输入命令
```bash
git add .
git commit 'xxxx'
git tag v0.0.1
git push origin v0.0.1
```
2.然后来到github项目平台上就可以看到相关步骤的运行情况了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201018174224535.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzA4MzE2,size_16,color_FFFFFF,t_70#pic_center)
2. 发布成功后,在releases中可以查看代码的提交记录,方便代码的回滚
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020101817441781.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzA4MzE2,size_16,color_FFFFFF,t_70#pic_center)
