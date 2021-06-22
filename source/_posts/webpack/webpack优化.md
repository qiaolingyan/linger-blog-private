## 内置优化

### DefinePlugin

 1. webpack内置插件

 2. production环境下默认启用

 3. 为代码注入全局成员

 4. process.env.NODE_ENV

 5. 使用

    ```
    const webpack = require('webpack')
    plugins: [
    	new webpack.DefinePlugin({
    		API_BASE_URL:'"https://api.example.com"'，
    		API_BASE_URL:JSON.stringfy('https://api.example.com')，
    		// 要求一段代码片段，如果是值的话需要 JSON.stringfy()转换
    	})
    ]
    
    // main.js
    console.log(API_BASE_URL)   => 将“https://api.example.com”直接替换到代码中，没有引号
    ```



### Tree Shaking

 1. 移除未引用代码（console等，未引用的代码）

 2. production环境下默认启用

 3. 不是某个配置选项，是一组功能搭配使用后的优化效果

    ```
    // 集中配置 webpack 内置的优化功能
    optimization:{
    	usedExports:true,  // Tree-shaking 输出结果中只导出外部使用了的成员
    	minimize:true   // 开启压缩功能，移除掉没有引用的代码
    }
    ```

    

### 合并模块

1. 尽可能将所有模块合并输出到一个函数中

2. 既提升了运行效率，又减少了代码的体积

3. Scope Hositing 作用域提升

   ```
   // 集中配置 webpack 内置的优化功能
   optimization:{
   	usedExports:true,  // 输出结果中只导出外部使用了的成员
   	concatenateModules:true, // 合并模块
   }
   ```

   

### Tree-shaking 与 Babel

 1.  Tree-shaking 前提是 ES Modules 打包代码，即交给 webpack 打包的代码必须使用 ESM 模块化

 2. 使用 babel-loader 的插件 ‘@babel/preset-env' 会使 ES Module 转换为 CommonJS ,所以 Tree-shaking 失效

 3. 在最新版本的  babel-loader  已经关闭了将 ES Module 转换为 CommonJS ，所以 Tree-shaking 不会失效

 4. 为了确保，可以强制 ['@babel/preset-env',{ modules: false }]

    ```
    // 试验 开启转换为 CommonJS ，Tree-shaking 失效
    {
        test:/.js$/,
        use:{
            loader:'babel-loader',
            options:{
            // presets里边是数组套数组
            	presets:[
            		['@babel/preset-env',{ modules: 'commonjs' }]
            		// ['@babel/preset-env',{ modules: false }] 确保不会转换
            	]
            }
        }
    }
    ```

    

### sideEffects

 1. sideEffects 一般用于 npm 包标记是否有副作用

 2. 解决 index 文件中引入了所有的依赖，这样就会都打包，但是只想打包其中一个，可以开启sideEffects

 3. production环境下默认启用

 4. 会根据 package.json 文件中配置的 "sideEffects" 判断这个模块是否有副作用，如果没有副作用，那么没有引用到的模块就不会打包

    ```
    // 集中配置 webpack 内置的优化功能
    optimization:{ 
    	sideEffects:true  // 开启这个功能
    }
    
    // package.json
    {
    	...
    	"sideEffects":false  // 以此判断这个模块是否有副作用，false-没有副作用
    }
    ```

    

 5. 副作用代码

    * 一个模块里给给数组 对象啥的扩展方法
    * 导入 css 代码
    * 解决， package.json 文件中配置 “sideEffects” 

    ```
    // package.json
    {
    	...
    	"sideEffects":[
    		"./src/extend.js",
    		"*.css"
    	]
    }
    ```

    

## Webpack 优化

### 代码分割 Code Splitting

 1. webpack 所有代码都打包一起，导致打包结果体积过大

 2. 并不是每个模块在启动时都是必要的

 3. 所以需要分包，按需加载

 4. 分包方式：

    * 多入口打包：适用于多页面应用程序，不同页面不同打包结果，公共部分单独提取

      缺点：有公共的部分，这样会导致不同的打包结果中会有相同的模块

      解决：提取公共模块

    * 动态导入：按需加载

      需要用到某个模块时再去加载，节省带宽和流量

      动态导入的模块会被自动分包，公共的模块会自动提取到单独的包

    ```
    // 1.多入口打包 -多页面应用程序
    // entry 定义为数组的话，会把多个页面打包到一起，所入口需要是个对象
    // 不使用插件配置 chunks，会导致每个 HTML 把所有打包好的 js 文件都引入了
    // new HtmlWebpackPlugin 里 chunks 里指定需要引入的 bundle
    // 提取公共模块
    module.exports = {
    	entry:{
    		index:'./src/index.js',
    		album:'./src/album.js'
    	},
    	output:{
    		filename:'[name].bundle.js' // name 动态获取入口文件的名称
    	},
    	optimization:{
    		splitChunks:{
    			chunks:'all'  // 将公共模块都提取到单独的bundle中
    		}
    	},
    	plugins:[
    		new HtmlWebpackPlugin({
    			title:'Multi Entry',
    			template:'./src/index.html',
    			filename:'index.html',
    			chunks:['index']
    		}),
    		new HtmlWebpackPlugin({
    			title:'Multi Entry',
    			template:'./src/album.html',
    			filename:'album.html',
    			chunks:['album']
    		})
    	]
    }
    
    ```

    ```
    // 动态导入 - 按需加载
    // 不需要修改配置，只需要将文件导入方式改变动态导入
    // 比如动态导入实现路由懒加载
    // 之前是 import posts from './posts/posts'
    // 改为在用到的地方动态导入
    
    import('./posts/posts').then(({ default: posts }) => {
    	mainElement.appendChild(posts())
    })
    
    ```

    

### 魔法注释 

 1. 动态导入的文件产生的打包结果文件名称默认只是一个序号

 2. 使用魔法注释可以给这些 bundle 命名，动态导入时添加 /* webpackChunkName:'posts' */

 3. 如果给相同的 webpackChunkName 的话，模块会被打包到一起

    ```
    import(/* webpackChunkName:'posts' */'./posts/posts').then(({ default: posts }) => {
    	mainElement.appendChild(posts())
    })
    ```

    

### MiniCssExtractPlugin 提取CSS 到单个文件

 1. 实现 css 模块的按需加载

 2. style-loader 将样式通过 style 标签注入，使用 MiniCssExtractPlugin  的话就不需要使用 style-loader ，直接 link 引入，需要使用 MiniCssExtractPlugin.loader

 3. 如果样式文件 超过 150kb左右才会考虑 提取CSS 到单个文件，否则样式文件太小的话会适得其反

    ```
    // MiniCssExtractPlugin 
    // style-loader 将样式通过 style 标签注入，使用 MiniCssExtractPlugin  的话就不需要使用 style-loader ，需要使用 MiniCssExtractPlugin.loader
    
    const MiniCssExtractPlugin = require('mini-css-extract-plugin')
    
    module:{
    	{
            test: /\.css$/,
            use: [
                MiniCssExtractPlugin.loader,
                'css-loader'
            ]
        },
    },
    plugins:[
    	new MiniCssExtractPlugin()
    ]
    ```

    

### OptimizeCssAssetsWebpackPlugin

 1. 生产模式下只会自动压缩js代码，不会自动压缩其他文件代码

 2. optimize-css-assets-webpack-plugin 压缩 css

 3. 

    ```
    const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin')
    const TerserWebpackPlugin = require('terser-webpack-plugin')
    
    // 配置到plugins中，任何情况下都会压缩
    plugins:[
    	new OptimizeCssAssetsWebpackPlugin()
    ]
    
    // 配置到 optimization 的minimizer 数组中,通过 minimize 选项统一控制
    // 生产环境中 minimize 会自动开启
    // 设置了 minimizer 后，webpack 不会自动压缩 js 代码，因为webpack认为我们要自定义 压缩器，内部的js压缩器就会被覆盖掉,需要手动添加
    
    optimization:{ 
    	minimizer:[
    		new TerserWebpackPlugin()  // js 压缩器
    		new OptimizeCssAssetsWebpackPlugin()  // css 压缩器
    	]
    }
    ```

    

### 输出文件名 hash

 1. 解决 缓存 问题，生产模式下，文件名使用 Hash

 2. filename 都支持 hash  值

 3. hash

    项目级别的，项目有任何一个地方发生改变，hash 值都会变

	4. chunkhash

    chunk 级别的，同一路的打包，他的 chunkhash 就相同

	5. contenthash

    根据输出文件的内容生成 hash 值，不同的文件就会有不同的 hash 值

	6. 执行 hash 的长度

    contenthash:8  -- 指定长度为 8

    ```
    output:{
    		filename:'[name]-[contenthash:8].bundle.js' // name 动态获取入口文件的名称
    	},
    plugins:[
    	new MiniCssExtractPlugin({
    		filename:'[name]-[contenthash:8].bundle.css'
    	})
    ]
    ```

    

