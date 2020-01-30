Dica #1: Bloquear refresh no Firefox Quantum
=============================================

*(Este texto foi atualizado em janeiro de 2020. As atualizações foram inseridas no final desta breve dica)*

Uma vez, pela primeira vez, dica rápida.

Para prevenir o carregamento automático de páginas - o ***auto refresh*** -, configure o Firefox para bloquear este tipo de requisição do site, impedindo que seu acesso gere *views* para anunciantes e portais. Sites e portais como UOL, Meio & Mensagem, Estadão, Folha, etc., adotam essa estratégia para transformar um único acesso em vários, enquando o usuário navega por um texto.

### Como?

Siga os passos, simples e rápidos.

```
# Acesse na barra de endereço o seguinte endereço
about:config
```

Clique no botão azul (*"I accept the risk!"*, em inglês), aceitando os riscos de alterar a configuração do navegador.

Em seguida, digite e busque pela seguinte opção:

```
# Coluna "Preferences Name"
accessibility.blockautorefresh
```

O objetivo é alterar o valor da última coluna, ***Value*** . Para isso, clique duas vezes na linha correspondente, e defina o valor de *false* para ***true*** .

~~E pronto! ;)~~
E quase... pronto.

---

## Quase... atualização sobre o texto (janeiro/2020):

Infelizmente, os sites e portais que comentei no texto mudaram a forma com que suas respectivas páginas são atualizadas. O *refresh* das páginas é realizada, agora, dinamicamente por *javascript*, o que dificulta a nossa tarefa de bloquear nativamente o *autorefresh* pelo navegador.

A dica no texto acima ***poderá*** funcionar, mas não é regra para **todos** os sites. A opção alterada acima, no **Firefox**, limitará apenas o *refresh* realizado pelas *tags* **META** presentes no *HTML* da página. A dica ainda é válida, e acho que vale a pena deixar a opção do bloqueio ativa.

O usuário ***bodhi***, nos comentários desta página, comentou e alertou sobre a dificuldade em bloquear o *autorefresh* do site **OGlobo**.

> No site do oglobo isso não funciona e o autorefresh continua. Alguma alternativa?

Infelizmente, não. O código *javascript*, inserido dentro de cada página do site, força a sua atualização automática. Este código é um pequeno *timer*: o ***tic-tac*** desse relógio é decrescente, e quando chega a ***0***, a página é carregada novamente, arruinando toda a experiência do usuário com o texto que está lendo.

Exemplo do código *javascript* no site **OGlobo**.

```
<script type="text/javascript">
/**
* Faz o refresh automático das sessoes do site
*/
var timeOutDoReloadAutomatico;
var propriedadeTempoDoRefreshAutomatico = 240000;

function reloadAutomatico(tempoRefreshAutomatico)
{
    if (tempoRefreshAutomatico != undefined)
    {
        window.clearTimeout(timeOutDoReloadAutomatico);
        
        timeOutDoReloadAutomatico = window.setTimeout(function()
        {
            if(tipoConteudoPiano === 'blogAnalitico')
            {
                window.location.href = window.location.origin+"/analitico"
            } else {
                location.reload(true);
            }
        }, tempoRefreshAutomatico);
    }
}
    
document.addEventListener('DOMContentLoaded', function (event) {
    reloadAutomatico(propriedadeTempoDoRefreshAutomatico);
});
</script>
```

### Como resolver?

No **Firefox** - já que estamos falando exclusivamente dele - não existe uma forma fácil e prática - e nativa! -, para desabilitar este trecho específico de código. Para resolver, ofereço três opções a contra-gosto do freguês.

1. Desabilitar o Javascript **completamente** do navegador: funciona, e muito! Utilizo o navegador com o *javascript* desativado em muitos momentos. É verdade que muitos sites deixarão de funcionar completamente, mas para outros, o bloqueio da funcionalidade será libertador: não haverão mais *paywalls* (aquelas famosas telas de *"para continuar lendo, assine..."**), não haverá *autorefresh*, e determinados conteúdos, fechados para assinantes, ficarão disponíveis. Para desativar por completo (em todos os sites), acesse `about:config` (como está descrito no começo do texto), busque e altere a opção `javascript.enabled` de `true` para `false`.
    
2. Desabilitar o Javascript **temporariamente**, enquanto navega por um texto/ página. Essa dica é valiosa e bem prática. Além de prevenir o *refresh* da página, também eliminará o *paywall* da página específica que você está acessando. Descreverei o mesmo processo de duas formas: uma detalhada e outra prática. A versão detalhada funciona da seguinte maneira: acesse o painel de desenvolvimento do Firefox, pressionando a tecla ***F12***; em seguida, clique no ícone, representado por ***três pontos***, localizado no canto superior direito desta janela que se abriu; agora, clique em ***Settings*** (*"Configurações"/ "Ajustes"*, em português); por fim, clique na opção ***Disable Javascript***. E o mesmo método, numa versão mais prática: pressione a tecla ***F12***; em seguida, pressione ***F1***; e clique na opção ***Disable Javascript***.

3. Instalar extensões, de terceiros, no navegador: também funciona, mas estávamos buscando uma opção nativa, certo? Por enquanto, as extensões **NoScript**, **AdBlock**, e outras também possibilitarão ao usuário desativar determinados trechos de códigos *javascript* do *site*. O problema desta alternativa é que os grandes portais realizam uma checagem (também em *javascript*) se há alguma destas extensões instaladas no seu navegador, enquanto a página é carregada.

Para o **Firefox Quantum** (versões 57 e superiores), acredito que estas sejam as opções mais práticas para lutar contra o recarregamento automático das páginas em portais de notícias.

Em outro navegador, no **Google Chrome**, existe uma configuração interessante disponível ao usuário: desabilitar o *javascript* em domínios (*URLs*) específicos, sem que o usuário precise ativá-lo ou desativá-lo a toda hora. Como o texto foi escrito, inicialmente, pensando no **Firefox**, resolvi permanecer no tema deste navegador, mas a configuração a que me refiro no **Chrome** está detalhada na seção de comentários deste texto.

Agora sim, por fim, é isso... e pronto. ;)
