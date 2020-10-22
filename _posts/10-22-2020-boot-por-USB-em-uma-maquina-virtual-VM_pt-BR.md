# Dica #3: boot por USB em uma máquina virtual (VM)

De uma vez, mais uma vez, dica rápida.

Por esses dias, precisei instalar um sistema operacional em uma máquina virtual **VMWare**, na qual trabalho de vez em quando. A mídia de instalação estava em uma imagem, no formato **ISO**, pois eu não estava encontrando o CD/DVD nessas bagunças de pandemia, por isso resolvi utilizar a *ISO* que salvei de *backup*. 

Depois de criar a máquina virtual, e configurá-la para inicializar por esta imagem *ISO* de instalação, percebi que, por algum motivo, a **BIOS** da máquina virtual não estava reconhecendo a mídia como *bootável* (uma possível falha na construção dos *headers* do *boot*, durante o backup). Seguidas tentativas fracassadas depois, foi então que pensei: por que não criar um USB, a partir desta imagem *ISO*, e inicializá-lo pelo assistente de configuração do *VMWare*? 

Criei o USB, coloquei no computador. Meu sistema operacional reconheceu a mídia no USB, e... mais outro problema: através das opções do *VMWare* não era possível definir o USB como uma mídia física de *boot*.

Pesquisando e pesquisando (e pesquisando!), encontrei uma alternativa para meu problema, bem recorrente com usuários do *VMWare*. A alternativa seria inicializar a máquina virtual (**VM**, *"virtual machine"*), e definir um gerenciador de *boot* como primeira imagem a ser carregada. Este gerenciador chama-se **Plop Boot Manager**.

Explicação rápida para quem não entendeu: por padrão, criamos uma máquina virtual pelo *VMWare*, em seguida definimos nas configurações o CD/DVD (ou imagem de um CD/DVD) que deverá ser inicializado pela *BIOS* da máquina virtual, certo? Ok, mas e se quisermos iniciar a máquina virtual através de um USB? Eis aí o meu problema, e a alternativa é o **Plop Boot Manager**.

### *Plop Boot Manager*, gerenciador de *boot* da Plop

O Plop (nome da empresa, de um funcionário só; como o *CEO*, e autor do projeto, diz) é um gerenciador de *boot*. Por este gerenciador, conseguimos selecionar o dispositivo por onde o sistema será inicializado/ *"bootado"*. Dentre as opções, estão partições do HD, CD/DVD, e USB!

Para utilizar o Plop na máquina virtual, é bem simples: baixe o arquivo **zipado** com os arquivos do gerenciador em seu computador, selecione a imagem *ISO* do gerenciador no *VMWare*, e inicie a máquina virtual.

A imagem (e arquivo necessário) do *Plop Boot Manager* está disponível na página oficial do projeto, acesso por este [link](https://www.plop.at/en/bootmanager/download.html). O arquivo compactado para *download* se chama `plpbt-5.0.15.zip`. Esta é a versão 5.0.15, lançada em 2013 (embora datada de vários anos, fique tranquilo, a versão é estável). 

Feito o download do arquivo `plpbt-5.0.15.zip`, descompacte o conteúdo em uma pasta, e, no assistente de configuração do *VMWare*, onde definimos a "imagem de CD/DVD", indique o arquivo `plpbt.iso`. Esta é a imagem do gerenciador. Desta forma, o primeiro passo na inicialização da máquina virtual será abrir o gerenciador de *boot* **Plop**. Antes, porém, vamos alterar outras configurações desta máquina virtual: na aba *Options ("Opções")*, configuração *General ("Geral")*, altere o sistema da máquina virtual para *Other ("Outro"), e versão, para Other ("Outro")*, também. Habilite a controladora USB e verifique se os dispositivos estão inseridos corretamente em seu computador, na máquina *host*.

### *Bootando* o **Plop** na máquina virtual

Agora, é bem fácil: depois das configurações definidas, inicie a máquina virtual, aguarde pela tela "espacial" aparecer, e *voilá*, o gerenciador de *boot* abriu! Precisamos "inserir" agora, o USB na máquina virtual: nas opções desta **VM**, habilite o USB correspondente ao drive onde está o sistema operacional que deseja instalar. Feito? Reinicie a máquina virtual, e de volta a tela inicial do **Plop**, escolha dentre as opções, o USB. E pronto! 

Outra dica, no entanto: paciência! 
A velocidade de leitura ficará comprometida e leeeenta.

Explicado e pronto! ;)

#### Post Scriptum

*P.S.: No final das contas, toda a **ISO** que eu estava tentando instalar estava completamente corrompida, e de nehuma forma consegui finalizar a instalação do sistema operacional. O lado bom do problema: descobri o **Plop** e estou repassando a dica para vocês. Este gerenciador de boot é uma excelente carta na manga para os problemas de BIOS e sistemas operacionais. A resolução do meu problema foi correr atrás da bagunça e encontrar o CD/DVD. Paciência sempre!*
