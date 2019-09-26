# ElementUILearn
本文主要研究了ElementUI的代码组织和构建,对于文档生成 测试 以及git和npm的管理策略并没有深入

## ElementUI的命令
### 基本命令
  - bootstrap 下载依赖包
  - build:file 整理icon 动态生成src/index.js 国际化 生成versions.json
  - build:theme 基础css引入支持 gulp打包sass 移动css文件至lib 
  - build:utils 移动src下文件至lib
  - build:umd 国际化
  - clean 清除lib
### 构建命令
  - dev 本地模拟elementui官网
  - dev:play 测试组件功能
  - dist 打包组件
  - i18n 国际化
  - lint 语法检测
  - pub 发布 部署
  - test 测试
### 基本命令介绍
#### build:file
  - 整理icon 使用postcss模拟构建packages/theme-chalk/src/icon.scss,得到生成的所有的class,匹配符合正则 /\.el-icon-([^:]+):before/，得到所有的类名，保存到examples/icon.json。
  - 动态生成src/index.js 根据components.json里面的组件目录动态生成src/index.js里面的内容
  - 国际化 略过
  - 生成versions.json 略过
#### build:theme
  - 基础css引入支持 根据components.json里面的组件,匹配到theme-chalk里面的sass文件,动态添加引入脚本
  - 打包sass Gulp脚本 略过
  - 移动css文件至lib 略过
#### build:utils
  - 这里使用babel来迁移，应该是希望在迁移的过程中进行babel构建
#### build:umd
  - 略过
#### clean
  - 清除文件
#### Makefile.new
  - 在新添加组件到packages里面的时候,执行此命令，可以更新组件引用到components.json src/index.js等
### 构建命令介绍
#### dev:play
##### 执行build:file命令
###### 执行build/webpack.demo.js
  - 入口文件./examples/play.js
  - 在入口文件引入的app.vue里面写入示例组件
  - 打开8085端口查看效果
#### dist
##### 执行clean命令
##### 执行build:file命令
##### 执行lint检测
##### 执行build/webpack.conf.js 入口文件./src/index.js 导出风格umd 生成文件 index.js
##### 执行build/webpack.common.js 入口文件./src/index.js 导出风格commonjs2 生成文件 element-ui.common.js
##### 执行build/webpack.component.js 入口文件是 components.json 生成文件 多个组件打包文件
##### 执行build:utils
##### 执行build:umd
##### 执行build:theme
#### pub
##### 执行bootstrap命令
##### 执行build/git-release.sh 检测git冲突
##### 执行 build/release.sh
  - 合并dev到master
  - 输入要发布的版本号:version
  - 推送到远程分支
  - 运行npm run dist编译前端资源
  - 使用npm version $version修改package.json里面的版本号
  - 如果版本号包含beta 发布到npm上,加上beta标签
  - 如果不包含beta标签 发布到npm上 npm publish
##### 执行build/bin/gen-indices.js 文档多语言支持
##### 执行build/deploy-faas.sh 发布站点

## ElementUI代码组织及管理
### Dev:play 本地预览
  作者编写了webpack.demo.js作为本地预览的webpack文件,同时添加examples/play.js作为入口文件,同时引入src/index.js作为所有组件的注册,我们只需要在play/index.vue中写入我们想要查看的组件 然后执行npm run dev:play即可(作者在webpack.demo.js里面配置了alias.main字段(在build/config.js中)使得我们引入的element-ui/src/支持文件可以转到当前项目中),因为css文件与组件文件完全隔离,所以我们看到作者在examples/play.js中 手动引入了css文件
### 开发代码组织
  开发人员将自己开发的组件放入packages文件,同时将组件里面的sass文件文件放入packages/theme-chalk文件,同时如果packages里面组件需要用到src目录里面方法,使用element-ui/src即可,因为作者同样在(build/config.js里面配置了别名 alias.element-ui)。
### 构建
#### 整体式构建
##### 风格：UMD commonjs2
##### 对应配置文件：UMD-webpack.conf.js commonjs2-webpack.common.js
##### 配置文件差异：
  - UMD添加压缩
  - commonjs2去掉压缩 同时添加对资源文件支持
##### 整体式构建是如何引入css文件的？
  - 如果查看官网就知道,在引入ElementUI的时候,我们还是需要手动引入css文件：import 'element-ui/lib/theme-chalk/index.css';所以其实是在gulp构建出css文件后,作者手动将css文件移至lib下。然后使用人员需要手动引入lib目录下css文件。
#### 按需引入式构建
##### 对应配置文件：webpack.component.js
##### 入口文件：components.json
##### 组件库中组件如果引入了src文件下的支持文件,打包后是如何引入的?
  - 我们看到build/config.js中把packages里面所有的针对src的引入全部忽略的,同时使用build:utils来把所有的utils文件移动到lib目录,所以作者在在.babelrc文件中,使用module-resolver来修改路径,使得打包后的组件依然可以引用src文件下支持文件。
### ElementUI代码组织
  packages 组件文件
  packages/theme-chalk 组件css文件
  src/组件支持文件
  src/index 整体构建入口文件
  components.json 按需打包入口文件

## 根据ElementUI的理解构建自己的组件库
### 组件库需要完成以下几件事情：
  - 开发人员如何将组件移动到组件库
  - 本地组件预览
  - 源代码如何组织
  - 如何构建
  - 发布流程
  - 文档生成
  - 组件测试
### 开发人员如何将组件移动到组件库
  1：将组件分为两部分,css文件和脚本文件
  2：脚本文件放入packages目录,css文件放入packages/theme-chalk目录
  3：执行build/new.js文件 更新components.json，packages/theme-chalk/src/index.sass, src/index.js的组件引用代码
### 本地组件预览
  模仿ElementUI模式即可
### 源代码如何组织
#### 构建脚本组织
  - build/pre 构建前脚本(一些文件的更新 创建等)
  - build/ready 构建脚本(webpack构建脚本)
  - build/after 构建后脚本(发布 部署等)
#### 源代码组织
  - build 构建脚本
  - examples 本地调试输出目录
  - packages 组件文件
  - packages/theme-chalk 组件css文件
  - src/ 组件基础支持文件
  - components.json 组件目录
#### 打包后目录(lib文件)组织
  - lib/theme-chalk css文件
  - lib/index.js 引入所有组件(umd风格)
  - lib/element-ui.common.js 引入所有组件(commonjs风格)
  - lib/其他组件文件 使用按需引入方式引入
  - lib/支持文件 ./src下文件
#### 如何构建
  - 参考ElementUI的构建过程即可
#### 发布流程
  - 参考ElementUI的构建过程即可
#### 文档生成
  - 暂不在讨论范围
#### 组件测试
  - 暂不在讨论范围