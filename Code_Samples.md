# Useful Code Samples



# Architecture


## Table of contents

- [Eloqent and DB](Eloquent.md)
- [Blade and Views](Blade_and_Views.md)
- [Livewire 3](Livewire_3.md)




### Business logic should be in service class

A controller must have only one responsibility, so move business logic from controllers to service classes.

```php
Bad:

    public function store(Request $request)
    {
        if ($request->hasFile('image')) {
            $request->file('image')->move(public_path('images') . 'temp');
        }
        
        ...
    }

Good:

    public function store(Request $request)
    {
        $this->articleService->handleUploadedImage($request->file('image'));
    
        ...
    }
    
    class ArticleService
    {
        public function handleUploadedImage($image): void
        {
            if (!is_null($image)) {
                $image->move(public_path('images') . 'temp');
            }
        }
    }
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



## Unset values fromo deeply-nested arrays

Let's take a look at how the data_forget helper works:

$data = [
  'people' => [
    'john' => ['address' => '123 main', 'state' => 'nc'],
    'michael' => ['address' => '34 east 5th', 'state' => 'ny']
  ]
];
 
data_forget($data, 'people.*.address');

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
