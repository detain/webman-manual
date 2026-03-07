# Yapılandırma dosyası

Eklenti yapılandırması normal bir webman projesiyle aynı şekilde çalışır, ancak genellikle yalnızca ilgili eklenti için geçerlidir ve ana projeyi etkilemez.
Örneğin, `plugin.foo.app.controller_suffix` değeri yalnızca eklentinin denetleyici sonekini etkiler, ana projeyi etkilemez.
Örneğin, `plugin.foo.app.controller_reuse` değeri yalnızca eklentinin denetleyiciyi yeniden kullanıp kullanmayacağını etkiler, ana projeyi etkilemez.
Örneğin, `plugin.foo.middleware` değeri yalnızca eklentinin ara katman yazılımını etkiler, ana projeyi etkilemez.
Örneğin, `plugin.foo.view` değeri yalnızca eklentinin kullandığı görünümleri etkiler, ana projeyi etkilemez.
Örneğin, `plugin.foo.container` değeri yalnızca eklentinin kullandığı kapsayıcıyı etkiler, ana projeyi etkilemez.
Örneğin, `plugin.foo.exception` değeri yalnızca eklentinin özel durum işleme sınıfını etkiler, ana projeyi etkilemez.

Ancak yönlendirme küresel olduğu için, eklentinin yapılandırdığı rotalar da küresel yönlendirmeyi etkiler.

## Yapılandırmayı alma
Bir eklentinin yapılandırmasını almak için `config('plugin.{plugin}.{özel_yapılandırma}');` kullanın. Örneğin, `plugin/foo/config/app.php` dosyasının tüm yapılandırmasını almak için `config('plugin.foo.app')` kullanın.
Benzer şekilde, ana proje veya diğer eklentiler de `config('plugin.foo.xxx')` kullanarak foo eklentisinin yapılandırmasını alabilir.

## Desteklenmeyen yapılandırmalar
Uygulama eklentileri `server.php` ve `session.php` yapılandırmalarını desteklemez; `app.request_class`, `app.public_path` ve `app.runtime_path` yapılandırmalarını da desteklemez.
