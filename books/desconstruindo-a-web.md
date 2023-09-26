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

Uma URI é um identificador de recursos, que pode ser um arquivo, uma imagem, um vídeo, um script, um documento, etc. A URI é composta por um esquema, um host, uma porta, um caminho e um fragmento. Já a URL é um tipo de URI que é composta por um esquema, um host, uma porta e um caminho. Para transformar uma URI em uma URL basta colocar o protocolo http ou https.

## 1.3 ESCOLHENDO O PROTOCOLO

Quando o endereço não tem um protocolo especificado então é usado o HTTP, o navegador tenta buscar o metadado HTTP Strict Transport Security (HSTS) que é um cabeçalho que indica que o site só pode ser acessado por HTTPS, caso o navegador não encontre o cabeçalho ele tenta acessar o site por HTTP.

## 1.4 O CAMINHO ATÉ A REDE

No fluxo de requisição o Chromium divide em vários processos para executar as tarefas, cada processo tem uma função específica, por exemplo, o processo de renderização (é o que vemos) é responsável por renderizar o HTML, CSS e JavaScript, já o processo de rede é responsável por fazer a requisição e receber a resposta do servidor.

A parte de I/O é executada em threads separadas para evitar que uma requisição bloqueie a outra, por exemplo, se uma requisição demorar para ser respondida, o usuário pode fazer outra requisição sem que a primeira esteja finalizada.

## 1.5 O CACHE DA URL
