---
title: "Lead machine"
date: 2021-02-25T14:55:27-03:00
draft: false
---

Igor Fagundes, LED Internet, 2021

NOTA: A documentação deve ser lida integralmente, não somente por consulta, uma vez que, conceitos que já foram expostos tendem a não serem repetidos ao longo da documentação, logo, é recomendada a leitura integral.

Este projeto consiste em um pacote que contém uma página para captar novos leads, e uma aplicação que deverá ser hospedada em um servidor para receber a solicitação da página e enviar os dados do `lead` captado por meio de uma mensagem para o Telegram.

## Instalação

### Página (pasta `www`)

Para a página foi utilizada uma linguagem de marcação chamada `pug` (pugjs). Feita para ser uma linguagem de marcação mais amigável e poderosa do que o HTML, o compilador de pug pode ser baixado por meio da `npm` que é o gerenciador de pacotes da linguagem Javascript, que é a linguagem de programação em que `pug` foi escrita. O motor de Javascript `node.js`, que irá dar suporte ao compilador de pug, e seu gerenciador de pacotes, o `npm`, estão disponíveis para a maioria dos sistemas operacionais, pelo menos dos grande utização. O compilador de pug, irá ler o arquivo de extensão `.pug` e gerar um `.html` correspondente. Basicamente o compilador irá tranformar um arquivo descrito na linguagem de marcação pug em um arquivo descrito na linguagem de marcação HTML, um vez que, os browsers (pelo menos a grande maioria deles) não suportam o pug para descrição das páginas web.

Após a compilação, ou seja, a geração do `index.html`, basta copiar a logo e o `index.html` para o servidor http em que deseje. 

Para instalarmos o `node.js`, o `npm` e o `pug`, respectivamente, no caso dos sistemas Linux, basta baixarmos o `node` e o `npm` via gerenciador de pacotes, e instalar o `pug` por meio do `npm` por meio do comando `npm install pug -g` e o comando `npm install pug-cli -g` (é necessário executar ambos os comandos). No caso do Arch Linux (EndeavourOS), que é o sistema operacional por onde eu desenvolvi a aplicação, é possível baixar o `node` e o `npm`, respectivamente, pelos comandos: `sudo pacman -S nodejs` e `sudo pacman -S npm`. Nota: atente-se às versões necessárias do `node.js` e `npm` na documentação do pugjs. O projeto utilizou estar versões do pug:

```
pug version: 2.0.4
pug-cli version: 1.0.0-alpha6
```

### Aplicação server-side

A instalação segue o modelo padrão da instalação das aplicações em Golang, basta mover o pasta do projeto para dentro da pasta `src/` do `$GOPATH` do seu sistema. No meu caso, que uso o Arch Linux (EndeavourOS) e que baixei o compilador do Golang pelo repositório padrão do sistema, a minha pasta de desenvolvimento é a `~/go`. No meu caso, basta eu baixar os arquivos do projeto, descomprimir (caso esteja comprimido), e mover a pasta do projeto para dentro da pasta `src` do meu diretório de desenvolvimento Go. No meu caso, o resultado final é algo como: `~/go/src/lead-machine`. Apesar da explicação, inúmeros tutoriais na internet expoem sobre este assunto, basta pesquisar algo como: `compile go hello world`, `go hello world tutorial` ou `compile golang project`.

## Configuração

### Página

É necessário referenciar a aplicação server-side na página, logo, é preciso editar a linha `128` colocando a url da aplicação server side do arquivo `.pug` antes da compilação.

### Aplicação server-side

Foi escolhido o uso de `variáveis de ambiente` e não de arquivos de configuração devido ao fato de ser mais natural o trabalho com `variáveis de ambiente` nos ambientes de cloud computing, permitindo assim, a alteração de diversos parâmetros de execução da aplicação sem a alteração do repositório da aplicação. Segue abaixo a lista das variáveis de ambiente requeridas pela aplicação.

- TELEGRAM_BOT_TOKEN -> O token do bot do Telegram por onde serão enviados os dados do novo `lead`.
- TELEGRAM_CHAT_ID -> O id do chat do Telegram por onde serão enviados os dados do novo `lead`.
- PORT -> A porta TCP por onde a aplicação à nível de servidor ficará hospedada, obviamente, deve ser a mesma porta em que o página enviará a solicitação. O atributo de ser uma string em que há um número, sem espaços ou símbolos. Ex: `"4000"` e `"5678"`.

## O deve ser feito e refeito

- Implementar o recurso do envio do lead via email.
- Gerenciar sobrecarga de envio, uma vez que, o envio via Telegram pode recusar o envio em caso de muitos envios em sequência.
