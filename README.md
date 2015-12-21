## Laravel EnvTenant 2.2.*

*Version 2.2.\* Changes:*

- Removed generic listener
- Removed public setDefaultConnection method
- Changed resolver registration to be app('tenant')
- Added TenantContract to enable custom models
- Updated documentation

The Laravel 5.2 EnvTenant package enables you to easily add multi-tenant capabilities to your application.
This package is designed using a minimalist approach providing just the essentials - no views, routes,
or configs. Just drop it in, run the migration, and start adding tenants. Your applications will
have access to current tenant information through the dynamically set config('tenant') values.
Optionally, you can let applications reconnect to the default master database so a tenant
could manage all tenant other accounts for example. And, perhaps the best part, Artisan
is completely multi-tenant aware! Just add the --tenant option to any command to
run that command on one or all tenants. Works on migrations, queueing, etc.!

Laravel EnvTenant was originally forked from the Laravel Tenantable project by @leemason
Lee is to be credited with doing a lot of the hard work to figure out how to globally
add the --tenant option to Artisan and for inspiration for the idea. Where this
project differs is in it's approach to managing database connection settings.
Tenantable stores settings in the database and offers unlimited domains.
EnvTenant relies on your ENV and Database config and stores just the
conneciton name in the table and only allows one subdomain and
domain per tenant, which is most often plenty for most apps.
EnvTenant also throws TenantNotResolvedException when
tenants are not found, which you can catch.


## Simple Installation & Usage

Composer install:

```
composer require thinksaydo/envtenant:2.1.*
```

Then run composer dump-autoload.

Service provider install:

```php
ThinkSayDo\EnvTenant\TenantServiceProvider::class,
```

Tenants database table install:

```php 
artisan migrate --path /vendor/thinksaydo/envtenant/migrations
```

Tenant creation (just uses a standard Eloquent model):

```php
$tenant = new \ThinkSayDo\EnvTenant\Tenant();
$tenant->name = 'ACME Inc.';
$tenant->email = 'person@acmeinc.com';
$tenant->subdomain = 'acme';
$tenant->alias_domain = 'acmeinc.com';
$tenant->connection = 'db1';
$tenant->meta = ['phone' => '123-123-1234'];
$tenant->save();
```

And you're done! Minimalist, simple. Whenever your app is visited via http://acme.domain.com or http://acmeinc.com
the default database connection will be set to "db1", the table prefix will switch to "acme_", and config('tenant')
will be set with tenant details allowing you to access values from your views or application.


## Advanced EnvTenant Usage

### Artisan

```php
// migrate master database tables
php artisan migrate

// migrate specific tenant database tables
php artisan migrate --tenant=acme

// migrate all tenant database tables
php artisan migrate --tenant=*
```

The --tenant option works on all Artisan commands.


### Tenant

The ```\ThinkSayDo\EnvTenant\Tenant``` class is a simple Eloquent model providing basic tenant settings.

```php
$tenant = new \ThinkSayDo\EnvTenant\Tenant();

// The unique name field identifies the tenant profile
$tenant->name = 'ACME Inc.';

// The non-unique email field lets you email tenants
$tenant->email = 'person@acmeinc.com';

// The unique subdomain field represents the subdomain portion of a domain and the database table prefix
$tenant->subdomain = 'acme';

// The unique alias_domain field represents an alternate full domain that can be used to access the tenant
$tenant->alias_domain = 'acmeinc.com';

// The non-unique connection field stores the Laravel database connection name
$tenant->connection = 'db1';

// The meta field is cast to an array and allows you to store any extra values you might need to know
$tenant->meta = ['phone' => '123-123-1234'];

$tenant->save();
```


### TenantResolver

The ```\ThinkSayDo\EnvTenant\TenantResolver``` class is responsible for resolving and managing the active tenant
during Web and Artisan access. You can access the resolver class using ```app('tenant')```.

```php
// get the resolver instance
$resolver = app('tenant');

// check if valid tenant
$resolver->isResolved();

// get the active tenant (returns Tenant model or null)
$tenant = $resolver->getActiveTenant();

// reconnect default connection enabling access to "tenants" table
$resolver->reconnectDefaultConnection();

// reconnect tenant connection disabling access to "tenants" table
$resolver->reconnectTenantConnection();
```

If you need even more flexibility, you can implement the ```ThinkSayDo\EnvTenant\Contracts\TenantContract```
interface on a custom model and inject that as the active tenant.

```php
// set the active tenant  to a
$activeTenant = new CustomTenant();
$resolver->setActiveTenant($activeTenant);
```


### Events

Throughout the lifecycle events are fired allowing you to listen and customize behavior.

Tenant activated:
```php
ThinkSayDo\EnvTenant\Events\TenantActivatedEvent
```

Tenant resolved:
```php
ThinkSayDo\EnvTenant\Events\TenantResolvedEvent
```

Tenant not resolved:
```php
ThinkSayDo\EnvTenant\Events\TenantNotResolvedEvent
```

Tenant not resolved via the Web, an exception is thrown:
```php
ThinkSayDo\EnvTenant\Events\TenantNotResolvedException
```


### Enjoy! Report issues or ideas.