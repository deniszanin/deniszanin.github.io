Configurando Split SSH no Qubes 4
=================================

Introdução
----------

O **Split SSH** é um conceito aplicado ao sistema operacional [Qubes](https://www.qubes-os.org/), parecido com o ***Split GPG***, introduzido e descrito na documentação oficial[^1] do sistema. O modelo de segurança consiste em separar a(s) chave(s) privada(s) do ambiente onde será utilizada, com isso, caso o ambiente seja comprometido, a chave GPG (ou SSH) não será afetada.

Na documentação oficial, encontrei e traduzi o parágrafo seguinte que explica melhor o conceito da divisão:

> **Split GPG** implementa um conceito similar em se ter um *smartcard* com suas chaves GPG privadas, só que a função do *smartcard* fica a cargo de outro ***AppVM*** (*domínio*) do Qubes. Dessa forma, um *domínio* não-muito-confiável, como onde o Thunderbird está sendo executado *[suscetível a ataques]*, por exemplo, poderá delegar funções, como encriptar/decriptar/assinar *[e-mails]* , para outro domínio mais confiável, isolado e sem conexão de Internet. Desta forma o comprometimento do *domínio* onde o Thunderbird (ou outro aplicativo) está rodando - sendo este um cenário possível e provável - não permite que o "malfeitor" roube suas chaves secretas. (*Nós* devemos fazer um comentário um tanto óbvio aqui que, as senhas das chaves privadas são "inúteis", pois um ataque pode instalar facilmente uma *backdoor* simples que aguarda até o usuário digitar sua senha e então roubá-la junto de sua chave.) [^longnote]

O mesmo conceito aplica-se ao **Split SSH**: as chaves públicas e privadas encontram-se armazenadas e configuradas num domínio para o SSH, isolado e sem conexão a Internet; outro domínio, com acesso a Internet, acessará o *VM* (domínio) que está isolado com as chaves. Esta técnica/*hack* foi proposta por Jason Hennessey, [@henn](https://github.com/henn), no projeto [qubes-app-split-ssh](https://github.com/henn/qubes-app-split-ssh) [^2]. A conexão será feita por um *socket* local (criado pelo **ssh-agent** de cada domínio) comunicando-se pelo *framework* do Qubes.

>Isto é feito pela utilização do **qrexec framework** do Qubes para conectar o *socket* local do SSH Agent de um AppVM ao socket do SSH Agent de dentro do VM secreto. [^longnote2]

A diferença entre o **Split GPG** e **Split SSH** dá-se na forma de implementar e configurar o sistema, sendo que o *Split GPG* é nativo, fácil e configurável pelo Qubes, enquanto o *Split SSH* precisa de algumas várias modificações, gerando toda essa dor de cabeça.

Configuração
------------

A princípio, a configuração do sistema, dos domínios e dos arquivos é tranquila, mas confusa se for mal organizada, portanto, recomendo seguir passo-a-passo para evitar qualquer esquecimento. Vamos trabalhar em quatro domínios (VMs), na respectiva ordem, são eles:

* ***fedora-27***: o *TemplateVM* que os domínios de SSH utilizarão como base.
* ***dom0***: o *domínio zero*, principal ambiente do sistema operacional, onde iremos definir as regras de acesso entre-domínios.
* ***ssh-secreto***: domínio isolado, sem acesso a Internet, onde estão instaladas as chaves públicas e privadas.
* ***ssh-cliente***: domínio com acesso a Internet e por onde você irá se conectar ao servidor destino. Neste domínio **NÃO** há qualquer chave (e segredos) instalados.


### 1. Instalar pacotes e *scripts* necessários no(s) *TemplateVM(s)* ###

Este primeiro passo é o mais importante, acho. Demorei muito para descobrir que os pacotes necessários não foram instalados no *TemplateVM* que utilizo: **Fedora 27** , no Qubes 4. Há poucos dias, vi o comentário de [@homotopycolimit](https://github.com/henn/qubes-app-split-ssh/issues/9) descrevendo a instalação dos *packages* e foi então que descobri qual foi o meu erro (depois de muitas horas tentando configurar o **Split SSH)** .

Portanto, agora, iremos focar **apenas** no ==TemplateVM== que será utilizado pelos outros VMs. No meu caso, é o ==fedora-27==, mas isso não significa que com outras imagens o mesmo procedimento não dará certo. O importante é instalar os pacotes e arquivos em todos aqueles que você utilizará para o **Split SSH**, independente da versão do sistema e da imagem utilizada, ok?

Abra o Terminal e instale no *template* ==fedora-27== os pacotes `nmap-ncat` e `openssh-askpass`, com o comando abaixo:

```
$ sudo dnf install nmap-ncat openssh-askpass
```

Se, e somente SE, o *template* que deseja utilizar for ==debian==, então será preciso instalar três pacotes com nomes diferentes do *Fedora*. São eles: `nmap`, `netcat` e `ssh-askpass`.

```
# Para debian, apenas:
$ sudo apt-get install nmap netcat ssh-askpass
```

Assim quando a instalação for concluída, continue com o Terminal aberto no ***fedora-27***, e crie o arquivo `/etc/qubes-rpc/qubes.SshAgent` com o seguinte conteúdo:

```
#!/bin/sh
# Qubes App Split SSH Script
#    for TemplateVM fedora-27
#
notify-send "[`qubesdb-read /name`] SSH agent access from: $QREXEC_REMOTE_DOMAIN"
ncat -U $SSH_AUTH_SOCK
```

A primeira parte está concluída e a configuração necessária foi realizada no *template* **fedora-27** . Seguimos agora para a segunda parte.

### 2. Configurar o **dom0**, o *domínio zero* ###

Agora, foquemos apenas no ==dom0==, o *domínio zero*. Crie e acrescente o seguinte conteúdo, com apenas uma linha, ao arquivo `/etc/qubes-rpc/policy/qubes.SshAgent`.

```
$anyvm $anyvm ask
```

Simples, não!? Pois concluída essa parte, será preciso criar os dois outros VMs restantes (caso ainda não os tenha criado): o ***ssh-cliente*** e o ***ssh-secreto***. Essa tarefa pode ser realizada pelo *Qubes Manager* ou pela linha de comando do **dom0** .

Pela linha de comando, continue com o Terminal do **dom0** aberto, e execute os seguintes comandos para criar o ==ssh-cliente== utilizando **fedora-27** como o *TemplateVM*.

```
$ qvm-create -t fedora-27 -l green ssh-cliente
```

O *VM* será criado tendo o **fedora-27** como *TemplateVM*, na cor de identificação verde, o **sys-firewall** como padrão de conexão a Internet, e com o nome que definimos, ***ssh-cliente*** .

E, por fim, criemos o ==ssh-secreto== com o comando:

```
$ qvm-create -t fedora-27 -l black ssh-secreto
```

E para isolar este *domínio* da Internet, execute:

```
$ qvm-prefs -s ssh-secreto netvm none
```

E pronto! Agora temos todos os *domínios* criados e prontos para a configuração.

### 3. Configurando o ***ssh-secreto*** ###

Lembrando que o ==ssh-secreto== armazenará as chaves públicas e privadas, sem conexão de Internet. Assim, crie as chaves neste *VM* ou copie para cá as chaves que deseja utilizar (as chaves são salvas no diretório `~user/.ssh` do *domínio* ).

Para a configuração, o arquivo necessário é o `~user/.config/autostart/ssh-add.desktop`. Se o diretório `.config/autostart` não existir, aproveite para criar com o comando `mkdir -pv ~user/.config/autostart`.

O arquivo `~user/.config/autostart/ssh-add.desktop` possui o seguinte conteúdo:

```
[Desktop Entry]
Name=ssh-add
Exec=ssh-add
Type=Application
```

Ok?

Para que as alterações sejam aplicadas no ***ssh-secreto*** , **interrompa** a execução do ***ssh-secreto*** pelo *Qubes Manager* . E deixe-o assim, parado (sem estar em execução).

### 4. Configurando o ***ssh-cliente*** ###

E para finalizar a saga da instalação, continuemos com a configuração do *VM* ==ssh-cliente== , que possui conexão de Rede, e irá se conectar com servidores SSH.

Os arquivos abaixo exigem atenção para a variável `SSH_VAULT_VM`, pois é quem dirá qual *domínio/VM* o **ssh-agent** irá se conectar; ou seja, o nome do *domínio* destino onde estão alocadas as chaves.

Acrescente o conteúdo abaixo no arquivo `/rw/config/rc.local` do ==ssh-cliente== . Este *script* será executado quando o *VM* for iniciado.

```
####################
# SPLIT SSH CONFIG
#   para o ssh-cliente VM
#   no arquivo /rw/config/rc.local
#
# Uncomment next line to enable ssh agent forwarding to the named VM
SSH_VAULT_VM="ssh-secreto"

if [ "$SSH_VAULT_VM" != "" ]; then
	export SSH_SOCK=~user/.SSH_AGENT_$SSH_VAULT_VM
	rm -f "$SSH_SOCK"
	sudo -u user /bin/sh -c "umask 177 && ncat -k -l -U '$SSH_SOCK' -c 'qrexec-client-vm $SSH_VAULT_VM qubes.SshAgent' &"
fi
```

Para torná-lo hábil para a executação, defina a permissão apropriada para o arquivo através do comando:

```
$ sudo chmod +x /rw/config/rc.local
```

O último a ser acrescido na configuração do ==ssh-cliente== é o código abaixo. Este deve ser adicionado no final do arquivo `~user/.bashrc`.

```
#####################
# SPLIT SSH CONFIG
#   para o ssh-cliente VM
#
# Append this to ~/.bashrc for ssh-vault functionality
# Set next line to the ssh key vault you want to use
SSH_VAULT_VM="ssh-secreto"

if [ "$SSH_VAULT_VM" != "" ]; then
	export SSH_AUTH_SOCK=~user/.SSH_AGENT_$SSH_VAULT_VM
fi
```

E pronto. A configuração está finalizada.

Para que as alterações sejam aplicadas no ***ssh-cliente*** , faça o mesmo procedimento do *domínio* anterior: **interrompa** a execução do ***ssh-cliente*** pelo *Qubes Manager* ou pela linha de comando. E deixe-o assim, parado (sem estar em execução). A tarefa pode ser feita também pelo Terminal do ==dom0==:

```
$ qvm-shutdown ssh-cliente
$ qvm-shutdown ssh-secreto
```

Conclusão e testes
------------------

A configuração do sistema operacional e dos *domínios* foi finalizada.

O ***Split-SSH*** está pronto para ser utilizado no **Qubes** permitindo o acesso a servidores utilizando diferentes chaves, isoladas e seguras de um *ambiente não-muito-confiável*.

Teste e certifique-se agora, se tudo funciona.

### 1. Primeiro, inicie o *domínio* ***ssh-secreto*** ###

Assim que o *domínio* for iniciado, abra o Terminal neste *VM*; o comando `openssh-askpass` solicitará as senhas das chaves que estão instaladas no diretório `~user/.ssh`; digite-as quando solicitadas.

Para verificarmos se o ***ssh-agent*** está rodando no ==ssh-secreto==, execute:

```
$ eval `ssh-agent -s`
# Retorno: Agent pid 000000
```

### 2. Inicie o *domínio* ***ssh-cliente*** ###

Abra o Terminal no ==ssh-cliente== para checar se o ***ssh-agent*** também está rodando:

```
$ eval `ssh-agent -s`
# Retorno: Agent pid 00000
```

### 3. Testando a conexão entre *domínios* ###

Com os dois *VMs* em execução, digite o comando abaixo no ==ssh-cliente== para testar a comunicação *interVMs*. Se as regras no ==dom0== foram inseridas corretamente, surgirá uma janela requisitando autorização para acesso ao ==ssh-secreto==.

```
$ ssh-add -L
```

O comando deverá listar as chaves instaladas no ***ssh-secreto***. Para finalizar, conecte-se a algum servidor SSH e verifique se o programa funcionará como previsto.

### 4. Erros? ###

Se não funcionar como previsto, tente algumas das soluções abaixo.

* Interrompa a execução dos dois *VMs*, ==ssh-secreto== e ==ssh-cliente==; e inicie novamente seguindo a sequência de, primeiro o ==ssh-secreto== e somente depois, inicie o *domínio* ==ssh-cliente==.
* Nos arquivos do ==ssh-cliente==, verifique se a variável `SSH_VAULT_VM` está definida com o nome corresponde ao *VM* que guarda as chaves.
* No ==ssh-cliente==, certifique-se que o comando `sudo chmod +x /rw/config/rc.local` foi executado.
* No ==ssh-cliente==, execute o script criado anteriormente neste mesmo *domínio*. E procure por erros no retorno do comando: `/rw/config/rc.local`.
* No ==ssh-cliente==, apague o arquivo `~user/.SSH_AGENT_******` e reinicie o *domínio*.
* Acrescente as chaves manualmente no ==ssh-secreto==, com o comando `ssh-add`.
* Para *debugar* a conexão no ==ssh-cliente== com o servidor, acrescente o parâmetro `-v` ao comando `ssh`.

Encontrou algum outro *bug*? Comente!

E fim! ;-)

##### Notas de rodapé #####

[^1]: Documentação oficial sobre Split GPG em https://www.qubes-os.org/doc/split-gpg/

[^2]: Disponível em https://github.com/henn/qubes-app-split-ssh

[^longnote]: Em tradução descomprometida feita por mim do parágrafo:
"Split GPG implements a concept similar to having a smart card with your private GPG keys, except that the role of the “smart card” plays another Qubes AppVM. This way one, not-so-trusted domain, e.g. the one where Thunderbird is running, can delegate all crypto operations, such as encryption/decryption and signing to another, more trusted, network-isolated, domain. This way the compromise of your domain where Thunderbird or another client app is running  arguably a not-so-unthinkable scenario  does not allow the attacker to automatically also steal all your keys. (We should make a rather obvious comment here that the so-often-used passphrases on private keys are pretty meaningless because the attacker can easily set up a simple backdoor which would wait until the user enters the passphrase and steal the key then.)". Disponível na documentação oficial do Qubes, em [https://www.qubes-os.org/doc/split-gpg/](https://www.qubes-os.org/doc/split-gpg/)

[^longnote2]: Em tradução do parágrafo: "This is done by using Qubes's qrexec framework to connect a local SSH Agent socket from an AppVM to the SSH Agent socket within the ssh-vault VM.". Disponível na introdução do projeto [qubes-app-split-ssh](https://github.com/henn/qubes-app-split-ssh) .
