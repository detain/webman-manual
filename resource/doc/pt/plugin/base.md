# Plugins básicos

Os plugins básicos são geralmente componentes comuns que são tipicamente instalados via Composer, com o código colocado no diretório vendor. Durante a instalação, configurações personalizadas (middleware, processos, rotas, etc.) podem ser copiadas automaticamente para o diretório `{projeto principal}config/plugin`. O webman reconhecerá automaticamente a configuração deste diretório e a fundirá com a configuração principal, permitindo que os plugins intervenham em qualquer fase do ciclo de vida do webman.

Para mais informações, consulte [Criação de plugins básicos](create.md).
