# Useful Code Samples








## Eloquent

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

