# grunt-contrib-concat [![Build Status](https://secure.travis-ci.org/gruntjs/grunt-contrib-concat.png?branch=master)](http://travis-ci.org/gruntjs/grunt-contrib-concat)

> Concatenate files.

_Note that this plugin has not yet been released, and only works with the latest bleeding-edge, in-development version of grunt. See the [When will I be able to use in-development feature 'X'?](https://github.com/gruntjs/grunt/blob/devel/docs/faq.md#when-will-i-be-able-to-use-in-development-feature-x) FAQ entry for more information._

## Getting Started
If you haven't used [grunt][] before, be sure to check out the [Getting Started][] guide, as it explains how to create a [gruntfile][Getting Started] as well as install and use grunt plugins. Once you're familiar with that process, install this plugin with this command:

```shell
npm install grunt-contrib-concat --save-dev
```

[grunt]: http://gruntjs.com/
[Getting Started]: https://github.com/gruntjs/grunt/blob/devel/docs/getting_started.md


## Concat task
_Run this task with the `grunt concat` command._

_This task is a [multi task][] so any targets, files and options should be specified according to the [multi task][] documentation._
[multi task]: https://github.com/gruntjs/grunt/wiki/Configuring-tasks


### Options

#### separator
Type: `String`  
Default: linefeed

Concatenated files will be joined on this string. If you're post-processing concatenated JavaScript files with a minifier, you may need to use a semicolon `';'` as the separator.

#### banner
Type: `String`  
Default: empty string

This string will be prepended to the beginning of the concatenated output. It is processed using [grunt.template.process][], using the default options.

_(Default processing options are explained in the [grunt.template.process][] documentation)_

#### stripBanners
Type: `Boolean` `Object`  
Default: `false`

Strip JavaScript banner comments from source files.

* `false` - No comments are stripped.
* `true` - `/* ... */` block comments are stripped, but _NOT_ `/*! ... */` comments.
* `options` object:
  * By default, behaves as if `true` were specified.
  * `block` - If true, _all_ block comments are stripped.
  * `line` - If true, any contiguous _leading_ `//` line comments are stripped.

#### process
Type: `Boolean` `Object`  
Default: `false`

Process source files as [templates][] before concatenating.

* `false` - No processing will occur.
* `true` - Process source files using [grunt.template.process][] defaults.
* `options` object - Process source files using [grunt.template.process][], using the specified options.

_(Default processing options are explained in the [grunt.template.process][] documentation)_

  [templates]: https://github.com/gruntjs/grunt/blob/devel/docs/api_template.md
  [grunt.template.process]: https://github.com/gruntjs/grunt/blob/devel/docs/api_template.md#grunttemplateprocess

### Usage Examples

#### Concatenating with a custom separator

In this example, running `grunt concat:dist` (or `grunt concat` because `concat` is a [multi task][]) will concatenate the three specified source files (in order), joining files with `;` and writing the output to `dist/built.js`.

```js
// Project configuration.
grunt.initConfig({
  concat: {
    options: {
      separator: ';'
    },
    dist: {
      src: ['src/intro.js', 'src/project.js', 'src/outro.js'],
      dest: 'dist/built.js'
    }
  }
});
```

#### Banner comments

In this example, running `grunt concat:dist` will first strip any preexisting banner comment from the `src/project.js` file, then concatenate the result with a newly-generated banner comment, writing the output to `dist/built.js`.

This generated banner will be the contents of the `banner` template string interpolated with the config object. In this case, those properties are the values imported from the `package.json` file (which are available via the `pkg` config property) plus today's date.

_Note: you don't have to use an external JSON file. It's also valid to create the `pkg` object inline in the config. That being said, if you already have a JSON file, you might as well reference it._

```js
// Project configuration.
grunt.initConfig({
  pkg: grunt.file.readJSON('package.json'),
  concat: {
    options: {
      stripBanners: true,
      banner: '/*! <%= pkg.name %> - v<%= pkg.version %> - ' +
        '<%= grunt.template.today("yyyy-mm-dd") %> */'
    },
    dist: {
      src: ['src/project.js'],
      dest: 'dist/built.js'
    }
  }
});
```

#### Multiple targets

In this example, running `grunt concat` will build two separate files. One "basic" version, with the main file essentially just copied to `dist/basic.js`, and another "with_extras" concatenated version written to `dist/with_extras.js`.

While each concat target can be built individually by running `grunt concat:basic` or `grunt concat:extras`, running `grunt concat` will build all concat targets. This is because `concat` is a [multi task][].

```js
// Project configuration.
grunt.initConfig({
  concat: {
    basic: {
      src: ['src/main.js'],
      dest: 'dist/basic.js'
    },
    extras: {
      src: ['src/main.js', 'src/extras.js'],
      dest: 'dist/with_extras.js'
    }
  }
});
```

#### Multiple files per target

Like the previous example, in this example running `grunt concat` will build two separate files. One "basic" version, with the main file essentially just copied to `dist/basic.js`, and another "with_extras" concatenated version written to `dist/with_extras.js`.

This example differs in that both files are built under the same target.

Using the `files` object, you can have list any number of source-destination pairs.

```js
// Project configuration.
grunt.initConfig({
  concat: {
    basic_and_extras: {
      files: {
        'dist/basic.js': ['src/main.js'],
        'dist/with_extras.js': ['src/main.js', 'src/extras.js']
      }
    }
  }
});
```

#### Dynamic filenames

Filenames can be generated dynamically by using `<%= %>` delimited underscore templates as filenames.

In this example, running `grunt concat:dist` generates a destination file whose name is generated from the `name` and `version` properties of the referenced `package.json` file (via the `pkg` config property).

```js
// Project configuration.
grunt.initConfig({
  pkg: grunt.file.readJSON('package.json'),
  concat: {
    dist: {
      src: ['src/main.js'],
      dest: 'dist/<%= pkg.name %>-<%= pkg.version %>.js'
    }
  }
});
```

#### Advanced dynamic filenames

In this more involved example, running `grunt concat` will build two separate files (because `concat` is a [multi task][]). The destination file paths will be expanded dynamically based on the specified templates, recursively if necessary.

For example, if the `package.json` file contained `{"name": "awesome", "version": "1.0.0"}`, the files `dist/awesome/1.0.0/basic.js` and `dist/awesome/1.0.0/with_extras.js` would be generated.

```javascript
// Project configuration.
grunt.initConfig({
  pkg: grunt.file.readJSON('package.json'),
  dirs: {
    src: 'src/files',
    dest: 'dist/<%= pkg.name %>/<%= pkg.version %>'
  },
  concat: {
    basic: {
      src: ['<%= dirs.src %>/main.js'],
      dest: '<%= dirs.dest %>/basic.js'
    },
    extras: {
      src: ['<%= dirs.src %>/main.js', '<%= dirs.src %>/extras.js'],
      dest: '<%= dirs.dest %>/with_extras.js'
    }
  }
});
```


## Release History

 * 2012-11-12   v0.1.1   Switch to this.file api internally.
 * 2012-10-02   v0.1.0   Work in progress, not yet officially released.

---

Task submitted by ["Cowboy" Ben Alman](http://benalman.com/)

*This file was generated on Wed Nov 28 2012 08:48:05.*
