
# Step Handler

## What's a step

We use a concept called "step" to keep track of the user's state and remember
what stage the user is in.

Whenever we need future messages from the user, we can save a "step" of
the current step; whenever the user sends a message, the tasks we want to be
performed.

A "step" can contain any type of information and store everything.
But for optimization, we prefer to store the least amount of data.

## Next Step Handler

The simplest possible type of step handler is this class.
All it does is take a function from you, and execute it on the next message
from the user.

```php
NextStepHandler::make()
    ->for(YourSection::class, 'yourMethod')
    ->keep($this->context);
```

The keep method saves this step in the user model.

If you are inside a Section, you can use the helper method:

```php
$this->nextStep('yourMethod');
```

Example:

```php
public function main()
{
    $number = rand(1, 100);

    $this->response("Guess the number:");
    
    $this->nextStep('guessNumber', $number);
}

public function guessNumber(int $actualNumber)
{
    try {
        Filter::make()
            ->unsignedInt()
            ->pass($this->context, $this->context->update, $number);

        if ($number == $actualNumber) {
            $this->response("Correct!");
        } else {
            $this->response("Wrong!");
        }
    } catch (FilterFailException $e) {
        $this->response($e->description);
    }
}
```

## Pipeline Step Handler

Pipelines are like magic! With pipelines, you can make the user be in
multiple steps at the same time!

In a pipeline, you store a set of steps, and the next step tries to execute
them in order. If a step fails and calls `skipHandler`,
it skips to the next step until a step can execute.

```php
public function main()
{
    $this->response("Enter a number or a text:");
    
    $pipe = PipelineStepHandler::make();
    $pipe->push(NextStepHandler::make()->for(static::class, 'number'));
    $pipe->push(NextStepHandler::make()->for(static::class, 'text'));
    $pipe->keep();
}

public function number()
{
    try {
        Filter::make()
            ->unsignedInt()
            ->pass($this->context, $this->context->update, $number);
    
        $this->response("Number: $number");
    } catch (FilterFailException $e) {
        $this->update->skipHandler();
    }
}

public function text()
{
    try {
        Filter::make()
            ->text()
            ->pass($this->context, $this->context->update, $text);
    
        $this->response("Text: $text");
    } catch (FilterFailException $e) {
        $this->update->skipHandler();
    }
}
```

**Example:**

```php
class MySection extends Section
{
    use CallbackControl;

    public function main()
    {
        $this->mainMenu->response();
    }

    public function mainMenu(Menu $menu)
    {
        $menu
            ->message("Main menu:")
            ->schema([
                [$menu->key("Hello World", function () {
                    $this->response("Hello World :)");
                })],
                [$menu->key("Change Number", 'changeNumber')],
            ]);
    }

    public function changeNumber()
    {
        $pipe = PipelineStepHandler::make();
        $pipe->pushCurrent($this->context);

        $pipe->listen($this->context, function () {
            $this->changeNumberPlain->invoke();
        });

        $pipe->keep($this->context);
    }

    public ?int $messageId = null;

    #[UseEvents('lost')]
    public function changeNumberPlain(Plain $plain)
    {
        $plain
            ->with('messageId')
            ->invoking(function () {
                $this->messageId = $this->response("Enter a number:", key: [
                    [static::keyInline("Cancel", 'cancel')],
                ])->id;
                
                // Note: The changed messageId property is saved
                //   because this function is executed before the keep method.
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
            ->lost(function () {
                $this->cancel();
            })
        ;
    }
    
    #[OnCallback]
    public function cancel()
    {
        $this->bot()->deleteMessage(
            chatId: $this->update->getChat()->id,
            messageId: $this->messageId,
            ignore: true,
        );
    }
}
```

## Extends

```php
use Mmb\Action\Memory\Attributes\StepHandlerAlias as Alias;
use Mmb\Action\Memory\Attributes\StepHandlerSerialize as Serialize;
use Mmb\Action\Memory\StepHandler;

class RunMyMethodStepHandler extends StepHandler
{
    
    #[Alias('a')]
    #[Serialize]
    public array $arguments = [];

    public function handle(Context $context, Update $update): void
    {
        HomeSection::invokes($context, 'myMethod', $this->arguments);
    }

}
```

Usage:

```php
public function main()
{
    $step = RunMyMethodStepHandler::make();
    $step->arguments = [1, 2, 3];
    $step->keep($this->context);
}
```

### Events

#### 1. Handle

This method is executed when this step is saved and
the user sends another message.

```php
public function handle(Context $context, Update $update): void
{
    HomeSection::invokes($context, 'myMethod', $this->arguments);
}
```

#### 2. Begin

When this step is saved and the user sends another message,
this function is executed in the first step.
Unlike `handle`, this method does not wait for its priority and
skips all handlers in the first step.

```php
public function onBegin(Context $context, Update $update): void
{
    $update->response("Hello!");
}
```

You can also prevent the process from continuing by setting `isHandled` to true.

```php
public function onBegin(Context $context, Update $update): void
{
    $this->response("You're stuck here!");
    $update->isHandled = true;
}
```

> This feature, like the example above,
> can be dangerous and cause the user to not be able to do anything else!
> Because no matter what the user does, only this method will be executed!

#### 3. End

At the end of the update management, this method is called in the saved step.

```php
public function onEnd(Context $context, Update $update): void
{
    $update->response("I'm with you!");
}
```

#### 4. Lost

This method is called if a step loses its storage, or in other words,
is replaced by another step.

The bad thing about this method is that it cannot predict that the step is going
to change in the future, so a full replacement is called. For example,
if you want to cancel the previous operation,
the new operation first shows its message to the user,
then this method is called and the cancellation message is sent,
which is not very attractive.

But anyway, it's very useful for deleting data you no longer need:

```php
public function onLost(Context $context, Update $update): void
{
    @unlink($this->cacheFile);
}
```
