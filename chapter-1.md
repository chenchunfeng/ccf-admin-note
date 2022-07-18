## 编程规则 


### ESLint+GitHooks

1. 搭建项目， 保证本地vue-cli 在4.5.13以上

```
# 查看版本指令
vue -V
# 升级指令
npm updatae -g @vue/cli
# 创建项目 vue create admin-template 
# 使用vue3
# 不使用history mode for router
# 使用css 预处理器 sass/scss(with dart-sass) 
# 使用eslint  standard config
# 使用lint and fix on commit  保存跟commit都进行lint
# 使用单独配置 in dedicated config files

# 升级vue vuex vue-router 版本
npm i vue@3.2.8 vue-router@4.0.11 vuex@4.0.2
```

2. eslint 配置
```javascript
module.exports = {
  // 表示本文件的当前目录为本项目根目录
  root: true,
  // 表示启用 eslint 检测的环境
  env: {
    // 表示node环境中 使用eslint检测
    node: true,
  },
  // 表示eslint基础配置需要继承的的配置
  extends: ["plugin:vue/vue3-essential", "@vue/standard"],
  // 解析器
  parserOptions: {
    parser: "babel-eslint",
  },
  // 检测规则，错误分为三个等级
  /**
   * off   0
   * warn  1
   * error 2
   */
  rules: {
    "no-console": process.env.NODE_ENV === "production" ? "warn" : "off",
    "no-debugger": process.env.NODE_ENV === "production" ? "warn" : "off",
  },
};

```

3. prettier 配置

- vscode 安装prettier formatter插件
- 项目根目录添加.prettierrc 配置文件
- vscode 配置文件添加 format on save true
- eslint prettier冲突配置可在 eslint rules里面调整

### git 提交规范

使用commitizen  定制提交规则

- type
- scope
- subject
- body
- footer 

> cz-cli

安装commit

1. 全局安装commitizen
> npm install -g commitizen@4.2.4
2. 项目中安装cz-customizable
> npm install cz-customizable@6.3.0 --save-dev
3. 配置cz-customizable

```javascript

// package.json

"config": {
  "commitizen": {
    "path": "node_modules/cz-customizable"
  }
}
```
4. 根目录配置.cz-config.js
```javascript 
module.exports = {
  // 可选类型
  types: [
    { value: 'feat', name: 'feat:     新功能' },
    { value: 'fix', name: 'fix:      修复' },
    { value: 'docs', name: 'docs:     文档变更' },
    { value: 'style', name: 'style:    代码格式(不影响代码运行的变动)' },
    {
      value: 'refactor',
      name: 'refactor: 重构(既不是增加feature，也不是修复bug)'
    },
    { value: 'perf', name: 'perf:     性能优化' },
    { value: 'test', name: 'test:     增加测试' },
    { value: 'chore', name: 'chore:    构建过程或辅助工具的变动' },
    { value: 'revert', name: 'revert:   回退' },
    { value: 'build', name: 'build:    打包' }
  ],
  // 消息步骤
  messages: {
    type: '请选择提交类型:',
    customScope: '请输入修改范围(可选):',
    subject: '请简要描述提交(必填):',
    body: '请输入详细描述(可选):',
    footer: '请输入要关闭的issue(可选):',
    confirmCommit: '确认使用以上信息提交？(y/n/e/h)'
  },
  // 跳过问题
  skipQuestions: ['body', 'footer'],
  // subject文字长度默认是72
  subjectLimit: 72
}
```

### git hooks

由于使用git cz才会使用上面的定制提交规则，如果开发者忘记了，直接使用git commit -m 'xxx'， 这个时候就必须要检查comment

> pre-commit  无参数    
> commit-msg  接收参数
> 都可以使用 --no-verify 参数绕过

这里要使用两个工具
- commitlint    检验commit
- husky      git hooks 工具


```shell

npm install --save-dev @commitlint/config-conventional@12.1.4 @commitlint/cli@12.1.4

# 创建commitlint.config, 这里可能自己添加，也可以使用插件，同步cz-customizable定制的
# 默认的commitlint.config
```
```javascript
module.exports = {
  // 继承的规则
  extends: ['@commitlint/config-conventional'],
  // 定义规则类型
  rules: {
    // type 类型定义，表示 git 提交的 type 必须在以下类型范围内
    'type-enum': [
      2,
      'always',
      [
        'feat', // 新功能 feature
        'fix', // 修复 bug
        'docs', // 文档注释
        'style', // 代码格式(不影响代码运行的变动)
        'refactor', // 重构(既不增加新功能，也不是修复bug)
        'perf', // 性能优化
        'test', // 增加测试
        'chore', // 构建过程或辅助工具的变动
        'revert', // 回退
        'build' // 打包
      ]
    ],
    // subject 大小写不做校验 
    'subject-case': [0]
  }
}

// commitlint.config.js 必须是utf8 格式的，不是的话会报错
// 可以使用cz适配器，不用再自定义rules, 安装 npm install commitlint-config-cz -D

module.exports = { 
  // extends: ['@commitlint/config-conventional']
  extends: ['cz']
};
```

```shell



npm install husky@7.0.1 --save-dev
npx husky install
# 需要npm大于7.x
npm set-script prepare "husky install" 
npm run prepare

npx husky add .husky/commit-msg 'npx --no-install commitlint --edit "$1"'

```
通过[文档](https://docs.npmjs.com/cli/v7/commands/npm-set-script), v7版本以上的npm，才有set-script 指令。可通过 npm i -g npm升级。

npm6.x
- 没有npm set-script，需要手动添加"prepare": "husky install"
- npx husky add .husky/commit-msg 'npx --no-install commitlint --edit "$1"' 要分成两步
  - npx husky add .husky/commit-msg 创建文件
  - 手动添加npx --no-install commitlint --edit "$1"


在pre-commit 校验eslint
> npx husky add .husky/pre-commit 'npx eslint --ext .js,.vue src'

自动格式修复
> npx eslint --fix

只检查本次修改更新的代码，并在出现错误的时候，自动修复并推荐
> npm install -D lint-staged@9.5.0
```javascript
  "lint-staged": {
    "src/**/*.{js,vue}": [
      "eslint --fix",
      "git add"
    ]
  }
```
## 根据commit记录生成changelog

生成日志
```shell
# 也可以全局安装
npm install conventional-changelog-cli -D
"changelog": "npx conventional-changelog -p angular -i CHANGELOG.md -s -r 0"
```

- 基本使用
  >-p angular | atom 用来指定使用的 commit message 标准
  >-i CHANGELOG.md 表示从 CHANGELOG.md 读取 changelog
  >-s 表示读写 changelog 为同一文件
  >-r 表示生成 changelog 所需要使用的 release 版本数量，默认为1，全部则是0

- 自定义参数
  > todo

  自定义changelog格式
  > npm i conventional-changelog-custom-config -D
  修改指令
  ```javascript
    "changelog": "conventional-changelog -p custom-config -i CHANGELOG.md -s -r 0  -n ./changelog-option.js"
      //在这指定配置文件位置，本人放在了根目录，也可以指定其他地方
      /*配置项说明：
      -p custom-config 指定风格 
      -i CHANGELOG.md 指定输出的文件名称
      -s -r 0 指定增量更新，不会覆盖以前的更新日志
      -n ./changelog-option.js 指定自定义配置
      */
  ```
 > 根目录添加配置文件


## 版本发布标准自动化（standard-version）

standard-version 是一款遵循语义化版本（ semver）和 commit message 标准规范 的版本和 changelog 自动化工具
todo  大概流程就是

参考 https://www.csdn.net/tags/NtTaYgwsMjQwNTctYmxvZwO0O0OO0O0O.html


### 总结

- 使用eslint检验 使用prettier格式化代码， 使用vscode 的配置 format on save true
- 使用 commitizen 来定制提交格式 git commit -m 'xxxx' -> git cz
- 但也有可能有同学不知道，继续使用git commit提求，这个就需要commit lint
- 使用husky管理git hooks 使用 commit-msg 处理
- 有一些旧项目的代码不符合eslint校验规则，这个时候只需要校验提求commit的文件，这又需要安装lint-staged
- 注意版本