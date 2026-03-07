# Qué es webman

Webman es un marco de servicios de alto rendimiento construido sobre Workerman, que integra HTTP, WebSocket, TCP, UDP y otros módulos. Mediante tecnologías como memoria residente, corutinas y grupos de conexiones, Webman no solo supera los cuellos de botella de rendimiento del PHP tradicional, sino que amplía considerablemente sus escenarios de aplicación.

Además, Webman ofrece un potente mecanismo de plugins que permite a los desarrolladores integrar y reutilizar rápidamente módulos funcionales creados por otros. Tanto si es para construir sitios web, desarrollar APIs HTTP, mensajería instantánea, sistemas IoT, juegos, servicios TCP/UDP, servicios Unix Socket u otros, Webman lo aborda con facilidad y ofrece un rendimiento y flexibilidad excepcionales.

# Filosofía de webman
**Máxima extensibilidad y rendimiento con un núcleo mínimo.**

Webman proporciona solo las funciones esenciales (enrutamiento, middleware, sesión e interfaz de procesos personalizados). Todo lo demás se reutiliza del ecosistema Composer, de modo que puedes usar los componentes que ya conoces. Por ejemplo, para bases de datos puedes elegir [illuminate/database](./db/tutorial.md) de Laravel, [ThinkORM](./db/thinkorm.md) de ThinkPHP u otros como `Medoo`. Integrarlos en webman es muy sencillo.

# Características de webman

1. Alta estabilidad. Webman está basado en Workerman, un marco de sockets con muy pocos errores y alta estabilidad en la industria.

2. Rendimiento muy elevado. Webman supera entre 10 y 100 veces a los marcos PHP-FPM tradicionales y rinde aproximadamente el doble que marcos Go como Gin o Echo.

3. Alta reutilización. Puedes reutilizar el ecosistema Composer existente sin modificaciones.

4. Alta extensibilidad. Soporta procesos personalizados y puede realizar todo lo que Workerman permite.

5. Muy fácil de usar, con un coste de aprendizaje bajo y un estilo de código similar al de marcos tradicionales.

6. Soporta [empaquetado binario](./others/bin.md) y puede ejecutarse directamente sin entorno PHP.

7. Utiliza la licencia MIT de código abierto, permisiva y amigable con el desarrollador.

# Enlaces del proyecto
GitHub: https://github.com/walkor/webman **¡No dudes en darle una estrella!**

Gitee: https://gitee.com/walkor/webman **¡No dudes en darle una estrella!**

# Datos de rendimiento de terceros

[![](../assets/img/benchmark1.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

Con consultas a base de datos, Webman alcanza hasta 390.000 QPS en una sola máquina, casi 80 veces más que Laravel con arquitectura PHP-FPM tradicional.

[![](../assets/img/benchmarks-go.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

Con consultas a base de datos, Webman rinde aproximadamente el doble que marcos web similares en Go.

Los datos proceden de [techempower.com](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf).
