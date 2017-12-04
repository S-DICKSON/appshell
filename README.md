# Konekt AppShell

Konekt AppShell is a [Concord box](https://github.com/artkonekt/concord/blob/master/docs/boxes.md) (thus a Laravel Extension) that serves as a foundation for Laravel business applications.

Incorporates the basics for:

- Users + profiles
- User impersonation
- Authentication, authorization (acl)
- Locations (countries, provinces, addresses)
- Customers
- Extensible Admin Interface
- Menu handling

The user/auth part is built on top of the Laravel facilities in a compatible manner.

## Create New AppShell Project

```bash
composer create-project laravel/laravel myapp
# Wait 1-4 minutes to complete ...
cd myapp
composer config minimum-stability dev
composer config prefer-stable true
composer require --prefer-source konekt/appshell:^0.9.1
touch config/concord.php
```

Edit `config/concord.php` and add this content to it:

```php
<?php

return [
    'modules' => [
        Konekt\AppShell\Providers\ModuleServiceProvider::class
    ]
];
```

Edit `config/app.php` and add this line to the `providers` array (below 'Package Service Providers', always above 'Application Service Providers')

(_Below tinker, as of v5.4_)

```php
Konekt\Concord\ConcordServiceProvider::class,
```

Test if it works by invoking the command

```bash
php artisan concord:modules
```

Now you should see this:

```
+----+---------------------+------+---------+------------------+-----------------+
| #  | Name                | Kind | Version | Id               | Namespace       |
+----+---------------------+------+---------+------------------+-----------------+
| 1. | Konekt AppShell Box | Box  | 0.9.1   | konekt.app_shell | Konekt\AppShell |
+----+---------------------+------+---------+------------------+-----------------+
```

> **TIP:** Try `php artisan concord:modules` to see ALL modules

Configure `.env`, along with a database.

Afterwards run the migrations:

```bash
php artisan migrate
```

AppShell contains ~10-15 migrations out of the box

> If running with linux + apache/nginx these commands are useful:
>
> `sudo chown -R .www-data storage/`
>
> `sudo chmod -R g+w storage/`

## Laravel Auth Support

First, Run `php artisan make:auth`

If the "final" user class is not going to be `App\User` then don't forget to modify model class this to your app's `config/auth.php` file:
```php
    //...
    'providers' => [
        'users' => [
            'driver' => 'eloquent',
            // 'model' => App\User::class <- change this to:
            'model' => Konekt\AppShell\Models\User::class,
        ],
    //...
```
**OR:**
Another approach is to keep `App\User` but modify the class to extend AppShell's user model:

```php
namespace App;

// No need to use Laravel default traits and properties as
// they're already present in the base class

class User extends \Konekt\AppShell\Models\User {}
```

And add this to you `AppServiceProviders`'s boot method:

```php
   $this->app->concord->registerModel(\Konekt\User\Contracts\User::class, \App\User::class);
```

### Create An Initial Super User

Run command `php artisan appshell:super`.

This will ask a several questions and create a proper superuser that you can start working with.


## Use AppShell's Admin Layout

Change the layout in the first line of `resources/views/home.blade.php` file to be:

```blade
@extends('appshell::layouts.default')
```

### Add AppShell CSS To Laravel Mix

In `webpack.mix.js` change:
```js
mix.js('resources/assets/js/app.js', 'public/js')
   // Add this line:
   .scripts([
           'public/js/app.js',
           'vendor/konekt/appshell/src/resources/assets/js/appshell.js'
       ], 'public/js/app.js')
   // And replace this line:
   //.sass('resources/assets/sass/app.scss', 'public/css');
   // With this one:
    .sass('vendor/konekt/appshell/src/resources/assets/sass/appshell.sass', 'public/css');
```

Remove the omnipresent Vue instance from Laravel's default app.js file:

`resources/assets/js/app.js`:

```javascript
/**
 * First we will load all of this project's JavaScript dependencies which
 * includes Vue and other libraries. It is a great starting point when
 * building robust, powerful web applications using Vue and Laravel.
 */

// REMOVE/COMMENT THIS LINE:
require('./bootstrap');
// ADD THIS LINE (jquery is needed):
window.$ = window.jQuery = require('jquery');

window.Vue = require('vue');

/**
 * Next, we will create a fresh Vue application instance and attach it to
 * the page. Then, you may begin adding components to this application
 * or customize the JavaScript scaffolding to fit your unique needs.
 */

Vue.component('example-component', require('./components/ExampleComponent.vue'));

// REMOVE/COMMENT THESE 3 LINES:
const app = new Vue({
    el: '#app'
});
```

and the compile the assets with mix: `yarn run dev`

> **TIP:** You may need to [install yarn](https://yarnpkg.com/en/docs/install)
> and run:
```bash
yarn install
```


### Configure PhpStorm For Properly Editing Sources In Vendor

You can set up PhpStorm to directly edit the source files in vendor, and have full IDE support.

#### Source Folders
First mark these folders as source folders, in PhpStorm's **Settings->Directories**.

![Source Folders](docs/phpstorm-source-folders.jpg)

Also add their namespace prefixes by clicking that small 'p' with the caret, and ticking [] for generated sources. Refer the Package prefix (namespace) on the screenshot above.

#### Git, Our Friend

Go to PhpStorm->**Settings->Version Control** and add the package folders you want to modify locally and push back to the repo:

![Source Folders](docs/phpstorm-vcs-folders.jpg)


## Built-in Facilities

### Menu

The menu functionality is built on top of [Konek Menu Component](https://github.com/artkonekt/menu). The component is automatically loaded, is fully available (incl. the `Menu` facade).

AppShell creates a menu named **appshellMenu** which is the main menu component, and is available in views as `$appshellMenu`.

