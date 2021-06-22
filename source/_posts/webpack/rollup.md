### Rollup

更为小巧，仅仅是一款 ESM 打包器

不支持 HMR 模块热替换等功能

 1. 安装 

    yarn add rollup --dev

2. 运行

   yarn rollup   --------不传递参数的情况下会打印出帮助信息

   yarn rollup ./src/index.js --format iife   ------format 指定输出格式（iife自调用函数）

   yarn rollup ./src/index.js --format iife --file dist/bundle.js  -----file 指定输出路径

3. rollup   默认会开启 tree-shaking 优化

4. 配置文件

   yarn rollup --config    ------运行读取配置文件

   yarn rollup --config rollup.config.js

   ```
   // rollup.config.js
   export default {
   	input:'src/index.js',   // 入口
   	output:{
   		file:'dist/bundle.js',  // 输出文件名
   		format:'iife'   // 输出格式
   	}
   }
   ```

5. 使用插件

   插件是 Rollup 的唯一扩展途径

   * rollup-plugin-json  -----在代码中通过 import 导入 json 文件
   * rollup-plugin-node-resolve -----加载 npm 模块
   * rollup-plugin-commonjs   ------- 加载 commonjs 模块

   ```
   import json from 'rollup-plugin-json'
   import resolve from 'rollup-plugin-node-resolve'
   import commonjs from 'rollup-plugin-commonjs'
   
   export default {
   	input:'src/index.js',   // 入口
   	output:{
   		file:'dist/bundle.js',  // 输出文件名
   		format:'iife'   // 输出格式
   	},
   	plugins:[
   		json(),
           resolve(),
           commonjs()
   	]
   }
   ```

6. 代码拆分

   动态导入-按需加载

   	* 必须使用 AMD 或者 commonjs 的标准
   	* 需要输出多个文件，需要修改配置文件

   ```
   import('./logger').then({ log }) => {
   	log('code')
   }
   
   // rollup.config.js
   export default {
   	input:'src/index.js',   // 入口
   	output:{
   		dir:'dist',  // 输出文件路径
   		format:'amd'   // 输出格式
   	},
   	plugins:[
   		json(),
           resolve(),
           commonjs()
   	]
   }
   ```

7. 多入口打包

   会将公共的地方提取出来作为单独的bundle

   将 input 设置为 数组[ ] ,或者对象 { }，此时输出格式不能为 iife,,可以设置成 amd

   amd 格式的文件不能直接引用到页面上，而是需要专门的库去加载

   ​	<script src="https://unpkg.com/requirejs@2.3.6/require.js" data-main="foo.js"></script>

   ```
   // rollup.config.js
   export default {
   	input:['src/index.js','logger.js'],   // 多入口
   	output:{
   		dir:'dist',  // 输出文件路径
   		format:'amd'   // 输出格式
   	},
   	plugins:[
   		json(),
           resolve(),
           commonjs()
   	]
   }
   ```

   

### rollup 与 webpack 选用原则

* rollup 优点：
  1. 输出结果更加扁平
  2. 自动移除未引用代码
  3. 打包结果依然完全可读
* rollup 缺点：
  1. 加载非 ESM 的第三方模块比较复杂
  2. 模块最终被打包到一个函数中，无法实现 HMR
  3. 浏览器环境中，代码拆分功能依赖 AMD

 * 如果我们正在开发应用程序 ：webpack

 * 如果我们正在开发一个框架或者类库 ： rollup

   例如：vue、react

* webpack 大而全，rollup 小而美



