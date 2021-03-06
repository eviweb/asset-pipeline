## What the heck is Asset Pipeline?

#### Watch an [introduction video](http://youtu.be/1iDB5BFsTw8) on the asset pipeline.

[![Introduction Video](http://i810.photobucket.com/albums/zz28/kelt4/youtube-assetpipeline_zpsd864d5d6.png)](http://youtu.be/1iDB5BFsTw8)


For those of you familar with Rails asset pipeline and sprockets, you will hopefully feel right at home using this package.

In a nutshell, this is what you will get from this package.

 - Update your layouts to use the asset pipeline.
 - Put your less/css/scss/javascript/coffeescript/jst/image/font/etc files into app/assets
 - Smile and let asset-pipeline automatically handle the compilation, concatenation, minification and even caching.

## Installation

Begin by installing this package through Composer. Edit your project's `composer.json` file to require `codesleeve/asset-pipeline`.

It might look something like:

```php
  "require": {
    "laravel/framework": "4.0.*",
    "codesleeve/asset-pipeline": "dev-master"
  }
```

Next, update Composer from the Terminal:

```php
    composer update
```

Once this operation completes, add the service provider. Open `app/config/app.php`, and add a new item to the providers array.

```php
    'Codesleeve\AssetPipeline\AssetPipelineServiceProvider'
```

Also, we need to make sure your environment is setup correctly because the asset pipeline caches and minfies assets on a production environment.

Inside `bootstrap/start.php`

```php
  $env = $app->detectEnvironment(array(
    'local' => array('your-machine-name'),
  ));
```

Run the `artisan` command from the Terminal for the `assets:generate` command.

```php
    php artisan assets:generate
```

Lastly, it is recommended to create a custom package config for [configuration of the asset pipeline.](#configuration)

```php
  php artisan config:publish codesleeve/asset-pipeline
```

## Usage

After running `php artisan assets:generate` you should notice two new directories with some files, `app/assets` and `vendor/assets`.

Be sure to check out the `app/assets/javascripts/application.js` and `app/assets/stylesheets/application.css` files. These are the default manifest files where you can adjust the files you want included into your manifest.

Let's verify you have everything working by going to:

    http://<your laravel project>/assets/application.js
    
Or for styles:

    http://<your laravel project>/assets/application.css

Now to bring everything in, (this is exactly how rails does it too) you should add to your layout

    <?= stylesheet_link_tag() ?>
    <?= javascript_include_tag() ?>

If you want to use different manifest files, these helper `tag` functions also take two parameters. 

    <?= stylesheet_link_tag($manifestFile = 'application', $attributes = array()) ?>
    <?= javascript_include_tag($manifestFile = 'application', $attributes = array()) ?>

So something like this...

```php
  <?= javascript_include_tag('frontend/application', ['data-requires' => 'foobar']); ?>
```

Would process the directives inside of app/assets/frontend/application.js and (on production environment) generate

```html
  <script src="assets/frontend/application.js" data-requires="foobar"></script>
```

### How Do I Compile My Coffee and Less, Man?

@felixkiss does a pretty job explaining how to get started with pre-compliation [here](https://github.com/CodeSleeve/asset-pipeline/issues/23#issuecomment-23089175).

Since the default manifest files that get generated by `php artisan assets:generate` have the directive 

```js
   require_tree .
```

then it is just a matter of placing a file with an extension of `.css.less` in the directory

```
   app/assets/stylesheets
```

**NOTE** that if your environment is `production` then it will [cache](#how-does-caching-work) your assets for you, so check your Laravel `App::environment()` if you're refreshing the page and not seeing any change.

### Available Directives?

A manifest file is where you put your sprocket's directives at. Here is a list of directives.

  - **require** filename
 
    This brings in a specific asset file within your path. You don't have to include the extension either, it will guess based on what extensions are in your `$filters` array inside of  `codesleeve/asset-pipeline/config.php`

  - **require_directory** some/directory
 
    This brings in assets only within some/directory (non-recurisve). You can also use '.' here which will resolve to the path that the manifest file is contained within.

  - **require_tree** some/directory

    This is almost like require_directory except it is recursive.

  - **require_self**

    This brings in the manifest file itself as an asset. This is already done on `require_tree .` if the manifest file is within that directory. Where you might want to use this is when you have a manifest file that does like `require_tree subdir/`

It is even possible to register your own custom directives or override existing ones. [More about that here.](#more-on-directives)

### Javascript Templates?

You could stick all your handlebar/eco/underscore/etc templates insde of the Laravel view but that adds up quickly. Any .html files you place within `app/assets/javascripts` will be accessible in a global javascript variable called JST. Just open up your javascript console and examine the `JST` object. Read more about [filters](#filters) below if you want your own custom filter (besides just .html files).

### Images? Fonts? Other files?

Place your fonts and images inside of the `app/assets/fonts` and `app/assets/images` folders. Alternatively you can place them inside of `app/assets/javascripts` or `app/assets/stylesheets` if that tickles your fancy. 

For you rails fans, can create image tags in your view using  

```php
  <?= image_tag('awesome/filename.png', ['alt' => 'The alt for the image', 'class' => 'img-responsive']) ?>
```

Assuming a file existed (e.g. `app/assets/stylesheets/awesome/filename.png`), this would return

```html
  <img src="assets/awesome/filename.png" alt="The alt for the image" class="img-response">
```


## Configuration

If you would like to configure the asset pipeline there are a few things you can change.

First run the command:

```php
  php artisan config:publish codesleeve/asset-pipeline
```

This will create a config file in your `app/config/packages/` directory. You can edit this config file without having to worry about composer overriding your changes.

Next open the config found in `app/config/packages/codesleeve/asset-pipeline/config.php` and you will see the following values.

### Routing 

If you want to point your application somewhere besides `http://<laravel project dir>/assets/javascripts.js` you can change this

```php
  'routing' => array(
    'prefix' => '/assets'
  ),
```

#### Filtering

You can place any kind of assets (fonts, images, mp3s) inside of the `app/assets` directory. This opens up the oppertunity for using Laravel route filters for your assets.

For example, say you don't want guest users looking at your assets, then you can add this to the config file.

```php
  'routing' => array(
    'prefix' => '/assets',
    'before' => 'auth',
  ),
```

The auth filter comes in Laravel 4 by default in the `filters.php` file but I'll put it here just in case you wanted to see it.

```php
  Route::filter('auth', function()
  {
    if (Auth::guest()) return Redirect::guest('login');
  });
```

Simple huh?

### Paths
These are the directories we search for files in relative to your laravel root base directory `base_path()`

```php
  'paths' => array(
    'app/assets/fonts',
    'app/assets/images',
    'app/assets/javascripts',
    'app/assets/stylesheets',
    'lib/assets/fonts',
    'lib/assets/images',
    'lib/assets/javascripts',
    'lib/assets/stylesheets',
    'vendor/assets/fonts',
    'vendor/assets/images',
    'vendor/assets/javascripts',
    'vendor/assets/stylesheets'
  ),
```

How is this used? Say you search for `/assets/foobar.js`, the asset pipeline will search for (in order) any path that has 'javascripts' in it.

So you might finally find '/assets/foobar.js` inside of `vendor/assets/javascripts/foobar.js`

But what if you had a path that did not have the string 'javascripts' in it? It is treated as an `other` resource. To get around this, you can use the word `javascripts` to tell asset pipeline that the resources in this directory are javascripts. The same applies for stylesheets.

```php
  'paths' => array(
      ... code omitted ...,
    'app/some/other/directory' => 'javascripts',
    'app/directory/with/style' => 'stylesheets',
      ... over even both ...,
    'app/some/mixed/directory' => 'javascripts,stylesheets',
  ),
```

You can also dynamically register your own paths (in say... your own Laravel package) to bring in your own additional assets.

```php
  Event::listen('assets.register.paths', function($paths) {
    $paths->add('another/custom/library/js', 'javascripts');
    $paths->add('another/custom/library/css', 'stylesheets');
  });
```

####[Watch me creating a handlebars package](http://youtu.be/IPgUUYb7SqU) for the asset pipeline.

### Filters

These filters are what determine 

  1) if we should consider the file in a manifest and 
  2) how to filter files of this extension type.

```php
  'filters' => array(
    '.min.js' => array(
      // don't minify files with this extension
    ),
    '.min.css' => array(
      // don't minify files with this extension
    ),
    '.js' => array(
      new Codesleeve\AssetPipeline\Filters\MinifyJS('production')
    ),
    '.css' => array(
      new Codesleeve\AssetPipeline\Filters\MinifyCSS('production')
    ),
    '.js.coffee' => array(
      new Codesleeve\AssetPipeline\Filters\CoffeeScriptFilter,
      new Codesleeve\AssetPipeline\Filters\MinifyJS('production')
    ),
    '.css.less' => array(
      new Assetic\Filter\LessphpFilter,
      new Codesleeve\AssetPipeline\Filters\MinifyCSS('production')
    ),
    '.css.scss' => array(
      new Assetic\Filter\ScssphpFilter,
      new Codesleeve\AssetPipeline\Filters\MinifyCSS('production')
    ),
    '.html' => array(
      new Codesleeve\AssetPipeline\Filters\JSTFilter,
      new Codesleeve\AssetPipeline\Filters\MinifyJS('production')
    )
  ),
```

You can even add your own, if you want to write your own filter or even change existing filters out.

```php
  'filters' => array(

    ... code omited ...

    '.jst.hbs' => array(
      new HandlebarsFilter
    )
  ),
```

Just like `paths` you can hook into asset pipeline's filters via Laravel's event system. This allows you to dynamically hook into filters (in say a seperate composer package or somewhere in your application).

Let's say we wanted to start filtering file.jst.hbs in the asset pipeline. (I'm making a video and package for this as well).

```php
  Event::listen('assets.register.filters', function($filters) {
    $filters->add('.jst.hbs', array(
      new HandlebarsFilter
    ));
  });
```

So what does a `HandlebarsFilter` look like? Filters are all handled via [Assetic](https://github.com/kriswallsmith/assetic/) so check them out to learn more. Basically though, it is just a class that implements `Assetic\Filter\FilterInterface`. Check out many of the [Assetic filters available](https://github.com/kriswallsmith/assetic/tree/master/src/Assetic/Filter) but be warned that some require external executables.


```php
  use Assetic\Asset\AssetInterface;
  use Assetic\Filter\FilterInterface;

  class HandlebarsFilter implements FilterInterface
  {
    public function filterLoad(AssetInterface $asset)
    {

    }
 
    public function filterDump(AssetInterface $asset)
    {
      $content = $asset->getContent();

      // do something with $content... 

      $asset->setContent($content);
    }
}
```

Check out [AssetInfterface](https://github.com/kriswallsmith/assetic/blob/master/src/Assetic/Asset/AssetInterface.php) for the list of functions you can do. I use `getSourcePath` and `getSourceDirectory` in some of my filters.


### Concat

If you want to turn on/off concatenation (for debugging perhaps?) you can do that easily here.

Leaving the value as null will fallback to the machine's environment to determine if assets are concatenated all within
the manifest file. Probably best to leave this alone if you're unsure.

```php
  'cache' => null,
```

## Cache

In the asset pipeline, assets can be cached for performance gains which is particularly helpful on production environments.

Leaving the value as null will fallback to the machine's environment to determine if assets are cached.

```php
  'cache' => null,
```

At anytime you can clear the cache by running

```php
  php artisan assets:clean
```

#### NOTE: Anytime you make a change to this config value you need to clear the cache!

## Creating Packages For Asset Pipeline

It is possible to hook into asset pipeline to dynamically add your own asset paths and filters. 

You might be wondering, what should I call this composer package? The convention I will follow is prefixing the package name with `l4-asset`. 

This way, it will be easy to search and find asset pipeline add-ons on [packagist.org](http://packagist.org). For example, if I was creating a handlebars add-on (I actually am going to create this later, will update with link) then I would name the package

   `codesleeve/l4-asset-handlebars`

Then in my package I could do something similar to this:

```php
   Event::listen('assets.register.filters', function($filters) {
      $filters->add('.jst.hbs', array(new Codesleeve\L4AssetHandlebars\Filters\HandlebarsFilter));
   });
```
and 

```php
   Event::listen('assets.register.paths', function($paths) {
      $paths->add(__DIR__ . '../../assets/javascripts', 'javascripts');
   });
```

and then finally in my `app/assets/javascripts/application.js` manifest file I could do 

```js
   //= require handlebars`
```

and any files with `.jst.hbs` extension would start showing up in my `JST` array as a compiled Handlebars template.

** TODO Make a video showing how to make your l4-asset-handlebars package including handlebars assets and .jst.hbs filter **

## More on Directives

Using laravel's event handlers we can override existing and create custom directives. We can use this to add in our own files for any given directive.

Here is an example of overriding the `require jquery` directive

```php
   Event::listen('assets.register.directive', function($directive) {
      if (!$directive->isJavascriptManifest()) {
         return;
      }
      
      if ($directive->name == 'require' && $directive->param == 'jquery') {
            $directive->add('jquery.js');
      }
   });
```

Or if I wanted to be crazy, I could even create my own custom stylesheet directive ... called `awesome_directive`

```php
   Event::listen('assets.register.directive', function($directive) {
      if (!$directive->isStylesheetManifest()) {
         return;
      }
      
      if ($directive->name == 'awesome_directive') {
         $directive->add('awesome/stuff1.js');
		       $directive->add('awesome/stuff2.js');
		       $directive->add('awesome/stuff3.js');
      }
   });
```

and in my application.css I could do

```css
   /**
    *= awesome_directive
    */
```

so this allows us to create whatever kind of directives we need for our application. However, custom directives should probably be rare.

## FAQ

### What about conditional includes?

You can go about conditional includes in several ways. Let me show you how I do them

I put this html tag inside of my layout(s) which in turn, tells me what laravel action I am looking at.

    <html lang="en" class="<?= $currentRoute ?>">

And in my `app/controllers/BaseController.php` I might have something like

```php

  protected function setupLayout()
  {
    if ( ! is_null($this->layout))
    {
      $this->layout = View::make($this->layout);
      View::share('currentRoute', $this->currentRoute());
    }
  }

  protected function currentRoute()
  {
    $controller = explode('@', Route::currentRouteAction())[0];
    $controller = strtolower(str_replace('Controller', '', $controller));
    $action = strtolower(explode('@', Route::currentRouteAction())[1]);

    return "$controller $action";
  }
```

And then inside of `app/assets/stylesheets/login.css.less`, assuming the route action was something like: `UsersController@login`

```css
   html.users.login {
      body {
         background-color: #abcdef;
      }
   }
```

For javascripts you can  do custom loads for specfic laravel views or check for the existence of that element

```js
   if ($('html.users.login').length) {
      // ... load things specific to the users.login page ...
   }

```

### How does caching work?

Out of the box, all script and stylesheet files are cached in production environment. However, you can always change this via config options. The cache will be built on the first time it is requested from the server. It lives forever, until the server admin runs a `php artisan assets:clean` to clear the cache. It is using Laravels' Cache facade, which uses the `file` driver out of the box, but you
can make it use `memory` or `redis` or whatever you fancy.

In case you skipped past this in the [installation](#installation) part I'll mention it again.

_Make sure your environment is setup correctly because the asset pipeline caches and minfies assets on a production environment_.

Inside `bootstrap/start.php`

```php
  $env = $app->detectEnvironment(array(
    'local' => array('your-machine-name'),
  ));
```

### How does asset pipeline work in development vs. production?

This is a pretty common question, especially for those who have never used rails asset pipeline. Let's look at an example.

In development the `stylesheet_link_tag()` will create a `link` tag for each asset you are including via asset pipeline manifest file `application.css`.

```html
   <link href="assets/awesome.css" rel="stylesheet" type="text/css">
   <link href="assets/some/other/asset.css" rel="stylesheet" type="text/css">
   ...
   <link href="assets/application.css" rel="stylesheet" type="text/css">
```

but on production all these files would be concatenated under `application.css` and you would just have

```html
   <link href="assets/application.css" rel="stylesheet" type="text/css">
```

### Want to watch some videos?
  - [Introduction To Asset Pipeline](http://youtu.be/1iDB5BFsTw8)
  - [Handlebars Asset Package](http://youtu.be/IPgUUYb7SqU)

## License

The codesleeve asset pipeline is open-source software licensed under the [MIT license](http://opensource.org/licenses/MIT)

## Support

Hopefully, you're satified with the pipeline. [You will be.](http://www.youtube.com/watch?v=I54wGOSg_BQ). But if you do see an problem then please file an issue.

Also, I do accept pull-requests for bug fixes and please place in issues

We use Travis CI for testing which you can see at: https://travis-ci.org/CodeSleeve/asset-pipeline

*Again*, enjoy! And have a nice day!
