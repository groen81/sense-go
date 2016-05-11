[![](https://david-dm.org/stefanwalther/sense-go.png)](https://david-dm.org/stefanwalther/sense-go)
[![](https://travis-ci.org/stefanwalther/sense-go.svg?branch=master)](https://travis-ci.org/stefanwalther/sense-go.svg?branch=master)

# sense-go

> Library to easily handle validation, deployment, packaging and testing web projects (e.g. Mashups for Qlik Sense, Qlik Sense Visualization Extensions or basically any other project ...).

***

<!-- toc -->

* [Installation](#installation)
* [Purpose](#purpose)
* [Deployment by convention](#deployment-by-convention)
* [Basic workflow](#basic-workflow)
* [Configuration](#configuration)
* [Tasks](#tasks)
  - [Deployment](#deployment)
  - [Bump](#bump)
  - [Clean](#clean)
  - [Copy](#copy)
  - [Minification/Optimization](#minification-optimization)
  - [Import](#import)
  - [Less](#less)
  - [Replace](#replace)
  - [Uglify](#uglify)
  - [Wbfolder](#wbfolder)
* [Task Chains](#task-chains)
* [Used Gulp plugins](#used-gulp-plugins)
* [Contributing](#contributing)
* [Author](#author)
* [License](#license)

_(Table of contents generated by [verb])_

<!-- tocstop -->

***

## Installation

**Step 1:** Install sense-go in your project

```js
npm install sense-go --save-dev
```

_Note:_

_Currently you have to save **_sense-go_** locally as a devDependency in your project. It is planned to get **_sense-go_** working as a global dependency, since **_sense-go_** is quite heavy because of the many gulp packages being installed. As an alternative install **_sense-go_** globally and do an `npm link` in your project as for now._

**Step 2:** Install Gulp4, to run your local gulpfile.js

```js
npm install git://github.com/gulpjs/gulp#4.0 --save-dev
```

_Note:_

_The beta of Gulp 4 is used in this project._

**Step 3:** Create a gulpfile.js in the root folder of your project, containing the following minimal code:

```js
var gulp = require('gulp');
var senseGo = require('sense-go');

var userConfig = {
    "packageName": "Your Package Name"
};

senseGo.init( gulp, userConfig,  function (  ) {
  // Your own gulp tasks or task-chains here
  // ...
});
```

Then run **_sense-go_** by:

* Running any of the single gulp commands,
* Run a predefined task chain or
* Define your own tasks/tasks-chains, combine them with pre-configured ones and run them.

## Purpose

Main purpose of this library is to provide a framework to easily

* validate
* package
* deploy and
* test

**Visualization Extensions** created for Qlik Sense.

The implementation is a based on the deployment functionality in the [Yeoman Generator for Visualization Extensions](https://github.com/stefanwalther/generator-qsExtension).

The main reason behind creating this library is that I am creating a lot of different visualization extensions for Qlik Sense, but in any of these projects I include some kind of deployment system (so far always using grunt). If I have to make changes to the general deployment approach I have to change every single visualization extension repository, which is not really ideal. So introducing this library centralizes the deployment needs and allows me to re-use a central approach.

Technically speaking sense-go is just a collection of configurable gulp tasks which can be easily re-used when developing your Qlik Sense visualization extensions.

## Deployment by convention

The entire concept follows **conventions** I am using when setting up a project (working on Qlik Sense visualization extensions or mashups):

```
| PROJECT-ROOT
|-- build           <= all builds, including source code or zipped files
    |-- dev         <= target for the development build
    |-- release     <= target for the release build
|-- docs            <= documentation files, then used by verb
|-- src             <= all source files
    |-- lib
        |-- css     <= see below *
        |-- less    <= less files
| .sense-go.yml     <= sense-go configuration file
| .verb.md          <= verbs readme template
| gulpfile.js       <= gulp file using sense-go
| package.json
```

* If using less files is preferred for a project I keep this folder empty, otherwise all the .css files will be place here

**_sense-go_** works best if you follow these conventions, otherwise everything is configurable, it's just a bit more work to get **_sense-go_** running immediately, but that's definitely NOT the idea behind this project.

## Basic workflow

The workflow of the pre-configured tasks can be summarized as follows:

* You **develop** in the `.src` folder
* Whenever you want to **test or deploy**, use a one-liner in your command line
* To
  - Convert `.less` files to `.css` files
  - Lint, Minify, Ugilify the output
  - Create a .zip file to distribute your visualization extension
  - ... a lot of other neat tasks ... fully customizable ...
* Then the **extension is being deployed**
  - To the local extension directory (Qlik Sense Desktop)
  - Imported to the Qlik Sense Server (using the Qlik Sense Repository API)
  - to other destinations, like via SSH

It is important to mention that you can by 100% re-define the workflow and also all default settings, but the idea of **sense-go** is really to get something up and running across different projects with as little configuration and development work as possible. So choose custom configurations wisely.

## Configuration

Considering the conventions mentioned above ***sense-go** comes with a default configuration based on these.

### Inline configuration

(TBD)

### Project configuration

When running in CLI define all configurations in the `sense-go.yml` file, located in the root of your project.

The easiest way to start with your custom configuration is to copy the [default configuration file](src/default-config.yml) and start modifying it.
But keep in mind, following the conventions, you should only need to adapt a few of the default configurations.

## Tasks

Get a list of all tasks by running

```
gulp --tasks
```

### Deployment

> Deploy to either the Extension folder on your local computer (using Qlik Sense Desktop), upload to a server via ssh or upload to a Qlik Sense Repository using the Qlik Sense Repository (QRS) API.

The following typical deployment tasks are available

### Qlik Sense Desktop

**`deploy:tmpToLocal`**

* Copies all files (except the excluded ones) from the `.mp` directory to the local extension directory, creating a new folder for the current package and eventually deleting any already existing files in the targeted folder.
* Options used:
  - `tmpDir`
  - `localExtensionDir`
* Excluded files:
  - `*.less`

Note: The path for the local deployment will be fetched automatically (using [sense-loc](https://github.com/stefanwalther/sense-loc)), if you want to override the path, use `localExtensionDir`.

### Upload to Qlik Sense server

Upload the zipped visualization extension to a Qlik Sense server (using the Repository API in behind).

(tbd)

### Upload via SSH

**`deploy:toSsh`**

* Deploy the final output via SSH to another computer
* Options used:
  - `deployment.toSsh.enabled`
  - `deployment.toSsh.host`
  - `deployment.toSsh.port`
  - `deployment.toSsh.username`
  - `deployment.toSsh.password`
  - `deployment.toSsh.dest`

Note: `deploy:toSsh` has mainly be tested with [mobaSSH](http://mobassh.mobatek.net/), using certificates is not tested, yet.

### Bump

> Bumps the version in your package.json file

**`gulp bump:patch`**

* Changes the version in package.json from `  0.2.1` to `  0.2.2`
* Shortcuts: `gulp b` or `gulp b:p`

**`gulp bump:minor`**

* Changes the version in package.json from `  0.2.1` to `  0.3.1`
* Shortcut: `gulp b:min`

**`gulp bump:major`**

* Changes the version in package.json from `  0.2.1` to `  1.0.0`
* Shortcut: `gulp b:maj`

**`gulp bump:version`**

* Set the package.json version to a specific value given by the parameter `--newversion` resp. `--nv`.
* Shortcut: `gulp b:v`
* Example:

```sh
gulp bump:version --newversion=0.1.0
gulp b:v --nv=0.1.0
```

**Possible command line parameters**

**`--tag`**

* Tags the current version of your commit with the newly created version created by any of the bump-tasks.

**`--commit="Your commit message"`**

* Commits all files with the given commit message, if no commit message is defined, "." will be committed as a message.

### Clean

> Cleaning and deleting folders.

**`gulp clean:tmp`**

* Delete the entire `.tmp` directory.
* Options used:
  - `tmpDir`

**`gulp clean:buildDev`**

* Deletes all files in the `./build/dev` directory.
* Options used:
  - `buildDevDir`

**`gulp clean:buildRelease`**

* Deletes all files in the `./build/release` directory.
* Options used:
  - `buildReleaseDir`

**`clean:localExtensionDir`**

* Deletes all files in the project's local extension folder. Only makes sense if working against Qlik Sense Desktop. Disabled if `deployment.toLocal.enabled === true`.
* Options used:
  - `deployment.toLocal.enabled`
  - `deployment.toLocal.extensionBaseDir`

**`clean:tmpIllegal`**

* Clean all files in the `.tmp` directory which are not supposed to be deployed to the extension directory.
* These are all files, except files with the following file extension:
  - `{png,jpg,jpeg,json,qext,txt,js,css,eot,svg,ttf,woff,html,htm,wbl,svg}`

### Copy

> Copy files to a specific directory on your system.

**`gulp copy:toTmp`**
* Copies all files (except the excluded ones) from the `src` folder to the `.tmp` folder.
* Options used:
  - `srcDir`
  - `tmpDir`

* Excluded files:
  - `*.less`

**`copy:tmpToDev`**
* Copies all files (except the excluded ones) from the `.tmp` folder to `.\build\dev` folder.
* Options used:
  - `tmpDir`
  - `buildDevDir`

* Excluded files:
  - `*.less`

**`copy:tmpToRelease`**
* Copies all files (except the excluded ones) from the `.tmp` folder to `.\build\release` folder.
* Options used:
  - `tmpDir`
  - `buildReleaseDir`

* Excluded files:
  - `*.less`

### Minification/Optimization

> Several minification tasks

**`minify:html:tmp`**

* Minifies all htm/html files in the `tmp` folder
* Options used:
  - `tmpDir`

**`minify:json:tmp`**
* Minify .json & .qext files
* Options used:
  - `tmpDir`

### Import

> Import files to the deployment.

`gulp import`
The main use-case behind this task is to be able to import "external" files from external dependencies (e.g. node_modules or bower) into the .tmp directory to use them in the solution.

Define the file you want to import in your `sense-go.yml` file as follows:

Example:

```
import:
  files:
    - ["./node_modules/d3/d3.min.js", "./.tmp/lib/external/d3/d3.min.js"]
    - ["./node_modules/moment/min/moment.min.js", "./.tmp/lib/external/moment/moment.min.js"]
```

### Less

> Converts .less files to .css files.

All less tasks automatically create a sourcemap (using [gulp-sourcemaps](http://github.com/floridoo/gulp-sourcemaps)) and autoprefix (using [gulp-autoprefixer](https://github.com/sindresorhus/gulp-autoprefixer))

**`gulp less:reduce`**

* Uses `/src/less/main.less`, resolves all its dependencies and creates `/.tmp/css/main.css`
* Options used:
  - srcDir
  - tmpDir

**`gulp less:each`**

* Converts every `.less` file from the source directory to a corresponding .css file in the .tmp directory.
* Options used:
  - srcDir
  - tmpDir

### Replace

> Replaces string patterns in text files across your project.

**Usage**
* Use @@ to prefix the key to be replaced with a given value in the source code
* Replacements will only be performed in the following file types:
  - .html
  - .js
  - .json
  - .qext
  - .txt
  - .xml
  - .yml

**Example:**

```js
console.log('Extension @@pkg.name, version @@pkg.version');
```

given the following package.json

```js
{
  "name": "my-extension",
  "version": "0.1.2"
}
```

will return

```js
Extension my-extension, version 0.1.2
```

```js

```

**Using data from package.json**

All keys from your package.json file are available out of the box if you use the prefix `pkg`

* To get the version, use `@@pkg.version`
* To the get name, use `@@pkg.name`
* etc.

**Builtin patterns**

The following patterns are available out of the box:

`@@timestamp` - Defaults to new Date().getTime()

**Adding replacement patterns**

Add new replacements patterns in your .sense-go.yml file:

(tbd)

### Uglify

> Uglify & minifies JavaScript files

**`uglify:tmp`**

* Uglify all JavaScript files.
* Options:
  - `tmpDir`
  - `uglify*` - All options directly passed to `gulp-uglify`, e.g.
    + `uglify.mangle`
    + `uglify.beautify`
    + `uglify.preserveComments`

* Excluded:
  - All files matching the pattern `*.min.js`

### Wbfolder

> Create a wbfolder.wbl file to be used in Dev Hub - Extension Editor.

**`gulp wbfolder`**
* Creates a wbfolder.wbl file in  the `.tmp` directory.
* Options used:
  - `wbfolder.enabled`- Whether the task is enabled or not, defaults to `true`.
  - `wbfolder.cwd` - Working directory, defaults to `./.tmp`.
  - `wbfolder.src` - Source mask, defaults to `./**/*.*`.
  - `wbfolder.dest` - wbfolder.wbl file destination, defaults to `./.tmp/wbfolder.wbl`.

Note: The wbfolder.wbl is only necessary if you want to allow users to open your visualization extension in Dev Hub. wbfolder.wbl is NOT required and necessary to let your visualization extension being used within Qlik Sense.

## Task Chains

Based on gulp tasks provided by **_sense-go_** you can then create your task chains.
Some are already predefined:

**`gulp build`**

```js
  gulp.task( 
    'build', 
    gulp.series( 
      'init', 
      'clean:tmp', 
      'copy:toTmp', 
      'replace:tmp', 
      'clean:buildDev', 
      'copy:tmpToDev', 
      'clean:localExtensionDir', 
      'deploy:tmpToLocal' 
      )
  );`
```

## Create your own task-chains

You can add additional tasks on top of sense-go, mixing your very own tasks with sense-go tasks, etc.

* Always initialize a task chain with the `init` task
* When creating your own tasks, note that sense-go relies on Gulp4

**Example:**

```js
'use strict';
var gulp = require('gulp');
var senseGo = require('./lib/');

var userConfig = {
    "packageName": "sense-go"
};

senseGo.init( gulp, userConfig,  function (  ) {
  
  // Create your own task chain, and overwrite the current 'build' task
  gulp.task( 'build', gulp.series( 
    'init', 
    'clean:tmp', 
    'copy:toTmp', 
    'myTask1', 
    'myTask2' 
  ) );
    
});
```

## Used Gulp plugins

sense-go is heavily relying on existing gulp plugins. A big thank you to the authors of these plugins!!

### General

* [gulp-concat](https://www.npmjs.com/package/gulp-concat): Concatenates files | [homepage](https://github.com/wearefractal/gulp-concat)
* [gulp-csslint](https://www.npmjs.com/package/gulp-csslint): CSSLint plugin for gulp | [homepage](https://github.com/lazd/gulp-csslint)
* [gulp-debug](https://www.npmjs.com/package/gulp-debug): Debug vinyl file streams to see what files are run through your gulp pipeline | [homepage](https://github.com/sindresorhus/gulp-debug)
* [gulp-less](https://www.npmjs.com/package/gulp-less): Less for Gulp | [homepage](https://github.com/plus3network/gulp-less)
* [gulp-load-plugins](https://www.npmjs.com/package/gulp-load-plugins): Automatically load any gulp plugins in your package.json | [homepage](https://github.com/jackfranklin/gulp-load-plugins)
* [gulp-replace](https://www.npmjs.com/package/gulp-replace): A string replace plugin for gulp | [homepage](https://github.com/lazd/gulp-replace)
* [gulp-rimraf](https://www.npmjs.com/package/gulp-rimraf): rimraf plugin for gulp | [homepage](https://github.com/robrich/gulp-rimraf)
* [gulp-simple-task-loader](https://www.npmjs.com/package/gulp-simple-task-loader): A simple task loader for gulp | [homepage](https://github.com/reaganthomas/gulp-simple-task-loader)
* [gulp-watch](https://www.npmjs.com/package/gulp-watch): Watch, that actually is an endless stream | [homepage](https://github.com/floatdrop/gulp-watch)

### Validation

* [gulp-csslint](https://www.npmjs.com/package/gulp-csslint): CSSLint plugin for gulp | [homepage](https://github.com/lazd/gulp-csslint)
* [gulp-htmllint](https://www.npmjs.com/package/gulp-htmllint): Check HTML code style with htmllint | [homepage](https://github.com/yvanavermaet/gulp-htmllint)
* [gulp-jsonlint](https://www.npmjs.com/package/gulp-jsonlint): A jsonlint plugin for Gulp | [homepage](https://github.com/rogeriopvl/gulp-jsonlint)

### Packaging

* [gulp-htmlmin](https://www.npmjs.com/package/gulp-htmlmin): gulp plugin to minify HTML. | [homepage](https://github.com/jonschlinkert/gulp-htmlmin)
* [gulp-uglify](https://www.npmjs.com/package/gulp-uglify): Minify files with UglifyJS. | [homepage](https://github.com/terinjokes/gulp-uglify/)
* [gulp-zip](https://www.npmjs.com/package/gulp-zip): ZIP compress files | [homepage](https://github.com/sindresorhus/gulp-zip)

### Testing

[gulp-mocha](https://www.npmjs.com/package/gulp-mocha): Run Mocha tests | [homepage](https://github.com/sindresorhus/gulp-mocha)

### Transpilation

[babel](https://www.npmjs.com/package/babel): Turn ES6 code into readable vanilla ES5 with source maps | [homepage](https://babeljs.io/)

## Contributing

Pull requests and stars are always welcome. For bugs and feature requests, [please create an issue](https://github.com/stefanwalther/sense-go/issues).
The process for contributing is outlined below:

1. Create a fork of the project
2. Work on whatever bug or feature you wish
3. Create a pull request (PR)

I cannot guarantee that I will merge all PRs but I will evaluate them all.

## Author

**Stefan Walther**

+ [qliksite.io](http://qliksite.io)
* [twitter/waltherstefan](http://twitter.com/waltherstefan)
* [github.com/stefanwalther](http://github.com/stefanwalther)

## License

Copyright © 2016 Stefan Walther<br/>
Released under the MIT license.

***

_This file was generated by [verb-cli](https://github.com/assemble/verb-cli) on May 11, 2016._