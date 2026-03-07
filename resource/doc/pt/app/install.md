# Instalação

Existem duas formas de instalar plugins de aplicativos:

## Instalação pelo mercado de plugins
Acesse o [painel de administração oficial webman-admin](https://www.workerman.net/plugin/82), abra a página de plugins de aplicativos e clique no botão de instalação para instalar o plugin correspondente.

## Instalação pelo pacote de código-fonte
Baixe o pacote compactado do plugin no mercado de aplicativos, descompacte e faça o upload do diretório descompactado para `{projeto principal}/plugin/` (crie o diretório plugin manualmente se não existir). Execute `php webman app-plugin:install nome_plugin` para concluir a instalação.

Por exemplo, se o pacote compactado se chama ai.zip, descompacte-o em `{projeto principal}/plugin/ai` e execute `php webman app-plugin:install ai` para concluir a instalação.


# Desinstalação

Da mesma forma, existem duas formas de desinstalar plugins de aplicativos:

## Desinstalação pelo mercado de plugins
Acesse o [painel de administração oficial webman-admin](https://www.workerman.net/plugin/82), abra a página de plugins de aplicativos e clique no botão de desinstalação para desinstalar o plugin correspondente.

## Desinstalação pelo pacote de código-fonte
Execute `php webman app-plugin:uninstall nome_plugin` para concluir a desinstalação. Em seguida, exclua manualmente o diretório do plugin correspondente em `{projeto principal}/plugin/`.
