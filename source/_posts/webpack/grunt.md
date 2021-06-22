## Grunt
### 
1. 安装yarn add grunt
2. 添加gruntfile.js文件
  * grunt.registerTask 用来注册任务
  * grunt.registerTask('default',['foo','bar']) 默认任务，可以默认执行多个任务
  * 异步任务
```javascript
/* Grunt入口文件
*  用于定义一些需要 Grunt 自动执行的任务
*  需要导出一个函数
*  此函数接收一个grunt的形参，内部提供一些创建任务时可以用到的 API
*
* */
module.exports = grunt => {
  grunt.registerTask('foo', () => {
    console.log('heoo')
    return false  // 任务执行失败
  })
  grunt.registerTask('bar', '任务描述',() => {
    console.log('other')
  })
  // grunt默认任务
  /*grunt.registerTask('default',() => {
    console.log('default')
  })*/
  // 默认执行数组中的任务 ['foo','bar']，前面的任务执行失败，后面的任务就不会执行
  grunt.registerTask('default',['foo','bar'])

  // 异步任务
  grunt.registerTask('async-task',function () {
    const done = this.async()
    setTimeout(() => {
      console.log('async task')
      // done()  // 任务已经完成，done完，grunt才会结束
      done(false) // 标记异步任务失败
    },1000)
  })
}
```
3. 运行 yarn grunt bar
4. 查看grunt帮助信息 yarn grunt --help
5. 执行默认任务 yarn grunt  //yarn grunt default
6. 异步任务 
  * 需要调用 this.async() 标识任务执行完了
7. 标记任务失败： return false
  * 通过default执行多个任务时，前面的任务失败了，后面的就不会执行了
  * 强制全部执行  yarn grunt default --force
  * 异步任务标记失败 done(false)

### 配置
1. grunt.initConfig 配置
2. 获取配置 grunt.config('foo')
```javascript
module.exports = grunt => {
  // 配置
  grunt.initConfig({
    // foo:'bar'
    foo:{
      bar:123
    }
  })
  grunt.registerTask('foo',() => {
    // 获取配置
    // console.log(grunt.config('foo'))
    console.log(grunt.config('foo.bar'))
    const foo = grunt.config('foo')
    console.log(foo.bar)
  })
}
```

### grunt多目标任务
* 运行目标任务  yarn grunt build
* 运行某个目标任务 yarn grunt build:foo
```javascript
// 多目标任务
module.exports = grunt => {
  grunt.initConfig({
    build:{
      // options 是配置，，其他的都是目标任务
      options:{
        foo:'bar',
        ddd:'34'
      },
      foo:{
        // 目标任务里的配置同个属性会覆盖build的配置
        options: {
          foo:123,
          css:'abc'
        }
      },
      bar:'abc'
    }
  })
  // 多目标模式，可以让任务根据配置生成多个任务
  grunt.registerMultiTask('build',function () {
    console.log(this.options())  // 获取配置
    console.log(`target: ${this.target},data: ${this.data}`) // this.target 当前执行的目标任务，this.data 目标任务的值
  })
}
```

### grunt插件的使用
1. 安装插件  
   yarn add grunt-contrib-clean  // 自动清除项目在开发过程中产生的临时文件
2. 加载任务
  grunt.loadNpmTasks('grunt-contrib-clean') 方法加载
3. 为任务添加配置选项
```javascript
module.exports = grunt => {
  grunt.initConfig({
    clean:{
      // temp:'temp/app.js'
      // temp:'temp/*.txt'  // 所有的txt文件
      temp:'temp/**'  // 所有的子目录及子目录里的文件,连同temp文件夹删除
    }
  })
  grunt.loadNpmTasks('grunt-contrib-clean')
}
```
### grunt常用插件
1. grunt-sass  
  * 安装 yarn add grunt-sass sass --dev
2. grunt-babel  
  * yarn add grunt-babel  @babel/core @babel/preset-env --dev
3. load-grunt-tasks  
  * yarn add load-grunt-tasks --dev  自动加载所有的 grunt 插件中的任务
  * loadGruntTasks(grunt)
4. grunt-contrib-watch
  * yarn add grunt-contrib-watch --dev
  * 监听文件变化

```javascript
const sass = require('sass')
const loadGruntTasks = require('load-grunt-tasks')
module.exports = grunt => {
  grunt.initConfig({
    clean:{
      // temp:'temp/app.js'
      // temp:'temp/*.txt'  // 所有的txt文件
      temp:'temp/**'  // 所有的子目录及子目录里的文件,连同temp文件夹删除
    },
    sass:{
      options:{
        sourceMap:true,
        implementation:sass
      },
      main:{
        files:{
          'dist/css/main.css':'src/scss/main.scss' // 键：输出路径，值：源码路径
        }
      }
    },
    babel:{
      options:{
        sourceMap:true,
        presets:['@babel/preset-env']
      },
      main:{
        files:{
          'dist/js/app.js':'src/js/app.js'// 键：输出路径，值：源码路径
        }
      }
    },
    watch:{
      js:{
        files:['src/js/app.js'],
        tasks:['babel']
      },
      css:{
        files:['src/scss/*.scss'],
        tasks:['sass']
      }
    }
  })
  // grunt.loadNpmTasks('grunt-contrib-clean')
  // grunt.loadNpmTasks('grunt-sass')
  loadGruntTasks(grunt)  // 自动加载所有的 grunt 插件中的任务
  grunt.registerTask('default',['sass','babel','watch'])
}
```
