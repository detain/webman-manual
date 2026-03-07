# O que é o webman

Webman é um framework de serviços de alto desempenho construído sobre Workerman, que integra HTTP, WebSocket, TCP, UDP e outros módulos. Com tecnologias como memória residente, corrotinas e pools de conexão, o Webman não só supera os gargalos de desempenho do PHP tradicional, como também amplia bastante os cenários de aplicação.

Além disso, o Webman oferece um poderoso mecanismo de plugins que permite aos desenvolvedores integrar e reutilizar rapidamente módulos funcionais criados por outros. Seja para construir sites, desenvolver APIs HTTP, mensagens instantâneas, sistemas IoT, jogos, serviços TCP/UDP, Unix Socket ou outros, o Webman lida com tudo isso com facilidade, oferecendo desempenho e flexibilidade notáveis.

# Filosofia do webman
**Máxima extensibilidade e desempenho com um núcleo mínimo.**

O webman fornece apenas as funcionalidades essenciais (roteamento, middleware, sessão, interface de processos personalizados). Todo o resto é reutilizado do ecossistema Composer. Assim, você pode usar os componentes que já conhece: por exemplo, para banco de dados, pode escolher [illuminate/database](./db/tutorial.md) do Laravel, [ThinkORM](./db/thinkorm.md) do ThinkPHP ou outros como `Medoo`. Integrá-los ao webman é bem simples.

# Características do webman

1. Alta estabilidade. O webman é baseado no Workerman, um framework de sockets muito estável e com poucos bugs na indústria.

2. Alto desempenho. O webman supera os frameworks PHP-FPM tradicionais em 10 a 100 vezes e fica cerca do dobro do desempenho de frameworks Go como Gin e Echo.

3. Alta reutilização. O ecossistema Composer existente pode ser reutilizado sem alterações.

4. Alta extensibilidade. Suporta processos personalizados e pode fazer tudo o que o Workerman permite.

5. Muito simples e fácil de usar, com baixo custo de aprendizado e estilo de código semelhante aos frameworks tradicionais.

6. Suporta [empacotamento binário](./others/bin.md) e pode ser executado diretamente sem ambiente PHP.

7. Utiliza a licença MIT open source, muito permissiva e amigável ao desenvolvedor.

# Links do projeto
GitHub: https://github.com/walkor/webman **Não hesite em dar uma estrela!**

Gitee: https://gitee.com/walkor/webman **Não hesite em dar uma estrela!**

# Dados de benchmark de terceiros

[![](../assets/img/benchmark1.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

Com operações de consulta a banco de dados, o webman atinge até 390.000 QPS em uma única máquina, cerca de 80 vezes mais que o framework Laravel com arquitetura PHP-FPM tradicional.

[![](../assets/img/benchmarks-go.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

Com operações de consulta a banco de dados, o webman apresenta cerca de duas vezes o desempenho de frameworks web similares em Go.

Os dados acima são de [techempower.com](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf).
