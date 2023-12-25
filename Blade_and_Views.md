

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
