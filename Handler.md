
# Handlers
```php
class PrivateHandler extends UpdateHandler
{

    public function handle(HandlerFactory $handler)
    {
        $handler
            ->match($this->update->getChat()?->type == 'private')
            ->recordUser(
                BotUser::class,
                $this->update->getUser()?->id,
                create: $this->createUser(...),
                validate: $this->validateUser(...),
                autoSave: true,
            )
            ->handle([
                Commands\StartCommand::class,
                $handler->inherit(),
                $handler->step(),
            ])
        ;
    }

    public function createUser()
    {
        $user = $this->update->getUser();

        return [
            'id' => $user->id,
            'name' => $user->name,
            'step' => '',
        ];
    }

    public function validateUser(BotUser $user)
    {
        // return $user->ban === false || $user->can('IgnoreBan');
        return true;
    }

}
```

## Example
```php
class ChannelPostHandler extends UpdateHandler
{

    public function handle(HandlerFactory $handler)
    {
        $handler
            ->match((bool) $this->update->channelPost)
            ->handle([
                PostHandling::class,
            ])
        ;
    }

}
```

## Custom Handling
```php
class PostHandling extends Section implements UpdateHandling
{
    public function handleUpdate(Context $context, Update $update)
    {
        if ($update->channelPost->id != MY_CHANNEL)
        {
            $update->skipHandler();
            return;
        }
        
        // Handle
    }
}
```

## Extends Handler

Use the `Handler` facade in service providers, to extends the handlers

### Add Update Handler
```php
Handler::add(MyCustomHandler::class);
```

### Extend

By using `extend`, you can extend an existing handler

```php
Handler::extend(PrivateHandler::class, function (HandlerExtends $extends) {
    $extends
        ->first(
            fn (HandlerFactory $handler) => $handler
                ->record(...)
                ->invoke(...)
        )
        ->last(
            fn (HandlerFactory $handler) => $handler
                ->record(...)
                ->invoke(...)
        )
        ->event(
            'custom',
            fn (HandlerFactory $handler) => $handler
                ->record(...)
                ->invoke(...)
        )
        ->handle(fn (HandlerFactory $handler) => [
            CustomHandling::class,
        ])
        ->handle(fn (HandlerFactory $handler) => [
            CustomHandling2::class,
        ], 'custom-name')
})
```

And use in the handler:

```php
class PrivateHandler extends UpdateHandler
{

    public function handle(HandlerFactory $handler)
    {
        $handler
            // Here automatically fire the "first"
            ->match($this->update->getChat()?->type == 'private')
            ->extends('custom')
            ->recordUser(
                BotUser::class,
                $this->update->getUser()?->id,
                create: $this->createUser(...),
                validate: $this->validateUser(...),
                autoSave: true,
            )
            // Here automatically fire the "last"
            ->handle([
                $handler->inherit('custom-name'),
                StartCommandHandler::class,
                $handler->inherit(),
                
                $handler->step(),
            ])
        ;
    }

}
```

## Handle Service Provider

```php
use Mmb\Support\Providers\HandleServiceProvider as ServiceProvider;

class HandleServiceProvider extends ServiceProvider
{

    /**
     * List of the handlers
     */
    protected array $handlers = [
        CustomHandler::class,
    ];

    /**
     * Map of extended handler and callback method name
     */
    protected array $extend = [
        PrivateHandler::class => 'extendPrivate',
    ];
    
    public function extendPrivate(HandlerExtends $extends)
    {
        $extends->handle(fn (HandlerFactory $handler) => [
            CustomCommand::class,
        ], 'commands')
    }
    
    /**
     * List of areas
     */
    protected array $areas = [
        CustomArea::class,
    ];

}
```
