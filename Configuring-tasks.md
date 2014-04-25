# 配置任务

这个指南解释使用`Gruntfile`时如何为你的项目配置任务。如果你还不知道`Gruntfile`是什么，请先阅读[新手上路](docs/configuring-tasks.html)指南，然后参考[Gruntfile实例](http://gruntjs.com/sample-gruntfile/)。

## Grunt配置

通常任务配置都是通过`grunt.initConfig`方法指定在`Gruntfile`中。这些配置信息主要是一些命名任务属性，但是理论上可以包含任意数据。只要这些属性不与任务需要的属性冲突即可，否则会被忽略。

此外，由于`Gruntfile`本身就是JavaScript，因此你不仅限于使用JSON；在这里可以使用任意有效的JavaScript。在必要的情况下，也可以使用编程的方式生成配置信息。

    grunt.initConfig({
        concat: {
            // 这里是concat任务的配置信息
        },
        uglify: {
            // 这里是uglify任务的配置信息
        },
        // 非任务特定的属性
        my_property: 'whatever',
        my_src_file: ['foo/*.js', 'bar/*.js']
    });
    
## 任务配置和目标

当一个任务运行时，Grunt会查找同名属性之下的配置信息。多个任务可以有多个配置，使用任意命名的"target"定义不同的任务即可。下面有一个例子，`concat`任务有`foo`和`bar`两个目标(targets)，而`uglify`任务只有一个`bar`目标。

    grunt.initConfig({
        concat: {
            foo: {
                // 这里是concat任务'foo'目标的选项和文件
            },
            bar: {
                // 这里是concat任务'bar'目标的选择和文件
            }
        },
        uglify: {
            bar: {
                // 这里是uglify任务'bar'目标的选项和文件
            }
        }
    });
    
像`grunt concat:foo`或者`grunt concat:bar`这样同时指定任务和目标时Grunt只会处理指定目标的配置，而运行`grunt concat`命令时，它会遍历所有定义在任务中的目标，然后依次处理每个目标。注意，如果任务使用[grunt.task.renameTask](http://gruntjs.com/grunt.task#grunt.task.renametask)重命名过，Grunt会在配置对象中使用新的任务名查找属性。

## options

在任务配置内，`options`属性可以用来重写内置选项的默认值。此外，任务的每个目标也有一个用于该目标的`options`属性。而目标级的选项会自动覆盖任务级的同名选项配置。

`options`对象是可选，如果不需要，可以省略。

    grunt.initConfig({
        concat: {
            options: {
                // 这里是任务级的Options，可以用来覆盖任务的默认值 
            },
            foo: {
                options: {
                    // 这里是'foo'目标的options，它可以用来覆盖任务级的options.
                }
            },
            bar: {
                // 没有指定options，这个目标将使用任务级的options
            }
        }
    });
    
## 文件

因为大多数的任务都是执行文件操作，Grunt提供了一个强大的抽象说明来描述任务应该操作哪些文件。这里有好几种定义**src-dest**（源文件-目标文件）文件映射的方式，都提供了不同程度的描述和操作文件操作功能。所有以下格式任意多任务配置都能理解，因此选择满足需求的格式即可。

所有的文件格式都支持设置`src`和`dest`，此外"Compact"[简洁]和"Files Array"[文件数组]格式还支持一些额外的属性：

- `filter`  使用一个有效的[fs.Stats方法名](http://nodejs.org/docs/latest/api/fs.html#fs_class_fs_stats)或者一个函数来传递匹配的`src`文件路径，它返回`true`或`false`。
- `nonull` 如果设置这个选项为`true`，操作将会包含非匹配模式（译注：这句文档没理解）。结合Grunt的`--verbose`标志，这个选项对调试文件路径问题很有帮助。
- `dot`允许模式匹配以点好开头的文件名，即使模式不确定文件名开头是否有句点。
- `matchBase` 如果设置了设个选项，如果文件路径包含斜线，而不带斜线的匹配模式将会匹配紧邻基础名称的路径。比如，a?b能匹配路径`/xyz/123/acb`，而不能匹配`/xyz/acb/123`。
- `expand` 处理动态src-dest文件映射，更多的信息请查看["动态构建文件对象"](http://gruntjs.com/configuring-tasks#building-the-files-object-dynamically)。
- 其他的属性将作为匹配项传递给底层的库。在[node-glob](https://github.com/isaacs/node-glob)和[minimatch](https://github.com/isaacs/minimatch)文档中可以查看更多选项。

### 简洁格式

这种形式允许每个目标对应一个**src-dest**文件映射。通常情况下它用于只读任务，比如[grunt-contrib-jshint](https://github.com/gruntjs/grunt-contrib-jshint), 它就只需要一个单一的`src`属性，而不需要关联的`dest`选项. 这种格式还支持为每个`src-dest`文件映射指定附加属性。

    grunt.initConfig({
        jshint: {
            foo: {
                src: ['src/aa.js', 'src/aaa.js']
            }
        },
        concat: {
            bar: {
                src: ['src/bb.js', 'src/bbb.js'],
                dest: 'dest/b.js'
            }
        }
    });
    
### 文件对象格式

这种形式支持每个任务目标对应多个`src-dest`形式的文件映射，属性名就是目标文件，源文件就是它的值(源文件列表则使用数组格式声明)。可以使用这种方式指定多个`src-dest`文件映射， 但是不能够给每个映射指定附加的属性。

    grunt.initConfig({
        concat: {
            foo: {
                files: {
                    'dest/a.js': ['src/aa.js', 'src/aaa.js'],
                    'dest/a1.js': ['src/aa1.js', 'src/aaa1.js']
                }
            },
            bar: {
                files: {
                    'dest/b.js': ['src/bb.js', 'src/bbb.js'],
                    'dest/b1.js': ['src/bb1.js', 'src/bbb1.js']
                }
            }
        }
    });
    
### 文件数组格式

这种形式支持每个任务目标对应多个`src-dest`文件映射，同时也允许每个映射拥有附加属性：

    grunt.initConfig({
        concat: {
            foo: {
                files: [
                    {src: ['src/aa.js', 'src/aaa.js'], dest: 'dest/a.js'},
                    {src: ['src/aa1.js', 'src/aaa1.js'], dest: 'dest/a1.js'}
                ]
            },
            bar: {
                files: [
                    {src: ['src/bb.js', 'src/bbb.js'], dest: 'dest/b/', nonull: true},
                    {src: ['src/bb1.js', 'src/bbb1.js'], dest: 'dest/b1/', filter: 'isFile'}
                ]
            }
        }
    });
    
### 较老的格式

**dest-as-target**文件格式在多任务和目标形式出现之前是一个过渡形式，目标文件路径实际上就是目标名称。遗憾的是, 由于目标名称是文件路径，那么运行`grunt task:target`可能不合适。此外，你也不能指定一个目标级的`options`或者给每个`src-dest`文件映射指定附加属性。

    grunt.initConfig({
        concat: {
            'dest/a.js': ['src/aa.js', 'src/aaa.js'],
            'dest/b.js': ['src/bb.js', 'src/bbb.js']
        }
    });
    
### 自定义过滤函数

`filter`属性可以给你的目标文件提供一个更高级的详细帮助信息。只需要使用一个有效的[fs.Stats方法名](http://nodejs.org/docs/latest/api/fs.html#fs_class_fs_stats)。下面的配置只会清理一个与模式匹配的真实的文件：

    grunt.initConfig({
        clean: {
            foo: {
                src: ['temp/**/*'],
                filter: 'isFile'
            }
        }
    });
    
或者创建你自己的`filter`函数根据文件是否匹配来返回`true`或者`false`。下面的例子只会清理空目录：

    grunt.initConfig({
        clean: {
            foo: {
                src: ['temp/**/*'],
                filter: function(filepath){
                    return (grunt.file.isDir(filepath) && require('fs').readdirSync(filepath).length === 0);
                }
            }
        }
    });
    
### 通配符模式

> 原文档标题为Globbing patterns，大意是指使用一些通配符形式的匹配模式快速的匹配文件。

通常分别指定所有源文件路径的是不切实际的(也就是将源文件-目标文件一一对应的关系列出来)，因此Grunt支持通过内置的[node-glob](https://github.com/isaacs/node-glob)和[minimatch](https://github.com/isaacs/minimatch)库来匹配文件名(又叫作`globbing`)。

当然这并不是一个综合的匹配模式方面的教程，你只需要知道如何在文件路径匹配过程中使用它们即可：

+ `*`匹配任意数量的字符，但不匹配`/`

+ `?`匹配单个字符，但不匹配`/`

+ `**`匹配任意数量的字符，包括`/`，只要它是路径中唯一的一部分

+ `{}`允许使用一个逗号分割的列表或者表达式

+ `!`在模式的开头用于否定一个匹配模式(即排除与模式匹配的信息)

大多数的人都知道`foo/*.js`将匹配位于`foo/`目录下的所有的`.js`结尾的文件, 而`foo/**/*.js`将匹配`foo/`目录以及其子目录中所有以`.js`结尾的文件。

此外, 为了简化原本复杂的通配符模式，Grunt允许指定一个数组形式的文件路径或者一个通配符模式。模式处理的过程中，带有`!`前缀模式不包含结果集中与模式相配的文件。 而且其结果集也是唯一的。

示例：

    //可以指定单个文件
    {src: 'foo/this.js', dest: …}
    //或者指定一个文件数组
    {src: ['foo/this.js', 'foo/that.js', 'foo/the-other.js'], dest: …}
    //或者使用一个匹配模式
    {src: 'foo/th*.js', dest: …}
    
    //一个独立的node-glob模式
    {src: 'foo/{a,b}*.js', dest: …}
    //也可以这样编写
    {src: ['foo/a*.js', 'foo/b*.js'], dest: …}
    
    //foo目录中所有的.js文件，按字母排序
    {src: ['foo/*js'], dest: …}
    //这里首先是bar.js，接着是剩下的.js文件按字母排序
    {src: ['foo/bar.js', 'foo/*.js'], dest: …}
    
    //除bar.js之外的所有的.js文件，按字母排序
    {src: ['foo/*.js', '!foo/bar.js'], dest: …}
    //所有.js文件按字母排序, 但是bar.js在最后.
    {src: ['foo/*.js', '!foo/bar.js', 'foo/bar.js'], dest: …}
    
    //模板也可以用于文件路径或者匹配模式中
    {src: ['src/<%= basename %>.js'], dest: 'build/<%= basename %>.min.js'}
    //它们也可以引用在配置中定义的其他文件列表
    {src: ['foo/*.js', '<%= jshint.all.src %>'], dest: …}
    
可以在[node-glob](https://github.com/isaacs/node-glob)和[minimatch](https://github.com/isaacs/minimatch)文档中查看更多的关于通配符模式的语法。

### 构建动态文件对象

当你希望处理大量的单个文件时，这里有一些附加的属性可以用来动态的构建一个文件. 这些属性都可以指定在`Compact`和`Files Array`映射格式中(这两种格式都可以使用)。

+ `expand` 设置`true`用于启用下面的选项：

+ `cwd` 相对于当前路径所匹配的所有`src`路径(但不包括当前路径。)

+ `src` 相对于`cwd`路径的匹配模式。

+ `dest` 目标文件路径前缀。

+ `ext` 使用这个属性值替换生成的`dest`路径中所有实际存在文件的扩展名(比如我们通常将压缩后的文件命名为`.min.js`)。

+ `flatten` 从生成的`dest`路径中移除所有的路径部分。

+ `rename` 对每个匹配的`src`文件调用这个函数(在执行`ext`和`flatten`之后)。传递`dest`和匹配的`src`路径给它，这个函数应该返回一个新的`dest`值。 如果相同的`dest`返回不止一次，每个使用它的`src`来源都将被添加到一个数组中。

在下面的例子中，`minify`任务将在`static_mappings`和`dynamic_mappings`两个目标中查看相同的`src-dest`文件映射列表, 这是因为任务运行时Grunt会自动展开`dynamic_mappings`文件对象为4个单独的静态`src-dest`文件映射--假设这4个文件能够找到。

可以指定任意结合的静态`src-dest`和动态的`src-dest`文件映射。

    grunt.initConfig({
        minify: {
            static_mappings: {
                //由于这里的src-dest文件映射时手动指定的, 每一次新的文件添加或者删除文件时，Gruntfile都需要更新
                files: [
                    {src: 'lib/a.js', dest: 'build/a.min.js'},
                    {src: 'lib/b.js', dest: 'build/b.min.js'},
                    {src: 'lib/subdir/c.js', dest: 'build/subdir/c.min.js'},
                    {src: 'lib/subdir/d.js', dest: 'build/subdir/d.min.js'}
                ]
            },
            dynamic_mappings: {
                //当'minify'任务运行时Grunt将自动在"lib/"下搜索"**/*.js", 然后构建适当的src-dest文件映射，因此你不需要在文件添加或者移除时更新Gruntfile
                files: [
                    {
                        expand: true, //启用动态扩展
                        cwd: 'lib/', //批匹配相对lib目录的src来源
                        src: '**/*.js', //实际的匹配模式
                        dest: 'build/', //目标路径前缀
                        ext: '.min.js' //目标文件路径中文件的扩展名.
                    }
                ]
            }
        }
    });
    
### 模板

使用`<% %>`分隔符指定的模板会在任务从配置中读取相应的数据时将自动填充。模板会以递归的方式填充，直到配置中不再存在遗留模板相关的信息(与模板匹配的)。

整个配置对象决定了属性上下文(模板中的属性)。此外，在模板中使用`grunt`以及它的方法都是有效的，例如： `<%= grunt.template.today('yyyy-mm-dd') %>`。

+ `<%= prop.subprop %>` 将会自动展开配置信息中的`prop.subprop`的值，不管是什么类型。像这样的模板不仅可以用来引用字符串值，还可以引用数组或者其他对象类型的值。

+ `<% %>`执行任意内联的JavaScript代码，对于控制流或者循环来说是非常有用的。

下面提供了一个`concat`任务配置示例，运行`grunt concat:sample`时将通过banner中的`/* abcde */`连同`foo/*.js`+`bar/*.js`+`bar/*.js`匹配的所有文件来生成一个名为`build/abcde.js`的文件。

    grunt.initConfig({
        concat: {
            sample: {
                options: {
                    banner: '/* <%= baz %> */\n' // '/* abcde */\n'
                },
                src: ['<%= qux %>', 'baz/*.js'], // [['foo/*js', 'bar/*.js'], 'baz/*.js']
                dest: 'build/<%= baz %>.js'
            }
        },
        //用于任务配置模板的任意属性
        foo: 'c',
        bar: 'b<%= foo %>d', //'bcd'
        baz: 'a<%= bar %>e', //'abcde'
        qux: ['foo/*.js', 'bar/*.js']
    });
    
## 导入外部数据

在下面的Gruntfile中，项目的元数据是从`package.json`文件中导入到Grunt配置中的，并且[grunt-contrib-uglify插件](http://github.com/gruntjs/grunt-contrib-uglify)的`uglify`任务被配置用于压缩一个源文件以及使用该元数据动态的生成一个banner注释。

Grunt有`grunt.file.readJSON`和`grunt.file.readYAML`两个方法分别用于引入JSON和YAML数据。

    grunt.initConfig({
        pkg: grunt.file.readJSON('package.json'),
        uglify: {
            options: {
                banner: '/* <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
            },
            dist: {
                src: 'src/<%= pkg.name %>.js',
                dest: 'dist/<%= pkg.name %>.min.js'
            }
        }
    });
