---
title: "Sistema de backups"
date: 2021-02-25T14:55:27-03:00
draft: false
---

Igor Fagundes, LED Internet, 2021

NOTA: A documentação deve ser lida integralmente, não somente por consulta, uma vez que, conceitos que já foram expostos tendem a não serem repetidos ao longo da documentação, logo, é recomendada a leitura integral.

Um sistema que se baseia em: coletar backups, sejam em SSH, Telnet ou outros métodos de diversos tipos de dispositivos e, à partir da coleta dos mesmos, construir um arquivo .zip que pode ser exportado via Dropbox, FTP e armazenamento local.

## Instalação

A instalação segue o modelo padrão da instalação das aplicações em Golang, basta mover o pasta do projeto para dentro da pasta `src/` do `$GOPATH` do seu sistema. No meu caso, que uso o Arch Linux (EndeavourOS) e que baixei o compilador do Golang pelo repositório padrão do sistema, a minha pasta de desenvolvimento é a `~/go`. No meu caso, basta eu baixar os arquivos do projeto, descomprimir (caso esteja comprimido), e mover a pasta do projeto para dentro da pasta `src` do meu diretório de desenvolvimento Go. No meu caso, o resultado final é algo como: `~/go/src/backup-system`. Apesar da explicação, inúmeros tutoriais na internet expoem sobre este assunto, basta pesquisar algo como: `compile go hello world`, `go hello world tutorial` ou `compile golang project`.

Para compilar a aplicação há um shell script chamado `build.sh` na pasta raiz do projeto. Basta executar-lo com `./build.sh` no terminal de sua preferência, estando você, obviamente, na pasta raiz do projeto. Se o sistema acusar algum problema com permissiões, basta executar `chmod +x build.sh`. O sistema foi desenvolvido no Linux, logo, muitas das ferramentas de compilação são exclusivas para o Linux/Unix. Entretanto, você pode usufruir de algo semelhante no Windows por meio do Git Bash ou dos pacotes do MinGW.

Após a compilação com o script `build.sh`, a aplicação ficará na pasta `dist` dentro da pasta do projeto. Lá dentro há as distribuições de Linux e Windows (ambos amd64), entretanto, a versão de Windows não foi devidamente testada. Para fazer uma montagem do aplicação, basta passar os arquivos referentes ao sistema operacional onde ficará hospedada a aplicação para a máquina escolhida. É importante frizar, que assim como na pasta das distribuições tanto o executável quanto os arquivos auxiliares devem ficar na mesma pasta. Ex: após copiar os arquivos da distribuição Linux para uma máquina Linux, os arquivos `make_tmp_dir.sh`, `ftp` e `backup-system` devem ficar na mesma pasta. Após montagem dos arquivos na máquina desejada é necessário configurar a execução da aplicação, algo que veremos à seguir.

## Configuração

A aplicação, ao ser executada lê três arquivos, `config.json` que contém as configurações gerais da aplicação, `devices.json` que contém a lista de dispositivos em que serão buscados os backups, e `exports.json` que contém a lista de lugares onde o arquivo de configuração será exportado.

Campos possíveis do JSON `config.json`:

- "public-ip" -> do tipo string, necessita de ser um IP válido. É o IP público da aplicação, campo necessário. Alguns dispositivos necessitam de um servidor FTP público para enviarem os backups, uma vez que a aplicação cria um servidor FTP temporário para coletar o backup de alguns dispositivos, que é criado logo antes de coletar o backup destes dispositivos e desligado em seguida. Vale ressaltar que o servidor FTP temporário não fica ativo durante toda a execução do programa, mas somente quando está sendo realizada o backup de um dispositivo que necessita de comunicação FTP para a coleta dos backups, e logo após a coleta do backup deste dispositivo, o servidor FTP temporário é desligado.
- "ftp-user" -> do tipo string. É o usuário (caso não queira usar o padrão) que será usado para os dispositivos autenticarem no servidor FTP temporário, campo não necessário.
- "ftp-password" -> do tipo string. É a senha (caso não queira usar a padrão) que será usado para os dispositivos autenticarem no servidor FTP temporário, campo não necessário.
- "ftp-port" -> do tipo string. É a porta (caso não queira usar a padrão) que será usado para os dispositivos se comunicarem com o servidor FTP temporário, campo não necessário. Ou seja é porta em que será criado o servidor FTP temporário.
- "enable-telegram-error-alerts" -> do tipo booleano. Caso o valor seja verdadeiro, o sistema habilitará o envio de erros de execução para o Telegram, caso ocorram, campo não necessário, o padrão é falso, ou seja, caso o campo não exista no arquivo de configuração o envio dos erros ao Telegram será desabilitado.
- "telegram-bot-token" -> do tipo string. O token do bot por onde será feito o envio das mensagens de erro, caso ocorram, campo necessário se o campo "enable-telegram-error-alerts" for verdadeiro.
- "telegram-chat-id" -> do tipo string. O id do chat por onde será feito o envio das mensagens de erro, caso ocorram, campo necessário se o campo "enable-telegram-error-alerts" for verdadeiro.

Campos possíveis para cada objeto do vetor que deverá haver no json `devices.json`:

- "host" -> do tipo string, necessita de ser um soquete de rede válido. É o soquete de rede por onde será acessado o dispositivo, alguns tipos de dispositivos são acessados via SSH, outros via TELNET, logo, deve-se conferir como é realizado o acesso ao dispositivo que deseja que seja efetuado o acesso, campo não necessário se o tipo do dispositivo for "SYSLOG".
- "name" -> do tipo string, necessita de ser um nome válido (que encaixe no REGEX `^([A-Z]|[a-z])([A-Z]|[a-z]|[-]|[_]|[0-9])*$`). É o nome do dispositivo que será efetuado o backup, campo necessário. Esse nome não é o nome do host na rede ou algo semelhante, é, na verdade, um rótulo à ser colocado no dispositivo para ser melhor identificado ao longo do processo de backup. Este nome será o nome do arquivo dentro do .zip referente ao dispositivo em questão, e também o nome que será exibido nos logs e nos erros que a aplicação emitir, lembrando que a aplicação emite este logs ou erros no STDOUT, ou seja, a saída do terminal, e como visto acima é possível enviar os erros, se ocorrerem, para o Telegram.
- "file-name" -> do tipo string, necessita de ser um nome de arquivo válido (que encaixe no REGEX `^([A-Z]|[a-z]|[{]|[.])([A-Z]|[a-z]|[-]|[_]|[:]|[ ]|[0-9]|[.]|[{]|[}])*$`, não confundir com o diretório do arquivo, portanto, não deverá conter o `/`). Nome do arquivo que a aplicação irá buscar caso o tipo do dispositivo for `SYSLOG`, campo nessário somente se o tipo do dispositivo for `SYSLOG`. Este nome poderá conter macros, que serão expostas abaixo.
- "directory" -> do tipo string, necessita de ser um diretório (que encaixe no REGEX `(([\w])*([\\]))*$`, o site ). Diretório do arquivo que a aplicação irá buscar caso o tipo do dispositivo for `SYSLOG`, campo nessário somente se o tipo do dispositivo for `SYSLOG`.
- "type" -> do tipo string, necessita encaixar entre um dos tipos válidos (veja mais a seguir). É o tipo do dispositivo em que será coletado o backup, campo requerido.
- "login" -> do tipo string. O login do dispositivo à ser acessado, campo não requerido se o dispositivo for do tipo `SYSLOG`. Se o dispositivo for de acesso via SSH, a login será para o acesso SSH, se for TELNET, será a login será para o acesso TELNET, e assim sucessivamente.
- "password" -> do tipo string. A senha do dispositivo à ser acessado, campo não requerido se o dispositivo for do tipo `SYSLOG`.
- "enable-password" -> do tipo string. A senha para entrar no modo `enable` do dispositivo à ser acessado, campo requerido se o dispositivo for do tipo `OLT_FIBERHOME` ou `OLT_V_SOLUTION`.
- "check-md5-checksum" -> do tipo booleano. Campo somente usado caso o dispositivo for do tipo `SYSLOG`, caso exista e seja verdadeiro, ao coletar o arquivo a aplicação irá checar um arquivo de nome igual ao nome do arquivo com o acréscimo do sulfixo `.md5` e verá se a hash contida no arquivo com o sulfixo `.md5` é equivalente a checksum md5 do arquivo que não tem o sulfixo `.md5`. Ex: O arquivo à ser coletado possui o nome `teste.txt`, caso o campo "check-md5-checksum" exista e seja verdadeiro (uma vez que, caso não exista a aplicação, por padrão, não irá checar o arquivo com o sulfixo `.md5`) a aplicação irá procurar o arquivo de nome `teste.txt.md5`, coletar a checksum (em formato de texto e não binário) presente no arquivo e irá comparar com a checksum do arquivo `teste.txt`. Caso o campo "check-md5-checksum" exista e seja verdadeiro o arquivo com o sulfixo `.md5` não exista, ou exista e contenha uma checksum não equivalente ao arquivo sem o sulfixo, a aplicação retornará um erro.
- "maintain-original-file-name" -> do tipo booleano. Campo somente usado caso o dispositivo for do tipo `SYSLOG`, caso exista e seja verdadeiro, a aplicação, ao invés de anexar um arquivo no `.zip` com o nome igual ao campo "name" do dispositivo, irá anexar ao .zip um arquivo de nome igual ao campo "file-name". Ex: Caso o dispositivo `SYSLOG` de nome "MEU_ARQUIVO_DE_LOG" e cujo "file-name" seja "logs.txt", ao invés de anexar no `.zip` um arquivo de nome "MEU_ARQUIVO_DE_LOG" irá anexar um arquivo com o nome "logs.txt".

Macros:

Alguns campos suportam macros temporais, estas são de extrema utilidade para, tanto produzir um output com nome personalizado, tanto para buscar arquivos (por meio do dispositivo `SYSLOG`) que tenham uma data específica sem ter a necessidade de ter que alterar todos os dias (ou meses, ou anos dependendo da situação) o arquivo `devices.json`. As macros devem ficar entre `{}`, e os `{}` devem conter dentro de sí somente a macro desejada, espaços ou qualquer outro símbolo não são permitidos. Ex: `{dd}` e `{mm}`. A data que as macros irão expandir é sempre a data do início da execução da aplicação (para evitar erros e comportamentos inesperados evite executar a aplicação na virada do dia), entretanto, é possível dar um offset desta data, ou seja, deslocar ela `x` unidades de tempo da frente ou para traz.

- `{day-offset:42}` -> Desloca a data a ser processada `x` dias para frente ou para traz. Importante frizar que o deslocamento ocorre somente para o processamento do nome em questão, ou seja, não irá afetar o processamento dos outros nomes. O número pode ser tanto positivo quanto negativo. Entre os `:` (dois pontos) e o `}`, deve conter somente o número, espaços e outros símbolos não são permitidos.
- `{d}` -> Retorna o número do dia no formato de uma casa decimal apenas. Ex: `9`, `2` e `3`. Entretanto, se o número do dia possuir mais duas casa decimais, irá retornar as duas casas decimais referentes ao dia em questão.
- `{dd}` -> Retorna o número do dia no formato de duas casas decimais. Ex: `09`, `31` e `10`.
- `{m}` -> Retorna o número do mês no formato de uma casa decimal apenas. Ex: `1` e `3`. Entretanto, se o número do mês possuir mais duas casas decimais, irá retornar as duas casas decimais referente ao mês em questão.
- `{mm}` -> Retorna o número do mês no formato de duas casas decimais. Ex: `09`, `12` e `10`.
- `{mon}` -> Retorna o nome do mês, em inglês, de forma abreviada (com 3 letras) e com a primeira letra em maiúsculo. Ex: `Jan` e `Feb`.
- `{month}` -> Retorna o nome do mês, em inglês, completo e com a primeira letra em maiúsculo. Ex: `January` e `February`
- `{yy}` -> Retorna o ano no formato de duas casas decimais, omitindo, portanto, as duas primeiras casas decimais (lendo do sentido da esquerda para a direita). Ex: `01` e `99`.
- `{yyyy}` -> Retorna o ano no formato de quatro casas decimais.

Os tipos:

- "OLT_V_SOLUTION" -> Dispositivo TELNET. Após autenticar e entrar no modo `enable` executa o comando `show running-config` e extrai o output do comando.
- "SW_HUAWEI" -> Dispositivo SSH. Após autenticar executa o comando `display current-configuration` e extrai o output do comando.
- "MIKROTIK" -> Dispositivo SSH. Após autenticar executa o comando `export` e extrai o output do comando.
- "NEXUS" -> Dispositivo SSH. Após autenticar executa o comando `show running-config` e extrai o output do comando.
- "JUNOS" -> Dispositivo SSH. Após autenticar executa o comando `show configuration` e extrai o output do comando.
- "OLT_FIBERHOME" -> Dispositivo TELNET. Necessita de um servidor FTP com IP público para enviar os dados de backup diretamente para este servidor FTP, logo, a aplicação cria um servidor FTP temporário, e depois coleta arquivo que OLT FIBERHOME enviou ao servidor. Após autenticar executa o comando `upload ftp config [IP_PUBLICO] [USUARIO_FTP] [SENHA_FTP] tmp.cfg`, por padrão o `[USUARIO_FTP]` e a `[SENHA_FTP]`, são `admin` e `123456` respectivamente. Por padrão a aplicação cria um servidor FTP na porta `21`, revisitar a seção que fala acerca da configuração caso deseje alterar o usuário, senha e porta padrões para a comunicação FTP, entretanto pode não funcionar a questão de mudança da porta padrão para a comunicação FTP, uma vez que, não foi devidamente testada. 
- "SYSLOG" -> Ao invés de buscar as configurações para backup nos hosts remotos, a aplicação irá buscar e anexar no .zip um arquivo presente na própria máquina onde a aplicação está hospedada. Este nome é mais por motivos "históricos" e de desenvolvimento, onde criamos este tipo de dispositivo devido a demanda de anexar nos backups syslogs da própria máquina onde está hospedada a aplicação, logo, é possível fazer backup das próprias configurações da aplicação de backup.

Campos possíveis para cada objeto do vetor que deverá haver no json `exports.json`:

- "method" -> do tipo string, necessita encaixar entre um dos métodos de exportação válidos (veja mais a seguir). É o método de exportação referente ao objeto JSON em questão, uma vez que o arquivo `exports.json` deve conter um vetor de objetos, permitindo assim multiplas exportações em apenas uma execução da aplicação, campo requerido.
- "directory" -> do tipo string. É o diretório para onde irá a exportação, campo requerido somente quando o método de exportação for do tipo `DROPBOX`. O diretório, mesmo se vazio, será concatenado com o nome do arquivo (já processado)
- "name" -> do tipo string, necessita de ser um nome de arquivo válido (que encaixe no REGEX `^([A-Z]|[a-z]|[{]|[.])([A-Z]|[a-z]|[-]|[_]|[:]|[ ]|[0-9]|[.]|[{]|[}])*$`, não confundir com o diretório do arquivo, portanto, não deverá conter o `/`). É o nome do arquivo que será exportado, campo requerido.
- "permissions" -> do tipo string, necessita de ser um octal de permissões de arquivo do Unix (padrão utilizado pelo `chmod` entretanto com um `0` na frente, que encaixe no REGEX `^([0])([0-7])([0-7])([0-7])$`). Campo utlizado somente no método de exportação `LOCAL`, a aplicação, ao salvar o arquivo no método de exportação `LOCAL`, irá aplicar os octais presentes no campo "permissions" ao arquivo exportado. Caso deseje saber mais sobre o assunto, basta pesquisar sobre `chmod` e `unix file permissions octal`.
- "token" -> do tipo string. É o token da aplicação do `DROPBOX` por onde a aplicação irá exportar o `.zip`, campo requerido somente no método de exportação `DROPBOX`.
- "host" -> do tipo string, necessita de ser um soquete de rede válido. É um campo utilizado somente no método de exportação `FTP`, e é o soquete do servidor `FTP` por onde a aplicação irá enviar o `.zip`, campo requerido somente no método de exportação `FTP`.
- "login" -> do tipo string. É o login do servidor `FTP` a ser acessado, caso o método de exportação for `FTP`, campo requerido somente no método de exportação `FTP`.
- "password" -> do tipo string. É a senha do servidor `FTP` a ser acessado, caso o método de exportação for `FTP`, campo requerido somente no método de exportação `FTP`.
- "export-md5-checksum" -> do tipo booleano. Caso o campo exista e seu valor seja verdadeiro, exportará além do arquivo `.zip`, um arquivo `.zip.md5`, que conterá o checksum `md5` (em texto, não em binário) do `.zip`. Se, por exemplo, a aplicação for exportar um arquivo de nome `a.zip`, e este campo existir e tiver valor verdadeiro, a aplicação irá calcular a hash `md5` do arquivo `a.zip` e colocará o valor calculado da hash no arquivo `a.zip.md5`, e exportará ambos os arquivos pelo método de exportação referente ao objeto JSON em questão, e do mesmo modo, ou seja, os mesmos campos que moldaram a exportação do `.zip` irão moldar a exportação do `.zip.md5`

Métodos de exportação:

- `LOCAL` -> Exporta o arquivo (ou os arquivos, caso deseje a exportação também do checksum `.zip.md5`) para o armazenamento local da máquina onde está hospedada a aplicação.
- `FTP` -> Realiza a exportação para um servidor `FTP` remoto.
- `DROPBOX` -> Realiza a exportação para o `DROPBOX`, para a integração é necessário haver uma aplicação `DROPBOX` criada no próprio sistema do `DROPBOX` (há materiais na internet que partilham mais a respeito deste assunto), a aplicação (de backups da LED Internet) requisita o token da aplicação para a integração deste método de exportação, conferir mais acima.

## Status

- É possível que na pasta `dist/` haja alguma build destualizada.
- Projeto todo comentado.
- Somente consigo compilar do lado de fora da pasta, quando compilo de dentro recebo o erro:

```
cannot find package "." in:
        /home/ifaresi/go/src/backup-system/vendor/backup-system
```

