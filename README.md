# grunt-usemin [![Build Status](https://secure.travis-ci.org/yeoman/grunt-usemin.png?branch=master)](http://travis-ci.org/yeoman/grunt-usemin)

> Replaces references to non-optimized scripts or stylesheets into a set of HTML files (or any templates/views).

Watch out, this task is designed for Grunt 0.4 and upwards.

## Getting Started
If you haven't used [grunt][] before, be sure to check out the [Getting Started][] guide, as it explains how to create a [gruntfile][Getting Started] as well as install and use grunt plugins. Once you're familiar with that process, install this plugin with this command:

```shell
npm install grunt-usemin --save-dev
```

[grunt]: http://gruntjs.com/
[Getting Started]: https://github.com/gruntjs/grunt/blob/devel/docs/getting_started.md

## Tasks

`usemin` is exporting 2 different tasks:

 - `useminPrepare` which purpose is to detect specific construction (blocks) in the scrutinized file, and to build the needed configuration to apply a transformation flow to the files referenced in the block.

 - `usemin` which purpose is to replace blocks by the file they referenced, and replace all references to assets by their revved version , if it is found on the disk. This target modifies the files it is working on.

Usually, `useminPrepare` is launched first, then the steps of the transformation flow (for example, `concat`, `uglify`, `cssmin` and `requirejs`), and then, in the end `usemin` is launched.


## The usemin task

The `usemin` task has 2 actions:

- First it replaces all the blocks with a single "summary" line, pointing at a file creating by the transformation flow.
- Then it looks for references to assets (i.e. images, scripts, ...), and tries to replace them with their revved version if it can find one on disk

### On directories

When `usemin` tries to replace referenced assets with their revved version it has to look at a collection of directories (asset search paths): for each of the directory of this collection it will look at the bellow tree, and try to find the revved version.
This asset search directories collection is by default set to the location of the file that is scrutinized but can be modified (see Options bellow).

#### Example 1: file `dist/html/index.html` has the following content:

``` html
<link rel="stylesheet" href="styles/main.css">
<img src="../images/test.png">
```
By default `usemin` will look under `dist/html` for revved versions of:

- `styles/main.css`: a revved version of `main.css` will be looked at under the `dist/html/styles` directory. For example a file `dist/html/styles/1234.main.css` will match (although `dist/html/1234.main.css` won't: the path of the referenced file is important)
- `../images/test.png`: it basically means that a revved version of `test.png` will be looked for under the `dist/images` directory

#### Example 2: file `dist/html/index.html` has the following content:

``` html
    <link rel="stylesheet" href="/styles/main.css">
    <img src="/images/test.png">
```
By default `usemin` will look under `dist/html` for revved versions of `styles/main.css` and `images/test.png`. Now let's suppose our assets are scattered in `dist/assets`. By changing the asset search path list to `['dist/assets']`, the revved versions of the files will be looked under `dist/assets` (and thus, for example, `dist/assets/images/875487.tes.png` and `dist/assets/styles/98090.main.css`) will be found.

### Options

#### assetsDirs

Type: 'Array'
Default: Single item array set to the value of the directory where the currently looked at file is.

List of directories where we should start to look for revved version of the assets referenced in the currently looked at file.

Example:
``` js
usemin: {
  html: 'build/index.html',
  options: {
    assetsDirs: ['foo/bar', 'bar']
  }
}
```

#### patterns

Type: 'Object'
Default: Empty

Allows for user defined pattern to replace reference to files. For example, let's suppose that you want for some reason replace
all references to `'image.png'` in your Javascript files by the revved version of `image.png` found bellow the directory `images`.
By specifying something along the lines of:

```js
usemin: {
  js: '*.js',
  options: {
    assetsDirs: 'images',
    patterns: {
      js: [[/(image\.png)/, 'Replacing reference to image.png']]
    }
  }
}
```

So in short:

* key in pattern should match the target (e.g `js` key for the target `js`)
* Each pattern is an array of arrays. These arrays are composed of 4 items (last 2 are optionals):
  * First one if the regexp to use. The first group is the one that is supposed to represent the file
    reference to replace
  * Second one is a logging string
    * FIXME
    * FIXME




## The useminPrepare task

`useminPrepare` task is updating the grunt configuration to apply a configured transformation flow to tagged files (i.e. blocks).
By default the transformation flow is composed of `concat` and `uglifyjs` for JS files, but it can be configured.

### Blocks
Blocks are expressed as:

```html
<!-- build:<type>(alternate search path) <path> -->
... HTML Markup, list of script / link tags.
<!-- endbuild -->
```

- **type**: either `js` or `css`
- ** alternate search path **: (optional) By default the input files are relative to the treated file. Alternate search path allow to change that
- **path**: the file path of the optimized file, the target output

An example of this in completed form can be seen below:

```html
<!-- build:js js/app.js -->
<script src="js/app.js"></script>
<script src="js/controllers/thing-controller.js"></script>
<script src="js/models/thing-model.js"></script>
<script src="js/views/thing-view.js"></script>
<!-- endbuild -->
```

### Transformation flow

The transformation flow is made of sequential steps: each of the step transform the file, and useminPrepare will modify the configuration in order to described steps are correctly performed.

By default the flow is: `concat -> uglifyjs`.
Additionnally to the flow, at the end, some postprocessors can be launched to alter further the configuration. The default processor used is 'requirejs': it alters the configuration to support requirejs correctly.

Let's have an example, using the default flow (we're just going to look at the steps), `app` for input dir, `dist` for output dir,  and the following block:

```html
<!-- build:js js/app.js -->
<script src="js/app.js"></script>
<script src="js/controllers/thing-controller.js"></script>
<script src="js/models/thing-model.js"></script>
<script src="js/views/thing-view.js"></script>
<!-- endbuild -->
```
The produced configurartion will look like:

```js
{
  concat: {
    '.tmp/concat/js/app.js': [
      'app/js/app.js',
      'app/js/controllers/thing-controller.js',
      'app/js/models/thing-model.js',
      'app/js/views/thing-view.js'
      ]
  },
  uglifyjs: {
    'dist/js/app.js': ['.tmp/concat/js/app.js']
  }
}
```



### Directories

Internally, the task parses your HTML markup to find each of these blocks, and initializes for you the corresponding Grunt config for the concat / uglify tasks when `type=js`, the concat / cssmin tasks when `type=css`.

The task also handles use of RequireJS, for the scenario where you specify the main entry point for your application using the "data-main" attribute as follows:

```html
<!-- build:js js/app.min.js -->
<script data-main="js/main" src="js/vendor/require.js"></script>
<!-- -->
```

One doesn't need to specify a concat/uglify/cssmin or RequireJS configuration anymore.

It is using only one target: `html`, with a list of the concerned files. For example, in your `Gruntfile.js`:

```js
'useminPrepare': {
  html: 'index.html'
}
```

### Options

### dest
Type: 'string'
Default: nil

Base directory where the transformed files should be output.

### flow
Type: 'object'
Default: { steps: ['concat', 'uglify'], post: ['requirejs']}

This allow you to configure the workflow, either on a per-target basis, or for all the targets.
You can change separately the `steps` or the post-processors (`post`).

For example:

* to change the `steps` and `post` for the target `html`:

```js
'useminPrepare', {
      html: 'index.html',
      options: {
        flow: {
          html: {
            steps: ['uglifyjs'],
            post: []
          }
        }
      }
    }
```

* to change the `steps` and `post` for all targets:

```js
'useminPrepare', {
      html: 'index.html',
      options: {
        flow: {
          steps: ['uglifyjs'],
          post: []
        }
      }
    }
```
The given steps or post-processors may be given by strings (for the default steps and post-processors), or as object (for the user-defined ones).

#### User-defined steps and post-processors

User-defined steps and post-processors must have 2 attributes:

* `name`: which is the name of the step or post-processors
* `createConfig` which is a 2 arguments function ( a `context` and the treated `block`)

##### `createConfig`

The `createConfig` function is responsible for creating (or updating) the configuration associated to the current step/post-processor.
It takes 2 arguments ( a `context` and the treated `block`), and returns a configuration object.

###### `context`
The `context` object represent the current context the step/post-processor is running in. As the step/post-processor is a step of a flow, it must be listed the input files and directory it must write a configuration for, potentially the already existing configuration. It must also indicate to the other steps/post-processor which files it will output in which directory. All this information is hold by the `context` object.
Attributes:

* `inDir`: the directory where the `input` file for the step/post-processors will be
* `inFiles`: the list of input file to take care of
* `outDir`: where the files created by the step/post-processors will be
* `outFiles`: the files that are going to be created
* `last`: whether or not we're the last step of the flow
* `options`: options of the `Grubntfile.js` for this step (e.g. if the step is named `foo`, holds configuration of teh `Gruntfile.js` associated to the attribute `foo`)

###### `block`


## The usemin task

This task is responsible for replacing in HTML and CSS files, references to non-minified files with reference to their minified/revved version if they are found on the disk.

```js
usemin: {
  html: ['**/*.html'],
  css: ['**/*.css'],
  options: {
    dirs: ['temp', 'dist']
  }
}
```
### dirs
Type: 'array of strings'
Default: nil

Used to limit the directories that will be looked for revved files when replacing reference. By default all subdirectories are looked at.

### basedir
Type: 'string'
Default: nil

Change the basedir that represent the location of the transformed file. For example, let's imagine you have someting like:

```
|
+--- styles
    \ main.css
+--- views
    \ index.html
```

By default, if the file to be transformed is `index.html`, the images, scripts, ... referenced by this file will be considered are being in the `views` directory, whereas they must be linked to the `styles` directory.

## On directories
The main difference to be kept in mind, regarding directories and tasks, is that for `useminPrepare`, the directories needs to indicate the input, transient and output path needed to output the right configuration for the processors pipeline, whereas in the case of `usemin` it only reflects the output paths, as all the needed assets should have been output to the destination dir (either transformed or just copied)

### useminPrepare
`useminPrepare` is trying to prepare the right configuration for the pipeline of actions that are going to be applied on the blocks (for example concatenation and uglify-cation). As such it needs to have the input directory, temporary directories (staging) and destination directory.
The files referenced in the block are either absolute or relative (`/images/foo.png` or `../../images/foo.png`).
Absolute files references are looked in a given set of search path (input), which by default is set to the directory where the html/css file examined is located (can be overriden per block).
Relative files references are also looked at from location of the examined file, unless stated otherwise.


### usemin
`usemin` target is replacing references to images, scrips, css, ... in the furnished files (html, css, ...). These references may be either absolute (i.e. `/images/foo.png`) or relative (i.e. `image/foo.png` or `../images/foo.png`).
When the reference is absolute a set of asset search paths should be looked at under the destination directory (for example, using the previous example, and `searchpath` equal to `['assets']`, `usemin` would try to find either a revved version of the image of the image bellow the `assets` directory: for example `dest/assets/images/1223443.foo.png`).
When the reference is relative, by default the referenced item is looked in the path relative *to the current file location* in the destination directory (e.g. with the preceding example, if the file is `build/bar/index.html`, then transformed `index.html` will be in `dist/bar`, and `usemin` will look for `dist/bar/../images/32323.foo.png`).

## License

[BSD license](http://opensource.org/licenses/bsd-license.php) and copyright Google
