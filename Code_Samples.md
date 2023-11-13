# Useful Code Samples








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



# Automatic Column Value When Creating Records

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




# Grouping by First Letter

You can group Eloquent results by any custom condition, here's how to group by first letter of user's name:

```php
    $users = User::all()->groupBy(function($item) {
        return $item->name[0];
    });
```



# Sub-selects in Laravel Way

From Laravel 6, you can use addSelect() in Eloquent statement, and do some calculation to that added column.

```php
    return Destination::addSelect(['last_flight' => Flight::select('name')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderBy('arrived_at', 'desc')
        ->limit(1)
    ])->get();
```



# Use DB Transactions

If you have two DB operations performed, and second may get an error, then you should rollback the first one, right?

For that, I suggest to use DB Transactions, it's really easy in Laravel:

```php
    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);
    
        DB::table('posts')->delete();
    });
```






# Reduce Memory

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



# Order JSON column attribute

With Eloquent you can order results by a JSON column attribute

```php
    // JSON column example:
    // bikes.settings = {"is_retired": false}
    $bikes = Bike::where('athlete_id', $this->athleteId)
            ->orderBy('name')
            ->orderByDesc('settings->is_retired')
            ->get();
```


# Directly convert created\_at date to human readable format

Did you know you can directly convert created\_at date to human readable format like 1 minute ago, 1 month ago using diffForHumans() function. Laravel eloquent by default enables Carbon instance on created\_at field.

```php
    $post = Post::whereId($id)->first();
    $result = $post->created_at->diffForHumans();
    
    /* OUTPUT */
    // 1 Minutes ago, 2 Week ago etc..as per created time
```



# Ordering by an Eloquent Accessor

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




## Laravel Scout with database driver

With laravel v9 you can use Laravel Scout (Search) with database driver. No more where likes!

```php
    $companies = Company::search(request()->get('search'))->paginate(15);
```



## Make use of the value method on the query builder

Make use of the `value` method on the query builder to execute a more efficient query when you only need to retrieve a single column.

```php
    // Before (fetches all columns on the row)
    Statistic::where('user_id', 4)->first()->post_count;
    
    // After (fetches only `post_count`)
    Statistic::where('user_id', 4)->value('post_count');
```


## Return the primary keys from models collection

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







