## Sentinel: Sentry Implementation for Laravel 

[![Build Status](https://travis-ci.org/rydurham/Sentinel.svg?branch=master)](https://travis-ci.org/rydurham/Sentinel)
[![Total Downloads](https://poser.pugx.org/rydurham/sentinel/downloads.svg)](https://packagist.org/packages/rydurham/sentinel)
[![License](https://poser.pugx.org/rydurham/sentinel/license.svg)](https://packagist.org/packages/rydurham/sentinel)

This package provides an implementation of  [Sentry 2](https://github.com/cartalyst/sentry) for [Laravel](https://github.com/laravel/laravel). By default it uses [Bootstrap 3.0](http://getbootstrap.com), but you can make use of whatever UI you want.  It is intended to be a very simple way to get up and running with User access control very quickly.  For simple projects you shouldn't need to do much more than drop it in and dial in the configuration.

Make sure you use the version most appropriate for the type of Laravel application you have: 

| Laravel Version  | Sentinel Version  | Packagist Branch |
|---|---|---|
| 4.2.*  | 1.4.*  | ```"rydurham/sentinel": "~1.4"``` |
| 5.0.*  | 2.0.*  | ```"rydurham/sentinel": "~2"```   |


### Laravel 5 Instructions
**Install the Package Via Composer:**

```shell
$ composer require rydurham/sentinel
```

Make sure you have configured your application's Database and Mail settings. 

**Add the Service Provider to your ```config/app.php``` file:**

```php
'providers' => array(
    ...
    'Sentinel\SentinelServiceProvider', 
    ...
)
```  

**Register the Middleware in your ```app/Http/Kernel.php``` file:**

```php
protected $routeMiddleware = [
    // ..
    'sentry.auth' => 'Sentinel\Middleware\SentryAuth',
    'sentry.admin' => 'Sentinel\Middleware\SentryAdminAccess',
];
```	

**Publish the Views, Assets, Config files and migrations:**
```shell
php artisan sentinel:publish
```

You can specify a "theme" option to publish the views and assets for a specific theme:  
```shell
php artisan sentinel:publish --theme="foundation"
```
Run ```php artisan sentinel:publish --list``` to see the currently available themes.

**Run the Migrations**
Be sure to set the appropriate DB connection details in your  ```.env``` file.  

Note that you may want to remove the ```create_users_table``` and ```create_password_resets_table``` migrations that are provided with a new Laravel 5 application. 

```shell
php artisan migrate
```

**Seed the Database:** 
```shell
php artisan db:seed --class=SentinelDatabaseSeeder
```
More details about the default usernames and passwords can be [found here](https://github.com/rydurham/Sentinel/wiki/Seeds).

**Set a "Home" Route.**  

Sentinel requires that you have a route named 'home' in your ```routes.php``` file: 
```php
// app/routes.php
 Route::get('/', array('as' => 'home', function()
{
    return View::make('home');
}));
```

### Basic Usage
Once installed and seeded, you can make immediate use of the package via these routes:
* ```yoursite.com/login``` 
* ```yoursite.com/logout``` 
* ```yoursite.com/register``` 
* ```yoursite.com/users``` - For user management.  Only available to admins
* ```yoursite.com/groups``` - For group management. Only available to admins.

Sentinel also provides these middlewares which you can use to [prevent unauthorized access](http://laravel.com/docs/routing#route-filters) to your application's routes & methods. 

* ```Sentinel\Middleware\SentryAuth``` - Require users to have an active session
* ```Sentinel\Middleware\SentryAdminAccess``` - Block access for everyone except users who have the 'admin' permission.  

### Advanced Usage
This package is intended for simple applications, but it is possible to integrate it into a large application on a deeper level:
* Turn off the default routes (via the config) and manually specify routes that make more sense for your application
* Create a new User model that extends the default Sentinel User Model ```Sentinel\Models\User```.  Be sure to publish the Sentinel and Sentry config files (using the ```sentinel:publish``` command) and change the User Model setting in the Sentry config file to point to your new user model. 
* Inject the ```SentryUserRepository``` and/or the ```SentryGroupRepository``` classes into your controllers to have direct access to user and group manipulation.  You may also consider creating custom repositories that extend the repositories that come with Sentinel. 

It is not advisable to extend the Sentinel Controller classes; you will be better off in the long run creating your own controllers from scratch. 

#### Using Sentinel in Tests
If you find yourself in the situation whereby you need to do tests with user logged in, then go to your ``` tests/TestCase.php `` and add below method.
```php
   /**
     * Login to sentry for Testing purpose
     * @param  $email
     * @return void
     */
    public function sentryUserBe($email='admin@admin.com')
    {
        $user = \Sentry::findUserByLogin($email);
        \Sentry::login($user);
       \Event::fire('sentinel.user.login', ['user' => $user]);
    }
```
After adding the above method, you can start testing your application with user logged in like below example
```php
class ExampleTest extends TestCase
{
    /**
     * Dashboard functional test example.
     *
     * @return void
     */
    public function testDashboardPage()
    {
        $this->sentryUserBe('admin@admin.com');
        $this->visit('/dashboard')
             ->see('dashboard;
    }
}
```


### Documentation & Questions
Check the [Wiki](https://github.com/rydurham/Sentinel/wiki) for more information about the package:
* Config Options
* Events & Listeners
* Seed & Migration Details
* Default Routes

Any questions about this package should be posted [on the package website](http://www.ryandurham.com/projects/sentinel/).

### Localization
Sentinel has been translated into several other languages, and new translations are always welcome! Check out the [Sentinel Page](https://crowdin.com/project/sentinel) on CrowdIn for more details.

### Tests
Tests are powered by Codeception.  Currently they must be run within a Laravel application environment.   To run the tests: 
* Pull in the require-dev dependencies via composer. 
* Navigate to the Sentinel folder
* Run ```vendor/bin/codecept run```

I would recommend turning on "Mail Pretend" in your testing mail config file.
