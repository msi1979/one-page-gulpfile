const {src, dest, series, parallel, watch, lastRun} = require('gulp'),
	plugins = require("gulp-load-plugins")(),
	del = require('del'),
	autoprefixer = require('autoprefixer'),
	pxtorem = require('postcss-pxtorem'),
	imagemin = require('gulp-imagemin'),
	browserSync = require('browser-sync').create(),
	reload = browserSync.reload,
	paths = {
		css: {
			src: 'src/skin/css/**',
			dev: 'dev/skin/css/'
		},
		img: {
			src: 'src/skin/images/**',
			dev: 'dev/skin/images/'
		},
		html: {
			src: 'src/*.html',
			dev: 'dev/'
		},
		copy: {
			src: 'src/skin/**',
			dev: 'dev/skin/'
		}
	};
let processors = [
	autoprefixer({
		cascade: false,
		grid: true
	}),
	pxtorem({
		rootValue: 20,
		unitPrecision: 5,
		propList: ['*'],
		selectorBlackList: [],
		replace: true,
		mediaQuery: false,
		minPixelValue: 0,
		exclude: /node_modules/i
	})
];

// 删除生成 build dev
function clean(cb) {
	del('dev/**')
	cb()
}

function delpic(cb) {
	del('dev/skin/images/**');
	cb()
}

function onError(error) {
	const title = error.plugin + ' ' + error.name
	const msg = error.message
	const errContent = msg.replace(/\n/g, '\\A ')
	plugins.notify.onError({
		title: title,
		message: errContent,
		sound: true
	})(error)
	
	this.emit('end')
}

// 复制所需素材
function copy(cb) {
	src([paths.copy.src, '!src/skin/css/**/*.less'], {allowEmpty: true}, {since: lastRun(exports.style)})
		.pipe(plugins.plumber(onError))
		.pipe(plugins.newer(paths.copy.dev))
		.pipe(dest(paths.copy.dev))
		.pipe(reload({stream: true}))
	cb()
}

function html(cb) {
	src(paths.html.src, {allowEmpty: true}, {since: lastRun(exports.html)})
		.pipe(plugins.plumber(onError))
		.pipe(plugins.newer(paths.html.dev))
		.pipe(dest(paths.html.dev))
		.pipe(reload({stream: true}))
	cb()
}

// 编译 style
function style(cb) {
	src(paths.css.src, {allowEmpty: true}, {since: lastRun(exports.style)}, {sourcemaps: true})
		.pipe(plugins.plumber(onError))
		.pipe(plugins.sourcemaps.init())
		.pipe(plugins.newer(paths.css.dev))
		.pipe(plugins.less())
		.pipe(plugins.postcss(processors))
		.pipe(plugins.sourcemaps.write('./maps/'))
		.pipe(dest(paths.css.dev), {sourcemaps: true})
		.pipe(plugins.filter('**/*.css'))
		.pipe(reload({stream: true}))
	cb()
}

//压缩图片
function img(cb) {
	src(paths.img.src, {allowEmpty: true}, {since: lastRun(exports.style)})
		.pipe(plugins.plumber(onError))
		.pipe(plugins.newer(paths.img.dev))
		.pipe(imagemin([
			imagemin.gifsicle({interlaced: true}),
			imagemin.mozjpeg({quality: 80}),
			imagemin.optipng({optimizationLevel: 5, errorRecovery: false, interlaced: 'null'}),
		]))
		.pipe(dest(paths.img.dev))
	cb()
}

//server
function dev(cb) {
	browserSync.init({
		server: {
			baseDir: paths.html.dev,
			index: 'login.html'
		},
		notify: false,
	});
	watch(paths.css.src, style);
	watch(paths.img.src, img);
	let watcher = watch([paths.copy.src, paths.html.src, '!src/skin/css/**/*.less'], copy);
	watcher.on('unlink', function (path) {
		plugins.newer(paths.copy.src)
		plugins.fileSync('src/', 'dev/');
	})
	cb()
}

function zip(cb) {
	let min = plugins.filter(['dev/**/*.*', '!dev/skin/css/maps/**'], {restore: true});
	src('dev/**', {allowEmpty: true}, {since: lastRun(zip)})
		.pipe(min)
		.pipe(plugins.zip('project-files.zip'))
		.pipe(dest(paths.html.dev))
	cb()
}

exports.clean = clean;
exports.style = style;
exports.html = html;
exports.dev = dev;
exports.zip = zip;
exports.copy = series(clean, copy);
exports.img = series(delpic, img);
exports.default = series(parallel(copy, style), series(html, dev));

