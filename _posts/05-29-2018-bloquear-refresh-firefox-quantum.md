Dica #1: Bloquear refresh no Firefox Quantum
=============================================

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

E pronto! ;)
