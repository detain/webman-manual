# Fluxo de execução

## Fluxo de inicialização do processo

Após executar `php start.php start`, o fluxo de execução é o seguinte:

1. Carregar as configurações em config/
2. Definir as configurações do Worker como `pid_file`, `stdout_file`, `log_file`, `max_package_size`, etc.
3. Criar o processo webman e escutar na porta (padrão: 8787)
4. Criar processos personalizados conforme a configuração
5. Após a inicialização do processo webman e dos processos personalizados, executar a seguinte lógica (tudo executado em `onWorkerStart`):
   ① Carregar os arquivos definidos em `config/autoload.php`, como `app/functions.php`
   ② Carregar os middlewares definidos em `config/middleware.php` (incluindo `config/plugin/*/*/middleware.php`)
   ③ Executar o método `start` das classes definidas em `config/bootstrap.php` (incluindo `config/plugin/*/*/bootstrap.php`) para inicializar módulos, como a conexão com o banco de dados Laravel
   ④ Carregar as rotas definidas em `config/route.php` (incluindo `config/plugin/*/*/route.php`)

## Fluxo de tratamento de requisições

1. Verificar se a URL da requisição corresponde a um arquivo estático em public. Se sim, retornar o arquivo (encerrar a requisição). Caso contrário, prosseguir para o passo 2.
2. Determinar se a URL corresponde a alguma rota. Se não corresponder, prosseguir para o passo 3; se corresponder, prosseguir para o passo 4.
3. Verificar se a rota padrão está desabilitada. Se estiver, retornar 404 (encerrar a requisição). Caso contrário, prosseguir para o passo 4.
4. Encontrar os middlewares do controlador correspondente à requisição, executar as operações de pré-processamento dos middlewares em ordem (fase de requisição do modelo de cebola), executar a lógica de negócios do controlador, executar as operações de pós-processamento dos middlewares (fase de resposta do modelo de cebola) e encerrar a requisição. (Consulte o [modelo de cebola dos middlewares](https://www.workerman.net/doc/webman/middleware.html#%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%B4%8B%E8%91%B1%E6%A8%A1%E5%9E%8B))
