# gulpfile
gulp配置
/* = Gulp组件
 -------------------------------------------------------------- */
// 引入gulp
var gulp			= require('gulp');					// 基础库

// 引入我们的gulp组件
var sass 			= require('gulp-ruby-sass'),			// CSS预处理/Sass编译
  uglify 			= require('gulp-uglify'),				// JS文件压缩
  imagemin 		= require('gulp-imagemin'),		// imagemin 图片压缩
  pngquant 		= require('imagemin-pngquant'),	// imagemin 深度压缩
  livereload 		= require('gulp-livereload'),			// 网页自动刷新（服务器控制客户端同步刷新）
  webserver 		= require('gulp-webserver'),		// 本地服务器
  rename 		= require('gulp-rename'),			// 文件重命名
  sourcemaps 	= require('gulp-sourcemaps'),		// 来源地图
  changed 		= require('gulp-changed'),			// 只操作有过修改的文件
  concat 			= require("gulp-concat"), 			// 文件合并
  clean 			= require('gulp-clean');				// 文件清理

/* = 全局设置
 -------------------------------------------------------------- */
var srcPath = {
  html	: 'src',
  css		: 'src/scss',
  script	: 'src/js',
  image	: 'src/images'
};
var destPath = {
  html	: 'dist',
  css		: 'dist/css',
  script	: 'dist/js',
  image	: 'dist/images'
};

/* = 开发环境( Ddevelop Task )
 -------------------------------------------------------------- */
// HTML处理
gulp.task('html', function() {
  return gulp.src( srcPath.html+'/**/*.html' )
    .pipe(changed( destPath.html ))
    .pipe(gulp.dest( destPath.html ));
});
// 样式处理
gulp.task('sass', function () {
  return sass( srcPath.css, { style: 'compressed', sourcemap: true })
    .on('error', function (err) {
      console.error('Error!', err.message); // 显示错误信息
    })
    .pipe(sourcemaps.write('maps')) // 地图输出路径（存放位置）
    .pipe(gulp.dest( destPath.css )); // 输出路径
});
// JS文件压缩&重命名
gulp.task('script', function() {
  return gulp.src( [srcPath.script+'/*.js','!'+srcPath.script+'/*.min.js'] ) // 指明源文件路径、并进行文件匹配，排除 .min.js 后缀的文件
    .pipe(changed( destPath.script )) // 对应匹配的文件
    .pipe(sourcemaps.init()) // 执行sourcemaps
    .pipe(rename({ suffix: '.min' })) // 重命名
    .pipe(uglify({ preserveComments:'some' })) // 使用uglify进行压缩，并保留部分注释
    .pipe(sourcemaps.write('maps')) // 地图输出路径（存放位置）
    .pipe(gulp.dest( destPath.script )); // 输出路径
});
// imagemin 图片压缩
gulp.task('images', function(){
  return gulp.src( srcPath.image+'/**/*' ) // 指明源文件路径，如需匹配指定格式的文件，可以写成 .{png,jpg,gif,svg}
    .pipe(changed( destPath.image ))
    .pipe(imagemin({
      progressive: true, // 无损压缩JPG图片
      svgoPlugins: [{removeViewBox: false}], // 不要移除svg的viewbox属性
      use: [pngquant()] // 深度压缩PNG
    }))
    .pipe(gulp.dest( destPath.image )); // 输出路径
});
// 文件合并
gulp.task('concat', function () {
  return gulp.src( srcPath.script+'/*.min.js' )  // 要合并的文件
    .pipe(concat('libs.js')) // 合并成libs.js
    .pipe(rename({ suffix: '.min' })) // 重命名
    .pipe(gulp.dest( destPath.script )); // 输出路径
});
// 本地服务器
gulp.task('webserver', function() {
  gulp.src( destPath.html ) // 服务器目录（.代表根目录）
    .pipe(webserver({ // 运行gulp-webserver
      livereload: true, // 启用LiveReload
      open: true // 服务器启动时自动打开网页
    }));
});
// 监听任务
gulp.task('watch',function(){
  // 监听 html
  gulp.watch( srcPath.html+'/**/*.html' , ['html'])
  // 监听 scss
  gulp.watch( srcPath.css+'/*.scss' , ['sass']);
  // 监听 images
  gulp.watch( srcPath.image+'/**/*' , ['images']);
  // 监听 js
  gulp.watch( [srcPath.script+'/*.js','!'+srcPath.script+'/*.min.js'] , ['script']);
});
// 默认任务
gulp.task('default',['webserver','watch']);

/* = 发布环境( Release Task )
 -------------------------------------------------------------- */
// 清理文件
gulp.task('clean', function() {
  return gulp.src( [destPath.css+'/maps',destPath.script+'/maps'], {read: false} ) // 清理maps文件
    .pipe(clean());
});
// 样式处理
gulp.task('sassRelease', function () {
  return sass( srcPath.css, { style: 'compressed' }) // 指明源文件路径、并进行文件匹配（编译风格：压缩）
    .on('error', function (err) {
      console.error('Error!', err.message); // 显示错误信息
    })
    .pipe(gulp.dest( destPath.css )); // 输出路径
});
// 脚本压缩&重命名
gulp.task('scriptRelease', function() {
  return gulp.src( [srcPath.script+'/*.js','!'+srcPath.script+'/*.min.js'] ) // 指明源文件路径、并进行文件匹配，排除 .min.js 后缀的文件
    .pipe(rename({ suffix: '.min' })) // 重命名
    .pipe(uglify({ preserveComments:'some' })) // 使用uglify进行压缩，并保留部分注释
    .pipe(gulp.dest( destPath.script )); // 输出路径
});
// 打包发布
gulp.task('release', ['clean'], function(){ // 开始任务前会先执行[clean]任务
  return gulp.start('sassRelease','scriptRelease','images'); // 等[clean]任务执行完毕后再执行其他任务
});

/* = 帮助提示( Help )
 -------------------------------------------------------------- */
gulp.task('help',function () {
  console.log('----------------- 开发环境 -----------------');
  console.log('gulp default		开发环境（默认任务）');
  console.log('gulp html		HTML处理');
  console.log('gulp sass		样式处理');
  console.log('gulp script		JS文件压缩&重命名');
  console.log('gulp images		图片压缩');
  console.log('gulp concat		文件合并');
  console.log('---------------- 发布环境 -----------------');
  console.log('gulp release		打包发布');
  console.log('gulp clean		清理文件');
  console.log('gulp sassRelease		样式处理');
  console.log('gulp scriptRelease	脚本压缩&重命名');
  console.log('---------------------------------------------');
});


*************************package.json*****************************************
{
  "name": "react-tutorial",
  "version": "0.0.0",
  "private": true,
  "license": "see LICENSE file",
  "description": "Code from the React tutorial.",
  "main": "server.js",
  "dependencies": {
    "body-parser": "^1.4.3",
    "express": "^4.4.5"
  },
  "devDependencies": {
    "gulp": "^3.9.1",
    "gulp-autoprefixer": "^3.1.0",
    "gulp-changed": "^1.3.0",
    "gulp-clean": "^0.3.2",
    "gulp-concat": "^2.6.0",
    "gulp-imagemin": "^3.0.2",
    "gulp-livereload": "^3.8.1",
    "gulp-minify-css": "^1.2.4",
    "gulp-rename": "^1.2.2",
    "gulp-ruby-sass": "^2.0.6",
    "gulp-sourcemaps": "^1.6.0",
    "gulp-uglify": "^1.5.4",
    "gulp-webserver": "^0.9.1",
    "imagemin-pngquant": "^5.0.0"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node server.js"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/reactjs/react-tutorial.git"
  },
  "keywords": [
    "react",
    "tutorial",
    "comment",
    "example"
  ],
  "author": "petehunt",
  "bugs": {
    "url": "https://github.com/reactjs/react-tutorial/issues"
  },
  "homepage": "https://github.com/reactjs/react-tutorial",
  "engines": {
    "node": "0.12.x"
  }
}
