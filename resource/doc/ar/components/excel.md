# جداول Excel

## عنوان المشروع

https://github.com/PHPOffice/PhpSpreadsheet
  
## التثبيت
 
  ```php
  composer require phpoffice/phpspreadsheet
  ```
  
## الاستخدام

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
        // حفظ الملف في مجلد public
        $writer->save($file_path);
        // تنزيل الملف
        return response()->download($file_path, 'اسم_الملف.xlsx');
    }

}
```

## المزيد من المحتوى

قم بزيارة https://phpspreadsheet.readthedocs.io/en/latest/
  
