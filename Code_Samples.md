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












