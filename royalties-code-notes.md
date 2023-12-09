
 

### create an invoice number
 
 ```php
 protected static function boot()
    {
        parent::boot();

        self::creating(function($invoice) {
            // Uncomment this line if you wish to use it instead of the service
            // $invoice->invoice_number = self::max('invoice_number') + 1;
        });
    }
```
