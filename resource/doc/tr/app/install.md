# Kurulum

Uygulama eklentisi kurmanın iki yolu vardır:

## Eklenti pazarından kurulum
[Resmi yönetim paneli webman-admin](https://www.workerman.net/plugin/82) sayfasına gidin, uygulama eklentileri sayfasını açın ve ilgili eklentiyi kurmak için kurulum düğmesine tıklayın.

## Kaynak paketinden kurulum
Uygulama pazarından eklenti arşivini indirin, açın ve açılan klasörü `{ana proje}/plugin/` dizinine yükleyin (plugin klasörü yoksa elle oluşturun). `php webman app-plugin:install eklenti_adı` komutunu çalıştırarak kurulumu tamamlayın.

Örnek: İndirilen arşiv ai.zip adlıysa, `{ana proje}/plugin/ai` dizinine açın ve `php webman app-plugin:install ai` ile kurulumu tamamlayın.


# Kaldırma

Uygulama eklentisini kaldırmanın da iki yolu vardır:

## Eklenti pazarından kaldırma
[Resmi yönetim paneli webman-admin](https://www.workerman.net/plugin/82) sayfasına gidin, uygulama eklentileri sayfasını açın ve ilgili eklentiyi kaldırmak için kaldırma düğmesine tıklayın.

## Kaynak paketinden kaldırma
`php webman app-plugin:uninstall eklenti_adı` komutunu çalıştırarak kaldırmayı tamamlayın. Ardından `{ana proje}/plugin/` dizinindeki ilgili eklenti klasörünü elle silin.
