---
title: "LED Services"
date: 2021-02-25T14:55:27-03:00
draft: false
---

Igor Fagundes, LED Internet, 2021

NOTA: A documentação deve ser lida integralmente, não somente por consulta, uma vez que, conceitos que já foram expostos tendem a não serem repetidos ao longo da documentação, logo, é recomendada a leitura integral.

O intuito desta aplicação é gerenciar diversos sistemas e APIs que estão integradas à LED Internet. A ideia básica da aplicação é iterar sobre cada cliente do banco de dados e à partir desta iteração realizar operações diversas, como: mandar um email, inserir os dados do cliente em outro sistema. Atualmente o sistema realiza duas funções: mandar um email para os clientes instalados (que ainda não receberam este email) sobre as credenciais do `LED Prime`, e sincronizar os clientes da LED Internet registrados no Hubsoft com o sistema do `Clube Certo`, removendo clientes que estão com algum (não é prática ideal) serviço inativo, e adicionando o cliente que está com algum serviço ativo (método que necessita de ser refeito).

## Instalação

A instalação segue o modelo padrão da instalação das aplicações em Golang, basta mover o pasta do projeto para dentro da pasta `src/` do `$GOPATH` do seu sistema. No meu caso, que uso o Arch Linux (EndeavourOS) e que baixei o compilador do Golang pelo repositório padrão do sistema, a minha pasta de desenvolvimento é a `~/go`. No meu caso, basta eu baixar os arquivos do projeto, descomprimir (caso esteja comprimido), e mover a pasta do projeto para dentro da pasta `src` do meu diretório de desenvolvimento Go. No meu caso, o resultado final é algo como: `~/go/src/led-services`. Apesar da explicação, inúmeros tutoriais na internet expoem sobre este assunto, basta pesquisar algo como: `compile go hello world`, `go hello world tutorial` ou `compile golang project`.

Para executar a aplicação é necessário criar um script de `run` para a aplicação, algo que veremos mais a seguir, uma vez que a aplicação opera com base no valor de diversas `variáveis de ambiente`, logo, necessitamos de criar algum arquivo que, antes de executar a aplicação, crie este ambiente personalizado. Pelas `variáveis de ambiente` a aplicação irá coletar informações importantes para execução da aplicação, como a token de acesso ao Hubsoft e a `URI` do banco de dados de cache da aplicação, por exemplo.

Se o seu desejo é somente produzir um executável da aplicação, basta somente executar o comando `go build led-services`, sendo `led-services` neste comando o nome da pasta do projeto dentro da pasta `src` dentro do `$GOPATH`, logo, se mudar o nome da pasta do projeto dentro da pasta `src`, basta, no entanto, executar o comando `go build nome_da_pasta`, sendo `nome_da_pasta`, obviamente, o nome da pasta do projeto dentro da pasta `src`. De todo modo, a compilação da aplicação segue o modelo padrão da compilação das aplicações em Golang. A sugestão de perquisa acima irá expor mais sobre este assunto.

## Configuração

Foi escolhido o uso de `variáveis de ambiente` e não de arquivos de configuração devido ao fato de ser mais natural o trabalho com `variáveis de ambiente` nos ambientes de cloud computing, permitindo assim, a alteração de diversos parâmetros de execução da aplicação sem a alteração do repositório da aplicação. Segue abaixo a lista das variáveis de ambiente requeridas pela aplicação.

- HUBSOFT_KEY -> A token de acesso à API do Hubsoft.
- LED_PRIME_PACKAGES -> O nome dos pacotes (no banco de dados do Hubsoft) que indentificam um cliente que possui o serviço do `LED Prime`, como as variáveis de ambiente não possuem um método padrão para a criação de arranjos e vetores, os nomes dos pacotes são separados por vírgula. Entre as vírgulas devem conter somente o nome dos pacotes, espaços adicionar e outros símbolos não são permitidos. Ex: `Cartoon Network,Esporte interativo,Esporte interativo ++`
- CLUBE_CERTO_PACKAGE -> O nome do pacote que indentifica um cliente que possui o serviço do `Clube Certo`.
- CLUBE_CERTO_USER -> O nome de usuário `Clube Certo` da LED Internet, o `Clube Certo` deve fornecer este dado.
- CLUBE_CERTO_TOKEN -> O token de acesso `Clube Certo` da LED Internet, o `Clube Certo` deve fornecer este dado.
- CLUBE_CERTO_KEY -> A key de acesso `Clube Certo` da LED Internet, o `Clube Certo` pode fornecer este dado. Na verdade este dado foi obtido por nós por meio de alguns processos na API do `Clube Certo`, entretanto é prudente dizer que o `Clube Certo` pode ajudar a conseguir este dado, tambem chamado por eles de "número de associado". 
- TELEGRAM_BOT_TOKEN -> O token do bot do Telegram por onde serão enviadas as mensagens de erro, caso ocorram.
- TELEGRAM_CHAT_ID -> O id do chat do Telegram por onde serão enviadas as mensagens de erro, caso ocorram.
- MONGO_URI -> A URI de acesso ao database MongoDB. A aplicação utiliza uma database MongoDB como um sistema de cache remoto. O MongoDB tem ótima velocidade de escrita e leitura, mesmo nos modelos `M0` do MongoDB Atlas, e a utlização de um cache remoto habilita a transferência do cache de uma instância (ou máquina que esteja hospedada a aplicação) desta aplicação para uma a outra de forma bem fácil, bastando somente a troca dos dados de integração com o MongoDB nas variáveis de ambiente. A aplicação consegue fazer `cold start` no database, ou seja, aplicação consegue pegar um cluster recém criado no MongoDB Atlas, sem ter havido a criação de índices, e utilizar de maneira adequada.
- MONGO_DATABASE -> O nome do database do MongoDB que será utilizado, irá sobrescrever a informação de qual database será usada contida no campo MONGO_URI.
- CNPJ -> Cnpj da LED Internet, com a máscara (seguindo o modelo `00.000.000/0000-00`). O `Clube Certo` necessita deste dado para algumas requisições.
- SMTP_USERNAME -> Username para autenticação no servidor SMTP que fará envio dos emails. Atualmente, é usado para o envio dos emails contendo as credenciais de acesso ao `LED Prime`, para que os clientes que possuem o serviço ativo, e que, obviamente, ainda não receberam o email com credenciais anteriormente.
- SMTP_PASSWORD -> Senha para autenticação no servidor SMTP que fará envio dos emails.
- SMTP_HOST -> Ip ou domínio do servidor SMTP que fará envio dos emails. As empresas que provém serviço de SMTP, provavelmente, lhe informarão facilmente este dado referente ao seu serviço contrato, se mensionar que deseja o atributo `host`. 
- SMTP_PORT -> É a porta por onde no `host` STMP, ou seja, no servidor SMTP, a aplicação irá comunicar e solicitar o envio do email. As empresas que provém serviço de SMTP, provavelmente, lhe informarão facilmente este dado referente ao seu serviço contrato, se mensionar que deseja o atributo `porta`. 
- SMTP_FROM -> É o endereço de email que será visivel para o cliente quando ele receber um email proveniente da aplicação, mais precisamente um email fornecendo as credenciais do `LED Prime`. Obviamente, não é possível informar qualquer endereço de email neste campo, o seu servidor SMTP tem que possuir a autorização para enviar emails sobre este endereço. Alguns serviços de SMTP já disponibilizam ao usuário um domínio em que ele poderá realizar o envio dos emails, outros, no entanto, não. Logo, para estes ultimos é necessário que eu, possuindo um domínio na internet, autorize o meu domínio a ser usado pelo servidor SMTP que eu contratar. Geralmente as empresas de SMTP possuem instruções para como realizar este processo, perguntas como: "Tenho um domínio proveniente de uma hospedagem que contratei por outra empresa, quero ver como uso este domínio no servidor SMTP de vocês" podem agilizar o processo. E claro, neste processo tome cuidado para não conflitar com algum servidor de email que já está utilizando o domínio seu, para estes casos, vale a pena criar subdomínios especialmente para o envio em massa via SMTP. Comumente os emails enviados pela aplicação, podem conter na visualização nas caixas de entrada, além do endereço contido no campo SMTP_FROM o endereço contido em SMTP_HOST, com SMTP_HOST em menor destaque, somente indicando que foi por meio deste servidor que o email foi enviado.
- SENDED_EMAILS_LIMIT -> Máximo de emails que podem ser enviados por execução da aplicação. Este é um mecanismo de segurança para evitar o envio demasiado de emails em apenas uma execução. Necessita de ser um número inteiro, entretanto, obviamente, uma string que contém um número inteiro (exclusivamente o número, espaços e outros símbolos não são permitidos) uma vez que todas variáveis de ambiente, pelo menos na maioria dos sistemas e ambientes, somente podem ser do tipo string.

## O deve ser feito e refeito

- A aplicação necessita de ser refatorada completamente, até refeita utilizando a biblioteca `httpigeon` presente no meu `github` (`github.com/igorifaresi/jda`).
- Pode-se modularizar a aplicação, e colocar um sistema de plugins e linkagem dinâmica, caso for oportuno.
- Uma interface web pode ser útil, com dashboards para acompanhar os status das execuções.
- Adicionar a necessidade de uma variável de ambiente para ditar o título do email que conterá as credenciais para o acesso ao `LED Prime`.
- Macros nos títulos dos emails.
- Talvez um modo de personalizar o corpo do email que contém as credenciais para o acesso ao `LED Prime`.
- Remover clientes cancelados do banco de dados.
