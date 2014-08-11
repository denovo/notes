**Setting up gulp build system.**

We currently use Grunt to build out our Sass and JS files aswell as for doing quick prototypes in Jade. There seems to be 
a new build system out every day, its hard to keep up with the Jones.

Why should we moved to Gulp? Our Grunt task works.  Good question. Gulp’s code-over-configuration makes it not only easy to write tasks for, 
but also much easier to read and maintain.

Gulp uses node.js streams, making it faster to build as it doesn’t need to write temporary files/folders to disk. 
If you want to learn more about streams—although not necessary—this article is a good read. 
Gulp allows you to input your source file(s), pipe them through a bunch of plugins and get an output at the end, 
rather than configuring each plugin with an input and output—like in Grunt.


So you’re interested, now what? Let’s install gulp and create a basic gulpfile with some core tasks to get started.

Installing gulp
Before we delve into configuring tasks, we need to install gulp:

```
$ npm install gulp -g
```
This installs gulp globally, giving access to gulp’s CLI. We then need to install it locally to the project. cd into your 
project and run the following (make sure you have an existing package.json file):

```
$ npm install gulp --save-dev
```

This installs gulp locally to the project and saves it to the devDependencies in the package.json file.

Installing gulp plugins
We are going to install some plugins to achieve the following tasks:


gulp (gulp),
gutil       = require('gulp-util'),
sass        = require('gulp-sass'),
csso        = require('gulp-csso'),
uglify      = require('gulp-uglify'),
jade        = require('gulp-jade'),
concat      = require('gulp-concat'),
livereload  = require('gulp-livereload'), // Livereload plugin needed: https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei
tinylr      = require('tiny-lr'),
express     = require('express'),
app         = express(),
marked      = require('marked'), // For :markdown filter in jade
path        = require('path'),
scsslint    = require('gulp-scsslint'),
compass     = require('gulp-compass'),
autoprefixer = require('gulp-autoprefixer'),
rename      = require('gulp-rename'),
coffee      = require("gulp-coffee"),
notify      = require('gulp-notify'),
clean       = require('gulp-clean'),
changed     = require('gulp-changed')
server      = tinylr(),
md5         = require('MD5');
    
To install these plugins, run the following command:

1
$ npm install gulp-ruby-sass gulp-autoprefixer gulp-minify-css gulp-jshint gulp-concat gulp-uglify gulp-imagemin gulp-clean gulp-notify gulp-rename gulp-livereload gulp-cache --save-dev
This will install all necessary plugins and save them to devDependencies in package.json. A full list of gulp plugins can be found here.

Load in the plugins
Next, we need to create a gulpfile.js and load in the plugins:

```
var gulp = require('gulp'),
    sass = require('gulp-ruby-sass'),
    autoprefixer = require('gulp-autoprefixer'),
    minifycss = require('gulp-minify-css'),
    jshint = require('gulp-jshint'),
    uglify = require('gulp-uglify'),
    imagemin = require('gulp-imagemin'),
    rename = require('gulp-rename'),
    clean = require('gulp-clean'),
    concat = require('gulp-concat'),
    notify = require('gulp-notify'),
    cache = require('gulp-cache'),
    livereload = require('gulp-livereload');
```
Phew! That seems a lot more work than Grunt, right? Gulp plugins are slightly different from Grunt plugins—they are designed to do one thing and one thing well. An example; Grunt’s imagemin uses caching to avoid re-compressing images that are already compressed. With gulp, this would be done with a cache plugin, which can also be used to cache other things too. This adds an extra layer of flexibility to the build process. Pretty cool huh?

We can auto load all installed plugins like in Grunt, but for the purpose of this post we’ll stick to the manual method.

Creating tasks
Compile Sass, Autoprefix and minify

Firstly, we will configure Sass compiling. We’re going to compile Sass as expanded, run it through Autoprefixer and save it to our destination. We’ll then create a minified .min version, auto-refresh the page and notify that the task is complete:

1
2
3
4
5
6
7
8
9
10
gulp.task('styles', function() {
  return gulp.src('src/styles/main.scss')
    .pipe(sass({ style: 'expanded' }))
    .pipe(autoprefixer('last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4'))
    .pipe(gulp.dest('dist/assets/css'))
    .pipe(rename({suffix: '.min'}))
    .pipe(minifycss())
    .pipe(gulp.dest('dist/assets/css'))
    .pipe(notify({ message: 'Styles task complete' }));
});
A little explanation before moving on.

1
gulp.task('styles', function() { ... )};
This is the gulp.task API which is used to create tasks. The above can run from Terminal with $ gulp styles.

1
return gulp.src('src/styles/main.scss')
This is the gulp.src API where we define the source file(s). It also can be a glob pattern, such as /**/*.scss to match multiple files. By returning the stream it makes it asynchronous, ensuring the task is fully complete before we get a notification to say it’s finished.

1
.pipe(sass({ style: 'expanded' }))
We use .pipe() to pipe the source file(s) into a plugin. Usually the options for a plugin are found on their respective GitHub page. I’ve linked them above for convenience.

1
.pipe(gulp.dest('dist/assets/css'));
The gulp.dest API is where we set the destination path. A task can have multiple destinations, one to output the expanded version and one to output the minifed version. This is demonstrated in the above styles task.

I’d recommend checking out the gulp API documentaiton to get a better understanding of these methods. It’s not as scary as it sounds!

JSHint, concat, and minify JavaScript

Hopefully you’ll now have a good idea of how to create a task for gulp. Next, we’ll set up the scripts task to lint, concat and uglify:

1
2
3
4
5
6
7
8
9
10
11
gulp.task('scripts', function() {
  return gulp.src('src/scripts/**/*.js')
    .pipe(jshint('.jshintrc'))
    .pipe(jshint.reporter('default'))
    .pipe(concat('main.js'))
    .pipe(gulp.dest('dist/assets/js'))
    .pipe(rename({suffix: '.min'}))
    .pipe(uglify())
    .pipe(gulp.dest('dist/assets/js'))
    .pipe(notify({ message: 'Scripts task complete' }));
});
One thing to note is that we need to specify a reporter for JSHint. I’m using the default reporter, which should be fine for most people. More on this can be found on the JSHint website.

Compress Images

Next, we’ll set up image compression:

1
2
3
4
5
6
gulp.task('images', function() {
  return gulp.src('src/images/**/*')
    .pipe(imagemin({ optimizationLevel: 3, progressive: true, interlaced: true }))
    .pipe(gulp.dest('dist/assets/img'))
    .pipe(notify({ message: 'Images task complete' }));
});
This will take any source images and run them through the imagemin plugin. We can go a little further and utilise caching to save re-compressing already compressed images each time this task runs. All we need is the gulp-cahce plugin—which we installed earlier. To set this up, we need to change this line:

1
.pipe(imagemin({ optimizationLevel: 3, progressive: true, interlaced: true }))
To this:

1
.pipe(cache(imagemin({ optimizationLevel: 5, progressive: true, interlaced: true })))
Now only new or changed images will be compressed. Neat!

Clean up!

Before deploying, it’s a good idea to clean out the destination folders and rebuild the files—just in case any have been removed from the source and are left hanging out in the destination folder:

1
2
3
4
gulp.task('clean', function() {
  return gulp.src(['dist/assets/css', 'dist/assets/js', 'dist/assets/img'], {read: false})
    .pipe(clean());
});
We can also pass an array of folders (or files) into gulp.src. As we don’t need to read files that are being deleted, we can add read: false to the options to prevent gulp from reading the file contents—making it much quicker.

The default task

We can create a default task, ran by using $ gulp, to run all three tasks we have created:


gulp.task('default', ['clean'], function() {
    gulp.start('styles', 'scripts', 'images');
});
Notice the additional array in gulp.task. This is where we can define task dependencies. In this example, the clean task will run before the tasks in gulp.start. Tasks in Gulp run concurrently together and have no order in which they’ll finish, so we need to make sure the clean task is completed before running additional tasks.

Note: It’s advised against using gulp.start in favour of executing tasks in the dependency arrary, but in this scenario to ensure clean fully completes, it seems the best option.

Watch

To watch our files and perform the necessary task when they change, we firstly need to create a new task, then use the gulp.watch API to begin watching files:


gulp.task('watch', function() {

  // Watch .scss files
  gulp.watch('src/styles/**/*.scss', ['styles']);

  // Watch .js files
  gulp.watch('src/scripts/**/*.js', ['scripts']);

  // Watch image files
  gulp.watch('src/images/**/*', ['images']);

});
We specify the files we want to watch via the gulp.watch API and define which task(s) to run via the dependency array. We can now run $ gulp watch and any changes to .scss, .js or image files will run their respective tasks.

LiveReload

Gulp can also take care of automatically refreshing the page on file change. We’ll need to modify our watch task, configuring the LiveReload server.

1
2
3
4
5
6
7
8
9
10
11
gulp.task('watch', function() {

  // Create LiveReload server
  var server = livereload();

  // Watch any files in dist/, reload on change
  gulp.watch(['dist/**']).on('change', function(file) {
    server.changed(file.path);
  });

});
To make this work, you’ll need to install and enable the LiveReload browser plugin. You can also place the snippet in manually.

Putting it all together
Here we have the full gulpfile:


// Load plugins
var gulp = require('gulp'),
    sass = require('gulp-ruby-sass'),
    autoprefixer = require('gulp-autoprefixer'),
    minifycss = require('gulp-minify-css'),
    jshint = require('gulp-jshint'),
    uglify = require('gulp-uglify'),
    imagemin = require('gulp-imagemin'),
    rename = require('gulp-rename'),
    clean = require('gulp-clean'),
    concat = require('gulp-concat'),
    notify = require('gulp-notify'),
    cache = require('gulp-cache'),
    livereload = require('gulp-livereload');

// Styles
gulp.task('styles', function() {
  return gulp.src('src/styles/main.scss')
    .pipe(sass({ style: 'expanded', }))
    .pipe(autoprefixer('last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4'))
    .pipe(gulp.dest('dist/styles'))
    .pipe(rename({ suffix: '.min' }))
    .pipe(minifycss())
    .pipe(gulp.dest('dist/styles'))
    .pipe(notify({ message: 'Styles task complete' }));
});

// Scripts
gulp.task('scripts', function() {
  return gulp.src('src/scripts/**/*.js')
    .pipe(jshint('.jshintrc'))
    .pipe(jshint.reporter('default'))
    .pipe(concat('main.js'))
    .pipe(gulp.dest('dist/scripts'))
    .pipe(rename({ suffix: '.min' }))
    .pipe(uglify())
    .pipe(gulp.dest('dist/scripts'))
    .pipe(notify({ message: 'Scripts task complete' }));
});

// Images
gulp.task('images', function() {
  return gulp.src('src/images/**/*')
    .pipe(cache(imagemin({ optimizationLevel: 3, progressive: true, interlaced: true })))
    .pipe(gulp.dest('dist/images'))
    .pipe(notify({ message: 'Images task complete' }));
});

// Clean
gulp.task('clean', function() {
  return gulp.src(['dist/styles', 'dist/scripts', 'dist/images'], {read: false})
    .pipe(clean());
});

// Default task
gulp.task('default', ['clean'], function() {
    gulp.start('styles', 'scripts', 'images');
});

// Watch
gulp.task('watch', function() {

  // Watch .scss files
  gulp.watch('src/styles/**/*.scss', ['styles']);

  // Watch .js files
  gulp.watch('src/scripts/**/*.js', ['scripts']);

  // Watch image files
  gulp.watch('src/images/**/*', ['images']);

  // Create LiveReload server
  var server = livereload();

  // Watch any files in dist/, reload on change
  gulp.watch(['dist/**']).on('change', function(file) {
    server.changed(file.path);
  });

});
You can also check out the gulpfile in this gist. I’ve also put together a Gruntfile that accomplishes the same tasks for a comparison in the same gist.

If you have any questions or issues, please leave a comment below or you can find me on Twitter.
