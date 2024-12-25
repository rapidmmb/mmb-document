
# Inline Actions
All inline actions should define in class of type Action, as a function.

## Shared Document

Inline actions are used for complex, structured behavior within a section.
Inline actions allow you to interact with the user at the section level while
still storing the information you need.

### Define

Inline actions are usually defined with a function inside a Section that takes
as its first input a parameter of that type of inline action,
and in the body of that function, the behaviors are defined.

For example:

```php
public function mainMenu(menu $menu)
{
    $menu
        ->creating(function () {
            $this->response("Menu is creating...");
        })
        ->loading(function () {
            $this->response("Menu is loading from database...")
        })
        ->schema([
            [$menu->key("Salam", 'salam')],
        ])
    ;
}
```

And then to create an inline action that has this structure,
just take the name of that function as a property:

```php
public function main()
{
    $defferMenu = $this->mainMenu;
}
```

Of course, this variable gives you an object of type `DefferInlineProxy<Menu>`.
To start creating the inline action, you need to call the `make` function:

```php
public function main()
{
    $menu = $this->mainMenu->make();
}
```

For example, if you want to call the `invoke` function, you have:

```php
public function main()
{
    $this->mainMenu->make()->invoke();
}
```

However, if your inline action is parameterless, you can use it without
calling the make function. That's exactly why this object is called a proxy!

```php
public function main()
{
    $this->mainMenu->invoke();
}
```

### Lifecycle

1. First of all, you create an inline action.
2. Then, you display it to the user or apply it, for example by `invoking` it
(depending on the type and behavior of the inline action).
3. Then with the help of StepHandler the current step is saved in the database.
4. Then the user wakes up the inline action again with their next update.
5. And finally, according to the inline action policies, a response 
is given to the user update.


> Note: The functions you define for inline actions are called once
> and are not kept in RAM whenever the inline action is required.
> 
> This powerful feature was introduced precisely because of the ability
> to reload the function you defined.

### Events

#### 1. Creating
```php
public function mainMenu(Menu $menu)
{
    $menu
        ->creating(fn() => $this->response("Menu is creating..."))
    ;
}
```

#### 2. Loading
```php
public function mainMenu(Menu $menu)
{
    $menu
        ->loading(fn() => $this->response("Menu is loading..."))
    ;
}
```

#### 3. Begin Update

> To use more advanced events than [StepHandler](StepHandler#events),
> you can introduce them to mmb with the `UseEvents` attribute.
> This is because for the events to be executed,
> we need the entire inline action to be loaded,
> and it is not always good to load it at BEGIN, for example.
> _**The choice is yours.**_

```php
#[UseEvents('begin')]
public function mainMenu(Menu $menu)
{
    $menu
        ->beginUpdate(fn() => $this->response("This is first thing that called"))
    ;
}
```

[See in StepHandler...](StepHandler.md#2-begin)

#### 4. End Update

```php
#[UseEvents('end')]
public function mainMenu(Menu $menu)
{
    $menu
        ->endUpdate(fn() => $this->response("This is last thing that called"))
    ;
}
```

[See in StepHandler...](StepHandler.md#3-end)

#### 5. Lost

```php
#[UseEvents('lost')]
public function editImageMenu(Menu $menu, string $imagePath)
{
    $menu
        ->lost(function () use ($imagePath) {
            @unlink($imagePath);
        })
    ;
}
```

[See in StepHandler...](StepHandler.md#4-lost)


### Variant Memory

#### 1. Argument Memory
```php
public function mainMenu(Menu $menu, int $number, #[FindById] Post $dbPost, Post $serializedPost)
```
If you define a Model like Post, you have two options.
1- Using #[Find] to save just `id` and reload model from database.
2- Empty using to save all attributes in user data (will not reload from database).

Usage:
```php
public function main()
{
    $dbPost = Post::find(27);
    $sPost = new Post(['title' => 'Hello world']);
    $this->mainMenu->make(number: 100, dbPost: $dbPost, serializedPost: $sPost)->response();
}
```

**To change:**
```php
public function mainMenu(Menu $menu, int $number)
{
    $menu
        ->schema([
            [$menu->key("Number: $number")],
            [$menu->key("Increase", 'increase')],
        ])
        ->on('increase', function () use ($number) {
            $this->mainMenu->make(number: $number + 1)->response("Increased");
        })
    ;
}
```

#### 2. Property Memory
```php
public int $number;
#[Find]
public Post $dbPost;
public Post $serializedPost;

public function mainMenu(Menu $menu)
{
    $menu
        ->with('number', 'dbPost', 'serializedPost')
    ;
}
```
Usage:
```php
public function main()
{
    $this->number = 100;
    $this->dbPost = Post::find(27);
    $this->serializedPost = new Post(['title' => 'Hello world']);
    
    $this->mainMenu->response();
}
```

**To change:**
```php
public int $number;

public function mainMenu(Menu $menu)
{
    $menu
        ->with('number')
        ->schema([
            [$menu->key("Number: {$this->number}")],
            [$menu->key("Increase", 'increase')],
        ])
        ->on('increase', function () {
            $this->number++;
            $this->mainMenu->response("Increased");
        })
    ;
}
```

#### 3. Local Variable

```php
public function mainMenu(Menu $menu)
{
    $menu
        ->have('random', $random, function () {
            return rand(1, 100);
        })
        ->schema([
            [$menu->key("Random: [$random]")]
        ])
    ;
}
```

The second parameter of the `have` method is an output variable that is set to
its value when the inline action is created, via the third parameter,
and loaded in subsequent times.

> Note: You cannot change the set value.



## Menu
```php
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
```
### Advanced Key
```php
public function mainMenu(Menu $menu)
{
    $menu
        ->schema([
            [
                // Invoke helloWorld() method
                $menu->key("Hello World!", 'helloWorld'),
                
                // Invoke helloWorld() method from other class
                $menu->key("Profile")->invoke(ProfileSection::class, 'main'),
                $menu->keyFor("Profile", ProfileSection::class, 'main'),
                
                // Inline action
                $menu->key("Help", fn() => $this->response("Information Text...")),
                
                // Inline action (way 2)
                $menu->key("Logout", 'logout'), // logout action defined below
                
                // Invoke with arguments
                $menu->key("Plus", 'addValue', 1),
                $menu->key("Mines", 'addValue', -1),
                
                // Add condition
                $menu->key("My Key")->if($id == 3),
                $menu->key("Invalid")->else(),
                
                // Visible / Hidden
                $menu->key("Visible")->visible(),
                $menu->key("Hidden")->hidden(),
                $menu->key("My Key")->visibleIf($id == 3),
                $menu->key("Invalid")->visibleElse(),
                
                // Other types
                $menu->key("Contact")->requestContact(),
                $menu->key("Location")->requestLocation(),
                $menu->key("Chat")->requestChat(1),
            ],
        ])
        ->on('logout', function() {
            $this->response("You're logged out!");
        })
    ;
}
```
### Regex Pattern
```php
public function mainMenu(Menu $menu)
{
    $menu
        ->schema([...])
        ->onRegex('/^My name is (\w+)$/', function ($matches) {
            $this->response("Hello {$matches[1]}");
        })
        // Or
        ->onRegex('/^My name is (\w+)$/', function ($name) {
            $this->response("Hello {$name}");
        }, 1)
    ;
}
```
### Default Message
```php
public function mainMenu(Menu $menu)
{
    $menu
        ->schema([...])
        ->message("Welcome =D")
    ;
}
```

## Inline Form
```php
public function testForm(InlineForm $form)
{
    $form
        ->input('amount', fn (Input $input) => $input
            ->unsignedInt()
            ->prompt("Enter amount:")
        )
        ->input('submit', fn (Input $input) => $input
            ->onlyOptions()
            ->options([[$input->key("Submit", true)]])
            ->prompt("Submit?")
        )
        ->cancel(fn() => $this->main())
        ->finish(function (Form $form) {
            $this->response("Form Submitted");
        })
    ;
}
```

Usage:
```php
public function main()
{
    $this->inlineForm('testForm')->request();
}
```

### Scope
Ready delete form:
```php
public function testForm(InlineForm $form)
{
    $form->deleteScope();
}
```

Custom scope:
```php
class IFMyScope extends InlineFormScope
{
    public function apply(InlineForm $form)
    {
        $form
            ->input('submit', fn (Input $input) => $input
                ->onlyOptions()
                ->add("Submit", true)
                ->prompt("Submit?")
            )
        ;
    }
}
```

Usage:
```php
public function testForm(InlineForm $form)
{
    $form
        ->input('amount', fn (Input $input) => $input
            ->unsignedInt()
            ->prompt("Enter amount:")
        )
        ->scope(new IFMyScope())
        ->cancel(fn() => $this->main())
        ->finish(function (Form $form) {
            $this->response("Form Submitted");
        })
    ;
}
```

## Dialog
```php
public bool $enabled = false;
public function toggleDialog(Dialog $dialog)
{
    $dialog
        ->use(DialogData::class)
        ->with('enabled')
        ->schema([
            [ $dialog->key("Status: " . ($this->enabled ? "Enabled" : "Disabled")) ],
            [ $dialog->key("Toggle", 'toggle') ],
        ])
        ->on('toggle', function() use($dialog) {
            $this->tell("Toggled!");
            $this->enabled = !$this->enabled;
            $dialog->reload();
        })
        ->message("Toggle Dialog [" . ($this->enabled ? "Enabled" : "Disabled") . "]");
}
```

### Key Id
Default id is key text, for example:
```php
$dialog->key("Toggle") // id = Toggle
```
Set custom id:
```php
$dialog->key("Toggle")->id('tg') // id = tg
```
Set action and id easily:
```php
$dialog->keyId("Info", 'info') // id = info, action = info
```
For identify pressed key, id should be unique.
```php
$dialog->schema(function() {
    foreach ($users as $user) {
        // All of these keys have same response, so use `keyId`:
        yield [ $dialog->keyId("User: {$user->name}", 'none') ];
        
        // Each key must run different action, but has same text! So set unique id:
        yield [ $dialog->key("Delete", 'delete', $user)->id("del-{$user->id}") ]; 
    }
});
```


### Reload
Manually reload dialog:
```php
$dialog->on('toggle', function() use($dialog) {
    $this->tell("Toggled!");
    $this->enabled = !$this->enabled;
    $dialog->reload();
})
```

Reload dialog after each action:
```php
$dialog->autoReload();
```

Cancel auto reloading with:
```php
$dialog->on('status', function() use($dialog) {
    $this->tell("Status: " . ($this->enabled ? "Enabled" : "Disabled"));
    $dialog->cancelReload();
})
```

### Data Model
```php
$dialog->use(DialogData::class);
```

Model:
```php
class DialogData extends Model
{
    use HasFactory, HasPresent;

    public function present(Present $present)
    {
        $present->id();
        $present->belongsTo(BotUser::class, 'user_id'); // Important
        $present->text('target')->cast(StepCasting::class)->nullable(); // Important
        $present->timestamps();
    }
}
```

### Example
```php
public int $number = 0;
public function selectNumber(Dialog $dialog)
{
    $dialog
        ->use(DialogData::class)
        ->with('number')
        ->schema([
            [ $dialog->key("> {$this->number} <") ],
            [
                $dialog->key("1", 'pass', 1),
                $dialog->key("2", 'pass', 2),
                $dialog->key("3", 'pass', 3),
                $dialog->key("<", 'backspace'),
            ],
            [
                $dialog->key("4", 'pass', 4),
                $dialog->key("5", 'pass', 5),
                $dialog->key("6", 'pass', 6),
                $dialog->key("C", 'clear'),
            ],
            [
                $dialog->key("7", 'pass', 1),
                $dialog->key("8", 'pass', 2),
                $dialog->key("9", 'pass', 3),
                $dialog->key("0", 'pass', 0),
            ],
        ])
        ->autoReload()
        ->on('pass', fn($number) => $this->number = $this->number * 10 + $number)
        ->on('backspace', fn() => $this->number = floor($this->number / 10))
        ->on('clear', fn() => $this->number = 0)
        ->message("Select number:")
    ;
}
```

## Fixed Dialog
```php
class ProfileSection extends Section
{
    use CallbackControl;
    
    #[FixedDialog('profile:{user:int}')]
    public function userProfile(Dialog $dialog, #[FindById] BotUser $user)
    {
        $dialog
            ->schema([
                [ $dialog->key("Info", fn() => $this->tell("User: {$user->name}")) ],
                [ $dialog->key("Add 500", 'add', 500) ],
            ])
            ->on('add', function ($count) use ($user)
            {
                $user->money += $count;
                $user->save();
                $this->tell("Added!");
            })
            ->message("Edit User {$user->id}:")
        ;
    }
}
```

> Warning: Telegram just accept 64 characters! So be careful in define variants and callback name.

### Variant
If you have a variant in dialog, you should define it in `FixedDialog` value:
```php
#[FixedDialog('profile:{user:int}')]
public function userProfile(Dialog $dialog, #[FindById] BotUser $user);
```

### Within
```php
public int $myVar;

#[FixedDialog('profile:{myVar:int}')]
public function userProfile(Dialog $dialog)
{
    $dialog->with('myVar');
}
```

### Argument

The arguments for the keys are the same as before and anything can be passed
to them! Because keys operate on the basis of `id`.

```php
$dialog->key("Something", 'foo', 1000, false, [], $user)->id('amazing');
```


## Plain

The Plain inline action is the simplest inline action because
it doesn't send any messages, doesn't monitor messages,
and doesn't enforce any specific policies. Everything is in your hands.

It only has two methods, invoking and handling, which are executed
when the request to execute this Plain is made,
and when the next user message is sent, respectively.

```php
public function enterNumberPlain(Plain $plain)
{
    $plain
        ->invoking(function () {
            $this->response("Enter a number:");
        })
        ->handling(function () {
            try {
                Filter::make()
                    ->unsignedInt()
                    ->pass($this->context, $this->context->update, $number);

                $this->response("You entered $number!");
            } catch (FilterFailException $e) {
                $this->response($e->description);
            }
        })
    ;
}
```

Usage:

```php
public function main()
{
    $this->enterNumberPlain->invoke();
}
```
