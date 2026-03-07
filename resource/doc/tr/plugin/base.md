# Temel eklentiler

Temel eklentiler genellikle ortak bileşenlerdir, tipik olarak Composer ile kurulur ve kod vendor dizininin altında bulunur. Kurulum sırasında özel yapılandırmalar (middleware, işlemler, rotalar vb.) `{ana proje}config/plugin` dizinine otomatik olarak kopyalanabilir. webman bu dizindeki yapılandırmayı otomatik olarak tanır ve ana yapılandırmayla birleştirir, böylece eklentiler webman'ın yaşam döngüsünün herhangi bir aşamasına müdahale edebilir.

Daha fazla bilgi için [Temel eklenti oluşturma](create.md) bölümüne bakın.
