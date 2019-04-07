Dica #2: descompactar arquivos MSI e DMG no Linux
=================================================

De uma vez, pela segunda vez, dica rápida.

Não é a primeira, nem é a segunda vez que preciso buscar como faço para descompactar arquivos com extensão **MSI** (aquele tipo de arquivo "Microsoft-padrão" para instalar programas no Windows). Mas por que descompactar? Simples: eu não tenho instalado Windows em nenhum ambiente nos quais trabalho, e para ter acesso aos arquivos do instalador e analizar o conteúdo dos executáveis, é necessário descompactá-los.

Esta solução é ainda mais interessante quando o mesmo comando, `7z` - que usaremos para descompactar **.msi* -, também pode ser usado para arquivos **DMG**, padrão do sistema da Apple, OS X.

### Como?

`7zip` resolverá a questão dos dois tipos de arquivo. Digite a linha de comando abaixo, por exemplo.

```
7z x nome_do_arquivo.msi
```

E para entendê-la melhor: `7z` é o comando; `x`, parâmetro para designar a extração do seu conteúdo; e `nome_do_arquivo.msi`, para o nome do arquivo (dãh!).

### Bytes e bytes

Extendendo um pouco mais a conversa, podemos conhecer o tipo de um determinado arquivo que baixamos, mesmo que ele não tenha extensão alguma definida (com a extensão omitida), ou seja, sem o *.msi* ou *.dmg*. O comando `file` é justamente o comando que identificará a que tipo de arquivo se trata. Por exemplo: `file nome_do_arquivo`; no caso de um arquivo MSI, o comando retornará, `Os: Windows, Version 10.0, MSI Installer`.

A maioria dos arquivos possuem uma assinatura digital, ou seja, uma forma com que aquele arquivo seja reconhecido e aceito no sistema ou programa ao qual foi destinado, sem que uma extensão correta seja definida (em análise forense utilizamos muito). Para isso, haverá no seu conteúdo (entre bytes e mais bytes), uma marcação no cabeçalho do arquivo. Por exemplo, no cabeçalho do arquivo `nome_do_arquivo.msi` que acabamos de identificar pelo comando `file`, este terá grafado 8 bytes iniciais, correspondendo a sua assinatura: `d0 cf 11 e0 a1 b1 1a e1`. No caso da extensão DMG, encontramos apenas 1 byte para identificá-lo: `78`.

Explicado e pronto! ;)
