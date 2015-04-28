# Gruntfile 实例

下面就针对一个 `Gruntfile` 案例做简单分析，也可以作为一个实例使用：

    module.exports = function(grunt) {

      grunt.initConfig({
        jshint: {
          files: ['Gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
          options: {
            globals: {
              jQuery: true
            }
          }
        },
        watch: {
          files: ['<%= jshint.files %>'],
          tasks: ['jshint']
        }
      });

      grunt.loadNpmTasks('grunt-contrib-jshint');
      grunt.loadNpmTasks('grunt-contrib-watch');

      grunt.registerTask('default', ['jshint']);

    };

在页面底部是这个 `Gruntfile` 实例的完整内容，如果你按顺序阅读本文的话，可以跟随我们一步步分析这个文件中的每一部分。我们会用到以下5个 Grunt 插件：

第一部分是"wrapper" 函数，它包含了整个Grunt配置信息。

    module.exports = function(grunt) {
    }

在这个函数中，我们可以初始化 configuration 对象：

    grunt.initConfig({
    });

接下来可以从`package.json` 文件读入项目配置信息，并存入`pkg` 属性内。这样就可以让我们访问到`package.json`文件中列出的属性了，如下：

    pkg: grunt.file.readJSON('package.json')

到目前为止我们就可以看到如下配置：

    module.exports = function(grunt) {
      grunt.initConfig({
        pkg: grunt.file.readJSON('package.json')
      });
    };

现在我们就可以为我们的每个任务来定义相应的配置(逐个定义我们为项目定义的任务配置)，然后每个任务的配置对象作为Grunt配置对象(即grunt.initConfig({})所接受的配置对象)的属性，并且这个属性名称与任务名相同。因此"concat"任务就是我们的配置对象中的"concat"键(属性)。下面便是我的"concat"任务的配置对象。

    concat: {
      options: {

        separator: ';'
      },
      dist: {

        src: ['src/**/*.js'],

        dest: 'dist/<%= pkg.name %>.js'
      }
    }

注意我是如何引用JSON文件(也就是我们在配置对象顶部引入的配置文件)中的`name`属性的。这里我使用`pkg.name`来访问我们刚才引入并存储在`pkg`属性中的`package.json`文件信息，它会被解析为一个JavaScript对象。Grunt自带的有一个简单的模板引擎用于输出配置对象(这里是指`package.json`中的配置对象)属性值，这里我让`concat`任务将所有存在于`src/`目录下以`.js`结尾的文件合并起来，然后存储在`dist`目录中，并以项目名来命名。

现在我们来配置uglify插件，它的作用是压缩（minify）JavaScript文件：

    uglify: {
      options: {

        banner: '/*! <%= pkg.name %> <%= grunt.template.today("dd-mm-yyyy") %> */n'
      },
      dist: {
        files: {
          'dist/<%= pkg.name %>.min.js': ['<%= concat.dist.dest %>']
        }
      }
    }

这里我们让`uglify`在`dist/`目录中创建了一个包含压缩结果的JavaScript文件。注意这里我使用了`<%= concat.dist.dest>`，因此uglify会自动压缩concat任务中生成的文件。

QUnit插件的设置非常简单。 你只需要给它提供用于测试运行的文件的位置，注意这里的QUnit是运行在HTML文件上的。

    qunit: {
      files: ['test/**/*.html']
    },

JSHint插件的配置也很简单：

    jshint: {

      files: ['gruntfile.js', 'src/**/*.js', 'test/**/*.js'],

      options: {

        globals: {
          jQuery: true,
          console: true,
          module: true
        }
      }
    }

JSHint只需要一个文件数组(也就是你需要检测的文件数组)， 然后是一个`options`对象(这个对象用于重写JSHint提供的默认检测规则)。你可以到[JSHint官方文档][1]站点中查看完整的文档。如果你乐于使用JSHint提供的默认配置，那么在Gruntfile中就不需要重新定义它们了.

最后，我们来看看watch插件：

    watch: {
      files: ['<%= jshint.files %>'],
      tasks: ['jshint', 'qunit']
    }

你可以在命令行使用`grunt watch`来运行这个任务。当它检测到任何你所指定的文件(在这里我使用了JSHint任务中需要检测的相同的文件)发生变化时，它就会按照你所指定的顺序执行指定的任务(在这里我指定了jshint和qunit任务)。

最后, 我们还要加载所需要的Grunt插件。 它们应该已经全部通过npm安装好了。

    grunt.loadNpmTasks('grunt-contrib-uglify');
    grunt.loadNpmTasks('grunt-contrib-jshint');
    grunt.loadNpmTasks('grunt-contrib-qunit');
    grunt.loadNpmTasks('grunt-contrib-watch');
    grunt.loadNpmTasks('grunt-contrib-concat');

最后设置了一些task。最重要的是default任务：

    grunt.registerTask('test', ['jshint', 'qunit']);

    grunt.registerTask('default', ['jshint', 'qunit', 'concat', 'uglify']);

下面便是最终完成的 `Gruntfile` 文件：

    module.exports = function(grunt) {

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
            banner: '/*! <%= pkg.name %> <%= grunt.template.today("dd-mm-yyyy") %> */n'
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
          files: ['Gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
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