webpack特点总结:

- 同时支持commonJs和AMD规范
- 一切都可以打包(各种文件)
- 可以分模块打包(灵活的配置)

webpack打包后的文件:

- vendor.js:项目依赖的第三方库的js压缩而成
- app.js: 项目的js代码压缩而成
- 通过文件后缀(md5戳)来判断文件内容是否发生改变,提高编译和加载效率

npm项目搭建:

- npm init 生成package.json
- 安装webpck插件webpack-dev-server(本地开发环境的服务器) 和 webpack,可以npm i webpack webpack-dev-server --save-dev来同时安装这两个插件
- --save和--save-dev的区别:分别将依赖记录到dependencies和devDependencies下面,前者记录是项目在运行是必须依赖的插件,需要打包上线的,后者只在开发中使用的插件
- scripts: 脚本代码(命令行),npm start和npm test是npm run start ,npm run test的简写,npm start :'webpack-dev-server --progess --colors --hot --inline -d'(-d ,dug模式),npm build:'webpack -progess --colors -p'(-p,调用unflifyJs压缩)
- npm i 安装项目依赖

###webpack.config.js:
webpack的开发环境配置文件,符合commomJs规范的一个普通js,js文件的最后输出一个配置对象,在node环境中运行

- require引入第三方模块和node核心模块
- entry:项目的入口,可以是单个,可以是多个入口(给不同的入口设置不同的名字)
- output:开发环境下的打包输出的出口,(开发环境中,一般是单页面程序,默认写成filename:'bundle.js',如果是多个入口,可以用webpcak的变量name来写,'[name].bundle.js',webpack打包的时候会自动读取entry里面的多个入口然后命名);pbulicPath,把打包后的文件放在指定的目录下
- noPath:设置避免查找所有的requre和import
- relove: 查找文件时,自动补全文件的后缀名,安装数组的顺序依次不全
- module>loaders,针对不同文件类型,使用不同的loader(加载器),file loader(把css,html中图片文件打包) , url loader(把icon,图片等文件打包成base64格式) ,style loader(把打包后的css文件写入html,用style标签内联),bable loader(打包es6语法成es5)
- module>postcss:在css 的loaders配置中(style!css!postcss!less,loader加载顺序从右往左) ,会调用autoprefixer插件,针对一些c3属性或者有兼容性的样式,加上各个浏览器内核的hack前缀写法,如: -webkit -mozila 
- .babelrc ,编译es6,jsx语法时候,需要加上.balelrc配置文件
- plugins:插件,常见:html模版插件htmlWebpackPlugin,将打包后的js引入到hmtl上,需要指定html文件路径;webpack.HotModuleReplacementPlugin(),热加载插件;OpenBrowerPlugin,自动打开浏览器
- devServer:开发环境的配置
###webpack.producttion.config.js:
开发环境下,可以不用考虑系统的性能,更多考虑的是如何增加开发效率,而上线的时候,就需要考虑项目的性能,加载速度,缓存等. 

- entry: 项目的入口有多个,app.js,vender.js
- output: path,打包生成的目录;filename,'/js/[name].[chunkhash:8].js',意思是生成md5的文件后缀名(8位字符)
- loaders: css,要分离出来(区别于开发环境,js,css,等所有文件都打包放在一个bundle.js里面),图片和字体要生成md5的后缀名且放在指定的目录下
