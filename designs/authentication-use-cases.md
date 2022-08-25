# Casos de uso de autenticação

Como especificação técnica, o Webauthn tem "muitas maneiras" de ser usado que *parece* ser
válido. No entanto, há um conjunto não escrito de combinações que se destinam, e combinações
que pode causar confusão aos usuários ou, na pior das hipóteses, permitir desvios de verificação.

Este documento detalha os casos de uso aos quais oferecemos suporte para entender como devemos estruturar nossa biblioteca
para garantir que nunca possa ser usado/mantido incorretamente.

> NOTA: Esta não pretende ser uma simples introdução ao webauthn, e assumirá extensa
> conhecimento.

## Termos chave

### Credencial não detectável

[credential-id w3c](https://www.w3.org/TR/webauthn-2/#credential-id)

Este é o tipo de credencial "comum" que é usado. Estes são comumente implementados com um esquema
conhecido como "key-wrapped-key". É aqui que o autenticador tem uma única chave privada e criptografa
uma credencial com essa chave. O ID da credencial é a credencial criptografada.

Para que o autenticador então funcione, deve ser apresentado com o id da credencial, que o autenticador
descriptografa e depois usa na cerimônia de autenticação.

### Credenciais detectáveis

[credencial detectável w3c](https://www.w3.org/TR/webauthn-2/#client-side-discoverable-credential)

Uma credencial detectável é anteriormente chamada de chave residente. É quando o *cliente* detém o
capacidade de encontrar credenciais localmente, em vez de exigir que elas sejam fornecidas no arquivo allowedCredentials
definido na autenticação.

### Verificação do usuário

[verificação de usuário w3c](https://www.w3.org/TR/webauthn-2/#user-verification)

O processo de fornecimento de autenticação extra ou suplementar ao autenticador. Isso melhora
a autenticação de "há uma pessoa presente" para "este proprietário específico do autenticador" é
presente. Conseder UV=True porque o autenticador é um dispositivo MFA self container, onde UV=false significa que
o dispositivo é um SFA e outra autenticação é necessária para compor um MFA completo.

No registro armazenamos a política e a verificação que foi fornecida na Credencial interna
tipo para que decisões posteriores possam ser tomadas para a política de segurança.

### Conhecimento Externo

É aqui que o uso de elementos da interação do usuário, fluxo de trabalho ou impressão digital do dispositivo, cookie
ou outro, o cliente ou RP pode decidir sobre qual credencial ou política usar antes
o desafio inicial do webauthn é enviado.

Este fluxo de trabalho pode ser uma página de login do usuário que solicita "como" o usuário deseja autenticar,
qual dispositivo eles querem usar. Também pode se basear no ID do dispositivo e entender que
o autenticador faz parte do dispositivo, por isso devemos dar preferência ao seu uso.

## Fluxos de trabalho de autenticação

### Credenciais homogêneas não detectáveis

Este é um conjunto de credenciais com uma política UV homogênea - todas desencorajadas ou todas necessárias.

Essa política é derivada do conjunto de sinalizadores UV=true/false na lista de credenciais de permissão. Se o
Os sinalizadores UV são inconsistentes e um erro é gerado.

No caso de todas essas credenciais serem desencorajadas, ainda afirmamos que o bit UV corresponde
a autenticação retornada, pois *é* válida para UV=true mesmo quando desencorajada e nós armazenamos isso
na credencial.

#### Exemplo

Um laptop em que o usuário possui várias yubikeys UV=false que podem ser usadas como fator de autenticação.

Um usuário indica que deseja autenticar com seu touchid como um MFA independente. O usuário possui
e registrou vários dispositivos que possuem TouchID para que qualquer um deles possa ser a credencial em uso
sem conhecimento externo.

#### Exemplo detalhado

Isso tem muitas interações com a política de UV no registro e quais os estados e resultados resultantes
são.

| Política UV | UV booleano retornado | Políticas UV válidas para autenticação | Boolean UV sempre verificado na autenticação |
| --------- | ------------------- | -------------------------- | --------------------------------- |
| desanimado | falso | desanimado | Não - o dispositivo não pode fazer UV |
| desanimado | verdadeiro | desencorajado, exigido | Sim - o dispositivo sempre faz UV |
| preferencial | falso | desanimado | Não - o dispositivo não pode fazer UV |
| preferencial | verdadeiro | desencorajado, exigido | Não - o dispositivo pode nem sempre enviar UV no modo desencorajado |
| necessário | falso | - | Não - o dispositivo não pode ser usado |
| necessário | verdadeiro | necessário | Sim - o dispositivo deve sempre executar UV |

A partir deles podemos então construir alguns cenários possíveis.

##### Cenário 1

* Política de registro = desencorajado.
* Um yubikey que não possui pin.
* Touch ID

Durante uma autenticação com política desencorajada, podemos verificar:

* O yubikey envia UV falso
* O touchID envia UV true devido à política de registro = desencorajado + uv true flag.

Se realizarmos uma autenticação com política exigida, somente o dispositivo TouchID poderá participar. Nós então
afirmar UV=true

##### Cenário 2

* Política de registro = preferencial
* Um
