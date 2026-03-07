# Empacotamento

Por exemplo, para empacotar o plugin da aplicação foo:

* Defina o número da versão em `plugin/foo/config/app.php` (**importante**)
* Remova os arquivos desnecessários em `plugin/foo`, especialmente os arquivos temporários de teste da funcionalidade de upload em `plugin/foo/public`
* Se o seu projeto inclui criação de tabelas de banco de dados e operações similares, configure corretamente `plugin/foo/install.sql`. Consulte a [seção de banco de dados](database.md)
* Se o seu projeto tem configuração independente de banco de dados e Redis, remova essas configurações primeiro. Elas devem ser solicitadas através de um assistente de instalação no primeiro acesso à aplicação (você precisa implementar isso), permitindo que o administrador preencha e gere manualmente.
* Se o seu projeto inclui menus de backend do webman admin, configure `plugin/foo/config/menu.php` para que esses menus sejam definidos automaticamente na instalação do plugin. Consulte [webman-admin importar menus](https://www.workerman.net/doc/webman-admin/app-development/menu.html)
* Restaure os outros arquivos que precisam ser restaurados ao estado original
* Após concluir as etapas acima, vá para o diretório `{projeto principal}/plugin/`
* Usuários Linux: execute o comando `zip -r foo.zip foo` para gerar foo.zip
* Usuários Windows: clique com o botão direito na pasta foo e selecione "Comprimir em arquivo ZIP" para gerar foo.zip

**foo.zip é o arquivo empacotado. Consulte o próximo capítulo [Publicar plugin](publish.md)**
