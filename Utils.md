
# Utils

## Query Matcher
```php
public function onCallback(QueryMatcher $matcher)
{
    $matcher->match('balance', 'balance');
}
```

### Pattern
Simple text:
```php
$matcher->match('my text');
```

#### Argument
```php
$matcher->match('My name is {name}');
```

#### Argument Type
```php
$matcher->match('/setAge {age:int}');
```

Valid types:

| Type      | Real Pattern    |
|-----------|-----------------|
| `slug`    | `[^\s\n\r\t]+`  |
| `slug?`   | `[^\s\n\r\t]*`  |
| `inline`  | `.+`            |
| `inline?` | `.*`            |
| `any`     | `[\s\S]+`       |
| `any?`    | `[\s\S]*`       |
| `int`     | `\d+`           |
| `int?`    | `\d*`           |
| `number`  | `\d+\.?\d*`     |
| `number?` | `(\d+\.?\d*\|)` |

Note: `slug` is default type.

#### Nullable Type
Add `?` after argument type:
```php
$matcher->match('{age:int?}');
```


### Match
```php
$matcher->match('pattern')
```

#### Action
```php
$matcher->match('pattern')->action('myMethod');
```

#### Action From
```php
$matcher->match('pattern {method}')->actionFrom('method');
```

#### Json
```php
$matcher->match('{method}:{args}')->json('args')->actionFrom('method');
```
All the input arguments will be compressed in {args} parameter.
And all the matched arguments named {args} will be uncompressed in action parameters.

#### Options
Ignore case:
```php
$matcher->match('IP')->ignoreCase(); // Accept 'IP', 'ip', 'Ip' and 'iP'
```

Whitespace:
```php
$matcher->match('hi mmb')->ignoreSpaces(); // Accept 'hi mmb', 'hi  mmb', 'hi   mmb', 'hi \n mmb' and ...
$matcher->match('hi mmb')->optionalSpaces(); // Accept 'himmb', 'hi mmb', 'hi  mmb', 'hi \n mmb' and ...
```

Custom spaces pattern:
```php
$matcher->match('hi mmb')->spaces('_+'); // Accept 'hi_mmb', 'hi__mmb', 'hi___mmb' and ...
```

### Special Value
#### Regex Pattern
```php
$matcher->match('{name}')->pattern('name', '\w{5}'); // Accept only words with 5 letters
```

#### Same
```php
$matcher->match('{name}')->same('name', 'Mahdi'); // Accept just 'Mahdi'
```

#### In
```php
$matcher->match('{name}')->in('name', ['Mahdi', 'Mmb']); // Accept 'Mahdi' and 'Mmb'
```

### Auto Match
```php
public function onCallback(QueryMatcher $matcher)
{
    $matcher->autoMatch($this);
}

#[OnCallback]
public function balance()
{
    $this->tell("Your balance: $0");
}
```

Note: `#[OnCallback]` only works in `CallbackControl` objects.


### Advanced
#### Find Pattern
Find first pattern that matched with input text
```php
$pattern = $matcher->findPattern('My name is Mahdi');

if ($pattern)
{
    $pattern->invoke($this);
}
```

#### Make Query
Make query using parameters (first matched pattern will use)
```php
$matcher->match('My name is {name}');

$text = $matcher->makeQuery('Mahdi');
echo $text; // My name is Mahdi
```


## POV
If you want to answer other users in the update handling, you can change POV
to second user and then response that. Why?

If you want to send a menu to first user:
```php
$this->myMenu->response();
```
That's easy! But what about second user? That's should be hard.

Don't worry! POV is here!
```php
pov()->user($secondUser)->run(function ($context)
{
    static::make($context)->myMenu->response();
});
```
That's so easy!

Example:
```php
class ChatQueueSection extends Section
{
    public function findChat()
    {
        if ($queue = ChatQueue::first())
        {
            $queue->delete();
            $chat = Chat::create([
                'user_a_id' => BotUser::current()->id,
                'user_b_id' => $queue->user_id,
            ]);

            pov()->user($queue->user)->run(
                fn (Context $context) => static::make($context)->safe->openedChat($chat)
            );
            $this->openedChat($chat);
        }
        else
        {
            ChatQueue::create(['user_id' => BotUser::current()->id]);
            $this->response("You are in queue...");
        }
    }

    public function openedChat(Chat $chat)
    {
        $this->chatMenu->make(chat: $chat)->response();
    }

    public function chatMenu(Menu $menu, #[FindById] Chat $chat)
    {
        $menu
            ->message("A user found:")
            ->schema([
                [ $menu->key("Exit", 'exit') ],
            ])
        ;
    }
}
```

Note: POV creates a new `Context` object that you should use this instead of old context.

So you should make new instance for new POV like this:
```php
static::make($newContext)->myFunc(); // Instead of $this->myFunc()
// Or
static::invokes($newContext, 'myFunc');
```


## Keyboard
### Keyboard Formatter
```php
KeyFormatter::wrap([
    [$menu->key('A'), $menu->key('B'), $menu->key('C')],
    [$menu->key('D')],
], 2)
```

#### Wrap
Wrap:
```php
KeyFormatter::wrap([
    [$menu->key('A'), $menu->key('B'), $menu->key('C')],
    [$menu->key('D'), $menu->key('E'), $menu->key('F'), $menu->key('G')],
], 2)
```
Result is:
```php
[
    [$menu->key('A'), $menu->key('B')],
    [$menu->key('C')],
    [$menu->key('D'), $menu->key('E')],
    [$menu->key('F'), $menu->key('G')],
]
```
Wrap Hidden:
```php
KeyFormatter::wrapHidden([
    [$menu->key('A'), $menu->key('B'), $menu->key('C')],
    [$menu->key('D'), $menu->key('E'), $menu->key('F'), $menu->key('G')],
], 2)
```
Result is:
```php
[
    [$menu->key('A'), $menu->key('B')],
    [$menu->key('D'), $menu->key('E')],
]
```

#### Resize
Resize:
```php
KeyFormatter::resize([
    [$menu->key('A'), $menu->key('B'), $menu->key('C')],
    [$menu->key('D'), $menu->key('E'), $menu->key('F'), $menu->key('G')],
], 2)
```
Result is:
```php
[
    [$menu->key('A'), $menu->key('B')],
    [$menu->key('C'), $menu->key('D')],
    [$menu->key('E'), $menu->key('F')],
    [$menu->key('G')],
]
```
Or:
```php
KeyFormatter::resize(function () use ($menu) {
    for ($i = 1; $i < 6; $i++)
    {
        yield [$menu->key($i)]; // or yield $menu->key($i);
    }
}, 3)
```
Result is:
```php
[
    [$menu->key(1), $menu->key(2), $menu->key(3)],
    [$menu->key(4), $menu->key(5)],
]
```
Resize Auto:

This method resize the key using their string length.
```php
KeyFormatter::resizeAuto([
    [
        $menu->key('Hello'),
        $menu->key('How are you baby?'),
        $menu->key('OK'),
        $menu->key('Good bye'),
        $menu->key('I\'m fine'),
        $menu->key('Let\'s go'),
        $menu->key('Hello my friend im in the bathroom'),
    ]
])
```
Result is:
```php
[
    [
        $menu->key('Hello'),
        $menu->key('How are you baby?'),
    ],
    [
        $menu->key('OK'),
        $menu->key('Good bye'),
        $menu->key('I\'m fine'),
        $menu->key('Let\'s go'),
    ],
    [
        $menu->key('Hello my friend im in the bathroom'),
    ],
]
```

#### Rtl
```php
KeyFormatter::rtl([
    [$menu->key('A'), $menu->key('B')],
    [$menu->key('C'), $menu->key('D'), $menu->key('E')],
])
```
Result is:
```php
[
    [$menu->key('B'), $menu->key('A')],
    [$menu->key('E'), $menu->key('D'), $menu->key('C')],
]
```


#### Builder
```php
$result = KeyFormatter::for([...])->resize(2)->rtl()->toArray();
```


## Text

### Modes
```php
$mode = Text::mode('none');
$mode = Text::mode('html');
$mode = Text::mode('markdown'); // Recommended to use 'markdown2'
$mode = Text::mode('markdown2');
```

```php
// Normal styles:
$mode->bold('Hello world'); // <b>Hello world</b>
$mode->italic('Text'); // <i>Text</i>

// Tag user:
$mode->user($user->name, $user->id); // <a href='tg://user?id=12345678'>Ahmad</a>

// Merge many styled text (StringContent):
$mode->build($mode->bold('A'), $mode->italic('B')); // <b>A</b><i>B</i>
$mode->build('Hello ', $mode->bold('Ahmad'), $mode->italic(' =D')); // Hello<b>Ahmad</b><i> =D</i>

// Deep:
$mode->bold($mode->italic('Styled')); // <b><i>Styled</i></b>
```

### Text Builder
```php
TextBuilder::make('html', prefix: '$ ', suffix: ' $', separate: ' : ', division: "---", subtitle: "The End")
    ->line('This is simple text')
    ->space1()
    ->line('Your name', $user->name)
    ->space()
    ->div()
    ->space()
    ->blank('Checkout your profile tomorrow')
    ->toString();
```
Result is:
```text
$ This is simple text $

$ Your name : Ahmad $


---


Checkout your profile tomorrow
The End
```

Using `build` helper method:
```php
TextBuilder::make('html', prefix: '# ', separate: ' : ')
    ->build(fn (TextBuilder $text, TextModeEncoding $mode) => $text
         ->blank($mode->bold("Your Profile:"))
         ->space()
         ->line('Name', $user->name)
         ->line('Age', $user->age)
         ->line('Break the', 'rules', prefix: '~ ', suffix: ' ~', separate: '  ')
    );
```

Result is:
```text
<b>Your Profile:</b>


# Name : Ahmad
# Age : 12
~ Break the  rules ~
```



