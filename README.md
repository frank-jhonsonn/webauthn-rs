Webauthn-rs
==========

Webauthn é uma abordagem moderna para autenticação baseada em hardware, consistindo em
um usuário com um dispositivo autenticador, um navegador ou cliente que interage com o
dispositivo, e um servidor capaz de gerar desafios e verificar a
validade do autenticador.

Os usuários podem registrar seus próprios tokens por meio de um processo de registro para
ser associados às suas contas e, em seguida, podem fazer login usando o token
que atua como uma autenticação criptográfica.

Esta biblioteca tem como objetivo fornecer funções e frameworks úteis, permitindo que você
integre webauthn em servidores web Rust. Isso significa que a biblioteca implementa o
Componente de terceira parte confiável do fluxo de trabalho FIDO2. Fornecemos modelo e
exemplos de ligações javascript e wasm para demonstrar as interações do navegador necessárias.

Documentação
-------------

Esta biblioteca consiste em duas partes principais.

Uma API segura e orientada a casos de uso, que é definida em [Webauthn-RS](https://docs.rs/webauthn-rs/)

As interações de nível de protocolo de baixo nível que são definidas em [Webauthn-Core-RS](https://docs.rs/webauthn-core-rs/)

É altamente recomendável que você use a API segura, pois o webauthn tem muitas arestas afiadas e maneiras de segurá-lo errado!

Demonstração
-------------

Você pode testar esta biblioteca em nosso [site de demonstração](https://webauthn.firstyear.id.au/)

Ou você pode executar a demonstração localmente com:

    cd demo_site/webauthn-rs-demo
    corrida de carga

Para opções de configuração adicionais:

    corrida de carga -- --help

Chaves Suportadas Conhecidas/Harwdare
-----------------------------

Testamos extensivamente uma variedade de chaves e dispositivos, não limitados a:

* Yubico 5c / 5ci / FIPS / Bio
* TouchID / FaceID (iPhone, iPad, MacBook Pro)
*Android
* Windows Hello (TPM)
* Softtokens

Se sua combinação de chave/navegador não funcionar (geralmente devido à falta de rotinas de criptografia)
faça um [teste de compatibilidade](https://webauthn.firstyear.id.au/compat_test) e abra
um problema para que possamos resolver o problema!

Chaves QUEBRADAS/Harwdare conhecidas
--------------------------

### Quebrado

* Pixel 3a / Pixel 4 + Chrome - Não envia certificados de atestado corretos,
  e ignora os algoritmos solicitados. Não resolvido.
* Windows Hello com TPMs mais antigos - geralmente usam assinaturas RSA-SHA1 em vez de atestado, o que pode permitir comprometimento/falsificação de credenciais.

### Fixo

* Windows 10 / Windows 11 + Firefox 98 - Quando se refere a aaguid
  para ser 16 bytes de 0, ele emite um único byte 0. Isso deve ser resolvido a partir de 17/04/2022
* BUG no Safari, NÃO Apple Passkeys (era: as senhas não se identificam como uma credencial transferível e devem ser consideradas flutuantes.)

Conformidade com os padrões
--------------------

Esta biblioteca foi cuidadosamente implementada para seguir o padrão w3c para processamento webauthn nível 3
para garantir um comportamento seguro e correto. Apoiamos a maioria das principais extensões e tipos de chave, mas não reivindicamos
para ser reclamação de padrões porque:

* Aplicamos restrições extras na biblioteca que vão além das garantias de segurança oferecidas pelo padrão.
* Não oferecemos suporte a certas opções esotéricas.
* Não suportamos todos os tipos primitivos criptográficos (apenas limitados aos seguros).
* Um grande número de recursos anunciados no webauthn não funciona no mundo real.

Esta biblioteca foi aprovada em uma auditoria de segurança realizada pela segurança do produto SUSE. Outras revisões de segurança
são bem-vindos!

Comentários
--------

O design atual da biblioteca está aberto a comentários sobre como
pode ser melhorado - por favor use esta biblioteca e entre em contato com o projeto sobre o que pode ser
melhorou!

Por que OpenSSL?
------------

Uma pergunta que espero é por que o OpenSSL em vez de algum outro criptográfico puro-ferrugem
provedores. Há duas justificativas principais.

A primeira é que, se essa biblioteca for usada em implantações corporativas ou de grande porte,
então pode ser necessário realizar auditorias criptográficas. É muito mais fácil apontar
em direção ao OpenSSL, que já passou por muito mais revisão e auditoria do que
usando uma série de caixas de ferrugem que (embora ainda sejam ótimas!)
nível de fiscalização.

A segunda é que o OpenSSL é a única biblioteca que encontrei que nos permite
reconstruir uma chave pública EC a partir de seus pontos X/Y ou uma chave pública RSA a partir de seus pontos
n/e para uso com verificação de assinatura.
Sem isso, não podemos analisar as credenciais do autenticador para realizar a autenticação.

Recursos
---------

* Especificação: https://www.w3.org/TR/webauthn-3
* Detalhes JSON: https://fidoalliance.org/specs/fido-v2.0-rd-20180702/fido-server-v2.0-rd-20180702.html
* Anote as interações: https://medium.com/@herrjemand/introduction-to-webauthn-api-5fd1fb46c285
