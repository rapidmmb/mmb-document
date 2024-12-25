
# Accessibility

## Area
```php
class AdminArea extends Area
{

    public function boot()
    {
        $this->authClass(EditProfileSection::class, 'AccessPanel');
        $this->authNamespace('App\Mmb\Sections\Panel', 'AccessPanel');
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
        $this->auth('AccessPanel'); // For the namespace
        $this->authNamespace('Profile', 'AccessProfile'); // For the Panel\Profile namespace
        $this->authClass('PanelSection', 'AccessPanel2'); // For the Panel\PanelSection class
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
class BotUser extends Authenticatable implements Stepping
{
    use HasRoles;
    
    protected $guard_name = 'bot';
}
```

### Setup Seeder
> This is optional. You can use this template to be faster.

Create a seeder in `database/seeders` like `RoleSeeder` and use this template:
```php
<?php

namespace Database\Seeders;

use App\Models\BotUser;
use Illuminate\Database\Seeder;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;

class RoleSeeder extends Seeder
{

    public string $guardName = 'bot';

    public array $permissions = [
        'AccessPanel',
        'ModifyAdmins',
    ];

    public array $roles = [
        'SuperAdmin' => '*',
        'Admin' => ['AccessAdmin'],
    ];

    public array $fixedUsers = [
        '370924007' => 'SuperAdmin',
    ];

    public function run(): void
    {
        foreach ($this->permissions as $permission) {
            Permission::findOrCreate($permission, $this->guardName);
        }

        foreach ($this->roles as $role => $permissions) {
            if ($permissions == '*') $permissions = $this->permissions;
            elseif (is_string($permissions)) $permissions = [$permissions];

            Role::findOrCreate($role, $this->guardName)->syncPermissions($permissions);
        }

        foreach ($this->fixedUsers as $user => $roles) {
            if ($roles === null) $roles = [];
            elseif (is_string($roles)) $roles = [$roles];

            BotUser::find($user)?->syncRoles($roles);
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
    $this->forNamespace('App\\Mmb\\Sections\\Panel', 'AccessAdmin');
}
```

### Introduce Area
Add area class to `config/mmb.php`:
```php
'areas' => [
    Areas\AdminArea::class,
],
```
