# Gruntfile示例

下面我们通过一个用了5个Grunt插件的示例来讨论一下`Gruntfile`：

+ [grunt-contrib-uglify](https://github.com/gruntjs/grunt-contrib-uglify)
+ [grunt-contrib-qunit](https://github.com/gruntjs/grunt-contrib-qunit)
+ [grunt-contrib-concat](https://github.com/gruntjs/grunt-contrib-concat)
+ [grunt-contrib-jshint](https://github.com/gruntjs/grunt-contrib-jshint)
+ [grunt-contrib-watch](https://github.com/gruntjs/grunt-contrib-watch)

完整的`Gruntfile`文件在本页面的底部，但是如果你继续阅读，我们会一步步来讨论它。

第一部分是“包装”(wrapper)函数，用于封装你的Grunt配置：

	module.exports = function(grunt){
		//在这里配置你的grunt任务
	}
	
在这个函数内你可以初始化你的配置对象：

	grunt.initConfig({
	});

接下来你可以从`package.json`文件中读取项目设置，然后存储到初始化配置对象的`pkg`属性中。这允许我们引用`package.json`文件中的属性值，正如很快就会看到的。

	pkg: grunt.file.readJSON('package.json');
	
到目前为止我们可以看到如下配置：

	module.exports = function(grunt){
		grunt.initConfig({
			pkg: grunt.file.readJSON('package.json')
		});
	}
	
现在你可以为你要用到的每个任务定义配置信息。任务的配置对象可以作为初始化配置对象的属性存在，并且（建议）与任务同名。因此，“concat”任务就再配置对象的“concat”属性中配置。下面是我的“concat”任务的配置对象。

	concat: {
		options: {
			// 定义一个会插入到合并输出的每个文件之间的字符串
			separator: ';'
		},
		dist: {
			// 要合并的文件
			src: ['src/**/*.js'],
			// 返回的JS文件的位置
			dest: 'dist/<%= pkg.name %>.js'
		}
	}
	
注意我是如何引用JSON文件中的`name`属性的。在这里我们使用`pkg.name`访问我们之前定义在`pgk`属性中载入的`package.json`文件的结果，这会被解析为一个JavaScript对象。Grunt有自己的模板引擎输出配置对象中的属性值。这里我的意思是告诉concat任务合并`src/`目录中所有以`.js`结尾的文件。

接下来让我们配置uglify插件，这个任务用于压缩我们的JavaScript：

	uglify: {
		options: {
			// 生成一个banner注释并插入到输出文件的顶部
			banner: '/*! <%= pkg.name %> <%= grunt.template.today("dd-mm-yyyy") %> */\n'
		},
		dist: {
			files: {
				'dist/<%= pkg.name %>.min.js': ['<%= concat.dist.dest %>']
			}
		}
	}

这个配置告诉uglify插件，在`dist/`目录中创建一个包含压缩过的JavaScript文件结果的文件。而这里我使用了`<%= concat.dist.dest>`，因此uglify会压缩合并任务生成的文件。

QUnit插件的设置非常简单。只需指定测试运行文件的位置，也就是运行QUnit测试的HTML文件。

	qunit: {
		files: ['test/**/*.html']
	}
	
JSHint插件也很容易配置：

	jshint: {
		// 定义要检测（lint）的文件
		files: ['gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
		//配置JSHint (参考文档: http://www.jshint.com/docs)
		options: {
			// 可以在这里重写更多的 JSHint 的默认选项
			globals: {
				jQuery: true,
				console: true,
				module: true
			}
		}
	}
	
JSHint接受一个文件数组和一个选项配置对象。在[JSHint官方站点](http://www.jshint.com/docs/)中可以找到所有的文档。如果你乐于使用JSHint的默认选项，那么无需在Gruntfile中重新定义它们。

最后我们还有一个watch插件:

	watch: {
		files: ['<%= jshint.files %>'],
		tasks: ['jshint', 'qunit']
	}

可以在命令行中使用`grunt watch`命令运行这个任务。当它检测到任意指定的文件发成变化时（这里，我就直接用了JSHint任务中要检测的文件），这个任务就会人按照指定的顺序运行你所指定的任务。

最后，必须将我们所需要的Grunt插件加载进来。它们应该已经全部使用npm安装好了。

	grunt.loadNpmTasks('grunt-contrib-uglify');
	grunt.loadNpmTasks('grunt-contrib-jshint');
	grunt.loadNpmTasks('grunt-contrib-qunit');
	grunt.loadNpmTasks('grunt-contrib-watch');
	grunt.loadNpmTasks('grunt-contrib-concat');
	
最终还要建立一些任务。最重要的是默认任务：

	// 在命令行输入 grunt test 后要运行的任务
	grunt.registerTask('test', ['jshint', 'qunit']);
	
	// 默认任务可以通过在命令行输入 grunt 来运行
	grunt.registerTask('default', ['jshint', 'qunit', 'concat', 'uglify']);
	
下面是最终完成的`Gruntfile.js`。
	
	module.exports = function(grunt){
		grunt.initConfig({
			pkg: grunt.file.readJSON('package.json'),
			concat: {
				options: {
					separator: ';'
				},
				dist: {
					src: ['src/**/*.js'],
					dest: 'dist/<%= pkg.name %>.js'
				}
			},
			uglify: {
				options: {
					banner: '/*! <%= pkg.name %> <%= grunt.template.today("dd-mm-yyyy") %> */\n'
				},
				dist: {
					files: {
						'dist/<%= pkg.name %>.min.js': ['<%= concat.dist.dest %>']
					}
				}
			},
			qunit: {
				files: ['test/**/*.html']
			},
			jshint: {
				files: ['gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
				options: {
					globals: {
						jQuery: true,
						console: true,
						module: true,
						document: true
					}
				}
			},
			watch: {
				files: ['<%= jshint.files %>'],
				tasks: ['jshint', 'qunit']
			}
		});
		
		grunt.loadNpmTasks('grunt-contrib-uglify');
		grunt.loadNpmTasks('grunt-contrib-jshint');
		grunt.loadNpmTasks('grunt-contrib-qunit');
		grunt.loadNpmTasks('grunt-contrib-watch');
		grunt.loadNpmTasks('grunt-contrib-concat');
		
		grunt.registerTask('test', ['jshint', 'qunit']);
		grunt.registerTask('default', ['jshint', 'qunit', 'concat', 'uglify']);
	};