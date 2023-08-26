# Livewire 3











#### Sharing additional data with the view

The difference between public properties and passing data in the render method is that public properties can be adjusted by the user, data passed with 'with' does not.


```php
<?php
 
namespace App\Livewire;
 
use Illuminate\Support\Facades\Auth;
use Livewire\Component;
 
class CreatePost extends Component
{
    public $title;
 
    public function render()
    {
        return view('livewire.create-post')->with([
            'author' => Auth::user()->name,
        ]);
    }
}
```
















