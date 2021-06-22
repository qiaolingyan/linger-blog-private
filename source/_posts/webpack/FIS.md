### 基本使用
1. 特点：高度集成
  拥有内置的任务，开发只需要配置
  调试
2. 安装：yarn global add fis3 
3. 构建：
  * 会自动将资源的引入路径变为绝对路径
  * fis3 release
  * fis3 release -d output
  * fis-conf.js文件配置
```javascript
fis.match('*.{js,css,png}',{
  release:'/assets/$0'
})

fis.match('**/*.scss',{
  rExt:'.css',
  parser:fis.plugin('node-sass'),
  optmizer:fis.plugin('clean-css')
})

fis.match('**/*.js',{
  parser:fis.plugin('babel-6.x'),
  optmizer:fis.plugin('uglify-js')
})
```
4. 编译与压缩
  * yarn add fis-parser-node-sass --dev
  * yarn add fis-parser-babel-6.x --dev
5. fis3 inspect
