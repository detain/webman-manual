# Estrutura de diretórios
```
.
├── app                           Diretório da aplicação
│   ├── controller                Diretório de controladores
│   ├── model                     Diretório de modelos
│   ├── view                      Diretório de visualizações
│   ├── middleware                Diretório de middleware
│   │   └── StaticFile.php        Middleware de arquivos estáticos incorporado
│   ├── process                   Diretório de processos personalizados
│   │   ├── Http.php              Processo Http
│   │   └── Monitor.php           Processo de monitoramento
│   └── functions.php             Funções de negócio personalizadas são escritas neste arquivo
├── config                        Diretório de configuração
│   ├── app.php                   Configuração da aplicação
│   ├── autoload.php              Os arquivos configurados aqui serão carregados automaticamente
│   ├── bootstrap.php             Configuração do callback executado em onWorkerStart ao iniciar o processo
│   ├── container.php             Configuração do contêiner
│   ├── dependence.php            Configuração de dependências do contêiner
│   ├── database.php              Configuração do banco de dados
│   ├── exception.php             Configuração de exceções
│   ├── log.php                   Configuração de log
│   ├── middleware.php            Configuração de middleware
│   ├── process.php               Configuração de processos personalizados
│   ├── redis.php                 Configuração do Redis
│   ├── route.php                 Configuração de rotas
│   ├── server.php                Configuração do servidor (porta, número de processos, etc.)
│   ├── view.php                  Configuração de visualização
│   ├── static.php                Configuração de arquivos estáticos e middleware
│   ├── translation.php           Configuração multilíngue
│   └── session.php               Configuração de sessão
├── public                        Diretório de recursos estáticos
├── runtime                       Diretório de tempo de execução da aplicação, requer permissão de escrita
├── start.php                     Arquivo de inicialização do serviço
├── vendor                        Diretório de bibliotecas de terceiros instaladas pelo Composer
└── support                       Adaptação de bibliotecas (incluindo as de terceiros)
    ├── Request.php               Classe de solicitação
    ├── Response.php              Classe de resposta
    ├── Setup.php                 Script do assistente de instalação
    └── bootstrap.php             Script de inicialização após o início do processo
```
