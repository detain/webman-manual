# Qu'est-ce que webman

Webman est un framework de services haute performance construit sur Workerman, intégrant HTTP, WebSocket, TCP, UDP et d'autres modules. Grâce à des techniques avancées telles que la mémoire résidente, les coroutines et les pools de connexions, Webman dépasse non seulement les limites de performance du PHP traditionnel, mais élargit considérablement ses domaines d'application.

De plus, Webman propose un mécanisme de plugins puissant permettant aux développeurs d'intégrer et de réutiliser rapidement des modules fonctionnels développés par d'autres. Que ce soit pour créer des sites web, des API HTTP, de la messagerie instantanée, des systèmes IoT, des jeux, des services TCP/UDP ou Unix Socket, Webman s'en acquitte avec aisance, offrant performance et flexibilité remarquables.

# Philosophie de webman
**Extensibilité et performances maximales avec un noyau minimal.**

Webman ne fournit que les fonctionnalités essentielles (routage, middleware, session, interface de processus personnalisés). Tout le reste est réutilisé depuis l'écosystème Composer. Vous pouvez donc utiliser vos composants habituels, par exemple [illuminate/database](./db/tutorial.md) de Laravel, [ThinkORM](./db/thinkorm.md) de ThinkPHP ou d'autres comme `Medoo` pour les bases de données. Les intégrer dans webman est très simple.

# Caractéristiques de webman

1. Haute stabilité. Webman repose sur Workerman, un framework socket connu dans l'industrie pour sa stabilité et le peu de bugs qu'il présente.

2. Très hautes performances. Webman dépasse les frameworks PHP-FPM traditionnels de 10 à 100 fois et environ le double des frameworks Go comme Gin ou Echo.

3. Forte réutilisabilité. L'écosystème Composer existant peut être réutilisé sans modification.

4. Grande extensibilité. Il prend en charge les processus personnalisés et peut faire tout ce que Workerman permet.

5. Très simple et facile à utiliser, avec un coût d'apprentissage faible et un code similaire aux frameworks traditionnels.

6. Supporte l'[empaquetage binaire](./others/bin.md) et peut s'exécuter directement sans environnement PHP.

7. Utilise la licence MIT open source, très permissive et adaptée aux développeurs.

# Liens du projet
GitHub: https://github.com/walkor/webman **N'hésitez pas à ajouter une étoile !**

Gitee: https://gitee.com/walkor/webman **N'hésitez pas à ajouter une étoile !**

# Données de performance par des tiers

[![](../assets/img/benchmark1.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

Avec des requêtes base de données, Webman atteint jusqu'à 390 000 QPS sur une seule machine, soit environ 80 fois plus que Laravel avec une architecture PHP-FPM traditionnelle.

[![](../assets/img/benchmarks-go.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

Avec des requêtes base de données, Webman surpasse d'environ deux fois les frameworks web similaires en Go.

Les données proviennent de [techempower.com](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf).
