# Ficheiro de configuração

A configuração dos plugins funciona como num projeto webman normal, mas geralmente aplica-se apenas ao plugin em causa e não afeta o projeto principal.
Por exemplo, o valor de `plugin.foo.app.controller_suffix` afeta apenas o sufixo do controlador do plugin e não tem efeito no projeto principal.
Por exemplo, o valor de `plugin.foo.app.controller_reuse` afeta apenas se o plugin reutiliza controladores e não tem efeito no projeto principal.
Por exemplo, o valor de `plugin.foo.middleware` afeta apenas os middleware do plugin e não tem efeito no projeto principal.
Por exemplo, o valor de `plugin.foo.view` afeta apenas as vistas utilizadas pelo plugin e não tem efeito no projeto principal.
Por exemplo, o valor de `plugin.foo.container` afeta apenas o contentor utilizado pelo plugin e não tem efeito no projeto principal.
Por exemplo, o valor de `plugin.foo.exception` afeta apenas a classe de tratamento de exceções do plugin e não tem efeito no projeto principal.

No entanto, como o encaminhamento é global, as rotas configuradas por um plugin também afetam o encaminhamento global.

## Obter a configuração
Para obter a configuração de um plugin, utilize `config('plugin.{plugin}.{config_específica}');`. Por exemplo, para obter toda a configuração de `plugin/foo/config/app.php`, utilize `config('plugin.foo.app')`.
Da mesma forma, o projeto principal ou outros plugins podem utilizar `config('plugin.foo.xxx')` para obter a configuração do plugin foo.

## Configurações não suportadas
Os plugins de aplicação não suportam as configurações `server.php` e `session.php`, nem as configurações `app.request_class`, `app.public_path` e `app.runtime_path`.
