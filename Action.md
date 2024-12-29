
# Action

Action is a base to execute logics


## Section

Section is an Action to execute your logics

```php
class HomeSection extends Section
{
    public function main()
    {
//        $this->menu('mainMenu')->response("Welcome!");

        // New syntax:
        $this->mainMenu->response("Welcome!");
    }

    public function mainMenu(Menu $menu)
    {
        $menu
            ->schema([
                [
                    $menu->key("Hello World!", 'helloWorld'),
                ],
            ])
        ;
    }
    
    public function helloWorld()
    {
        $this->response("Hi =D");
    }
}
```

### Safe Call

```php
public function main()
{
    // Don't use this!
    PanelSection::make($this->context)->main();
    
    // Use this instead:
    PanelSection::make($this->context)->safe->main();
}
```

The magical `safe` property ensures permissions and handles errors.

```php
public function main()
{
    denied(404);
}

public function denied404()
{
    $this->response("An error occurred");
}
```

The `denied404` method only calls if the `main` method calls using safe proxy.

Also, you can pass a closure to `safe`:

```php
$this->safe(function () {
  $this->response("This is safe");
});
```

If you want to handle the errors, but turning off authorizations, use `unsafe`:

```php
public function main()
{
    PanelSection::make($this->context)->unsafe->main();
}
```



## Form
```php
class TestForm extends Form
{

    protected $inputs = [
        'amount',
        'submit',
    ];

    public function amount(Input $input)
    {
        $input
            ->unsignedInt()
            ->ask("Enter amount:")
        ;
    }

    public function submit(Input $input)
    {
        $input
            ->onlyOptions()
            ->options([
                [$input->key("Submit", true)]
            ])
            ->ask("Submit?");
    }

    public function onFinish()
    {
        $this->response("Your amount is: {$this->amount}");
    }

    public function onCancel()
    {
        $this->response("Cancelled");
    }

}
```

Inputs:
```php
protected $inputs = [
    'amount',
    'submit',
];

public function amount(Input $input);

public function submit(Input $input);
```

Path:
```php
protected $path = [
    'amount',
    'submit',
];
```

#### Settings
```php
protected $cancelKey = false;
protected $defaultFormKey = false;
protected $previousKey = true;
protected $skipKey = true;
protected $mirrorKey = true;

protected $previousKey = "<< Previous <<";
```

#### Attributes
```php
public function amount(Input $input);

#[AsAttribute]
public $customAttr;

#[AsAttribute]
#[Find]
public BotUser $user;

public function onFinish()
{
    $amount = $this->amount;
    $customAttr = $this->customAttr;
    $user = $this->user;
}
```

#### Requests
```php
TestForm::make($context)->request();
// Or
TestForm::make($context)->request([
    'customAttr' => 'Test',
    'user' => $user,
]);
// Or
TestForm::make($context, [
    'customAttr' => 'Test',
    'user' => $user,
])->request();
```

## Chunked Form
```php
class TestForm extends Form
{
    use HasFormChunks;
}
```

### Inputs
```php
protected $inputs = [
    'name',
    'bankChunk' => [
        'card',
        'amount',
    ],
];
```

### Requests
```php
TestForm::make($context)->requestChunk('bankChunk');
TestForm::make($context)->requestChunk(['name', 'bankChunk']);
TestForm::make($context, ['user' => $user])->requestChunk('name');
TestForm::make($context)->requestChunk('name', [
    'user' => $user,
]);
TestForm::make($context, ['user' => $user])->withChunk('name')->request();
```

## With Records

```php
class EditUser extends Form
{
    use HasRecords;
    
    #[AsAttribute]
    #[Find]
    public BotUser $user;
    
    protected function record(): Model
    {
        return $this->user;
    }
    
    public function name(Input $input)
    {
        // ...
    }
    
    public function age(Input $input)
    {
        // ...
    }
    
    public function onFinish()
    {
        $this->updateRecord(['name', 'age']);
        $this->response("Updated!");
        $this->back();
    }
}
```

Update advanced parameters:

```php
// Simple, input names
$this->updateRecord([
    'name',
    'age',
]);

// Rename inputs
$this->updateRecord([
    'name' => 'user_name',
    'age' => 'user_age',
]);

// Modify values
$this->updateRecord([
    'name' => function (string $name) {
        return "[[$name]]";
    },
    'age' => function (int $age) {
        return $age + 14;
    },
]);

// Expand values
$this->updateRecord([
    'name' => [
        'first_name' => function (string $name) {
            return explode(' ', $name)[0];
        },
        'last_name' => function (string $name) {
            return explode(' ', $name)[1];
        },
    ],
]);

// Extra values
$this->updateRecord(['name', 'age'], ['vip_until' => now()]);
```

## Input
### Prompt
```php
$input->prompt("Enter number:");
// Or
$input->ask("Enter number:");
```

### Keyboards
```php
$input->schema([
    [ $input->key("Hello world") ],
]);
$input->header([
    [ $input->key("Page 2/10") ],
]);
$input->footer([
    [ $input->key("Next Page") ],
]);
```

### Filters
```php
$input->textSingleLine()
$input->unsignedInt()
$input->min(100)
$input->minLength(5)
$input->clamp(100, 999)
$input->length(5, 1000)
$input->divisible(1000)
$input->messageBuilder()
```

### Advanced Filters
```php
$input->useFilter(
    fn(Filter $filter) => $filter
        ->textSingleLine()
        ->regex('/^Token: (.+)$/', 1)
        ->or()
        ->unsignedInt()
        ->clamp(1, 1000)
)
```

### Events
```php
$input
    ->passing(function ($update) {})
    ->then(function () {})
    ->filled(function() {})
    ->on('customEvent', function() {})
```

### Settings
```php
$input
    ->disableCancelKey()
    ->disableDefaultFormKey()
    ->disableMirrorKey()
    ->disablePreviousKey()
    ->disableSkipKey()
    
    ->cancelKey()
    ->defaultFormKey()
    ->mirrorKey()
    ->previousKey()
    ->skipKey()
    
    ->cancelKey("Back To Main Menu")
```

## Form Key
```php
FormKey::make('Test');
// Or
$input->key('Test');
```

### Replacement
```php
$input->key('Test', 'Replacement');
```

### Action
```php
$input->keyAction('Click me', fn() => $this->response("Clicked"))
// Or
$input->keyAction('Click me', 'customEvent')
$input->on('customEvent', fn() => $this->response("Clicked"))
```

### Action With Pass Value
```php
$input->keyAction('Click me', fn($pass) => $pass("Hello World"))
```

### Enabled Condition
```php
$input->key('Some key')->if($id == 3)
```

## Command
```php
class BalanceCommand extends CommandAction
{

    protected $command = '/balance';

    public function handle()
    {
        $this->response("Your balance: $0");
    }

}
```

### Command Value
```php
protected $command = '/balance';
protected $command = '/balance {user:any}';
protected $command = ['/balance', '/myAccount'];
```

### Parameter
```php
protected $command = '/balance {user:any}';

public function handle($user)
{
    $this->response("You entered: $user");
}
```

### Separate Method
```php
protected $command = [
    '/balance',
    '/balance {user:any}' => 'handleFor',
];

public function handle()
{
    $this->response("Your balance: $0");
}

public function handleFor($user)
{
    $this->response("You entered: $user");
}
```

### Start Command
```php
class StartCommand extends StartCommandAction
{

    protected $command = ['/start', '/start {code:any?}' => 'invite'];

    protected $ignoreSpaces = true;

    public function handle()
    {
        HomeSection::make($this->context)->safe->main();
    }
    
    public function invite($code)
    {
        if ($code != User::current()->id && ($target = User::findCache($code)))
        {
            $target->inviteAccepted(User::current());
        }

        $this->handle();
    }

}
```

And use these methods for interacting with start command:

```php
public function main()
{
    $botUrl = StartCommand::make($this->context)->url();
    $inviteUrl = StartCommand::make($this->context)->url('invite-code');
}
```

## Callback Controlling
```php
class BalanceSection extends Section
{
    use CallbackControl;
}
```

> Callback control is used to show inline keyboards with reactions.
This feature is used for `keyInline()` and [Dialog](InlineAction.md#Dialog).


## Auto Match
If you don't define `onCallback` method, it will be automatically!
```php
#[OnCallback]
public function balance()
{
    $this->tell("Your balance: $0");
}
```

Custom pattern:
```php
#[OnCallback('myBalance')]
public function balance();
```

Full pattern:
```php
#[OnCallback('balance', true)]
public function balance();
```

Full pattern with method name (_ replaced with method name or $name argument)
```php
#[OnCallback('myMethod:{_}', true)]
public function balance();
```

Custom pattern with method name:
```php
#[OnCallback('{_}:{id:any}', true)]
public function balance($id);
```

### Custom Match
Define the `onCallback` method:

```php
public function onCallback(QueryMatcher $matcher)
{
    $matcher->match('balance', 'balance');
}

public function balance()
{
    $this->tell("Your balance: $0");
}
```

Enable auto matching:
```php
public function onCallback(QueryMatcher $matcher)
{
    $matcher->match('balance', 'balance');
    $matcher->autoMatch($this);
}
```

### Inline Key
```php
$this->keyInline("Key Text", ...args);
```

Inline for auto matched method:
```php
$this->keyInline("Text", 'method', ...args);
```

Inline for customized auto matched method:
(In this example, we don't have any matching parameter)
```php
#[OnCallback('balance')]
public function balance();

public function main()
{
    $key = $this->keyInline('My Balance');
}
```
Or use {_} to identify method:
```php
#[OnCallback('{_}')]
public function balance();

public function main()
{
    $key = $this->keyInline('My Balance', 'balance');
}
```

Pass arguments:
```php
#[OnCallback]
public function balance($id);

public function main()
{
    $id = 1000;
    $key = $this->keyInline('My Balance', 'balance', $id);
}
```
In customized method:
```php
#[OnCallback('{_}:{id:any}')]
public function balance($id);

public function main()
{
    $id = 1000;
    $key = $this->keyInline('My Balance', 'balance', $id);
}
```

Usage:
```php
public function main()
{
    $this->response("Select:", key: [
        [ $this->keyInline("My Balance", 'balance') ],
    ]);
}
```

## Middle Action
```php
class EnterAgeMiddleAction extends MiddleAction
{
    public function isRequired()
    {
        return !BotUser::current()->age;
    }

    public function main()
    {
        $this->ageForm->request();
    }

    public function ageForm(InlineForm $form)
    {
        $form
            ->input('age', fn (Input $input) => $input
                ->unsignedInt()->clamp(10, 150)
                ->prompt("Enter your age:")
            )
            ->cancel(fn() => HomeSection::make($this->context)->safe->main())
            ->finish(function (Form $form)
            {            
                BotUser::current()->update(['age' => $form->age]);
                $this->redirect();
            });
    }
}
```

### Request
Request runs anyway
```php
public function enterAge()
{
    EnterAgeMiddleAction::make($this->context)->redirectTo(static::class, 'ageEntered')->request();
}

public function ageEntered()
{
    $this->response("Good Job");
    $this->main();
}
```

### Required
Required runs when class is required (and stop the code)
```php
public function withdraw()
{
    EnterAgeMiddleAction::make($this->context)->requiredHere();

    if (BotUser::current()->age < 20)
    {
        $this->response("Access Denied");
        return;
    }
    
    $this->inlineForm('withdrawForm')->request();
}
```
> Note: `requiredHere` uses backtrace and detected current method! So don't use it on helper functions.
```php
public function withdraw()
{
    EnterAgeMiddleAction::make()->required([static::class, 'withdraw']);

    if (BotUser::current()->age < 20)
    {
        $this->response("Access Denied");
        return;
    }
    
    $this->inlineForm('withdrawForm')->request();
}
```

### Arguments
```php
class TestAction extends MiddleAction
{
    public function main(BotUser $user)
    {
        $this->myForm->make(user: $user)->request();
    }

    public function myForm(InlineForm $form, #[FindById] User $user);
}

class AnotherClass extends Section
{
    public function withdraw()
    {
        TestAction::make($this->context)->requiredHere($user);
    }
}
```

### Required Globally
Add class name to update handlers:
```php
class PrivateHandler extends UpdateHandler
{
    public function handle(HandlerFactory $handler)
    {
        $handler
            ->handle([
                MiddleActions\EnterAgeMiddleAction::class,
                $handler->afterMiddles(Sections\HomeSection::class, 'main'),
    
                $handler->step(),
            ])
        ;
    }
}
```
Mmb support multiple middle actions:
```php
class PrivateHandler extends UpdateHandler
{
    public function handle(HandlerFactory $handler)
    {
        $handler
            ->handle([
                MiddleActions\EnterAgeMiddleAction::class,
                MiddleActions\EnterNameMiddleAction::class,
                MiddleActions\EnterPhoneMiddleAction::class,
                $handler->afterMiddles(Sections\HomeSection::class, 'main'),
    
                $handler->step(),
            ])
        ;
    }
}
```
Set which method should run after the middle actions handled (and redirected):
```php
$handler->afterMiddles(Sections\HomeSection::class, 'main'),
```
Set different methods for different middle actions:
```php
class PrivateHandler extends UpdateHandler
{
    public function handle(HandlerFactory $handler)
    {
        $handler
            ->handle([
                MiddleActions\EnterAgeMiddleAction::class,
                MiddleActions\EnterNameMiddleAction::class,
                MiddleActions\EnterPhoneMiddleAction::class,
                $handler->afterMiddles(Sections\HomeSection::class, 'ageEntered')
                    ->only(MiddleActions\EnterAgeMiddleAction::class),
                $handler->afterMiddles(Sections\HomeSection::class, 'main'),
    
                $handler->step(),
            ])
        ;
    }
}
```
Category:
```php
class EnterAgeMiddleAction extends MiddleAction
{
    protected $category = 'user-profile';
}
class EnterNameMiddleAction extends MiddleAction
{
    protected $category = 'user-profile';
}
class EnterPhoneMiddleAction extends MiddleAction
{
    protected $category = 'phone';
}

class PrivateHandler extends UpdateHandler
{
    public function handle(HandlerFactory $handler)
    {
        $handler
            ->handle([
                MiddleActions\EnterAgeMiddleAction::class,
                MiddleActions\EnterNameMiddleAction::class,
                MiddleActions\EnterPhoneMiddleAction::class,
                $handler->afterMiddles(Sections\HomeSection::class, 'profileUpdated')->for('user-profile'),
                $handler->afterMiddles(Sections\HomeSection::class, 'phoneUpdated')->for('phone'),
                $handler->afterMiddles(Sections\HomeSection::class, 'main'), // Will not run in this case
    
                $handler->step(),
            ])
        ;
    }
}
```
> Note: `afterMiddles` runs just one method! How it selects? When last middle actions run, first `afterMiddles` that match the middle action will run.
