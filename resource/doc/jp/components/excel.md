# Excel表

## プロジェクトのアドレス

https://github.com/PHPOffice/PhpSpreadsheet
  
## インストール
 
  ```php
  composer require phpoffice/phpspreadsheet
  ```
  
## 使い方

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
        // publicディレクトリにファイルを保存
        $writer->save($file_path);
        // ファイルをダウンロード
        return response()->download($file_path, 'ファイル名.xlsx');
    }

}
```

## 詳細情報

https://phpspreadsheet.readthedocs.io/en/latest/ をご覧ください
  
