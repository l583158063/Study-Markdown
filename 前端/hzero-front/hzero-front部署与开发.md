HZERO-FRONT
---

### 一、环境配置

#### （1）Node（10.0.x 版本以上）

- 准备 Node
```shell
# 下载：
wget https://nodejs.org/dist/v10.15.0/node-v10.15.0-linux-x64.tar.xz
# 解压：
tar -xvf node-v10.15.0-linux-x64.tar.xz
# 移动到/usr/local/：
mv node-v10.15.0-linux-x64 /usr/local/node-v10.15.0
```

- 创建软链接：

```shell
ln -s /usr/local/node-v10.15.0/bin/node /usr/local/bin/node
ln -s /usr/local/node-v10.15.0/bin/npm /usr/local/bin/npm
ln -s /usr/local/node-v10.15.0/bin/npx /usr/local/bin/npx
验证：
node -v
npm -v
npx -v
```

- 解决 npm 连接国外镜像下载模块速度慢（使用淘宝服务器）：

```shell
# 临时使用
npm --registry https://registry.npm.taobao.org

# 永久使用
npm config set registry https://registry.npm.taobao.org

# 查看配置
npm config list
```



#### （2）yarn & lerna

```shell
npm install -g yarn
ln -s /usr/local/node-v10.15.0/bin/yarn /usr/local/bin/yarn

npm install -g lerna
ln -s /usr/local/node-v10.15.0/bin/lerna /usr/local/bin/lerna
```



#### （3）Windows10 配置 WSL + VS Code

链接：http://hzero-front-docs.hft.jajabjbj.top/zh/docs/quick-start/init-project/wsl/



### 二、HZERO-CLI 构建前端

#### （1）生成项目

```shell
# 创建名为 sample 的 hzero 前端项目
hzero-cli new sample

# 进入目录并查看当前工程信息
cd sample && hzero-cli info
```

按照提示选择对应版本的服务并生成项目。



#### （2）创建子模块并创建一个页面

1. 创建子模块

```shell
# 创建名为 test-module 的子模块
hzero-cli g sub-module test-module
```

2. 在子模块下创建一个页面

```shell
# 进入子模块目录 ***为自动补全的父模块
cd packages/***-test-module

# 创建一个简单页面 test-page (react)
hzero-cli g simple-page test-page
```

3. 启动调试

```shell
# 在子模块目录下启动
yarn start
```



#### （3）全量编译

```shell
# 进入父工程目录 sample
yarn run build
```

全量编译会删掉之前编译的**所有代码**。生成 dist 文件夹，所有子模块的代码会被编译到 `dist/packages` 下以子模块名字命名的文件夹中。



#### （4）增量编译（编译子模块）

```shell
# 进入父工程目录后，增量编译 test-module 子模块
yarn run build:ms sample-test-module
```

想要让父工程启动的时候能访问到子模块，需要先编译子模块。



### 三、项目配置

#### （1）.hzerorc.js 工程包含模块说明

该文件下包含一个数组，数组内存放了该工程的所有模块名字，供编译打包时读取。编译打包时首先在 **packages** 文件夹下寻找对应的模块，若找不到则会去 **node_modules** 文件夹下寻找，最终编译各个包至 **dist** 文件夹中。



#### （2）.env.yml 环境变量配置

文件路径为 /sample/src/config/.env.yml。首先需要修改 **API_HOST** 属性为**后端接口调用地址（网关）**。

```yaml
API_HOST: http://dev.hzero.org:8080
```

环境变量介绍：

```yaml
API_HOST: 网关地址
BASE_PATH: 浏览器地址栏的 router 前缀
PUBLIC_URL: index.html 中的资源加载路径地址的前缀
LOGIN_URL: 单点登录跳转的 url
LOGOUT_URL: 注销登录跳转的 url
PLATFORM_VERSION:
WEBSOCKET_HOST: 实时通讯消息服务的 ws 地址
BPM_HOST: 工作流服务地址
WFP_EDITOR: 工作流你编辑器地址
CLIENT_ID: oauth 访问 客户端id
```



### 四、启动项目

1. 按需修改     `../src/config/.env.yml` 中的环境变量
2. 安装依赖     `lerna bootstrap --registry http://nexus.saas.hand-china.com/content/groups/hzero-npm-group/`
3. 编译子模块   lerna run transpile
4. 打包dll      yarn build:dll
5. 启动本地环境  yarn start





### 五、部署项目

#### （1）步骤

1. 编译项目:
   1. 安装依赖      yarn bootstrap
   2. 编译子模块    lerna run transpile
   3. 打包dll      yarn build:dll
   4. 编译项目      yarn build
2. 将build完成的`dist`所有文件 复制到 `/usr/share/nginx/html(服务器部署目录)`
3. 替换环境变量
4. [执行脚本](http://hzerodoc.saas.hand-china.com/zh/docs/development-guide/front-develop-guid/start/#run-sh)



#### （2）run.sh

`/usr/share/nginx/html`: 服务器部署目录

```bash
#!/bin/bash
set -e

find /usr/share/nginx/html -name '*.js' | xargs sed -i "s BUILD_BASE_PATH $BUILD_BASE_PATH g"
find /usr/share/nginx/html -name '*.js' | xargs sed -i "s BUILD_API_HOST $BUILD_API_HOST g"
find /usr/share/nginx/html -name '*.js' | xargs sed -i "s BUILD_CLIENT_ID $BUILD_CLIENT_ID g"
find /usr/share/nginx/html -name '*.js' | xargs sed -i "s BUILD_WEBSOCKET_HOST $BUILD_WEBSOCKET_HOST g"
find /usr/share/nginx/html -name '*.js' | xargs sed -i "s BUILD_PLATFORM_VERSION $BUILD_PLATFORM_VERSION g"
find /usr/share/nginx/html -name '*.js' | xargs sed -i "s BUILD_IM_ENABLE $BUILD_IM_ENABLE g"
find /usr/share/nginx/html -name '*.js' | xargs sed -i "s BUILD_IM_WEBSOCKET_HOST $BUILD_IM_WEBSOCKET_HOST g"
find /usr/share/nginx/html -name '*.js' | xargs sed -i "s BUILD_CUSTOMIZE_ICON_NAME $BUILD_CUSTOMIZE_ICON_NAME g"


exec "$@"
```

> 注意：根据安装的版本按需设置变量。



#### （3）环境变量说明

- BUILD_BASE_PATH:

  ```text
    如果`/app/`二级目录部署 需要修改 `srm-front/config/compileBuildEnv.js` 中的 
      BASE_PATH 为 `/app/(二级目录)`
      PUBLIC_URL 为 `/app(二级目录不要后面的/)`
  ```

- BUILD_API_HOST(必填): 网关地址

- BUILD_CLIENT_ID(必填): oauth 认证客户端id

- BUILD_WEBSOCKET_HOST: websocket 地址

- BUILD_PLATFORM_VERSION(必填): OP/SAAS

- BUILD_IM_ENABLE：true/false，是否启用IM

- BUILD_IM_WEBSOCKET_HOST: IM websocket地址，需要和后端对应

- BUILD_CUSTOMIZE_ICON_NAME: “，客制化图标名称，详情请查看[icons 组件](http://hzerodoc.saas.hand-china.com/zh/docs/development-guide/front-develop-guid/component/icons/)



#### （4）jenkins & 开发服务器

jenkins 点击构建后 触发对应 服务器上的 `run.sh` 脚本

- 服务器需要安装运行环境(以及编译环境)
  - nginx, node
  - yarn, lerna
- 脚本目录在 `**/srm-fornt/**.sh`
- nginx root 指向 `**/srm-fornt/html`

```bash
#!/bin/sh

git pull

gitPullErrorCode=$?

if [ 0 -ne $gitPullErrorCode ]; then
  echo "git pull error, try back yarn.lock, and pull again";
  mv yarn.lock "yarn.lock.`date +"%Y-%m-%d_%H-%M-%S"`.bakk" # decide yarn.lock has conflict
  git pull;
fi;

if [ 0 -ne $? ]; then
  exit "git pull error";
fi;

export PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true

yarn --registry http://nexus.saas.hand-china.com/content/groups/hzero-npm-group/
#yarn run build:dll
yarn build:dll
lerna run transpile
yarn build

buildErrorCode=$?

if [ 0 -ne $buildErrorCode ]; then
  echo $buildErrorCode
  echo 'build error';
  exit $buildErrorCode;
fi
cp -r dist dist.bak
# 环境变量根据项目情况更改
find dist -name '*.js' | xargs sed -i "s BUILD_API_HOST http://hzeronb.saas.hand-china.com g"
find dist -name '*.js' | xargs sed -i "s BUILD_CLIENT_ID hzero-front-dev g"
find dist -name '*.js' | xargs sed -i "s BUILD_WEBSOCKET_HOST http://hzeronb.saas.hand-china.com/hpfm/sock-js g"
find dist -name '*.js' | xargs sed -i "s BUILD_PLATFORM_VERSION SAAS g"
find dist -name '*.js' | xargs sed -i "s BUILD_IM_ENABLE true g"
find dist -name '*.js' | xargs sed -i "s BUILD_IM_WEBSOCKET_HOST ws://192.168.16.150:9876 g"
find dist -name '*.js' | xargs sed -i "s BUILD_CUSTOMIZE_ICON_NAME customize-icon g"
rm -rf html
mv dist html
```



### 六、问题与解决

#### （1）本地可以访问前端而同一局域网内的机器无法访问

跳转到错误页面`/exception/500`，错误信息如下：

【火狐浏览器】已拦截跨源请求：同源策略禁止读取位于 http://dev.hzero.org:8080/iam/hzero/v1/users/self 的远程资源。（原因：CORS 请求未能成功）。

【谷歌浏览器】连接超时。

解决：需要将项目 build 生成的 `dist` 文件夹下的所有文件放入 `/usr/share/nginx/html` (服务器部署目录) ，并替换环境变量。