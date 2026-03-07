# Yürütme Akışı

## İşlem Başlatma Akışı

`php start.php start` komutu çalıştırıldıktan sonra yürütme akışı şöyledir:

1. config/ altındaki yapılandırmaları yüklemek
2. `pid_file`, `stdout_file`, `log_file`, `max_package_size` vb. Worker ile ilgili yapılandırmaları ayarlamak
3. webman işlemini oluşturup portu dinlemek (varsayılan: 8787)
4. Yapılandırmaya göre özel işlemler oluşturmak
5. webman işlemi ve özel işlemler başlatıldıktan sonra aşağıdaki mantık çalıştırılır (tümü `onWorkerStart` içinde):
   ① `config/autoload.php` içinde tanımlanan dosyaları yüklemek, örn. `app/functions.php`
   ② `config/middleware.php` (dahil `config/plugin/*/*/middleware.php`) içinde tanımlanan middleware'leri yüklemek
   ③ `config/bootstrap.php` (dahil `config/plugin/*/*/bootstrap.php`) içinde tanımlanan sınıfların `start` metodunu çalıştırmak; Laravel veritabanı bağlantısı gibi modülleri başlatmak için
   ④ `config/route.php` (dahil `config/plugin/*/*/route.php`) içinde tanımlanan rotaları yüklemek

## İstek İşleme Akışı

1. İstek URL'sinin public altındaki statik dosyaya karşılık gelip gelmediğini kontrol etmek. Evetse dosyayı döndürmek (isteği sonlandırmak). Hayırsa 2. adıma geçmek.
2. URL'nin bir rotayla eşleşip eşleşmediğini belirlemek. Eşleşmezse 3. adıma; eşleşirse 4. adıma geçmek.
3. Varsayılan rotanın devre dışı olup olmadığını kontrol etmek. Evetse 404 döndürmek (isteği sonlandırmak). Hayırsa 4. adıma geçmek.
4. İsteğe karşılık gelen denetleyicinin middleware'lerini bulmak, sırayla middleware ön işlemlerini çalıştırmak (soğan modeli istek aşaması), denetleyici iş mantığını çalıştırmak, middleware son işlemlerini çalıştırmak (soğan modeli yanıt aşaması) ve isteği sonlandırmak. (Bkz. [Middleware Soğan Modeli](https://www.workerman.net/doc/webman/middleware.html#%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%B4%8B%E8%91%B1%E6%A8%A1%E5%9E%8B))
