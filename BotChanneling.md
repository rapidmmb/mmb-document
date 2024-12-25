
# Bot Channeling
Single source, multiple bot! Change the `configs/mmb.php` file.

## Default Channeling
Use this for single bot:
```php
return [
    'channeling'     => Core\DefaultBotChanneling::class,

    'channels'       => [
        'default' => [
            'token'     => env('BOT_TOKEN'),
            'username'  => env('BOT_USERNAME'),
            'hookToken' => 'YOUR_HOOK_TOKEN',
            // 'guard'     => 'bot',

            'handlers' => [
                Handlers\PrivateHandler::class,
                Handlers\GroupHandler::class,
            ],
        ],
    ],
];
```

Or for multiple bots:
```php
return [
    'channeling'     => Core\DefaultBotChanneling::class,

    'channels'       => [
        'default' => [
            'token'     => env('BOT_TOKEN'),
            'username'  => env('BOT_USERNAME'),
            'hookToken' => 'YOUR_HOOK_TOKEN',
            // 'guard'     => 'bot',

            'handlers' => [
                Handlers\PrivateHandler::class,
                Handlers\GroupHandler::class,
            ],
        ],
        'second' => [
            'token'     => 'SECOND_TOKEN',
            'username'  => 'SECOND_USERNAME',
            'hookToken' => 'SECOND_HOOK_TOKEN', // should different!
            // 'guard'     => 'bot',

            'handlers' => [
                Handlers\PrivateHandler::class,
                Handlers\GroupHandler::class,
            ],
        ],
    ],
];
```

## Creative Channeling
Creative channeling use a database to find robot. So you can create multiple bots in one source code and modify bots in real time.
```php
return [
    'channeling'         => Core\CreativeBotChanneling::class,

    'channels'           => [
        'database' => [
            'model'          => BotModel::class,
            'tokenColumn'    => 'token', // default is 'token'
            'nameColumn'     => 'name', // default is 'name'
            'hookColumn'     => 'hook_token', // should be unique! default is 'hook_token'
            'usernameColumn' => 'username', // optional
        ],
        'default' => [
            'token'     => env('BOT_TOKEN'),
            'username'  => env('BOT_USERNAME'),
            'hookToken' => 'YOUR_HOOK_TOKEN',

            'handlers' => [
                Handlers\PrivateHandler::class,
                Handlers\GroupHandler::class,
            ],
        ],
        'foo_cat' => [
            'handlers' => [
                FooHandlers\PrivateHandler::class,
            ],
        ],
    ],
];
```
