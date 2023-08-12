# Useful Code Samples








## Eloquent

#### Latest of Many


```
/**
 * Get the user's most recent order.
 */
public function latestOrder()
{
    return $this->hasOne(Order::class)->latestOfMany();
}
```

