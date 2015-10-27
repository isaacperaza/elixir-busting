# Elixir Busting

## Introduction

Browser cache busting solution that allows to use laravel-elixir version functionality for all projects/frameworks.

## Why I created this?

Main reason is that Laravel Elixir is awesome, and I want to use it in my projects. Laravel Elixir provides a clean, fluent API for defining basic Gulp tasks for your Laravel application. Elixir supports several common CSS and JavaScript pre-processors, and even testing tools.

But when you use a Laravel Elixir "version"  task, all versioned files created are placed inside a new folder called "build", that broke all resources relative paths.

## How use it?

First you need to install node, npm and gulp, please follow [Laravel website Documentation](http://laravel.com/docs/elixir).

Create a package.json file in your application root path as the following:

```javascript
{
  "devDependencies": {
    "gulp": "^3.8.8",
    "laravel-elixir": "^3.0.0",
    "elixir-busting": "^1.0.0"
  }
}
```


Now execute the following command (from same path where your package.json) in order to install nodejs packages:

```shell
npm install
```


Next, open and edit your gulpfile.js:

```javascript
// Load Laravel elixir
var Elixir = require('laravel-elixir');
// Load elixir busting, this append a new Elixir task called "busting"
require('elixir-busting');

// Override elixir configuration
var config = Elixir.config;

// Change your assets path
config.assetsPath = './public';

Elixir(function(mix) {
    // Instead used a mix.version() task, use a mix.busting() task
    mix.busting([
        // replace css files path with yours
        config.assetsPath + '/css/user.css',
        config.assetsPath + '/css/contact.css',

        // replace script files path with yours
        config.assetsPath + '/js/user.js',
        config.assetsPath + '/js/contact.js'
    ]);
});
```

Finally execute "gulp" command:

```shell
gulp
```

So now you will see an output like this:

```
public
└─ css
 └─ user.css
 └─ user-7ab1e8e50c.css
 └─ contact.css
 └─ contact-8df1e8e80x.css
└─ js
 └─ user.js
 └─ user-9ba1e9e15n.js
 └─ contact.js
 └─ contact-6df1e1h12b.js
└─ rev-manifest.json
  ```
  
Now you will have 5 more files, 2 .css versioned files, 2 .js versioned files and rev-manifest.json file. The versioned files has the same content as the original file, them are just a copy of the original file, but with a different name to avoid browser cache nightmares.

The rev-manifest.json file is a one-to-one relationship, between the original file name with versioned file name:

```json
{
  "css/user.css": "css/user-7ab1e8e50c.css",
  "css/contact.css": "css/contact-8df1e8e80x.css",
  "js/user.js": "js/user-9ba1e9e15n.js",
  "js/contact.js": "js/contact-6df1e1h12b.js"
}
```


Now you can use your new versioned file in your script tag, for example:

```html
<!-- Before -->
<script src="js/user.js"></script>

<!-- After -->
<script src="js/user-9ba1e9e15n.js"></script>
```

But that is totally a bad practice, because every time that you execute **gulp** command you will get a new file names, so you will need to change all script and style tags to point to the new versioned file, so that is not an good solution. Instead that we going to use an PHP elixir function, that will set the correct versioned file without effort :).

PHP elixir function should looks like this:

```php
/**
 * Get the path to a versioned Elixir file.
 *
 * @param  string  $file
 * @return string
 *
 * @throws \InvalidArgumentException
 */
function elixir($file)
{
    static $manifest = null;

    if (is_null($manifest)) {
        $manifest = json_decode(file_get_contents('public/rev-manifest.json'), true);
    }

    if (isset($manifest[$file])) {
        return '/'.$manifest[$file];
    }

    throw new InvalidArgumentException("File {$file} not defined in asset manifest.");
}
```

**Note**: Add PHP elixir function to your general php file, something like utils.php, functions.php or another else and be sure to include it in your php files where you want to use it.


Now lets request our resource using **elixir** function:

```html

<script src="<?php echo elixir('js/user.js'); ?>"></script>

```


So after php was intepreted you will see an output like this:

```html

<script src="js/user-9ba1e9e15n.js"></script>

```


As you can see, you requested the original resource file **js/user.js**, but you got a versioned file **js/user-9ba1e9e15n.js**, this is because PHP elixir function read the rev-manifest.json file to provide to you the versioned file name and avoid that you need to change the src attribute value after every gulp command.

After every code release, you will be sure that browser cache problems are gone.


Also if you don't want to run **gulp** command after every change that you did in original source file you can use this command:

```shell
gulp watch
```

The **gulp watch** command is waiting for any change in your sources files.


## Official Documentation

Basically that was the documentation for Elixir Busting, but if you want to know more about Laravel Elixir features please visit [Laravel website](http://laravel.com/docs/elixir).


### License

Same license as Laravel Elixir, is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT)


### Thanks to Laravel Elixir and Gulp developers

Thanks to Taylor Otwell and Jeffrey Way that created this awesome gulp wrapper and thanks so much to Gulp team (http://wearefractal.com) to create awesome javascript tool kit.

