## ESLint

### ESLint 介绍

	1. 最为主流的js lint工具，监测 js 代码质量
 	2. 很容易统一开发者的编码风格
 	3. 帮助开发者提升编码能力

### ESLint 安装

 1. 初始化项目：npm init --yes

 2. 安装 ESLint 模块为开发依赖：npm install eslint --save-dev

    --save：将保存配置信息到pacjage.json。默认为dependencies节点中。

    --dev：将保存配置信息devDependencies节点中。

 3. 通过 cli 命令验证安装结果

    npx eslint --version ----查看版本

### ESLint 快速上手

 1. 编写”问题“代码

 2. 使用 eslint 执行检测   ----    

     npx eslint .\01-prepare.js

    npx eslint --init    -------添加配置文件

     npx eslint .\01-prepare.js --fix      ------自动修复绝大多数代码风格的问题

 3. 完成 eslint 使用配置  

### ESLint 配置文件

 1. .eslintrc.js (npx eslint --init 生成 )

    ```
    // .eslintrc.js
    module.exports = {
    	// 标记当前代码运行环境
    	env:{
    		// 每个选项对应着一组预定义的全局变量
    		browser: true,
    		es2020: true,
    		es6: false
    	},
    	// 继承共享的配置
    	extends: [
    		'standard'
    	],
    	// 设置语法解析器的配置，控制是否允许使用某个 es 版本的语法
    	// 只代表语法检测
    	parserOptions:{
    		ecmaVersion: 2015
    	},
    	// 检测规则
    	rules:{
    		'no-alert':"off"  // off:关闭，error:报错，warn:警告
    	},
    	// 最新版本已经没有体现了,额外声明代码中可以使用的全局成员
    	globals: {
    		'jquery':"readonly"
    	}
    }
    ```

    eslint-env 配置

    ![eslint-env](./img/eslint-env.jpg)

### ESLint 配置注释

http://eslint.cn/docs/user-guide/configuring#configuring-rules

 1. 将配置通过注释的方式写到代码中

    * 忽略所有规则校验

      const str1 = "${name} is a coder"   // eslint-disable-line

    * 忽略指定规则校验

      const str1 = "${name} is a coder"  // eslint-disable-line no-template-curly-in-string

### ESLint 结合自动化工具

 1. ##### 结合 gulp

    在 babel 转换前先检查

    ```
    // gulpfile.js
    const script = () => {
      return src('src/assets/scripts/*.js', {
          base: 'src'
        })
        .pipe(plugins.eslint())
        .pipe(plugins.eslint.format())
        .pipe(plugins.eslint.failAfterError())
        .pipe(plugins.babel({
          presets: ['@babel/preset-env']
        }))
        .pipe(dest('temp'))
        .pipe(bs.reload({
          stream: true
        }))
    }
    ```

	2. ##### 结合 webpack

    ```
    // webpack.config.js
    {
        test: /\.js$/,
        exclude: /node_modules/,
        use: 'babel-loader'
    },
    {
        test: /\.js$/,
        exclude: /node_modules/,
        use: 'eslint-loader',
        enforce:'pre'   // 最开始执行
    }
    ```

    * 解决引入 react ，未引用报错，使用插件  eslint-plugin-react，

      ```
      // .eslintrc.js
      module.exports = {
        env: {
          browser: true,
          es2021: true
        },
        extends: [
          'standard'
        ],
        parserOptions: {
          ecmaVersion: 12
        },
        rules: {
        	'react/jsx-uses-react':2,  // eslint-plugin-react 插件中的规则
        	'react/jsx-uses-vars':2   // eslint-plugin-react 插件中的规则
        },
        plugins:[
          'react'    // 使用 eslint-plugin-react 插件，这样插件中所有的规则就可以用了
        ]
      }
      
      // 2.继承 eslint-plugin-react 插件 的所有规则配置，一般用这个
      module.exports = {
        env: {
          browser: true,
          es2021: true
        },
        extends: [
          'standard',
          'plugin:react/recommended'  // 继承 eslint-plugin-react 插件
        ],
        parserOptions: {
          ecmaVersion: 12
        },
        rules: {
        }
      }
      ```

	3. ##### vue 项目中继承 ESLint

	4. ##### ESLint 检查 TypeScript

    npx eslint --init  初始化时 选择 TypeScript

    指定语法解析器

    ```
    // .eslintrc.js
    module.exports = {
      env: {
        browser: true,
        es2021: true
      },
      extends: [
        'standard'
      ],
      parser:'@typescript-eslint/parser', // 语法解析器
      parserOptions: {
        ecmaVersion: 12
      },
      rules: {
      }
    }
    ```

    

## Stylelint

 	1. 对 css 代码 lint
 	2. 提供默认的代码检查规则
 	3. 提供 cli 工具，快速调用
 	4. 通过插件支持 Sass Less PostSCC
 	5. 支持 Gulp 或 Webpack 集成

### 安装

​	npm install stylelint -D

### 运行

 1. 添加配置文件 

    .stylelintrc.js

	2. 添加配置，npm i stylelint-config-standard

	3. 添加配置 sass 语法，npm install stylelint-config-sass-guidelines

    ```
    // .stylelintrc.js
    module.exports = {
    	extends:[
    		"stylelint-config-standard",  // 继承stylelint-config-standard，需要完成的名
    		"stylelint-config-sass-guidelines"
    	]
    	
    }
    ```

    

## Prettier

所有类型的格式化

 1. 安装

    npm insatll prettier -D

2. 运行

   npx prettier style.css                ---》 默认将格式化后的代码输出到控制台

   npx prettier style.css  --write   ---》 将格式化后的代码输出到文件，覆盖原文件

   npx prettier .   --write     ---》将所有代码格式化并覆盖源文件

### Git Hooks 介绍

 1. Git Hook 也称为 git 钩子，每个钩子都对应一个任务

 2. 通过 shell 脚本可以编写钩子任务触发时要具体执行的操作

    复制一份  pre-commit.sample

    删除后缀名 sample，就可以编写文件

    删除所有的内容，除了最上面的注释   #!/bin/sh

    ```
    // pre-commit
    #!/bin/sh
    echo "before commit"
    ```

    

    ![git hooks](./img/git-hooks.jpg)

### ESLint  结合 Git Hooks

 1. 编写 shell 脚本

 2. Husky 可以实现 Git Hooks 的使用需求，可以检查代码

    * 安装

      npm install husky -D

      ```
      // package.json 添加字段“husky"
      "scripts":{
      	"test":"eslint ./index.js"
      },
      "husky":{
      	"hooks":{
      		"pre-commit":"npm run test"
      	}
      }
      ```

	3.   lint-staged 代码格式化

    * 安装

      npm install lint-staged -D

      ```
      // package.json 添加字段“lint-staged"
      "scripts":{
      	"test":"eslint ./index.js",
      	"precommit":"lint-staged"
      },
      "husky":{
      	"hooks":{
      		"pre-commit":"npm run precommit"
      	}
      },
      "lint-staged":{
      	"*.js":[
      		"eslint",
      		"git add"
      	]
      }
      ```

      

#### 推荐使用 Husky  +   lint-staged