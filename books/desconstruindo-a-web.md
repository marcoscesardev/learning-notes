# [Desconstruindo a Web: As tecnologias por trás de uma requisição]()

por Willian Molinari

# E NO COMEÇO, HAVIA O NAVEGADOR

Imagine que um usuário vai digitar uma URI no navegador, como o software interpreta o hardware? Caso o sistema seja GNU/Linux aqui está o esboço de como isso pode acontecer:

1. Teclado
2. Kernel
3. Driver do teclado
4. Sistema de janelas (xorg, wayland)
5. Gerenciador de janelas (gnome, kde)
6. Navegador

Mas o foco do livro é focar em conceitos da web, então quando a URI é digitada e o enter é pressionado o fluxo dentro do navegado acontece da seguinte forma:

1. Aguarda o unload (navigationStart)

2. (redirectStart) Redirect (redirectEnd)

   2.1. (unloadStart) Unload (unloadEnd)

3. (fetchStart) App cache

4. (domainLookupStart) DNS (domainLookupEnd)

5. (connectStart) (secureConnectionStart) TCP (connectEnd)

6. (requestStart) Request (responseStart) (responseEnd)

7. (domLoading) Processing (domInteractive) (domContentLoaded) (domComplete)

8. (loadEventStart) onLoad (loadEventEnd)

## 1.1 O NAVEGADOR DE ESTUDO

Até então o Chrome é o navegador mais usado no mundo, mais de 57% de usuários, no estudo será usando o Chromium que é navegador de código aberto e que é base para o Chrome.

## 1.2 O QUE É UMA URI?

Uma URI é um identificador de recursos, que pode ser um arquivo, uma imagem, um vídeo, um script, um documento, etc. A URI é composta por um esquema, um host, uma porta, um caminho e um fragmento. Já a URL é um tipo de URI que é para identificar um recurso na web. Em comparação a URI pode ser o nome de uma pessoa e a URL o endereço dela, que além de identificar a pessoa também indica onde ela está. Para tentar deixar mais claro, olhe os exemplos abaixo:

URI (Uniform Resource Identifier):

- mailto:nome@email.com (identifica um endereço de e-mail)
- tel:+55-45-1234-5678 (identifica um número de telefone)
- urn:isbn:978-85-7522-420-4 (identifica um livro pelo ISBN)
- geo:48.8566,2.3522 (identifica uma localização geográfica)

URL (Uniform Resource Locator):

- <https://pt.wikipedia.org/wiki/Brasil>
- <http://www.example.com/imagens/logo.png>
- <ftp://ftp.example.com/arquivos/documento.txt>
- file:///C:/Users/Usuario/Documentos/arquivo.pdf

URI de sites (que também são URLs):

- <https://pt.wikipedia.org/wiki/Brasil>
- <https://www.youtube.com>

URI mas não URLs completa:

- [pt.wikipedia.org/wiki/Brasil]() (falta o protocolo)
- [www.youtube.com]() (falta o protocolo)

## 1.3 ESCOLHENDO O PROTOCOLO

Quando o endereço não tem um protocolo especificado então é usado o HTTP, o navegador faz uma request do tipo HEAD para servidor e busca na resposta o metadado HTTP Strict Transport Security (HSTS) que é um cabeçalho que indica que o site só pode ser acessado por HTTPS, caso o navegador não encontre o cabeçalho ele tenta acessar o site por HTTP.

## 1.4 O CAMINHO ATÉ A REDE

No fluxo de requisição o Chromium divide em vários processos para executar as tarefas, cada processo tem uma função específica, por exemplo, o processo de renderização (é o que vemos) é responsável por renderizar o HTML, CSS e JavaScript, já o processo de rede é responsável por fazer a requisição e receber a resposta do servidor.

A parte de I/O é executada em threads separadas para evitar que uma requisição bloqueie a outra, por exemplo, se uma requisição demorar para ser respondida, o usuário pode fazer outra requisição sem que a primeira esteja finalizada.

## 1.5 O CACHE DA URL

Após descobrir a URL completa o navegador procura cache relacionado a ela. O cache é uma cópia dos arquivos que ficam salvos localmente no computador. Existem formas de avisarem ao navegador que o cache deve ser mantido como os cabeçalhos de _Cache-Control, Expires e Etag_.

Para validar um cache pode ser usado o max-age (parte do cabeçalho de _Cache-Control_), não não haja _Cache-control_, o navegador procura pelo cabeçalho 'Expires' que terá a data de expiração. Quando o cache é dado como expirado mas contém exatamente a mesma response do servidor, então é usado a e-tag com _if-None-Match_, com isso o servidor pode retornar 304 (_Not Modified_).

## 1.6 O NAVEGADOR E A RESOLUÇÃO DE NOME

Para obter o conteúdo do DNS é necessário saber o IP do site. O navegador pode possuir (O Chrome possui) um cache interno com o mapa de domínios para evitar passar essa atividade para algum agente externo e sem precisar ir no sistema operacional

# 2. O SISTEMA OPERACIONAL E A RESOLUÇÃO DE NOMES

Nesse capítulo é explicado como o navegador consegue encontrar o endereço do servidor que foi solicitado e como funciona a função getaddrinfo para traduzir o domínio em um endereço IP.

## 2.1 DEFININDO O SISTEMA OPERACIONAL

Nesse capítulo será usado o GNU/Linux como sistema operacional para o estudo.

### Observação: Por que o GNU/Linux e não só Linux?

Porque o Linux de hoje é a união do kernel Linux com o sistema GNU que foi criado por Richard Stallman.

## 2.2 A `GLIBC` E AS CHAMADAS DE SISTEMA

A `GLIBC` é a libc do GNU, todo sistema Unix precisa de uma libc para funcionar, a `glibc` é responsável por fazer a ponte entre o kernel e o software, e é nela que estão alguns comandos mais primitivos do sistema (printf, malloc, etc). Mas não só comandos primitivos estão nela, um exemplo é o `getaddrinfo`.

## 2.3 A FUNÇÃO QUE RESOLVE NOMES

Antes da getaddrinfo existia a função gethostbyname, os programas usavam ela para retornar o endereço IP de um domínio, mas implementação era diferente para cada versão do protocolo IP.

Um dos passos parar traduizr um domínio é verificar no sistemas de hosts do SO (/etc/hosts).

## 2.4 O PROTOCOLO IP E SUAS VERSÕES

Atualmente existem duas versões, conhecidas por IPv4 e IPv6, a IPv4 tem 32 bits e a IPv6 tem 128 bits. Mas cadê o IPv5? Ele é o protocolo chamado também de ST-II+, ou Internet Stream Protocol, que foi criado para transmitir áudio e vídeo. Ele ficou em fase de testes e foi abandonado.

A necessidade do IPv6 surgiu porque o IPv4 tem um limite de 4 bilhões de endereços, e com a popularização da internet esse número foi se esgotando. O IPv6 tem 128 bits e pode mais de 340 undecilhões de endereços de IP.

Os dois protocolos são diferentes e não são compatíveis, então há sites que possuem uma infra para IPv4 e outra para IPv6.

## 2.5 HAPPY EYEBALLS

O Happy Eyeballs é um algoritmo para deixar a conexão mais rápida, ele tenta conectar com o servidor usando IPv6 e IPv4 ao mesmo tempo, e a primeira requisição que responder terá o protocolo usado. Esse algoitmo é implementado no navegador e no sistema operacional.

## 2.6 RESUMO

Após separa o domínio a ser acessado da URI (remover o protocolo da requisição e o caminho), o navegador precisa procurar pelo endereço IP do domínio. O navegador delega essa função ao SO, através da função getaddrinfo da glibc. A função faz requisições usando IPv4 e IPv6 poara o servidor de DNS. Com o(s) endereço(s) IP o navegador escolhe qual usar atraves do algoritmo Happy Eyeballs.

## 3 RESOLUÇÃO DE NOMES NA REDE
