
 

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
or like this

```php
 private function getNextInvoiceNumber()
    {
        // Or you can move this logic to the Model's boot() method
        return Invoice::max('invoice_number') + 1;
    }
```


### Grouping Data foor Reporting

```php
 public static function getReportByMonth()
    {
        return Invoice::withSum('products', \DB::raw('invoice_product.price * invoice_product.quantity'))
            ->orderBy('invoice_date', 'desc')
            ->get()
            ->groupBy(function($invoice) {
                return $invoice->invoice_date->format('Y-m');
            });
    }

    public static function getReportByProduct()
    {
        return \DB::table('invoice_product')
            ->select('products.name', \DB::raw('sum(invoice_product.price * invoice_product.quantity) as total_revenue'))
            ->join('products', 'invoice_product.product_id', '=', 'products.id')
            ->groupBy('products.name')
            ->orderBy('total_revenue', 'desc')
            ->get();
    }
```


##  Services

```php
namespace App\Services\Reports;

use App\Models\Invoice;
use Illuminate\Http\Response;

class InvoiceReportService {

    public function getReportByMonth()
    {
        return Invoice::withSum('products', \DB::raw('invoice_product.price * invoice_product.quantity'))
            ->orderBy('invoice_date', 'desc')
            ->get()
            ->groupBy(function($invoice) {
                return $invoice->invoice_date->format('Y-m');
            });
    }

    public function getReportByProduct()
    {
        return \DB::table('invoice_product')
            ->select('products.name', \DB::raw('sum(invoice_product.price * invoice_product.quantity) as total_revenue'))
            ->join('products', 'invoice_product.product_id', '=', 'products.id')
            ->groupBy('products.name')
            ->orderBy('total_revenue', 'desc')
            ->get();
    }

    public function downloadAsPdf($report): Response
    {
        // to be implemented - download as PDF
    }

    public function downloadAsCSV($report): Response
    {
        // to be implemented - download as CSV
    }

    public function downloadAsXLS($report):Response
    {
        // to be implemented - download as XLS
    }
}
```
