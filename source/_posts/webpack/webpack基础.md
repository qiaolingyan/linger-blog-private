### webpack

#### 一、快速上手

	1. yarn init
	2. yarn add webpack webpack-cli --dev
	3. yarn webpack   (打包)默认将 ‘src/index.js' 文件作为打包入口输出到 ’dist/main.js'

#### 二、配置文件

​	**webpack.config.js**

 1. entry 打包入口

    字符串：单入口  ， 相对路径的话  ./  不能省略

    数组：会把多个文件打包到一起

    对象：多入口打包，代码分包

 2. output  打包输出路径

    output.path  必须是绝对路径

    

 3. mode

    * production
    * development
    * none

 4. module

    * loader ---  webpack的核心特性

      借助于loader就可以加载任何类型的资源

 5. plugins

    * 解决除了资源加载的其他自动化工作

    * 清除dist目录

    * 拷贝静态文件至输出目录

    * 压缩输出代码
    * 实现前端工程化



```
const path = require('path')
module.exports = {
  entry:'./src/index.js', // 相对路径的话  ./  不能省略
  output:{
    filename:'bundle.js',
    path:path.join(__dirname,'output') // 必须是绝对路径
  }
}

// module.exports也可以是一个数组，每一项都是单独的打包配置
module.exports = [
	{
		entry:'./src/main.js',
		output:{
			filename:'a.js'
		}
	},
	{
		entry:'./src/main.js',
		output:{
			filename:'b.js'
		}
	}
]

// module.exports也可以导出一个函数，返回配置对象
// env 环境参数，argv 运行cli过程中传递的所有参数
module.exports = (env,argv) => {
	const config = {}
	if(env === 'production'){
		config.mode = 'production'
		config.devtool = false
		config.plugins = [
			...config.plugins,
			new CleanWebpackPlugin(),
			new CopyWebpackPlugin(['public'])
		]
	}
	return config
}
```

6. 资源加载器

   * 编译转换类：加载的资源模块转换为 js 模块
   * 文件操作类：加载的资源模块拷贝的输出目录，将访问路径向外导出
   * 代码检查类：代码校验，统一代码风格，提高代码质量

   * 样式：css-loader / style-loader    

   * 文件：file-loader

     问题：404资源找不到，j解决：配置 output 的 publicPath：’dist/', / 不能省略

     **文件加载器工作过程**：webpack在打包时遇到了图片，然后根据配置文件中的配置匹配到对应的文件加载器，文件加载器就开始工作，先将导入的文件拷贝到输出的目录，再将拷贝到输入目录的路径作为当前的模块的返回值返回，对于应用来说，所需要的资源就被发布出来了，可以通过模块的导出成员拿到这个成员的访问路径

   * URL加载器（图片）：url-loader

     配置选项 options 中的 limit 来限制大小，小文件使用 Data URLs, 减少请求次数

     大文件单独提取存放，提高加载速度

     ```
     {
         test:/\.png$/,
         use:{
             loader:'url-loader',
             options:{
             	limit:10 * 1024 // 10 kb
         	}
         }
     }
     // 这种使用方式必须安装 file-loader,超出大小限制的会使用file-loader
     ```

   * 代码转换：babel-loader  （@babel/core  @babel/preset-env)

     webpack只是打包工具

     加载器可以用来编译转换代码（将es6语法编译转换为es5）

     ```
     {
         test:/\.js$/,
         use:{
             loader:'babel-loader',
             options:{
             	presets:['@babel/preset-env']
             }
         }
     }
     ```

   * html：html-loader

     ```
     {
         test:/\.html$/,  // 必须有，用来匹配文件路径
         use:{
             loader:'html-loader',
             options:{
             	attrs:['img:src','a:href'] // 配置打包资源
             }
         }
     }
     ```

7. 常用插件

   * 清除 dist 输出目录：**clean-webpack-plugin**

   * 自动生成 Html  文件：**html-webpack-plugin**

     自动将打包的 文件 添加到 html 文件中，路径引用是动态的

     创建多个 HtmlWebpackPlugin 实例对象就可以生成多个 html 文件

   * 拷贝文件：**copy-webpack-plugin**

     开发阶段一般不会用，会在上线前使用

   * 提取公共代码：**commons-chunk-plugin**
   
   * 压缩 es6 代码：**uglifyjs-webpack-plugin**
   
   * 定义环境变量：**define-plugin**
   
     ```
   plugins:[
         new CleanWebpackPlugin(),
         // 用于生成 index.html
         new HtmlWebpackPlugin ({
           title:'Webpack Plugin Sample',
           meta:{
             viewport:"width=device-width"
           },
           template:'./src/index.html'
         }),
         // 用于生成 about.html
         new HtmlWebpackPlugin ({
           filename:'about.html'
         }),
      new CopyWebpackPlugin([
           // 'public/**'
           'public'
         ])
     ]
     ```
     
     



#### 三、打包结果运行原理

 1. 定义给一个对象 ____webpack_module_cache____ = {}  缓存加载后的模块

 2. 定义一个  ____webpack_require____ 函数，专门加载模块，给模块上定义了一些功能

 3. 定义了一个____webpack_exports____ = {} 

 4.  调用____webpack_require____（ ____webpack_require____.s = 0)，

    0 =》 模块数组下标，开始加载源代码中定义的入口模块

	5. 调用____webpack_require____.r 方法，给模块加了一个 ‘__esModule' 的标记

	6. 调用____webpack_require____（ ____webpack_require____.s = 0)，正式加载第一个模块

	7. 最后document.body.append(heading)

#### 四、webpack 加载资源的方式

 1. 遵循 ES Module 标准的 import 声明

 2. 遵循 CommonJS 标准的 require 函数

 3. 遵循 AMD 标准的 define 函数 和 require 函数

 4. 样式代码中的 @import 指令和 url 函数

 5. HTML 代码中图片标签的 src 属性

    HTML 代码中 a 标签的 href 属性不会默认处理，需要配置  options{attrs['img:src','a:href']}

    ```
    import footerHtml from './footer.html' // 接收到的是字符串
    document.write(footerHtml)
    ```

#### 五、webpack 核心工作原理

根据配置的入口文件，顺着入口文件的代码，根据import或者require的语句，解析其依赖的资源模块，分别解析每个资源模块对应的依赖，形成了所有文件的依赖树，webpack递归遍历依赖数树，找到每个节点对应的资源文件，根据配置的rules属性找到这个模块对应的加载器，加载器去加载这个模块，将加载到的结果放到打包结果中。

#### 六、Loader 工作原理

​	loader 负责资源文件从输入到输出的转换，类似一个管道，对于同一个资源可以依次使用多个 loader

#### 七、开发一个 markdown-loader

 1. 得到的结果是 markdown 转换过后的 html 字符串

 2. use:'./markdown-loader'    // use 可以使用模块的路径

 3. webpack最终的结果 必须是一段 JavaScript 代码，可以最后使用加载器转换 html-loader

    因为其将loader最后加载的结果直接拼接到 打包结果中，所以不是 js 代码的话会导致语法不通过

4. 安装 marked 模块 解析 markdown 为 html 字符串

5. 将 html 变为 js 代码 

   ```
   // markdown-loader.js
   const marked = require('marked')
   module.exports = source => {
   	const html = marked(source)
   	// return `module.exports = "${html}"` // 会导致html字符串里的一些换行符，引号出错
   	// return `module.exports = ${JSON.stringfy(html)}` // 第一种方法
   	// return `export default = ${JSON.stringfy(html)}` // 第二种方法
   	return html // 第三种方法，直接返回 html 字符串，安装 html-loader 解析
   }
   
   // webpack.config.js
   {
       test:/.md$/,  // 必须有，用来匹配文件路径
       use:[
       	'html-loader',
       	'./marked-loader'
       ]
   }
   ```

   

#### 八、plugins插件工作原理

​	Plugin 通过钩子机制实现，webpack给每个节点都预先定义了钩子。

​	1. plugin 是一个函数或者是一个包含 apply 方法的对象

#### 九、开发一个 plugins

 1. 通过在生命周期的钩子中挂载函数实现扩展

    ```
    // 实现清除bundle.js文件的注释
    class MyPlugin {
      apply (compiler) {
        console.log('MyPlugin启动')
        // emit钩子：输出 asset 到 output 目录之前执行。
        compiler.hooks.emit.tap('MyPlugin',compilation => {
          // compilation 可以理解为此次打包的上下文
          for(const name in compilation.assets){
            // console.log(compilation.assets[name].source())
            if(name.endsWith('.js')){
              const contents = compilation.assets[name].source()
              const withoutComments = contents.replace(/\/\*\*+\*\//g,'')
              compilation.assets[name] = {
                source:() => withoutComments,
                size:() => withoutComments.length
              }
            }
          }
        })
      }
    }
    ```

