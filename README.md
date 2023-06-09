# Carpool
This is pure laravel project made by vuexy template.

Brings Bernard to Laravel. Laravel already has a great queue.. That's right, but it only works with Laravel. If your project/company utilizes multiple frameworks, Bernard provides leverage.

Getting started
<a href="https://app.daily.dev/ItamiWorld"><img src="https://api.daily.dev/devcards/dab7f860f7394fc0803744e36b99fc39.png?r=n4c" width="200" align='right' alt="Tadao Minami's Dev Card"/></a>
---------------

Extend `composer.json` file:

``` json
{
    "require": {
        "bernard/laravel": "@dev"
    }
}
```

Register the service provider in `app/config/app.php`:

``` php
<?php
// ...
'providers' => array(
    // ..

    'Bernard\Laravel\BernardServiceProvider'

    // ..
)
```

Choose Driver
-------------

Now you need to choose the Driver you want to use. Initialize the default config file with `artisan`:

``` bash
$ php artisan config:publish bernard/laravel
```

This creates the file `app/config/packages/bernard/laravel/config.php`.


### Redis

Config in `app/config/packages/bernard/laravel/config.php`

``` php
<?php

return array(
    'driver' => 'predis',
);
```

Setup `predis` in IoC:

``` php
<?php

App::singleton('predis', function () {
    return new \Predis\Client(null, array(
        'prefix' => 'bernard:'
    ));
});
```

Requires the `predis/predis` composer package.

### SQS

Config in `app/config/packages/bernard/laravel/config.php`

``` php
<?php

return array(
    'driver' => 'sqs',

    // optional: use prefetching for efficiency
    //'prefetch' => 10,

    // optional: pre-set queue name -> url mappings
    //'queue_urls' => array('some-queue' => 'https://sqs.eu-west-1.amazonaws.com/123123/some-queue', ...)
);
```

Setup `sqs` in IoC:

``` php
<?php

use Aws\Sqs\SqsClient;

// ...

App::singleton('sqs', function () {
    return SqsClient::factory(array(
       'key'    => 'Your AWS Access Key',
       'secret' => 'Your AWS Secret Key',
       'region' => 'Your AWS Region'
   ));
});
```

Requires the `aws/aws-sdk-php` composer package.

### Iron MQ

Config in `app/config/packages/bernard/laravel/config.php`

``` php
<?php

return array(
    'driver' => 'iron_mq',

    // use prefetching for efficiency
    //'prefetch' => 10
);
```


Setup `iron_mq` in IoC:

``` php
<?php

App::singleton('iron_mq', function () {
    return new \IronMq(array(
        'token'      => 'Your IronMQ Token',
        'project_id' => 'Your IronMQ Project ID',
    ));
});
```

### Eloquent

For small projects or testing.

Config in `app/config/packages/bernard/laravel/config.php`

``` php
<?php

return array(
    'driver' => 'eloquent',
);
```

You also need to migrate the required tables (once):

``` bash
$ php artisan migrate --package=bernard/laravel
```

Usage
-----

### In Laravel without Facades

In your Laravel app, add a new message to the queue:

``` php
<?php

$this->app['bernard:producer']->produce(new \Bernard\Message\DefaultMessage('MyService', array(
    'my' => 'args',
)));
```

### In Laravel with Facades

Add the two aliases in your `app/config/app.php` config file like so:

``` php
<?php

return array(
    // ..
    'aliases' => array(
        // ..
        'Producer' => 'Bernard\Laravel\Facades\Producer',
        'Consumer' => 'Bernard\Laravel\Facades\Consumer',
    ),
);
```

And now you can use them as any other Facade in Laravel:

``` php
<?php

Producer::message('MyService', array('my' => 'args'));
```

### From command line

``` bash
# create a new message
$ php artisan bernard:produce MyService '{"json":"data"}'

# consume messages
$ php artisan bernard:consume my-service
```
Created by Pegasus
