### 不同环境不同配置

 1. 中小型项目

    **yarn webpack --mode production**

    ```
    // module.exports也可以导出一个函数，返回配置对象
    // env 环境参数，argv 运行cli过程中传递的所有参数
    module.exports = (env,argv) => {
    	const config = {
            mode: 'development',
            entry: './src/main.js',
            output: {
              filename: 'js/bundle.js'
            },
            devtool: 'cheap-eval-module-source-map',
            devServer: {
              hot: true,
              contentBase: 'public'
            },
            module: {
              rules: [
                {
                  test: /\.css$/,
                  use: [
                    'style-loader',
                    'css-loader'
                  ]
                },
                {
                  test: /\.(png|jpe?g|gif)$/,
                  use: {
                    loader: 'file-loader',
                    options: {
                      outputPath: 'img',
                      name: '[name].[ext]'
                    }
                  }
                }
              ]
            },
            plugins: [
              new HtmlWebpackPlugin({
                title: 'Webpack Tutorial',
                template: './src/index.html'
              }),
              new webpack.HotModuleReplacementPlugin()
            ]
          }
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

    

 2. 大型项目 不同环境对应不同配置文件

    **yarn webpack --config webpack.prod.js **    指定所使用的配置文件

    * webpack.common.js

      ```
      const HtmlWebpackPlugin = require('html-webpack-plugin')
      
      module.exports = {
        entry: './src/main.js',
        output: {
          filename: 'js/bundle.js'
        },
        module: {
          rules: [
            {
              test: /\.css$/,
              use: [
                'style-loader',
                'css-loader'
              ]
            },
            {
              test: /\.(png|jpe?g|gif)$/,
              use: {
                loader: 'file-loader',
                options: {
                  outputPath: 'img',
                  name: '[name].[ext]'
                }
              }
            }
        },
        plugins: [
          new HtmlWebpackPlugin({
            title: 'Webpack Tutorial',
            template: './src/index.html'
          })
        ]
      }
      
      ```
      
      
      
    * webpack.dev.js
    
      ```
      const webpack = require('webpack')
      const { merge } = require('webpack-merge')
      const common = require('./webpack.common')
      
      module.exports = merge(common, {
        mode: 'development',
        devtool: 'cheap-eval-module-source-map',
        devServer: {
          hot: true,
          contentBase: 'public'
        },
        plugins: [
          new webpack.HotModuleReplacementPlugin()
        ]
      })
      ```

      

    * webpack.prod.js

      ```
      const { merge } = require('webpack-merge')
      const { CleanWebpackPlugin } = require('clean-webpack-plugin')
      const CopyWebpackPlugin = require('copy-webpack-plugin')
      const common = require('./webpack.common')
      
      module.exports = merge(common, {
        mode: 'production',
        plugins: [
          new CleanWebpackPlugin(),
          new CopyWebpackPlugin(['public'])
        ]
      })
      
      ```
    
      

    