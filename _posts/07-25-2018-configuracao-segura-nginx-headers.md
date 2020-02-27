Configuração segura dos cabeçalhos no Nginx
============================================

***Nota:** Este texto foi atualizado em 22/02/2020 para incluir dúvidas de outros usuários sobre a configuração dos cabeçalhos e a solução para estas dúvidas.*

Enquanto pesquisava sobre segurança digital, vasculhando em vários sites pelo Google, encontrei aleatoriamente o *blog* de [Scott Helme](https://scotthelme.co.uk/). Por este, descobri que é autor de um projeto de *cybersecurity*, cujo objetivo é analisar e avaliar gratuitamente o nível de segurança de um determinado site, baseando-se ***apenas*** nos cabeçalhos do servidor *(HTTP response headers)*. O projeto está disponível em [https://securityheaders.io/](https://securityheaders.io/).

Ao realizar o teste por aqui, descobri que a minha configuração definida no ***Nginx*** estava com vários cabeçalhos mal configurados. Resolvi, então, escrever este tutorial de análise e correção dos cabeçalhos *(somente para **Nginx**; no Google você encontrará os passo-a-passos para outros servidores)*.

Configuração
-------------

#### 1. Análise ####

Acesse [https://securityheaders.io/](https://securityheaders.io/) e realize um teste, digitando o endereço do site que deseja analisar (selecione a opção ***Hide results*** para esconder a *URL* do site busca da página principal do projeto).

No meu primeiro teste, recebi uma nota **D** em ***Report Security Summary***, ou seja, muito vulnerável a diversos ataques. O *header* ***HTTP Strict Transport Security (HSTS)*** era o único cabeçalho configurado corretamente por aqui, pois eu havia configurado *SSL* anteriormente, com um *script* de automação e configuração de servidor que criei (postarei isso depois no [Github](https://github.com/deniszanin)). O restante dos *headers* estavam todos vermelhos neste teste, nenhum configurado: `Content-Security-Policy`, `X-Frame-Options`, `X-XSS-Protection`, `X-Content-Type-Options`, `Referrer-Policy`, `HTTP Public Key Pinning` e `Feature-Policy`.

`Content-Security-Policy` e `X-XSS-Protection` são exemplos de *headers* que evitarão ataques, como o ***Cross site scripting (XSS)*** , e sua implementação ajudará a proteger os usuários do seu site, além dele próprio. Na [CryptoRave](https://cryptorave.org/) de 2015 eu palestrei sobre a customização do Firefox, pensando na privacidade e segurança do usuário, e em uma parte dela comento sobre diversos ataques (e casos reais!) citados aqui, que são proferidos durante a navegação. 

Se ficou curioso(a) sobre a palestra, o vídeo na íntegra está publicado no [YouTube](https://www.youtube.com/watch?v=JGBU3s9BLD4), e pode ser visto aqui (a fala dos *ataques virtuais* começa em ***21m05s*** do vídeo).

Agora, voltando do desvio de assunto e indo ao tópico do *post* sobre a configuração... então, o quê fazer?

#### 2. Configuração ####

Para começar, edite o arquivo de configuração do **Nginx**, localizado em `/etc/nginx/nginx.conf`. Lembrando que dependendo do seu sistema e estrutura organizacional, os arquivos de configuração poderão encontrar-se em diferentes diretórios.

O objetivo desse texto é priorizar *apenas* a configuração dos cabeçalhos, portanto suponho que o seu servidor ***Nginx*** está com o **SSL** e **certificados** devidamente configurados e funcionais.

#### a. *HTTP Strict Transport Security (HSTS)* ####

O **HTTP Strict Transport Security**, ou *HSTS*, é essencial para incluir em seu site. A funcionalidade fortalecerá a conexão entre *usuário-site*, adicionando uma camada extra de criptografia TLS nesta conexão, para usar *HTTPS*.

Caso ainda não esteja configurado em seu servidor, configure o *HTTP Strict Transport Security*. Acrescente a linha abaixo no arquivo:

```
add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";
```

#### b. *Content Security Policy* ####

O **Content Security Policy**, ou *CSP*, é uma medida efetiva contra ataques *Cross Site Scripting (XSS)*, através deste cabeçalho são definidas as fontes de conteúdos aprovados que serão carregados no site, para evitar (*whitelisting*) que o navegador acesse elementos não-autorizados. Exemplos deste ataque foram ditos no vídeo da palestra anexada acima.

Para isso, acrescente a linha abaixo ao arquivo de configuração do ***Nginx***.

```
add_header Content-Security-Policy "default-src 'self'";
```

Ou, um exemplo mais completo na linha abaixo com maiores restrições (e que possivelmente irá quebrar toda a estrutura do site).[^1]

```
add_header Content-Security-Policy "upgrade-insecure-requests; block-all-mixed-content; default-src 'self' https; style-src 'self'; img-src 'self'; script-src 'self'";
```

#### c. *X-Frame Options* ####

**X-Frame Options** dirá ao navegador se o site poderá, ou não, ser inserido dentro de um *frame* (interno, ou externo). Inicialmente, pode não fazer sentido, mas há diversos ataques com o nome de *clickjacking* rolando por aí. O *roubo de clicks* acontece quando o usuário clica em um botão ou link, sem intenção do clique. O valor `SAMEORIGIN` na linha abaixo, determinada que o *frame* será aceito se a origem (*domínio*) for a mesma que a do site.

```
add_header X-Frame-Options SAMEORIGIN;
```

---

***Atualização sobre o uso do X-Frame Options (22/02/2020)***

Como já citei, o **X-Frame-Options** dirá ao navegador se o site permite, ou não, ser inserido em um *frame* (ou *iframe*). Este *header* aceita dois valores: `DENY`, para bloquear toda requisição em *frames*; ou, como no exemplo, `SAMEORIGIN`. 

Esta semana, o usuário David entrou em contato para tirar uma dúvida sobre um terceiro valor para o *header* **X-Frame-Options**: `ALLOW-FROM`. Este valor permitia especificar quais *URIs* poderiam abrir um *frame*; uma lista de domínios *seguros* e autorizados a requisitar *frames*. Mesmo definindo de forma correta na configuração do ***Nginx***, David não estava conseguindo liberar seus *URIs* específicos.

O problema que encontramos foi que o valor `ALLOW-FROM` está **obsoleto**. De acordo com a documentação da **Mozilla**[^5], *headers* definidos com este valor são ignorados pelos navegadores *modernos*. Os navegadores que **não** suportam este valor de cabeçalho, `ALLOW-FROM` são: **Chrome** (todas as versões), **Firefox** (versão 18-70), **Opera** (todas as versões), **Safari** (todas as versões), enquanto `SAMEORIGIN` é amplamente aceito pelos navegadores.[^6]

##### Então... qual a solução?

Para resolver o problema de David, precisamos configurar um outro *header*: **Content-Security-Policy**. A descrição sobre este cabeçalho já foi detalhada neste texto, mas faltou dizer sobre um valor: o `frame-ancestors`.[^7]

Para permitir requisições de *frame, iframe, object, embed ou applet*, de domínios específicos, não vamos usar o `ALLOW-FROM`, que está obsoleto, e vamos configurar o cabeçalho **Content-Security-Policy** com o valor `frame-ancestors`. Exemplo:

```
add_header Content-Security-Policy "frame-ancestors 'self' outrodominio.com;";
```

Lembrando que `'self'` refere-se ao próprio domínio.
Ou, uma outra configuração possível:

```
add_header Content-Security-Policy "frame-ancestors seudominio.com outrodominio.com;";
```

Para incluir estes valores de configuração juntamente com outros valores do **Content-Security-Policy**, separe-os com um ponto-e-vírgula, `;`, como no exemplo do item **b** deste texto.

O valor `frame-ancestors` é amplamente aceito pelos navegadores modernos com exceção do *Internet Explorer*.

Mas... o *IE* é um navegador moderno?
Deixo a pergunta no ar... 

---

#### d. *X-XSS-Protection* ####

**X-XSS-Protection** dirá ao navegador para habilitar, ou não, o filtro de *Cross-Site Scripting*, ou seja, se o navegador do usuário poderá relacionar *scripts* entre outros domínios. 

```
add_header X-XSS-Protection "1; mode=block";
```

#### e. *X-Content-Type-Options* ####

**X-Content-Type-Options** é o *header* para evitar que os navegadores obtenham uma leitura superficial do tipo de um arquivo. O *"tipo de um conteúdo"*, o **MIME**, é avaliado previamente pelo navegador[^2]. Exemplo: um referencial no site para uma *stylesheet* será avaliado pelo navegador, e este não será carregado **a não ser que** o tipo (MIME) seja compatível com "text/css". 

```
add_header X-Content-Type-Options nosniff;
```

#### f. *Referrer-Policy* ####

**Referrer-Policy** é o *header* para controlar a informação que será enviada como referencial para outro *link* destino. Ou seja, o referencial origem num clique para o destino do clique. Para a especificação abaixo, restringimos o referencial: o referencial de origem, proveniente de https, de um clique será enviado para o destino *https*; a origem não será enviada se o destino for *http*, sem conexão segura.

```
add_header Referrer-Policy "no-referrer-when-downgrade";
```

#### g. *HTTP Public Key Pinning (HPKP)* ####

**HTTP Public Key Pinning** protege seu site de ataques MiTM, *man-in-the-middle*, quando há "alguém-no-meio-do-caminho", um *Big Brother* monitorando suas ações e simulando conexões seguras (ataques bem comuns em *wi-fi* públicas)[^3].

As identidades dos certificados (ou as *assinaturas digitais*) são definidas explicitamente no cabeçalho, dizendo ao navegador do usuário quais certificados **SSL** o navegador deverá confiar naquele site, desta forma, ainda que exista "alguém-no-meio-do-caminho" monitorando as conexões seguras do usuário, as ***assinaturas digitais*** dos certificados estão claramente especificadas e o navegador não irá confiar naquela conexão.

Antes de acrescentar o cabeçalho ao **Nginx** é preciso recuperar a identidade dos certificados, sua assinatura digital. Para isso, substitua `SEU_CERTIFICADO_AQUI`, no comando abaixo, com o caminho do arquivo do seu certificado (o valor da opção `ssl_certificate`, definida no seu arquivo de configuração).

```
openssl x509 -in SEU_CERTIFICADO_AQUI -pubkey -noout | openssl pkey -pubin -outform der | openssl dgst -sha256 -binary | openssl enc -base64
```

A execução desse comando resultará em algo como `vrqHdf0qoWVMi01inbU4HLRBLBe9Lx2JCkyU12suxLA=`, e que deverá ser acrescido ao código do cabeçalho abaixo. Este resultado é a assinatura do seu certificado em ***sha-256***, e depois transformada em uma *string* em ***base64***.

```
add_header Public-Key-Pins 'pin-sha256="RESULTADO_AQUI"; includeSubdomains; max-age=10';
```

E **atenção**: sempre que o seu certificado for renovado, este cabeçalho no **Nginx** deverá ser atualizado com a nova assinatura digital. Isso pode ser automatizado com um *script*, como fiz por aqui, ao gerar os certificados do ***Let's Encrypt***.

#### h. *Feature-Policy* ####

**Feature-Policy** é o mais novo *header*, que permite ao site controlar quais funcionalidades e APIs poderão ser usadas pelo navegador do usuário. Na linha abaixo está um exemplo bem restritivo[^4].

```
add_header Feature-Policy "geolocation 'none'; midi 'none'; notifications 'none'; push 'self'; sync-xhr 'none'; microphone 'none'; camera 'none'; magnetometer 'none'; gyroscope 'none'; speaker 'none'; vibrate 'self'; fullscreen 'self'; payment 'self'";
```

Finalizando
-----------

Depois de acrescentar os 8 (oito!!) cabeçalhos ao arquivo de configuração do **Nginx**, execute o comando `nginx -t` para conferir se não há erros nos arquivos modificados.

Depois do teste de sintaxe, reinicie o serviço do servidor:

```
# Para ubuntu/ debian:
systemctl restart nginx.service
```

Agora realize novamente o teste de avaliação dos cabeçalhos em [https://securityheaders.io/](https://securityheaders.io/). Depois destas alterações descritas aqui, o resultado da avaliação foi **A+**.

E fim! ;-)

##### Notas de rodapé #####

[^1]: Em inglês, *Content Security Policy (CSP)
Quick Reference Guide*, [fonte externa](https://content-security-policy.com/).

[^2]: Em inglês, *Reducing MIME type security risks*, [fonte externa](https://docs.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/compatibility/gg622941(v=vs.85)).

[^3]: Em inglês, *Optimising NGINX and Server Security*, [fonte externa](https://gregorykelleher.com/nginx_security).

[^4]: Em inglês, *A new security header: Feature Policy*, [fonte externa](https://scotthelme.co.uk/a-new-security-header-feature-policy/).

[^5]: Em inglês, *MDN web docs: X-Frame-Options*, [fonte externa](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)

[^6]: A tabela completa de suporte dos navegadores está disponível na documentação do X-Frame-Options (em inglês), [fonte externa](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)

[^7]: Em inglês, *MDN web docs: CSP: frame-ancestors*, [fonte externa](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors)

