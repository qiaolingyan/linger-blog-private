### Parcel

 1. 安装

    yarn add parcel-bundler --dev

	2. 入口文件

    src/index.html   (官方推荐，支持是任意类型的文件)

	3. 运行

    yarn parcel src/index.html

    根据命令找到 index.html 文件，再根据 script 标签找到 main.js 文件，再根据 import 语句找到对应的 foo 模块，从而完成整体项目的打包

	4. 不仅打包了应用，还开启了一个服务器（相当于 server . )，可以使用自动刷新

	5. 支持模块热替换

    ```
    // main.js
    
    if(module.hot){
    	// accept只接收一个参数（callback），当前模块更新或当前模块依赖的模块更新后会自动执行
    	module.hot.accept(() => {
    		console.log('hmr')
    	})
    }
    ```

	6. 自动安装依赖

    不用手动依赖，直接导入，文件保存后parcel会自动帮我们安装这个依赖

    ```
    import $ from 'jquery'
    ....
    ```

	7. 可以加载其他资源文件，直接导入使用，不用具体配置

	8. 支持动态导入，内部会自动拆分代码

    ```
    import('jquery').then(() => {
    ....
    })
    ```

	9. parcel 以生产模式运行打包

    yarn parcel **build** src/index.html

### parcel 与 webpack

 1. parcel 构建速度比webpack 快很多，因为parcel 内部实现了多进程同时去工作

    

