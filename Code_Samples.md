# Useful Code Samples








## Eloquent


Models should contain only Laravel native things (relations, scopes...) and database-related code.
Huge business logic should be written into Support or Action classes.
Use mass assignment where possible.



#### Latest of Many


```
public function latest_transaction() 
{
    return $this->hasOne(Transaction::class)->latest();
} 
```
or
```
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
````
public function destroy(Project $project) {
    DB::transaction(function () {
        $project->tasks()->delete();
        $project->delete();
    });
}
```

Database Transaction is important because it would rollback any deleted task if something goes wrong with the following deleting of other tasks or parent project object. So, it's "delete all or nothing".
