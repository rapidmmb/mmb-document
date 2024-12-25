
# Async

mmb uses the amphp package for async.

```php
$this->response("Wait for 5 seconds...");
delay(5);
$this->response("Wow!");
```

```php
$promises = [];

foreach (BotUser::all() as $user) {
    $promises[] = async(function () use ($bot, $user) {
        $bot->sendMessage(
            text: "Hello!",
            chatId: $user->id,
            ignore: true,
        );
    });
}

await($promises);
```

## Update Handling

The update handling policy is such that in a chat,
updates are executed in order and the next update is not handled until
the response to one update is completed.

```php
// Bad:
$this->response("Wait for 5 seconds...");
delay(5);
$this->response("Wow!");

// Good:
$this->response("Wait for 5 seconds...");
dispatch(new ResponseAfter5s)->delay(now()->addSeconds(5));
```

## Thunder

You can use this package for more powerful update management.
This package gives you the ability to run part of the application in
the background without the need for a queue:

https://github.com/rapidmmb/thunder
