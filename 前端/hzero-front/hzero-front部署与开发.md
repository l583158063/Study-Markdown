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

```bash
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



#### （3）路由配置

页面一般位于模块内，只需要关心模块内路由配置即可。

路由配置路径 `根目录/packages/模块名/src/config/routers.ts` ，配置示例：

```typescript
import { RoutersConfig } from 'hzero-boot/lib/typings/IRouterConfig';

const routerConfig: RoutersConfig = [
  // Insert New Router
  {
    path: '/demo1/hello',
    component: () => import('../routes/hello/HelloDemo1Page'),
    authorized: true,
    title: 'Hello JianQiaoFront Demo1',
  },
  {
    // 路由
    path: '/demo1/demo-page',
    // 组件位置
    component: () => import('../routes/hello/DemoPage'),
    // 可直接通过路由访问
    authorized: true,
    // 页面标签标题
    title: 'Sample Demo1',
  }
];

export default routerConfig ;
```



### 四、启动项目

1. 按需修改     `根目录/src/config/.env.yml` 中的环境变量
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



### 六、菜单和权限配置

- 创建菜单（子菜单）
- 添加页面路由
  - 对应 `根目录/packages/[modules]/src/config/routers.ts` 内的 `path` 属性。

- 维护菜单权限
- 角色管理
- 测试页面



### 七、DataSet 详解

#### （1）API

[DataSet API](https://choerodon.github.io/choerodon-ui/components-pro/data-set-cn/)

#### （2）重点理解

理解 DataSet、Field、Record 三个对象概念，明确需求以及使用场景下需要操作的对象及方法，就能很好的使用c7n-ui pro。

![img](http://hzero-front-docs.hft.jajabjbj.top/img/docs/_assets/2019-10-16-01-12-23.png)

- DataSet -- 数据源

  基于 mobx 状态管理 Store 封装成的一个数据集

  - field         字段属性数组
  - records   所有记录
  - data         数据（不包括删除状态的 Record） 

- Field -- 字段
- Record -- 记录（一般指当前操作的记录）



#### （3）DataSet 文件示例说明

![img](http://hzero-front-docs.hft.jajabjbj.top/img/docs/_assets/2019-10-16-01-12-26.png)

- primaryKey、autoQuery、pageSize、name 等 -- DataSet 的 props 属性

- fields -- 展示列的属性，可自定义如序号等仅供前端使用的字段
- queryFields -- Table 上方查询字段，自动生成查询组件



#### （4）DataSet.props.transport

可自定义的 Axios 请求配置。重点关注 `read` 和 `submit` 属性，编写这两个适配器即可完成对表单的 CRUD 。

| 属性       | 说明                                                         | 类型                                                         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| create     | 新建请求的 axios 配置或 url 字符串                           | AxiosRequestConfig \| ({ data, params, dataSet }) => AxiosRequestConfig \| string |
| **read**   | 查询请求的 axios 配置或 url 字符串                           | AxiosRequestConfig \| ({ data, params, dataSet }) => AxiosRequestConfig \| string |
| update     | 更新请求的 axios 配置或 url 字符串                           | AxiosRequestConfig \| ({ data, params, dataSet }) => AxiosRequestConfig \| string |
| destroy    | 删除请求的 axios 配置或 url 字符串                           | AxiosRequestConfig \| ({ data, params, dataSet }) => AxiosRequestConfig \| string |
| validate   | 唯一性校验请求的 axios 配置或 url 字符串。当字段配了 unique 属性时，在当前数据集中没有重复数据的情况下，则会发起远程唯一性校验。校验的请求 data 格式为 { unique: [{fieldName1: fieldValue1,fieldName2: fieldValue2...}] }，响应格式为 boolean \| boolean[]。 | AxiosRequestConfig \| ({ data, params, dataSet }) => AxiosRequestConfig \| string |
| **submit** | create, update, destroy 的默认配置或 url 字符串。            | AxiosRequestConfig \| ({ data, params, dataSet }) => AxiosRequestConfig \| string |
| tls        | 多语言数据请求的 axios 配置或 url 字符串。UI 接收的接口返回值格式为：[{ name: { zh_CN: '简体中文', en_US: '美式英语', ... }}]， 其中 name 是字段名。请使用全局配置 transport 的 tls 钩子统一处理。 | AxiosRequestConfig \| ({ data, params, dataSet, record, name }) => AxiosRequestConfig \| string |
| exports    | 导出的配置或 url 字符串                                      | AxiosRequestConfig \| ({ data, params, dataSet }) => AxiosRequestConfig \| string |
| adapter    | CRUD 配置适配器                                              | (config: AxiosRequestConfig, type: string) => AxiosRequestConfig |

##### AxiosRequestConfig

![AxiosRequestConfig](D:\Documents\Study-Markdown\前端\hzero-front\AxiosRequestConfig.png)

DataSet 默认请求为 `POST` 方法，可修改上述配置中的 `method` 属性切换。



### 八、使用 DataSet 与后端交互（查询）

#### （1）配置 DataSet 文件 -- transport.read

```javascript
// import ...

export default () => new DataSet({
  // 自动查询
  autoQuery: true,
  
  transport: {
    // 查询接口配置
    read: (config: AxiosRequestConfig): AxiosRequestConfig => {
      // 返回 axiosConfig 对象
      return {
        // config 对象中的所有参数
        ...config,
        // 请求接口地址
        url: `${
          commonConfig.TODO_API
          }/v1/${getCurrentOrganizationId()}/tasks`,
        // 请求方式
        method: 'GET',
      };
    },
    // {submit}
  },
  
  // 分页大小
  pageSize: 2,
  // 复选框
  selection: 'multiple' as DataSetSelection,
  // 数据表主键
  primaryKey: 'id',
      
  // 展示列属性定义（后端返回字段格式）
  /**
   * name: 列名定义
   * label: 前端展示文字
   * type: 数据类型
   * lovCode: 值集视图编码，配置以后可弹出值集视图选择框
   * lookupCode: 值集编码，配置以后自动展示 *下拉列表*
   * ignore: 是否作为参数传递
   * required: 是否必输
   * bind: 绑定 object 类型的 field 的某一个字段传输给后端
   * pattern: 校验正则表达式
   * defaultValidationMessages: 校验不通过默认提示
   */
  fields: [
    {
      name: 'employeeObject',
      label: '员工',
      type: 'object' as FieldType,
      lovCode: 'TODO.USER',
      ignore: 'always' as FieldIgnore,
      required: true,
    },
    {
      name: 'employeeId',
      label: '员工ID',
      type: 'number' as FieldType,
      bind: 'employeeObject.id',
    },
    {
      name: 'employeeName',
      label: '员工姓名',
      type: 'string' as FieldType,
      bind: 'employeeObject.employeeName',
    },
    {
      name: 'employeeNumber',
      label: '员工编号',
      type: 'string' as FieldType,
      bind: 'employeeObject.employeeNumber',
    },
    {
      name: 'state',
      label: '任务状态',
      type: 'string' as FieldType,
      lookupCode: 'TODO.TODO_STATE',
      required: true,
    },
    {
      name: 'taskNumber',
      label: '任务编号',
      type: 'string' as FieldType,
      required: true,
      pattern: /^[\dA-Z]*$/,
      defaultValidationMessages: {
        patternMismatch: '只能输入大写字母和数字, 例如: A123', // 正则不匹配的报错信息
      }
    },
  ],
  
  // 查询列属性定义（前端传给后端参数）
  queryFields: [
    {
      name: 'employeeObject',
      label: '员工',
      type: 'object' as FieldType,
      lovCode: 'TODO.USER',
      ignore: 'always' as FieldIgnore,
    },
    {
      name: 'employeeId',
      label: '员工ID',
      type: 'number' as FieldType,
      bind: 'employeeObject.id',
    },
    {
      name: 'employeeName',
      label: '员工姓名',
      type: 'string' as FieldType,
      bind: 'employeeObject.employeeName',
    },
    {
      name: 'state',
      label: '任务状态',
      type: 'string' as FieldType,
      lookupCode: 'TODO.TODO_STATE',
    },
  ],
});
```



#### （2）页面 Table 和 DataSet 结合

```javascript
// import ...
import TodoDS from 'xxx.js';

const HelloWorldPage: React.FC = () => {

  /* tableDS = new DataSet({
    ...TodoDS(),
  }) */
  /* const tableDS = useDataSet(todoTableDataFactory, HelloWorldPage); */
  const tableDS = TodoDS();
  const currentPage = useDataSetCurrentPage(tableDS);
  const isSelected = useDataSetIsSelected(tableDS);
  useDataSetEvent(tableDS, 'load', () => {
    // tslint:disable-next-line: no-console
    console.log('数据加载完成！');
  });

  // 定义 Table 需要展示的列
  const columns: ColumnProps[] = [
    { name: 'taskNumber', width: 320, editor: true },
    { name: 'taskDescription', editor: true },
    { name: 'state', editor: true },
    { name: 'employeeObject', editor: true },
  ];

  // 定义 Table 表头按钮
  const buttons = [
    TableButtonType.add,
    'delete' as Buttons,
  ];

  return (
    <>
      <Header title='Hello World'>
        {/* DataSet 主按钮：提交 */}
        <Button
          color={'primary' as ButtonColor}
          onClick={() => tableDS.submit()}
        >
          提交
        </Button>
      </Header>
      <Content >
        <Table
          queryFieldsLimit={3}
          dataSet={tableDS}
          queryFields={{
            state2: <SelectBox mode={ViewMode.button} />
          }}
          columns={columns}
          buttons={buttons}
          pagination={{
            showQuickJumper: true,
            // pageSize: 20,
            pageSizeOptions: ['20', '40'],
            // page: 4,
          }}
        />
        <pre>
          当前在第 {currentPage} 页
        </pre>
        <p>{isSelected ? '当前勾选了数据' : '当前没有勾选数据'}</p>
        <p>css modules 测试: <span className={styles['test-cls']}>{styles['test-cls']}</span></p>
      </Content>
    </>
  );
};

export default HelloWorldPage;
```



### 九、使用 DataSet 与后端交互（增删改）

#### （1）配置 DataSet transport 实现增删改

1. ##### 使用单独的 create + delete + update

   在 `transport` 属性中单独编写这三个属性对应的函数（声明接口 url 和 params 等），并通过 DataSet 对象调用自身的 create、delete、update，如 `tableDS.update()` ，DataSet 会经过一些处理后调用对应接口（多次调接口）。

2. ##### 使用数据由状态控制的 submit（常用）

   操作 DataSet 的数据不会马上调用接口，而是给数据的 `_status` 属性打上标记，点击提交后触发接口调用，后端批量处理。

```javascript
transport: {
  submit: ({ data, params }): AxiosRequestConfig => {
    return {
      url: `${
        commonConfig.TODO_API
        }/v1/${getCurrentOrganizationId()}/tasks/submit`,
      data,
      params,
      method: 'POST',
    };
  },
},
```

```javascript
// 调用时：
const res = await tableDS.submit();
// 返回值为 undefined 表示未作修改，此时不会调用接口
if (undefined === res) {
  notification.warning({
    message: '请先修改数据',
  });
  return;
} else if (res && res.failed && res.message) {
  notification.error({
    message: res.message,
  });
  throw new Error(res);
} else {
  // 提交后操作成功，自动查询
  await tableDS.query();
}
```



#### （2）页面交互逻辑

1. 使用 Table 内置按钮
2. 自定义 Table 操作按钮

```javascript
// 定义 Table 表头按钮
const buttons = [
  // 内置按钮 -- 新增
  TableButtonType.add,
  // 内置按钮 -- 删除
  'delete' as Buttons,
  // 自定义按钮
  <Button
    key="create-field"
    icon="playlist_add"
    color="primary"
    funcType="flat"
    onClick={() => { addLine() }}
  >
    自定义新增
  </Button>,
];

const addLine = () => {
  tableDS.create({}, 0);
}
```

3. 自定义 Table 行操作

```javascript
// 定义 Table 需要展示的列
const columns: ColumnProps[] = [
  { name: 'taskNumber', width: 320, editor: true },
  { name: 'taskDescription', editor: true },
  { name: 'state', editor: true },
  { name: 'employeeObject', editor: () => <Lov noCache /> },
  { /* 自定义的操作列 */
    header: '操作',
    width: 150,
    command: ({ record }) => {
      const state = record.get('state');
      return [
        TableButtonType.delete,
        <Button
          key="delete-value"
          icon="delete"
          color="red"
          funcType="flat"
          onClick={() => tableDS.remove(record)}
        />,
        <Tooltip title="撤回">
          <Button
            key="replay"
            icon="replay"
            title="abcd"
            disabled={state !== 'CODE_3'}
            funcType="flat"
            onClick={() => handleReStart([record])}
          />
        </Tooltip>
      ]
    }
  }
];
```



### 十、DataSet 实现校验控制

#### （1）保存之后的数据不可变更 taskNumber

```javascript
  fields: [
    {
      name: 'taskNumber',
      label: '任务编号',
      type: 'string',
      required: true,
      pattern: /^[\dA-Z]*$/,
      defaultValidationMessages: {
        patternMismatch: '只能输入大写字母和数字, 例如: A123', // 正则不匹配的报错信息
      },
      dynamicProps: {
        // 非新增行要设置为只读
        readOnly: ({ record }) => {
          //   return record.get('id');
          return record.status !== 'add';
        },
      },
    },
  ],
```



#### （2）完成状态的待办事项详情页不可编辑

通过对 `<Form>` 组件的属性 `pristine` 或 `disabled` 设置 true/false 完成，`pristine = true` 显示原始值，`disabled = true` 禁用表单。

```javascript
<Form pristine={this.state.readOnly} dataSet={this.detailDS} columns={3}>
  {/* 表单内容 */}
</Form>
```



#### （3）列表页根据不同状态显示不同颜色

```javascript
get columns() {
  return [
    { name: 'taskNumber', width: 320, editor: true },
    { name: 'taskDescription', editor: true },
    { name: 'state', editor: true },
    { name: 'employeeObject', editor: () => <Lov noCache /> },
    {
      header: '自定义列',
      width: 150,
      // 自定义渲染条件判断
      renderer: ({ record }) => {
        // console.log(record.get('taskDescription'));
        let color = 'gray';
        const state = record.get('state');
        if (state === 'CODE_1') {
          color = 'purple';
        } else if (state === 'CODE_2') {
          color = 'green';
        } else if (state === 'CODE_3') {
          color = 'dark';
        }
        return (
          <Button
            
            color={color}
        
            onClick={() => {
              this.gotoDetailPage(record, { otherField: 'test', rand: Math.random() });
            }}
          >
            ##{record.get('taskDescription')}##
          </Button>
        );
      },
      lock: 'right',
      align: 'center',
    },
  ];
}
```



#### （4）进行中状态的待办事项，可设置进度

```javascript
// 增加监听：页面发生修改时触发
componentWillUnmount() {
  this.detailDS.removeEventListener('update', this.handleChange);
}

// 当修改的 field = 'state' 且 state = '进行中' 时显示进度条
@Bind()
handleChange({ name, value }) {
  if (name === 'state') {
    if (value === 'CODE_2') {
      this.setState({
        percentVisible: true,
      });
    } else {
      this.setState({
        percentVisible: false,
      });
    }
  }
}
```



#### （5）假设数据不做分页，每个人最多只能存在一个进行中待办事项

通过 SQL 值集查询时用条件语句限制查询结果即可。



#### （6）如何使用表单自定义校验

1. **字段值属性校验**，编写 `DataSet.field.validator` 属性，当返回值为 false 或 涵盖错误信息的字符串，则为校验失败：

```javascript
fields: [
  {
    name: 'taskDescription',
    label: '任务描述',
    type: 'string',
    required: true,
    // (value, name, record) => boolean | string | undefined
    validator: value => {
      if (value && value.length <= 5) {
        return '长度必须超过5个字符';
      }
    },
  },
],
```

2. **字段正则表达式校验**，编写 `DataSet.field.pattern` 以及 `DataSet.field.defaultValidationMessages` ：

```javascript
fields: [
  {
    name: 'taskNumber',
    label: '任务编号',
    type: 'string',
    required: true,
    pattern: /^[\dA-Z]*$/,
    defaultValidationMessages: {
      patternMismatch: '只能输入大写字母和数字, 例如: A123', // 正则不匹配的报错信息
    },
    dynamicProps: {
      // 非新增行要设置为只读
      readOnly: ({ record }) => {
        //   return record.get('id');
        return record.status !== 'add';
      },
    },
  },
],
```

> 默认校验值 defaultValidationMessages 详见 [ValidationMessages](https://choerodon.github.io/choerodon-ui/components/configure-cn/#ValidationMessages)



### 十一、DataSet.children 关联级联行数据集

![](D:\Documents\Study-Markdown\前端\hzero-front\DataSet Props children.png)

- todoList -- 自定义的 key
- children 可以关联多个子 DataSet

```javascript
childrenDS1 = new DataSet({
  autoQuery: false,
  ...TodoDS(),
});

detailDS = new DataSet({
  autoQuery: false,
  ...UserDS(),
  children: {
    todoList: this.childrenDS1,
  },
});
```

- 可以为子 DataSet 设置查询参数：

```javascript
async componentDidMount() {
  this.detailDS.queryParameter = {
    employeeNumber: this.props.match.params.id,
  };
  this.childrenDS1.queryParameter = {
    employeeNumber: this.props.match.params.id,
  };
  await this.detailDS.query();
}
```

- 页面可以再新增一个 `<Table>` 展示子集



### 十二、页面间跳转和通信

#### （1）列表页和详情页间跳转

1. 自定义跳转按钮

```javascript
get columns() {
  return [
    {
      header: '操作',
      width: 150,
      align: 'center',
      renderer: ({ record }) => {
        return (
          <Button 
            onClick={() => this.handleGotoDetail(record)}
          >
            跳转详情 {record.get('taskNumber')}
          </Button>
        );
      }
    }
  ];
}
```

2. 实现跳转方法，调用 dispatch 函数

```javascript
@Bind()
handGotoDetail(record, otherData) {
  const { dispatch } = this.props;
  const pathname = record
    ? `/todo-module/todo-feature/detail/${record.get('taskNumber')}`
    : '/todo-module/todo-feature/create';
  dispatch(
    routerRedux.push({
      pathname,
      search:
        otherData &&
        querystring.stringify({
          otherData: encodeURIComponent(JSON.stringify(otherData)),
        }),
    })
  );
}
```

3. 更新路由

```javascript
const routerConfig: RoutersConfig = [
  // Insert New Router
  {
    path: '/todo-module/todo-feature',
    components: [
      {
        path: '/todo-module/todo-feature/list',
        component: 'todo/list/TodoListPage',
      },
      {
        path: '/todo-module/todo-feature/create',
        component: 'todo/detail/TodoDetailPage',
      },
      {
        // 带有 taskNumber 参数的详情页
        path: '/todo-module/todo-feature/detail/:taskNumber',
        component: 'todo/detail/TodoDetailPage',
      },
    ],
  },
];
export default routerConfig;
```

获取 `pathVariable` ：`this.props.match.params.taskNumber`

4. 更改菜单配置

在对应菜单中将原来设置的路由改为不带确定页面的 `/todo-module/todo-feature` ，进入菜单后会自动重定向至 `components` 数组的**第一个**组件。

5. 编写详情页

一般在 `async componentDidMount()` 函数中添加监听、处理创建/查询逻辑、获取上个页面传来的数据以及处理保存事件等。

```javascript
async componentDidMount() {
  // 前提 props 接口中声明 match: any
  this.detailDS.queryParameter.taskNumber = this.props.match.params.taskNumber;
  await this.detailDS.query();
}
```

`backPath` 属性可以指定返回的页面路由。

```javascript
<Header title="待办事项明细" backPath="/todo-module/todo-feature/list">
```



#### （2）两个页面跳转需要携带大量数据如何处理

1. 传递配置

比如想要通过 `/todo-module/todo-feature/detail/1?a=1&b=2` 路由传递数据。在上文 `handGotoDetail` 函数的 `dispatch` 属性中配置 `search` 属性，需要注意进行编码转换，保证所有需要的字符不被浏览器误认为关键字处理。

```javascript
@Bind()
gotoDetailPage(record, extData) {
  const { dispatch } = this.props;
  const pathname = record
    ? `/todo-module/todo-feature/detail/${record.get('taskNumber')}`
    : '/todo-module/todo-feature/create';
  dispatch(
    routerRedux.push({
      pathname,
      search:
        extData &&
        querystring.stringify({
          otherData: encodeURIComponent(JSON.stringify(extData)),
        }),
    })
  );
}
```



2. 接收与使用

传递过来的数据可以在 `this.props.location.search` 中读取并解析，并放入 `state` 中：

```javascript
// 初始化 state
state = {};

// ...

async componentDidMount() {
  const { search } = this.props.location;
  // 浏览器自带的解析参数对象
  const otherDataQueryStr = new URLSearchParams(search).get('otherData');
  if (otherDataQueryStr) {
    const otherData = JSON.parse(decodeURIComponent(otherDataQueryStr));
    // console.log('拿到上个页面传过来的参数：', otherData);
    this.setState({
      receiveData: otherData,
    });
  }
}
```



#### （3）如何获取 dva 中的数据

dva 保存了当前页面的 state 等数据，可以从这获取诸如当前登录用户 currentUser 信息等。取出后会注入当前页面的 `props` 中。

```javascript
// 入参为整个 state
@connect(({ user }) => {
  return {
    userName: user.currentUser.realName,
  }
})
export default class DetailPage extends PureComponent {
    console.log(this.props.userName);
};
```



### 问题与解决

#### （1）本地可以访问前端而同一局域网内的机器无法访问

跳转到错误页面 `/exception/500` ，错误信息如下：

【火狐浏览器】已拦截跨源请求：同源策略禁止读取位于 http://dev.hzero.org:8080/iam/hzero/v1/users/self 的远程资源。（原因：CORS 请求未能成功）。

【谷歌浏览器】连接超时。

解决：需要将项目 build 生成的 `dist` 文件夹下的所有文件放入 `/usr/share/nginx/html` (服务器部署目录) ，并替换环境变量。



#### （2）DataSet field.type = 'object' 无法自动填充值集视图

```javascript
fields: [
  {
    name: 'categoryObject',
    label: '商品类型',
    type: 'object',
    lovCode: 'JIANQIAO.PRODUCT_CATEGORY',
    ignore: 'always',
    required: true,
  },
  {
    name: 'categoryId',
    label: '商品类型ID',
    type: 'number',
    bind: 'categoryObject.categoryId',
  },
  {
    name: 'categoryCode',
    label: '商品类型编码',
    type: 'string',
    bind: 'categoryObject.categoryCode',
    ignore: 'always',
  },
  {
    name: 'categoryName',
    label: '商品类型名称',
    type: 'string',
    bind: 'categoryObject.categoryName',
    ignore: 'always',
  },
],
```

后端回传 productSpu 只有 `categoryId` ，缺少  `categoryCode` 和 `categoryName` 数据，因此需要添加非数据库字段并返回对应值：

```java
@Transient
private String categoryCode;
@Transient
private String categoryName;
```



#### （3）tsx 与 js 用法区别

1. ##### 属性匹配问题

由以下对比可见一斑。

```typescript
// ts
get columns(): ColumnProps[] {
  return [
    {
      header: '查看',
      renderer: ({ record }) => {
        return (
          <Button onClick={() => this.handleGotoDetail(record)}>
            详情
          </Button>
        );
      },
      lock: ColumnLock.right,
      align: ColumnAlign.center,
    },
  ];
}
```

```javascript
// js
get columns(): {
  return [
    { name: 'isActive', editor: true, align: 'center', lock: 'right', },
  ];
}
```

2. ##### React props

TS 参考 https://www.jianshu.com/p/9897c11f74b9 。

当我们想要使用如图所示 props 而直接在代码中编写如 `this.props.match` 等，将会被警告当前 props 中没有该属性，直接读取返回 `undefined` 。

![React Props](D:\Documents\Study-Markdown\前端\hzero-front\React Props.png)

TS + React 需要先编写 props 接口，在接口中声明将要使用的属性数据：

```typescript
import React, { Component } from 'react';
import { Dispatch } from 'redux';
import { connect } from 'dva';

// 声明 props 接口
interface ProductSpuDetailPageProps {
  dispatch: Dispatch<any>;
  // 想要使用 match，要先声明
  match: any;
}

// (alias) class Component<P = {}, S = {}, SS = any> 
// 其中 P --> props, S --> state, SS --> setState
@connect()
export default class ProductSpuDetailPage extends Component<ProductSpuDetailPageProps> {
  async componentDidMount() {
    this.match = this.props.match;
    console.log(this.match.params);
  }
}
```

**常用 props 属性：**

- match.params -- route 上的 pathVariable

- location.search -- 请求报文内携带的 data



#### （4）读取环境变量

默认读取最外层环境变量： `process.env.XXX` 。例如自定义 Axios 请求时访问域名可用：

```javascript
const url = `${process.env.API_HOSE}${commonConfig.XXX}/v1/xxx`;
```



#### （5）组装 headers.Authorization 并自定义 Axios 请求

```typescript
import { getAccessToken } from 'utils/utils';
import { DataSet, Axios, } from 'choerodon-ui/pro';

const accessToken = getAccessToken();
const headers = {
  Authorization: '',
};
if (accessToken) {
  headers.Authorization = `bearer ${accessToken}`;
}

const url = `${process.env.API_HOST}${commonConfig.HJQG_BACKEND}/v1/product-spus/on-shelf`;

const axiosConfig = {
  headers: headers,
  params: {
    isOnShelf: isOnShelf,
  },
};

Axios
  .post(url, productSpuIds, axiosConfig)
  .then(response => {
    console.log('response: ' + response);
    notification.success({
      message: '操作成功',
      description: '',
    });
  })
  .catch(error => {
    console.log('error: ' + error);
    notification.error({
      message: '操作失败',
      description: '',
    })
  });
```

`Axios.post(url, data, axiosConfig)` 其中 `data` 会默认自动转换成 Json 格式数据，直接使用 `dataSet.current` 会造成字段闭环，无法生成 Json ，**必须使用 `dataSet.current.toData()` 进行数据提取转换**。

















