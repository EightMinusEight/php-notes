# Useful Code Samples




# DB Models and Eloquent



## Eloquent


Models should contain only Laravel native things (relations, scopes...) and database-related code.
Huge business logic should be written into Support or Action classes.
Use mass assignment where possible.


#### Default model

You can assign a default model in `belongsTo` relationship, to avoid fatal errors when calling it like `{{ $post->user->name }}` if $post->user doesn't exist.

```php
public function user()
{
    return $this->belongsTo(User::class)->withDefault();
}
```






#### Latest of Many


```php
public function latest_transaction() 
{
    return $this->hasOne(Transaction::class)->latest();
} 
```
or
```php
/**
 * Get the user's most recent order.
 */
public function latestOrder()
{
    return $this->hasOne(Order::class)->latestOfMany();
}
```


#### Cascading Deletion of Children -  Deleting Manually with DB Transactions

hasMany relationship in Laravel and want to delete children records when the parent is deleted, there are a few ways to set that up.

```php
public function destroy(Project $project) {
    DB::transaction(function () {
        $project->tasks()->delete();
        $project->delete();
    });
}
```

Database Transaction is important because it would rollback any deleted task if something goes wrong with the following deleting of other tasks or parent project object. So, it's "delete all or nothing".




### Eloquent scopes inside of other relationships

Did you know that you can use Eloquent scopes inside of defining other relationships?

```php
app/Models/Lesson.php:

public function scopePublished($query)
{
     return $query->where('is_published', true);
}

app/Models/Course.php:

public function lessons(): HasMany
{
     return $this->hasMany(Lesson::class);
}
 
public function publishedLessons(): HasMany
{
     return $this->lessons()->published();
}
```



### Automatic Column Value When Creating Records

If you want to generate some DB column value when creating record, add it to model's `boot()` method.
For example, if you have a field "position" and want to assign the next available position to the new record (like `Country::max('position') + 1)`, do this:

```php
    class Country extends Model {
        protected static function boot()
        {
            parent::boot();
     
            Country::creating(function($model) {
                $model->position = Country::max('position') + 1;
            });
        }
    }
```




### Grouping by First Letter

You can group Eloquent results by any custom condition, here's how to group by first letter of user's name:

```php
    $users = User::all()->groupBy(function($item) {
        return $item->name[0];
    });
```



### Sub-selects in Laravel Way

From Laravel 6, you can use addSelect() in Eloquent statement, and do some calculation to that added column.

```php
    return Destination::addSelect(['last_flight' => Flight::select('name')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderBy('arrived_at', 'desc')
        ->limit(1)
    ])->get();
```



### Use DB Transactions

If you have two DB operations performed, and second may get an error, then you should rollback the first one, right?

For that, I suggest to use DB Transactions, it's really easy in Laravel:

```php
    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);
    
        DB::table('posts')->delete();
    });
```






### Reduce Memory

Sometimes we need to load a huge amount of data into memory. For example:

    $orders = Order::all();

But this can be slow if we have really huge data, because Laravel prepares objects of the Model class.
In such cases, Laravel has a handy function `toBase()`

```php
    $orders = Order::toBase()->get();
    //$orders will contain `Illuminate\Support\Collection` with objects `StdClass`.
```
By calling this method, it will fetch the data from the database, but it will not prepare the Model class.
Keep in mind it is often a good idea to pass an array of fields to the get method, preventing all fields to be fetched from the database.



### Order JSON column attribute

With Eloquent you can order results by a JSON column attribute

```php
    // JSON column example:
    // bikes.settings = {"is_retired": false}
    $bikes = Bike::where('athlete_id', $this->athleteId)
            ->orderBy('name')
            ->orderByDesc('settings->is_retired')
            ->get();
```


### Directly convert created\_at date to human readable format

Did you know you can directly convert created\_at date to human readable format like 1 minute ago, 1 month ago using diffForHumans() function. Laravel eloquent by default enables Carbon instance on created\_at field.

```php
    $post = Post::whereId($id)->first();
    $result = $post->created_at->diffForHumans();
    
    /* OUTPUT */
    // 1 Minutes ago, 2 Week ago etc..as per created time
```



### Ordering by an Eloquent Accessor

Ordering by an Eloquent Accessor! Yes, that's doable. Instead of ordering by the accessor on the DB level, we order by the accessor on the returned Collection.

```php
    class User extends Model
    {
        // ...
        protected $appends = ['full_name'];
     
        // Since Laravel 9
        protected function full_name(): Attribute
        {
    
            return Attribute::make(
                get: fn ($value, $attributes) => $attributes['first_name'] . ' ' . $attributes['last_name'];),
            );
        }
     
        // Laravel 8 and lower
        public function getFullNameAttribute()
        {
            return $this->attribute['first_name'] . ' ' . $this->attributes['last_name'];
        }
        // ..
    }
```




### Laravel Scout with database driver

With laravel v9 you can use Laravel Scout (Search) with database driver. No more where likes!

```php
    $companies = Company::search(request()->get('search'))->paginate(15);
```



### Make use of the value method on the query builder

Make use of the `value` method on the query builder to execute a more efficient query when you only need to retrieve a single column.

```php
    // Before (fetches all columns on the row)
    Statistic::where('user_id', 4)->first()->post_count;
    
    // After (fetches only `post_count`)
    Statistic::where('user_id', 4)->value('post_count');
```


### Return the primary keys from models collection

Did you know `modelsKeys()` eloquent collection method? It returns the primary keys from models collection.

```php
    $users = User::active()->limit(3)->get();
    
    $users->modelsKeys(); // [1, 2, 3]
```

----



# Model Relations





## JSON Where Clauses

Laravel offers helpers to query JSON columns for databases that support them.

```php
    // To query a json column you can use the -> operator
    $users = User::query()
                ->where('preferences->dining->meal', 'salad')
                ->get();
    
    // You can check if a JSON array contains a set of values
    $users = User::query()
                ->whereJsonContains('options->languages', [
                    'en', 'de'
                   ])
                ->get();
    
    // You can also query by the length a JSON array
    $users = User::query()
                ->whereJsonLength('options->languages', '>', 1)
                ->get();
```



## Get the newest (or oldest) item of another relation

Since Laravel 8.42, in an Eloquent model, you can define a relation that will get the newest (or oldest) item of another relation.

```php
    /**
     * Get the user's latest order.
     */
    public function latestOrder()
    {
        return $this->hasOne(Order::class)->latestOfMany();
    }
    
    /**
     * Get the user's oldest order.
     */
    public function oldestOrder()
    {
        return $this->hasOne(Order::class)->oldestOfMany(); 
    }
```


## Raw DB Queries: havingRaw()

You can use RAW DB queries in various places, including `havingRaw()` function after `groupBy()`.

```php
    Product::groupBy('category_id')->havingRaw('COUNT(*) > 1')->get();
````



## Default model

You can assign a default model in `belongsTo` relationship, to avoid fatal errors when calling it like `{{ $post->user->name }}` if $post-\>user doesn't exist.

```php
    public function user()
    {
        return $this->belongsTo(User::class)->withDefault();
    }
```


## Extra Filter Query on Relationships

If you want to load relationship data, you can specify some limitations or ordering in a closure function. For example, if you want to get Countries with only three of their biggest cities, here's the code.

```php
    $countries = Country::with(['cities' => function($query) {
        $query->orderBy('population', 'desc');
    }])->get();
```



## Load Relationships Always, but Dynamically

You can not only specify what relationships to ALWAYS load with the model, but you can do it dynamically, in the constructor method:

```php
    class ProductTag extends Model
    {
        protected $with = ['product'];
     
        public function __construct() {
            parent::__construct();
            $this->with = ['product'];
     
            if (auth()->check()) {
                $this->with[] = 'user';
            }
        }
    }
```


## Filter hasMany relationships

Just a code example from my project, showing the possibility of filtering hasMany relationships.

    TagTypes -\> hasMany Tags -\> hasMany Examples

And you wanna query all the types, with their tags, but only those that have examples, ordering by most examples.

```php
    $tag_types = TagType::with(['tags' => function ($query) {
        $query->has('examples')
            ->withCount('examples')
            ->orderBy('examples_count', 'desc');
        }])->get();
```


## Filter by many-to-many relationship pivot column

If you have a many-to-many relationship, and you add an extra column to the pivot table, here's how you can order by it when querying the list.

```php
    class Tournament extends Model
    {
        public function countries()    
        {
            return $this->belongsToMany(Country::class)->withPivot(['position']);
        }
    }

    class TournamentsController extends Controller
    {
        public function whatever_method() {
            $tournaments = Tournament::with(['countries' => function($query) {
                $query->orderBy('position');
            }])->latest()->get();
        }
    }
```


## You can add conditions to your relationships

```php
    class User
    {
        public function posts()
        {
            return $this->hasMany(Post::class);
        }
     
        // with a getter
        public function getPublishedPostsAttribute()
        {
            return $this->posts->filter(fn ($post) => $post->published);
        }
     
        // with a relationship
        public function publishedPosts()
        {
            return $this->hasMany(Post::class)->where('published', true);
        } 
    }
```


## \`whereHas()\` multiple connections

```php
    // User Model
    class User extends Model
    {
        protected $connection = 'conn_1';
     
        public function posts()
        {
            return $this->hasMany(Post::class);
        }
    }
     
    // Post Model
    class Post extends Model
    {
        protected $connection = 'conn_2';
     
        public function user()
        {
            return $this->belongsTo(User::class, 'user_id');
        }
    }
     
    // wherehas()
    $posts = Post::whereHas('user', function ($query) use ($request) {
          $query->from('db_name_conn_1.users')->where(...); 
      })->get();
```




# Blade and Views

### $loop variable in foreach

Inside of foreach loop, check if current entry is first/last by just using `$loop` variable.

```php
    @foreach ($users as $user)
         @if ($loop->first)
            This is the first iteration.
         @endif
     
         @if ($loop->last)
            This is the last iteration.
         @endif
     
         <p>This is user {{ $user->id }}</p    
    @endforeach
```

There are also other properties like `$loop->iteration` or `$loop->count`.


### Cleanup loops

Did you know the Blade `@each` directive can help cleanup loops in your templates?

```php
    // good
    @foreach($item in $items)
        <div>
            <p>Name: {{ $item->name }}
            <p>Price: {{ $item->price }}
        </div>
    @endforeach
     
    // better (HTML extracted into partial)
    @each('partials.item', $items, 'item')
```


### Checked blade directive

In Laravel 9, you'll be able to use the cool new "checked" Blade directive.

This is going to be a nice addition that we can use to clean up our Blade views a little bit

```php
    // Before Laravel 9:
    <input type="radio" name="active" value="1" {{ old('active', $user->active) ? 'checked' : '' }}/>
    
    <input type="radio" name="active" value="0" {{ old('active', $user->active) ? '' : 'checked' }}/>
     
    // Laravel 9   
    <input type="radio" name="active" value="1" @checked(old('active', $user->active))/>
 
    <input type="radio" name="active" value="0" @checked(!old('active', $user->active))/>
```

### Selected blade directive

In Laravel 9, you'll be able to use the cool new "selected" Blade directive for HTML select elements.

This is going to be a nice addition that we can use to clean up our Blade views a little bit

```php
    // Before Laravel 9:
    <select name="country">
        <option value="India" {{ old('country') ?? $country == 'India' ? 'selected' : '' }}>India</option>
        <option value="Pakistan" {{ old('country') ?? $country == 'Pakistan' ? 'selected' : '' }}>Pakistan</option>
    </select>
     
    // Laravel 9
    <select name="country">
        <option value="India" @selected(old('country') ?? $country == 'India')>India</option>
        <option value="Pakistan" @selected(old('country') ?? $country == 'Pakistan')>Pakistan</option> 
    </select>
```


### Calculate Sum with Pagination

How to calculate the sum of all records when you have only the PAGINATED collection? Do the calculation BEFORE the pagination, but from the same query.

```php
    // How to get sum of post_views with pagination?
    $posts = Post::paginate(10);
    
    // This will be only for page 1, not ALL posts
    $sum = $posts->sum('post_views');
     
    // Do this with Query Builder
    $query = Post::query();
    
    // Calculate sum
    $sum = $query->sum('post_views');
    
    // And then do the pagination from the same query    
    $posts = $query->paginate(10);
```

# Mail


## Preview Mailables

If you use Mailables to send email, you can preview the result without sending, directly in your browser. Just return a Mailable as route result:

```php
    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);
        return new App\Mail\InvoicePaid($invoice); 
    });
```

## Preview Mail without Mailables

You can also preview your email without Mailables. For instance, when you are creating notification, you can specify the markdown that may be use for your mail notification.

```php
    use Illuminate\Notifications\Messages\MailMessage;
     
    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);
        return (new MailMessage)->markdown('emails.invoice-paid', compact('invoice'));    
    });
```
You may also use other methods provided by `MailMessage` object such as `view` and others.

# Set conditional object properties

You can use the `when()` or `unless()` methods in your MailMessage notifications to set conditional object properties like a call to action.

```php
    class InvoicePaid extends Notification
    {
        public function toMail(User $user)
        {
            return (new MailMessage)
                ->success()
                ->line('We\'ve received your payment')
                ->when($user->isOnMonthlyPaymentPlan(), function (MailMessage $message) {
                    $message->action('Save 20% by paying yearly', route('account.billing'));
                })
                ->line('Thank you for using Unlock.sh');
        } 
    }
```











# Other



## Use Model Events or Observers to Recaalculate Stuff

```php
// Recalculate invoice when the invoice item is saved.
public function saved(InvoiceItem $invoiceItem): void
{
  $invoiceItem->invoice()->recalculate();
}
 
// Set default state when the order is created.
public function creating(Order $order): void
{
  $order->state = OrderState::NEW;
}
 
// Delete relations before the model is deleted.
public function deleting(Order $order): void
{
  $order->products()->delete();
}
```









## Schedule Laravel job based on time zone

Do you know you can schedule laravel job based on time zone

Setting Timezone for One Command:

```php
    $schedule->command('reportg:generate')
             ->timezone('America/New_York') 
             ->at('2:00');
```

If you are repeatedly assigning the same timezone to all of your schedule tasks, you may wish to define a `scheduleTimezone` method in you `app\Console\Kernel` class:

```php
    protected function scheduleTimezone()
    {
         return 'America/Chicago'; 
    }
```


## Dealing with deeply-nested arrays

If you have a complex array, you can use `data_get()` helper function to retrieve a value from a nested array using "dot" notation and wildcard.

```php
    $data = [
      0 => ['user_id' => 1, 'created_at' => 'timestamp', 'product' => {object Product}],
      1 => ['user_id' => 2, 'created_at' => 'timestamp', 'product' => {object Product}],
      2 => etc
    ];
     
    // Now we want to get all products ids. We can do like this:
    data_get($data, '*.product.id');
     
    // Now we have all products ids [1, 2, 3, 4, 5, etc...]

In the example below, if either `request`, `user` or `name` are missing then you'll get errors.

    $value = $payload['request']['user']['name'];
     
    // The data_get function accepts a default value, which will be returned if the specified key is not found.
      
    $value = data_get($payload, 'request.user.name', 'John')
```


## Customize how your exceptions are rendered

You can customize how your exceptions are rendered by adding a 'render' method to your exception.

For example, this allows you to return JSON instead of a Blade view when the request expects JSON.

```php
    abstract class BaseException extends Exception
    {
        public function render(Request $request)
        {
            if ($request->expectsJson()) {
                return response()->json([
                    'meta' => [
                        'valid'   => false,
                        'status'  => static::ID,
                        'message' => $this->getMessage(),
                    ],
                ], $this->getCode());
            }
     
            return response()->view('errors.' . $this->getCode(), ['exception' => $this], $this->getCode());
        }
    }

    class LicenseExpiredException extends BaseException
    {
        public const ID = 'EXPIRED';
        protected $code = 401;
        protected $message = 'Given license has expired.' 
    }
```


## Use through instead of map when using pagination

When you want to map paginated data and return only a subset of the fields, use `through` rather than `map`. The `map` breaks the pagination object and changes it's identity. While, `through` works on the paginated data itself

```php
    // Don't: Mapping paginated data
    $employees = Employee::paginate(10)->map(fn ($employee) => [
        'id' => $employee->id,
        'name' => $employee->name
    ])
     
    // Do: Mapping paginated data
    $employees = Employee::paginate(10)->through(fn ($employee) => [
        'id' => $employee->id,
        'name' => $employee->name 
    ])
```

## Copy file or all files from a folder

You can use the `readStream` and `writeStream` to copy a file (or all files from a folder) from one disk to another keeping the memory usage low.

```php
    // List all the files from a folder
    $files = Storage::disk('origin')->allFiles('/from-folder-name');
     
    // Using normal get and put (the whole file string at once)
    foreach($files as $file) {
        Storage::disk('destination')->put(
            "optional-folder-name" . basename($file),
            Storage::disk('origin')->get($file)
        );
    }
     
    // Best: using Streams to keep memory usage low (good for large files)
    foreach ($files as $file) {
        Storage::disk('destination')->writeStream(
            "optional-folder-name" . basename($file),
            Storage::disk('origin')->readStream($file)
        ); 
    }
```


## Specify what to do if a scheduled task fails or succeeds

You can specify what to do if a scheduled task fails or succeeds.

```php
    $schedule->command('emails:send')
            ->daily()
            ->onSuccess(function () {
                // The task succeeded
            })
            ->onFailure(function () {
                // The task failed 
            });
```


## Perform Action when Job has failed

In some cases, we want to perform some action when job has failed. For example, send an email or a notification.

For this purpose, we can use `failed()` method in the job class, just like the `handle()` method:

```php
    namespace App\Jobs\Invoice;
    use Illuminate\Bus\Batchable;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;
    use Throwable;
     
    class CalculateSingleConsignment implements ShouldQueue
    {
        use Batchable, Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
     
        // ... __construct() method, handle() method, etc.
     
        public function failed(Throwable $exception)
        {
            // Perform any action here when job has failed
         } 
    }
```












# Log and Debug





### Benchmark class

In Laravel 9.32 we have a Benchmark class that can measure the time of any task.

It's a pretty useful helper:

```php
    class OrderController
    {
         public function index()
         {
              return Benchmark::measure(fn () => Order::all()),
         } 
    }
```


## Log Long Running Laravel Queries

```php
    DB::enableQueryLog();
     
    DB::whenQueryingForLongerThen(1000, function ($connection) {
         Log::warning(
              'Long running queries have been detected.',
              $connection->getQueryLog()
         ); 
    });
```

## Log all the database queries during development

If you want to log all the database queries during development add this snippet to your AppServiceProvider

```php
    public function boot()
    {
        if (App::environment('local')) {
            DB::listen(function ($query) {
                logger(Str::replaceArray('?', $query->bindings, $query->sql));
            });
        } 
    }
```