# gulp-inline-ng2-template
This is a fork of [gulp-inline-ng2-template](https://github.com/ludohenin/gulp-inline-ng2-template).

__note:__

* 4.0.1 -
  * fixed for partial process
  * added original url to processors - if you want to use a bundler and to leave the files processing to that bundler, replace the text with require
  * grave accent for style is optional - useStyleGraveAccent option, default useStyleGraveAccent = true


Inline Angular HTML and CSS files into JavaScript ES5/ES6 and TypeScript files (and possibly more - not tested).

This plugin uses the [ES6 template strings](https://github.com/lukehoban/es6features#template-strings) syntax by default _(which requires the use of a transpiler -typescript, babel, traceur- to produce valid ES5 files)_ but you can opt-in for ES5 one.

Very convenient to unit test your component or bundle your components/application (avoid extra HTTP request and keeps your source clean).

## TOC

* [Installation](#installation)
* [Configuration](#configuration)
  * [Options](#options)
  * [Processors configuration](#processors-configuration)
  * [Processor Examples](#processor-examples)
  * [Template function](#template-function)
  * [CustomFilePath configuration](#customfilepath-configuration)
* [Example usage](#example-usage)
* [Browserify transform example](#browserify-transform-example)
* [How it works](#how-it-works)
* [Contribute](#contribute)
* [Contributors](#contributors)
* [Todo](#todo)
* [Licence](#licence)

## Installation

```bash
npm install gulp-inline-ng2-template --save-dev
```

## Configuration

### Options

You can pass a configuration object to the plugin.
```javascript
defaults = {
  base: '/',                  // Angular2 application base folder
  target: 'es6',              // Can swap to es5
  indent: 2,                  // Indentation (spaces)
  useRelativePaths: false     // Use components relative assset paths
  removeLineBreaks: false     // Content will be included as one line
  templateExtension: '.html', // Update according to your file extension
  templateFunction: false,    // If using a function instead of a string for `templateUrl`, pass a reference to that function here
  templateProcessor: function (path, ext, file, url, callback) {/* ... */},
  styleProcessor: function (path, ext, file, url, callback) {/* ... */},
  customFilePath: function(ext, file) {/* ... */},
  supportNonExistentFiles: false // If html or css file do not exist just return empty content
};
```

### Processors configuration

```typescript
/**
 *  Processor function call signature and type return
 *
 * @Param{String}   file path
 * @Param{String}   file extension (type)
 * @Param{String}   file content
 * @Param{Function} callback function (err, result) => void
 * @Return{void}
 */
function processor(path, ext, file, url, cb) {
  // async implementation of your source files processing goes here ...
  cb(null, file);
}
```

### Processor Examples

Minify template file before inlining them

```javascript
import inlineTemplate from 'gulp-inline-ng2-template';
import htmlMinifier from 'html-minifier';

const pluginOptions = {
  base: mySrcPath,
  templateProcessor: minifyTemplate
};

function minifyTemplate(path, ext, file, url, cb) {
  try {
    var minifiedFile = htmlMinifier.minify(file, {
      collapseWhitespace: true,
      caseSensitive: true,
      removeComments: true,
      removeRedundantAttributes: true
    });
    cb(null, minifiedFile);
  }
  catch (err) {
    cb(err);
  }
}
```

_Credit [@lcrodriguez](https://github.com/lcrodriguez)_

### Template function

Inside your component: `templateUrl: templateFunc('app.html')`

```es6
/**
 *  Template function call signature and type return
 *
 * @Param{String}   filename
 * @Return{String}  returned filename
 */
templateFunction: function (filename) {
  // ...
  return newFilename;
}
```

### CustomFilePath configuration

```typescript
/**
 *  Custom function name call signature and type return
 *
 * @Param{String}   file extension (type)
 * @Param{String}   file path
 * @Return{String}  returned file path updated
 */
function customFilePath(ext, file) {
  return file;
}
```

## Example usage

```javascript
//...
var inlineNg2Template = require('gulp-inline-ng2-template');

var result = gulp.src('./app/**/*.ts')
  .pipe(inlineNg2Template({ base: '/app' }))
  .pipe(tsc());

return result.js
  .pipe(gulp.dest(PATH.dest));
```

## Browserify transform example

Example transform function to use with Browserify.

```javascript
// ng2inlinetransform.js
var ng2TemplateParser = require('gulp-inline-ng2-template/parser');
var through = require('through2');
var options = {target: 'es5'};

function (file) {
  return through(function (buf, enc, next){
    ng2TemplateParser({contents: buf, path: file}, options)((err, result) => {
      this.push(result);
      process.nextTick(next);
    });
  });
}
```

```javascript
// gulp task
return browserify('main.ts', {} )
  .add(config.angularApp.additionalFiles)
  .plugin(require('tsify'), {target: 'es5'})
  .transform('./ng2inlinetransform')
  .bundle()
  .pipe(gulp.dest(config.rootDirectory))
```

_Thanks to [@zsedem](https://github.com/zsedem)_

## How it works

__app.html__
```html
<p>
  Hello {{ world }}
</p>
```

__app.css__
```css
.hello {
  color: red;
}
```

__app.ts__
```javascript
import {Component, View} from 'angular2/angular2';
@Component({ selector: 'app' })
@View({
  templateUrl: './app.html',
  styleUrls: ['./app.css'],
  directives: [CORE_DIRECTIVES]
})
class AppCmp {}
```

__result (app.ts)__
```javascript
import {Component, View} from 'angular2/angular2';
@Component({ selector: 'app' })
@View({
  template: `
    <p>
      Hello {{ world }}
    </p>
  `,
  styles: [`
    .hello {
      color: red;
    }
  `],
  directives: [CORE_DIRECTIVES]
})
class AppCmp {}
```

## Contributors

![Contributors](https://webtask.it.auth0.com/api/run/wt-ludovic_henin-yahoo_com-0/contributors-list/ludohenin/gulp-inline-ng2-template.svg)

## Todo

- [ ] Append styles into `styles` View config property if it exist
- [ ] Add support for source maps
- [ ] Add option `skipCommented`

## Licence

MIT
