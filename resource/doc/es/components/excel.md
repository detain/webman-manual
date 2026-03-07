# Tablas Excel

## Dirección del proyecto

https://github.com/PHPOffice/PhpSpreadsheet
  
## Instalación
 
  ```php
  composer require phpoffice/phpspreadsheet
  ```
  
## Uso

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
        // Guardar el archivo en el directorio public
        $writer->save($file_path);
        // Descargar el archivo
        return response()->download($file_path, 'nombre_archivo.xlsx');
    }

}
```

## Más información

Visite https://phpspreadsheet.readthedocs.io/en/latest/
  
