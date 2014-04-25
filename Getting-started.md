# 新手上路

Grunt和Grunt插件都可以通过[Node.js](http://nodejs.org/)的包管理器[npm](https://npmjs.org/)来安装和管理。

*Grunt 0.4.x要求Node.js的版本`>=0.8.0`(也就是0.8.0及以上版本的Node.js才能很好的运行Grunt)。奇数版本的Node.js通常被认为是不稳定的开发版本。*

## 安装CLI

**如果你要从Grunt 0.3升级，请查看[Grunt 0.3升级说明](http://gruntjs.com/upgrading-from-0.3-to-0.4#grunt-0.3-notes)。**

为了便于开始，有需要全局安装Grunt的命令行接口(CLI)。你可能需要使用sudo权限（OSX，*nix，BSD系列的系统等等），或者你要以超级管理员的身份运行shell命令（Windows系统）才能做到这一点。

	npm install -g grunt-cli
	
这条命令将会把`grunt`命令植入到你的系统路径中，这样你就可以在你机器上的任意目录中运行`grunt`。

注意，安装`grunt-cli`并不等于安装了Grunt任务运行器！而Grunt CLI的工作很简单：运行已配置好的`Gruntfile`中指定版本的Grunt。它还允许在同一机器上同时安装多个版本的Grunt。

## CLI如何工作

每次运行`grunt`命令时，它会使用Node的`require()`系统查找本地已安装好的Grunt。正因为这样，你才可以从项目的任意子目录中运行`grunt`命令。

运行`grunt`命令时，如果找到本地已安装的Grunt，CLI就会加载本地安装的Grunt库，然后应用定义在你的`Gruntifile`中的配置信息，同时执行你要求它运行的所有任务。

*想要真正的了解这里发生了什么，[可以阅读源码](https://github。com/gruntjs/grunt-cli/blob/master/bin/grunt)。这份代码很短。*

## 使用一个现有的Grunt项目

假设你已经成功安装了Grunt CLI，并且也已经为你的项目配置好了`package.json`和`Gruntfile`，那么开始使用Grunt是非常容易的：

1. 首先定位到项目的跟目录（译注：通常`package.json`和`Gruntfile`建议存放在项目的根目录）。
2. 然后使用`npm install`命令安装项目依赖（译注：通常在`package.json`配置，推荐这么做）。
3. 最后使用`grunt`命令运行Grunt。

> 译注：可以在终端/cmd面板中使用命令行进入项目目录，对于Windows系统也可以进入项目(根)目录之后按住Shift，按下鼠标右键，在右键菜单中选择“在此处打开命令窗口(W)”。

这就是全部操作。安装好的Grunt任务可以使用`grunt --help`命令列出来，但是通常情况下最好先查看一下文档。

## 准备一个新的Grunt项目

典型的设置就是为你的项目添加两个文件：`package.json`和`Gruntfile`。

**package.json**：这个文件被用来使用npm存储作为npm模块发布的项目元数据。你的项目需要使用的Grunt和Grunt插件都应该列在这个文件的[开发依赖](https://npmjs.org/doc/json.html#devDependencies)中。

**Gruntfile**：这个文件通常被命名为`Gruntfile.js`或者`Gruntfile.coffee`，它用来配置或者定义任务，以及加载Grunt插件。

### package.json

`package.json`文件应该存放在项目的根目录中，与`Gruntfile`相邻，并且它应该与项目的源代码一起提交到服务器。在同一目录中运行`npm install`时，它会安装在`package.json`列出的每个依赖的正确版本。

这里有好几种方式可以为你的项目创建一个`package.json`文件：

- 大多数的[grunt-init](http://gruntjs.com/project-scaffolding)都会自动创建一个项目特定的`package.json`文件。
- 也可以使用[npm init](https://npmjs.org/doc/init.html)创建一个基础的`package.json`文件。
- 也可以以下面的示例为蓝本，然后按照[规范](https://npmjs.org/doc/json.html)按需扩展。

======

    {
        "name": "my-project-name", 
        "version": "0.1.0", 
        "devDependencies": { 
            "grunt": "~0.4.1",
            "grunt-contrib-jshint": "~0.6.0",
            "grunt-contrib-nodeunit": "~0.2.0",
            "grunt-contrib-uglify": "~0.2.2"
        }
    }

> 译注：更多关于`package.json`文件格式请自行参考规范。

#### 安装Grunt和grunt插件

为项目中已有的`package.json`文件添加Grunt和Grunt插件最简单的方式就是使用`npm install <module> --save-dev`命令。这不仅会在本地安装指定的`<module>`，它还会自动将这个模块作为依赖添加到`package.json`文件的devDependencies部分，添加进去的时候会使用一个[波浪形字符形式的版本范围标识](https://npmjs.org/doc/misc/semver.html#Ranges)。

> 例如：
> `"grunt-contrib-xxx": "~0.2.6"`

例如下面这条命令将会安装最新版本的Grunt到你的项目目录中，同时也会将它添加为你的devDependencies：

    npm install grunt --save-dev
    
同样的方式也可以用于Grunt插件和其他Node模块。当完成操作之后，确保将这个更新过的`package.json`文件与项目源代码一起提交。

> 译注：多人协作的时候这一点非常重要。

### Gruntfile

`Gruntfile.js`或者`Gruntfile.coffee`文件就是一个有效的JavaScript或者CoffeeScript文件，并且它应该存放在项目的根目录中，紧邻`package.json`文件，同时它也应该与项目的源码一起提交到服务器。

`Gruntfile`由以下几个部分组成：

- “包装”函数
- 项目和任务配置
- Grunt插件和任务加载
- 自定义任务

#### 一个Gruntfile示例

在下面这个`Gruntfile`中，我们从`package.json`文件中导入了项目的元数据到Grunt配置中，这个[grunt-contrib-uglify](http://github.com/gruntjs/grunt-contrib-uglify)插件的`uglify`任务配置用来压缩源代码文件并使用导入的元数据动态生成一个横幅注释。当我们在命令行中运行`grunt`时，`uglify`任务会默认运行。

	module.exports = function(grunt){

		// 项目配置
		grunt.initConfig({
			pkg: grunt.file.readJSON('package.json'),
			uglify: {
				options: {
					banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
				},
				build: {
					src: 'src/<%=pkg.name %>.js',
					dest: 'build/<%= pkg.name %>.min.js'
				}               
			}
		});

		// 加载提供"uglify"任务的插件
		grunt.loadNpmTasks('grunt-contrib-uglify');

		// 默认任务
		grunt.registerTask('default', ['uglify']);
	}

至此，你看到的就是一个完整的`Gruntfile`文件，接下来让我们一起看看它的组成部分：

#### "wrapper"（包装）函数

每个`Gruntfile`（以及Grunt插件）就是使用这个基本的格式，并且你所有的Grunt配置代码都必须指定在这个函数内部：

	module.exports = function(grunt) {
		// 在这里处理Grunt相关的事情
	}

#### 项目和任务配置

大多数任务都依赖于传递给[grunt.initConfig](http://gruntjs.com/grunt#grunt.initconfig)方法的对象中定义的配置数据。

在下面的例子中，`grunt.file.readJSON('package.json')`将存储在`package.json`中的JSON元数据导入到了Grunt配置中。因为`<% %>`模板字符串可以引入任意配置属性，像文件路径和文件列表这类配置数据都可以使用这种方式指定，已减少重复。

你还可以在配置对象内存储任意数据，只要它不与你的任务中需要的属性冲突，否则会被忽略。此外，由于这个文件本身就是JavaScript，因此你不仅限于可以使用JSON，实际上在这里可以使用任意有效的JS代码。必要的情况下甚至可以以编程的方式动态生成配置信息。

与大多数任务一样，[grunt-contrib-uglify](http://github.com/gruntjs/grunt-contrib-uglify)插件 的`uglify`任务期望将它的配置信息指定在一个同名属性之中。下面的自立中，我们指定了一个`banner`选项，在旁边还定义了一个名为`build`的单一uglify目标，用于压缩源代码文件为目标文件。

	// 项目配置
	grunt.initConfig({
		pkg: grunt.file.readJSON('package.json'),
		uglify: {
			options: {
				banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
			},
			build: {
				src: 'src/<%=pkg.name %>.js',
				dest: 'build/<%= pkg.name %>.min.js'
			}
		}
	});

#### 加载grunt插件和任务

许多常用任务，比如[concatenation](https://github.com/gruntjs/grunt-contrib-concat)，[minification](http://github.com/gruntjs/grunt-contrib-uglify)和[linting](https://github.com/gruntjs/grunt-contrib-jshint)都可以以常见的方式启用。只要一个常见被作为依赖定义在`package.json`文件中，并且已经通过`npm install`命令安装好，那么就可以在你的`Gruntfile`使用一条简单的命令来启用它：

	// 加载提供"uglify"任务的插件
	grunt.loadNpmTasks('grunt-contrib-uglify');

**注意**: `grunt --help`命令可以列出所有可用的任务。

#### 自定义任务

可以通过定义一个`default`任务的方式配置Grunt默认运行一个或者多个任务。在下面的例子中，在命令行中运行`grunt`而不指定任务时默认会运行`uglify`任务。这一功能与显示地运行`grunt uglify`甚至是`grunt defult`一样。其实在这个数组中可以指定任意数量的任务（带参数或者不带参数的都可以）。

	// 默认任务
	grunt.registerTask('default', ['uglify']);
	
如果你的项目需要使用的任务没有可用的Grunt插件，你可以在`Gruntfile`内定义自定义任务。例如，下面这个`Gruntfile`中就定义了一个完整的自定义的`default`任务，甚至都没有使用任务配置：

	module.exports = function(grunt) {
		// 一个非常基础的default任务
		grunt.registerTask('default', 'Log some stuff.', function() {
			grunt.log.write('Logging some stuff...').ok();
		});
	};
	
自定义项目特定的任务可以不定义在`Gruntfile`中；它们可以定义在外部的`.js`文件中，并且也可以使用[grunt.loadTasks](http://gruntjs.com/grunt#grunt.loadtasks)方法来加载。

## 扩展阅读

+ [安装Grunt](installing_grunt.html)指南中有关于安装特定版本的，发布版本的或者开发中版本的Grunt和grunt cli的更多详细信息。

+ [配置任务](configuring_tasks.html)指南中详细解释了如何在`Gruntfile`内配置任务，目标，选项和文件；此外还解释了模板，匹配模式和导入外部数据的相关信息。

+ [创建任务](creating_tasks.html)指南列出了Grunt任务类型之间的区别，还展示了许多任务实例和配置。

+ 关于如何编写自定义任务或者Grunt插件的更多信息请参考[开发者文档](http://gruntjs.com/grunt)。