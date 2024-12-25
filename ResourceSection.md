
# Resource Section
```php
class BotUserResourceSection extends ResourceSection
{

    protected $for = BotUser::class;

    public function resource(ResourceMaker $maker)
    {
        $maker->list()
            ->label(fn (BotUser $record) => "#{$record->id} ($record->name)")
            ->creatable(
                fn (ResourceCreateModule $module) => $module
                    ->textSingleLine('name', "نام کاربر را وارد کنید:")
                    ->attribute('step', '')
            )
            ->searchable(
                fn (ResourceSearchModule $module) => $module
                    ->by('id')
                    ->message("آیدی کاربر را جستجو کنید:")
            )
        ;
        $maker->info()
            ->editable(
                fn (ResourceEditModule $module) => $module
                    ->textSingleLine('name', "نام جدید کاربر را وارد کنید:")
            )
            ->editableSingle("نام", 'name', left: true)
            ->deletable()
        ;
    }
    
}
```

Be pretty:
```php
class BotUserResourceSection extends ResourceSection
{

    protected $for = BotUser::class;

    public function resource(ResourceMaker $maker)
    {
        $this->list($maker->list());
        $this->info($maker->info());
    }

    public function list(ResourceListModule $module)
    {
        $module
            ->label(fn (BotUser $record) => "#{$record->id} ($record->name)")
            ->creatable($this->create(...))
            ->searchable($this->search(...))
        ;
    }

    public function info(ResourceInfoModule $module)
    {
        $module
            ->editable($this->edit(...))
            ->editableSingle("نام", 'name', left: true)
            ->deletable()
        ;
    }

    public function create(ResourceCreateModule $module)
    {
        $module
            ->textSingleLine('name', "نام کاربر را وارد کنید:")
            ->attribute('step', '')
        ;
    }

    public function edit(ResourceEditModule $module)
    {
        $module
            ->textSingleLine('name', "نام جدید کاربر را وارد کنید:")
        ;
    }

    public function search(ResourceSearchModule $module)
    {
        $module
            ->by('id')
            ->message("آیدی کاربر را جستجو کنید:")
        ;
    }

}
```

## Resource List
A list to show all records.
```php
$maker->list()
    // Set message
    ->message("List:")
    // Set key label:
    ->label(fn (BotUser $record) => "#{$record->id} ($record->name)")
    // Add create key:
    ->creatable($this->create(...))
    // Add search key:
    ->searchable($this->search(...))
    // Add order key:
    ->orderable($this->order(...))
    // Customize query:
    ->query(fn () => BotUser::query()->role('admin'))
    // Customize query 2:
    ->filter(fn ($builder) => $builder->role('admin'))
    // Custom key:
    ->addTopKey("My Key", fn () => $this->response("Hey!"))
    // Add simple filters:
    ->simpleFilter('testFilter', $this->testFilter(...))
    // Customize empty-state
    ->notFound(fn () => $this->response("Empty state"))
    ->notFoundLabel("Empty")
    // Select action
    ->select(fn ($record) => $this->response("Selected #{$record->id}"))
    // Customize menu
    ->top(fn (Menu $menu) => ...)
    ->bottom(fn (Menu $menu) => ...)
    // Customize paginate
    ->perPage(20)
    ->pageHide()
    ->paginateHide()
    ->pageLabel(fn ($page, $lastPage) => "Page $page/$lastPage")
    ->paginateRow(...)
    ->paginateOnTop()
    ->paginateOnBottom()
    ->paginateOnBoth()
;
```

Available parameters:
```php
function action(
    $record, // Current record (only when one record is using)
    $all, $paginate, // All records in current page
    $page, $current, // Current page number
    $lastPage, // Last page number
    $menu, // Current menu (only for top() and bottom())
    $module, // Module
)
```

## Resource Info
```php
$maker->info()
    // Set message
    ->message(fn ($record) => "User #{$record->id}:")
    // Add edit key
    ->editable($this->edit(...))
    // Add single attribute edit key
    ->editableSingle("نام", 'name', left: true)
    // Add delete key
    ->deletable($this->delete(...))
    // Add key
    ->schema(...)
;
```

Available parameters:
```php
function action(
    $record, // Current record
    $module, // Module
)
```

## Resource Create/Edit
```php
$module
    // Add input
    ->input(...)
    ->textSingleLine('name', "نام کاربر را وارد کنید:")
    // Set custom attribute value
    ->attribute('step', '')
    ->attributes(['step' => ''])
    // Events
    ->creating(...)
    ->created(...)
    ->createdOpenInfo('info')
;
```

Available parameters:
```php
function action(
    $record, // Current record
    $module, // Module
)
```

## Resource Search
```php
$module
    // Set key
    ->by('id')
    // Set message
    ->message("آیدی کاربر را جستجو کنید:")
    // Search key
    ->keyLabel("Search")
    ->keySearchingLabel(fn ($query) => "Searching [$query]")
    // "Show All" key
    ->allKeyEnabled()
    ->allKeyLabel("Show All")
    // Set algorithm
    ->simple()
    ->query(fn ($builder, $query, $key))
;
```

## Resource Simple Filter
```php
$module
    // Reset filters
    ->reset()
    // Set message
    ->message(...)
    // Set key
    ->keyLabel("My Filter")
    ->prefix("Prefix")
    ->suffix("Suffix")
    // Add filters
    ->addNone("None", default: true)
    ->add("Admins", fn ($builder) => $builder->role('admin'))
    // Toggle mode
    // When press filter in the list, automatically select next filter
    ->toggle()
;
```

Set key label dynamically:
```php
$module
    ->prefix("My Filter : ")
    ->keyLabelAuto()
```

## Resource Order
Order is an simple filter.
```php
$module
    // Delete default orders
    ->reset()
    // Set message
    ->message(...)
    // Add order
    ->addOrder("Name", 'name', asc: true)
    ->addAsc("Name Asc", 'name')
    ->addDesc("Name Desc", 'name')
;
```

## Resource Delete
```php
$module
    // Set message
    ->message("Are you sure to delete?")
    // Set confirm message
    ->confirm("Yes, Delete!")
```
