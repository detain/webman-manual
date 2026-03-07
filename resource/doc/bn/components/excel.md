# এক্সেল টেবিল

## প্রজেক্ট অ্যাড্রেস

https://github.com/PHPOffice/PhpSpreadsheet
  
## ইনস্টলেশন
 
  ```php
  composer require phpoffice/phpspreadsheet
  ```
  
## ব্যবহার

```php
<?php
namespace app\controller;

use PhpOffice\PhpSpreadsheet\Spreadsheet;
use PhpOffice\PhpSpreadsheet\Writer\Xlsx;

class ExcelController
{
    public function index($request)
    {
        $spreadsheet = new Spreadsheet();
        $sheet = $spreadsheet->getActiveSheet();
        $sheet->setCellValue('A1', 'Hello World !');

        $writer = new Xlsx($spreadsheet);
        $file_path = public_path().'/hello_world.xlsx';
        // public ফোল্ডারে ফাইল সংরক্ষণ করুন
        $writer->save($file_path);
        // ফাইল ডাউনলোড করুন
        return response()->download($file_path, 'ফাইলের_নাম.xlsx');
    }

}
```

## আরও তথ্য

ভিজিট করুন https://phpspreadsheet.readthedocs.io/en/latest/
  
