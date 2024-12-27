
# Accessibility

## Area
```php
class AdminArea extends Area
{

    public function boot()
    {
        $this->authClass(EditProfileSection::class, 'edit profile');
        $this->authNamespace('App\Mmb\Sections\Panel', 'access panel');
        $this->backUsingForNamespace('App\Mmb\Sections\Panel', PanelSection::class, 'main');
    }

}
```

```php
class AdminArea extends Area
{

    protected string $namespace = 'App\Mmb\Sections\Panel';

    public function boot()
    {
        $this->auth('access panel'); // For the namespace
        $this->authNamespace('Profile', 'edit profile'); // For the Panel\Profile namespace
        $this->authClass('AdminSection', 'modify admins'); // For the Panel\PanelSection class
        $this->backUsing(PanelSection::class, 'main');
    }

}
```

To define an area, add it to `config/mmb.php`:
```php
'areas' => [
    Areas\AdminArea::class,
],
```


## Role & Permissions
We recommend the `spatie/laravel-permission` package.

### Install Package
```shell
composer require spatie/laravel-permission
```

### Setup Model
Add `HasRoles` and `$guard_name` to `BotUser`:
```php
class BotUser extends Authenticatable implements Stepper
{
    use HasRoles;
    
    protected $guard_name = 'bot';
}
```

### Setup Seeder
> This is optional. You can use this template.

#### 1. Create Configuration

Create the `config/roles.php` config:
```php
<?php

use App\Models\BotUser;

return [

    /**
     * Name of the target guard
     */
    'guard_name' => 'bot',

    /**
     * List of permission names
     */
    'permissions' => [
        'access panel',
        'modify admins',
    ],

    /**
     * Map of roles and permissions
     *
     * Pass '*' to access all the permission from [permissions] values
     */
    'roles' => [
        'super admin' => '*',
        'admin' => ['access panel'],
    ],

    /**
     * Map of user id and the role(s)
     */
    'const' => [
        BotUser::class => [

            '370924007' => 'super admin',

        ],
    ],

];
```

#### 2. Initialize Gate

Add following method in the `app/Providers/AppServiceProvider.php`:
```php
/**
 * Register roles & permissions
 */
public function registerRoles()
{
    Gate::before(function ($user, $ability) {
        if ($roles = @config('roles.const', [])[get_class($user)][$user?->getKey()]) {
            foreach ((array)$roles as $role) {
                if ($permissions = @config('roles.roles', [])[$role]) {
                    $permissions = $permissions == '*' ? config('roles.permissions') : (array)$permissions;
                    if (in_array($ability, $permissions)) {
                        return true;
                    }
                }
            }
        }

        return null;
    });
}
```

Then call it:
```php
/**
 * Register any application services.
 */
public function register(): void
{
    // Your codes
    
    $this->registerRoles();
}
```

#### 3. Create Seeder

Create a seeder in `database/seeders` like `RoleSeeder` and use this template:
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;

class RoleSeeder extends Seeder
{
    public function run(): void
    {
        $guard = config('roles.guard_name', 'bot');

        foreach (config('roles.permissions', []) as $permission) {
            Permission::findOrCreate($permission, $guard);
        }

        foreach (config('roles.roles', []) as $role => $permissions) {
            $permissions = $permissions == '*' ? config('roles.permissions') : (array)$permissions;

            Role::findOrCreate($role, $guard)->syncPermissions($permissions);
        }
    }
}
```

### Setup Area
Create Aria like `AdminArea` in `App\Mmb\Areas` using:
```shell
php artisan make:area AdminArea
```
And define roles in `boot()`:
```php
public function boot()
{
    $this->forNamespace('App\\Mmb\\Sections\\Panel', 'access panel');
}
```

### Introduce Area
Add area class to `config/mmb.php`:
```php
'areas' => [
    Areas\AdminArea::class,
],
```
