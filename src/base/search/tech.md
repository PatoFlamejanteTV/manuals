# Do Raso ao Fundo, Introdução aos Princípios de Busca

Esta vez trazemos uma introdução ao negócio de busca de e-commerce, a série de busca de e-commerce é dividida em dois artigos:

- Introdução ao Negócio de Busca em E-commerce
- Do Raso ao Fundo, Introdução aos Princípios de Busca

> Este artigo toma como exemplo o mecanismo de busca de código aberto ES (Elasticsearch), doravante referido como ES.

Primeiro, para os alunos que estão entrando em contato pela primeira vez, este artigo envolverá muitos conceitos, principalmente os seguintes:

Conceito de Termo de Busca|Descrição
------|------
Documento (Doc)|?
Termo (Term)|?
Índice Invertido (Inverted Index)|?
Palavra-chave (Query)|?
Recall (Recall)|?
Frequência do Termo (tf:Term Frequency)|?
Frequência Inversa do Documento (idf:Inverse Document Frequency)|?
Ordenação Bruta (Rough Sort/Coarse Rank)|?
Ordenação Fina (Fine Sort/Fine Rank)|?

Este artigo introduz os princípios de busca do simples ao complexo e gradualmente revela esses conceitos. A estrutura deste artigo é a seguinte:

- O nascimento do mecanismo de busca ES
- Processo de busca versão simplificada
    + Processo de indexação
    + Processo de consulta
- Processo de busca versão avançada
    + Processo de indexação
        * O que é um Documento (Doc)
        * O que é um Termo (Term)
        * O que é um Índice Invertido (Inverted Index)
        * Análise de Documento (Doc)
            - Filtro de caracteres
            - Tokenizador (Analisador Léxico)
            - Filtro de token
        * Criar índice invertido
    + Processo de consulta
        * Análise de Palavra-chave (Query)
            - Filtro de caracteres
            - Tokenizador
            - Filtro de token
        * Recall (Recall)
            - O que é Recall
        * Ordenação (Ranking)
            + O que é Frequência do Termo (tf:Term Frequency)
            + O que é Frequência Inversa do Documento (idf:Inverse Document Frequency)
            + Ordenação Bruta/Ordenação Fina
    + Resumo do processo de busca
- Mecanismo de busca ES avançado
    + Estrutura básica do índice (substantivo)
    + Estrutura lógica do mecanismo de busca ES

## O nascimento do mecanismo de busca ES

O ES nasceu de uma biblioteca JAVA de código aberto `Lucene`. Através da descrição do site oficial do `Lucene`, podemos descobrir que o `Lucene` possui as seguintes capacidades:

- `Lucene` é uma biblioteca JAVA
- `Lucene` implementa verificação ortográfica
- `Lucene` implementa destaque de caracteres correspondentes (highlighting)
- `Lucene` implementa funções de análise e tokenização

Capacidades que o `Lucene` não possui:

- Distribuído
- Alta disponibilidade
- Pronto para uso (Out of the box)
- Etc.

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220215203335.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220215203335.png" style="width:66%">
    </a>
</p>

Então, muitos anos atrás, um desenvolvedor chamado `Shay Banon` implementou um mecanismo de busca distribuído de código aberto e alta disponibilidade `Elasticsearch` através do `Lucene`.

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220215203346.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220215203346.png" style="width:66%">
    </a>
</p>

As funções de busca comuns são baseadas em "mecanismos de busca". Em seguida, vamos ver o **processo de busca versão simplificada**.

## Processo de busca versão simplificada

O processo de busca versão simplificada é o seguinte:

- Passo ①: Processo de indexação, os dados de origem que precisam ser pesquisados são indexados (verbo) no mecanismo de busca, e um índice (substantivo) é estabelecido.
- Passo ②: Processo de consulta, o usuário insere uma palavra-chave (Query), o mecanismo de busca analisa a Query e retorna o resultado da consulta.

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220306221130.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220306221130.png" style="width:39%">
    </a>
</p>

## Processo de busca versão avançada

### Processo de indexação

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220129183509.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220129183509.png" style="width:50%">
    </a>
</p>

#### O que é um Documento (Doc)

Por exemplo, se o conteúdo da página da web "Manual de Design de E-commerce | SkrShop" precisa ser pesquisado, todo o conteúdo desta página da web é chamado de um `Documento Doc`.

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220222131248.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220222131248.png" style="width:80%">
    </a>
</p>

O conteúdo do `Documento Doc` é o seguinte:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Manual de Design de E-commerce | SkrShop</title>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="description" content="Deve ser o manual de design de sistema de e-commerce mais completo, detalhado e prático">
  <!-- Omitido... -->
  <p>Seckill é um meio de marketing de e-commerce</p>
  <!-- Omitido... -->
```

Conceito de Termo de Busca|Descrição
------|------
Documento (doc)|O conteúdo de texto que precisa ser pesquisado, pode ser informações detalhadas de um produto, informações de uma página da web, etc.

#### O que é um Termo (Term)

Continue com o `Documento Doc` acima como exemplo. Para simplificar o entendimento de `Termo (Term)`, simplifique o conteúdo do `Documento Doc` acima para uma frase `Seckill é um meio de marketing de e-commerce`.

`Termo (Term)` é o conjunto de resultados de termos obtidos após o processamento de tokenização do `Documento Doc`. Por exemplo, `Seckill é um meio de marketing de e-commerce` após a tokenização em chinês (ou outra língua) resulta em:

```
Seckill / é / e-commerce / de / um / marketing / meio
```

Seckill, é, e-commerce, de, um, marketing, meio são chamados respectivamente de `Termo (Term)`, e esse conjunto é chamado de `Terms`.

Conceito de Termo de Busca|Descrição
------|------
Termo (Term)|O texto Doc pesquisado é desmontado em N frases padrão pelo tokenizador.

#### O que é um Índice Invertido (Inverted Index)

"Índice Invertido" é a implementação específica do índice (substantivo) criado ao indexar (verbo) os dados de origem.

Usamos o seguinte Documento (Doc) como exemplo para explicar o índice invertido:

ID do Documento|Conteúdo do Documento (Doc)
------|------
1|Manual de Design de E-commerce SkrShop
2|Seckill é um meio de marketing de e-commerce
3|O carrinho de compras é a etapa mais importante do processo de compra de e-commerce

Tokenizador: Documento (Doc) desmontado em múltiplos termos independentes (Doc -> Terms).

Tokenizadores chineses de código aberto:

- IK Analyzer
- jieba
- Etc.

Tomando a demonstração online do tokenizador jieba como exemplo: https://app.gumble.pw/jiebademo/

ID do Documento|Conteúdo do Documento (Doc)|Resultado da Tokenização (Terms)
------|------|------
1|Manual de Design de E-commerce SkrShop|E-commerce / Design / Manual / SkrShop
2|Seckill é um meio de marketing de e-commerce|Seckill / é / E-commerce / de / um / marketing / meio
3|O carrinho de compras é a etapa mais importante do processo de compra de e-commerce|Carrinho de compras / é / E-commerce / compra / processo / mais / importante / de / uma etapa

Os IDs de documentos correspondentes a cada termo são os seguintes:

Termo|IDs de Documentos
------|------
E-commerce|1, 2, 3
Design|1
Manual|1
SkrShop|1
Seckill|2
é|2, 3
de|2, 3
um|2
marketing|2
meio|2
Carrinho de compras|3
compra|3
processo|3
mais|3
importante|3
uma etapa|3

Este é o processo básico de criação de um índice invertido.

Após a criação do índice invertido, simulamos o processo de busca, supondo:

- Buscar `E-commerce`, pode encontrar rapidamente os documentos 1, 2, 3
- Buscar `Marketing`, pode encontrar rapidamente o documento 2

(Este processo é chamado de "Recall")

Isso é o conceito de "Índice Invertido".

Conceito de Termo de Busca|Descrição
------|------
Índice Invertido (Inverted Index)|A implementação específica do índice (substantivo) criado ao indexar (verbo) os dados de origem.

#### Análise de Documento (Doc)

A análise é o processo de padronização do texto do Documento (Doc) e o processo de conversão do Documento (Doc) em Termos (Term) padronizados. A implementação do processo de análise do mecanismo de busca ES depende do analisador.

Composição básica do analisador:

- Filtro de caracteres
- Tokenizador
- Filtro de token

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220129183541.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220129183541.png" style="width:60%">
    </a>
</p>

##### Filtro de caracteres

> Um analisador corresponde a um filtro de caracteres.

Formata para texto padrão (text -> standard text), por exemplo, removendo tags html do texto.

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220129183701.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220129183701.png" style="width:60%">
    </a>
</p>

Por exemplo `<p>Manual de Design de E-commerce SkrShop</p>` ---> `Manual de Design de E-commerce SkrShop`

##### Tokenizador

> Um analisador corresponde a um tokenizador.

O processo de desmontar o Documento (Doc) em múltiplos termos independentes (doc -> terms). Por exemplo:
Por exemplo `Manual de Design de E-commerce SkrShop` ---> `E-commerce / Design / Manual / SkrShop`

O que também precisa ser mencionado aqui é o **Dicionário Personalizado**: vocabulário que o dicionário original não possui, como novos vocabulários da internet criados recentemente.

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220129183714.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220129183714.png" style="width:60%">
    </a>
</p>

##### Filtro de token

> Um analisador corresponde a N filtros de token.

- Conversão unificada para minúsculas
- Conversão de sinônimos
- Stop words (Palavras de parada)
- Extração de radical (Stemming)
- Correção de erros
- Autocompletar
- Etc...

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220215205418.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220215205418.png" style="width:60%">
    </a>
</p>

Filtro de token|Exemplo
------|------
Conversão unificada para minúsculas|Adequado para inglês, etc. Por exemplo, converter letras inglesas para minúsculas, ex `Word -> word`
Conversão de sinônimos|Adequado para vários idiomas. Ex `espaçoso -> amplo`
Stop words|Adequado para vários idiomas. Remover palavras com significado amplo e não representativo e outras palavras designadas manualmente para parar, ex `de`, `é`. Dicionário de stop words em chinês: https://github.com/goto456/stopwords
Extração de radical|Adequado para inglês, etc. Ex `words -> word`
Correção de erros|Adequado para vários idiomas. Ex `espaçoso -> espaçoso (corrigindo grafia)`
Autocompletar|Adequado para vários idiomas.

#### Resumo do processo de indexação

### Processo de consulta

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220215205523.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220215205523.png" style="width:30%">
    </a>
</p>

Conceito de Termo de Busca|Descrição
------|------
Palavra-chave (Query)|A palavra-chave inserida pelo usuário ao iniciar a busca

#### Análise de Palavra-chave (Query)

A Palavra-chave (Query) também precisa passar pelo `analisador`, e é o mesmo `analisador` do processo de indexação de documentos.

Mesmo analisador:

- Mesmo filtro de caracteres
- Mesmo tokenizador
- Mesmo filtro de token

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220220211920.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220220211920.png" style="width:60%">
    </a>
</p>

Tokenizador:

Palavra-chave (Query)|Resultado da Tokenização (Terms)
------|------
Design do sistema de seckill|Seckill / sistema / de / design

|Termo (Terms)|
|------|
|Seckill|
|sistema|
|de|
|design|

Filtro de token:

Aqui tomamos o filtro de token de stop words como exemplo para explicar o processo do filtro de token. Exemplo de dicionário de stop words usado neste artigo: https://github.com/goto456/stopwords/blob/master/cn_stopwords.txt

Conjunto de termos (Terms) obtido após a remoção da stop word `de`:

|Termo (Terms)|
|------|
|Seckill|
|sistema|
|design|

#### Recall (Recall)

##### O que é Recall

Usando o conteúdo do documento e o resultado da tokenização do documento acima:

ID do Documento|Conteúdo do Documento (Doc)|Resultado da Tokenização (Terms)
------|------|------
1|Manual de Design de E-commerce SkrShop|E-commerce / Design / Manual / SkrShop
2|Seckill é um meio de marketing de e-commerce|Seckill / é / E-commerce / de / um / marketing / meio
3|O carrinho de compras é a etapa mais importante do processo de compra de e-commerce|Carrinho de compras / é / E-commerce / compra / processo / mais / importante / de / uma etapa

Usando ainda o filtro de token para filtrar os resultados da tokenização, tomando o mesmo filtro de token de stop words como exemplo. Exemplo de dicionário de stop words usado neste artigo: https://github.com/goto456/stopwords/blob/master/cn_stopwords.txt

Por exemplo, acertou a stop word `é`:
  
<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220302203921.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220302203921.png" style="width:39%">
    </a>
</p>

Resultado após passar pelo filtro de token de stop words:

ID do Documento|Conteúdo do Documento (Doc)|Resultado da Tokenização (Terms)
------|------|------
1|Manual de Design de E-commerce SkrShop|E-commerce / Design / Manual / SkrShop
2|Seckill é um meio de marketing de e-commerce|Seckill / E-commerce / um / marketing / meio
3|O carrinho de compras é a etapa mais importante do processo de compra de e-commerce|Carrinho de compras / E-commerce / compra / processo / importante / uma etapa

Obtendo ainda a estrutura de índice invertido:

Termo|IDs de Documentos
------|------
E-commerce|1, 2, 3
Design|1
Manual|1
SkrShop|1
Seckill|2
um|2
marketing|2
meio|2
Carrinho de compras|3
compra|3
processo|3
importante|3
uma etapa|3

Em seguida, simulamos o processo de busca, supondo que o usuário pesquise `Design do sistema de seckill`:

Palavra-chave (Query)|Resultado da Tokenização (Terms)
------|------
Design do sistema de seckill|Seckill / sistema / de / design

|Termo (Terms)|
|------|
|Seckill|
|sistema|
|de|
|design|

Filtro de token, usando o mesmo `filtro de token de stop words` do processo acima como exemplo, obtém-se o conjunto de termos (Terms) após a remoção da stop word `de`, chamado de conjunto de termos da Palavra-chave (Query):

|Termo (Terms)|
|------|
|Seckill|
|sistema|
|design|

- O termo `Seckill` da Palavra-chave (Query), através do índice invertido acima, pode-se encontrar facilmente o `Documento 2`
- O termo `sistema` da Palavra-chave (Query), através do índice invertido acima, não encontrou nenhum documento
- O termo `design` da Palavra-chave (Query), através do índice invertido acima, pode-se encontrar facilmente o `Documento 1`

Assim, a pesquisa do usuário por `Design do sistema de seckill` encontrou os seguintes documentos:

- `Documento 2`: Seckill é um meio de marketing de e-commerce
- `Documento 1`: Manual de Design de E-commerce SkrShop

O processo acima é `Recall`.

Conceito de Termo de Busca|Descrição
------|------
Recall (Recall)|O processo pelo qual o mecanismo de busca usa o índice invertido para obter documentos relevantes através de termos.

No processo de recall acima, o usuário encontrou os documentos 1 e 2 pesquisando `Design do sistema de seckill`.

```
Complemento: O acima é baseado no método de recall de texto de índice invertido. Além disso, existem métodos de recall baseados na mesma categoria, outros atributos semelhantes, e recall de vetor baseado em aprendizado profundo.
```

Então vem a pergunta:

> Documentos 1 e 2 recuperados, quem vem antes e quem vem depois, como a ordem é decidida?

A seguir, falaremos sobre a implementação da ordenação do mecanismo de busca.

#### Ordenação

Introduzindo a pergunta acima:

> Documentos 1 e 2, quem vem antes e quem vem depois, como a ordem é decidida?

Resposta: Decidida pela relevância do documento, o mecanismo de busca pontuará (score) a relevância do documento. Geralmente, os dois principais indicadores que determinam essa pontuação score são:

- Taxa de termo no documento: tf (Term Frequency)
- Taxa inversa de documento: idf (Inverse Document Frequency)

Pode-se entender simplesmente que score de relevância = taxa de documento * taxa inversa de documento, quanto maior o valor do score de relevância, mais alta a classificação. Em seguida, vamos ver o significado dos conceitos relacionados separadamente.

##### O que é Frequência do Termo (tf:Term Frequency)

Ainda usando os documentos acima:

ID do Documento|Conteúdo do Documento (Doc)|Resultado da Tokenização (Terms)
------|------|------
1|Manual de Design de E-commerce SkrShop|E-commerce / Design / Manual / SkrShop
2|Seckill é um meio de marketing de e-commerce|Seckill / E-commerce / um / marketing / meio
3|O carrinho de compras é a etapa mais importante do processo de compra de e-commerce|Carrinho de compras / E-commerce / compra / processo / importante / uma etapa

Aqui tomamos os termos: `E-commerce/Seckill` como exemplo.

Algoritmo simples de frequência do termo: Frequência do termo = número de vezes que o termo aparece em um único documento / número total de termos no documento. Quanto maior o valor da frequência do termo, mais relevante, caso contrário, menos relevante.

Por exemplo, a frequência da palavra `Seckill` no documento 1, tomando todos os termos de um único documento como dimensão, obtemos simplesmente a frequência da palavra `Seckill` em cada documento:

ID do Documento|Conteúdo do Documento (Doc)|Resultado da Tokenização (Terms)|Número de vezes que o termo aparece em um único documento|Frequência do Termo (Seckill)
------|------|------|------|------
1|Manual de Design de E-commerce SkrShop|E-commerce / Design / Manual / SkrShop|0|0/4=0
2|Seckill é um meio de marketing de e-commerce|Seckill / E-commerce / um / marketing / meio|1|1/5=0.2
3|O carrinho de compras é a etapa mais importante do processo de compra de e-commerce|Carrinho de compras / E-commerce / compra / processo / importante / uma etapa|0|0/6=0

Da mesma forma, obtemos simplesmente a frequência da palavra `E-commerce` em cada documento:

ID do Documento|Conteúdo do Documento (Doc)|Resultado da Tokenização (Terms)|Número de vezes que o termo aparece em um único documento|Frequência do Termo (E-commerce)
------|------|------|------|------
1|Manual de Design de E-commerce SkrShop|E-commerce / Design / Manual / SkrShop|1|1/4=0.25
2|Seckill é um meio de marketing de e-commerce|Seckill / E-commerce / um / marketing / meio|1|1/5=0.2
3|O carrinho de compras é a etapa mais importante do processo de compra de e-commerce|Carrinho de compras / E-commerce / compra / processo / importante / uma etapa|1|1/6=0.167

Conceito de Termo de Busca|Descrição
------|------
Frequência do Termo (tf:Term Frequency)|Número de vezes que o termo aparece em um único documento / número total de termos no documento

##### O que é Frequência Inversa do Documento (idf:Inverse Document Frequency)

Para um único documento, quanto maior o valor da frequência do termo, mais relevante.

> Pense em uma questão, se um determinado termo aparece em todos os documentos, a relevância é melhor ou pior?

```
Resposta: Pior, certo.
```

Essa é a taxa de documento: `Taxa de documento = Número de documentos contendo um determinado termo / Número total de documentos`, quanto maior o valor da taxa de documento, menos relevante, caso contrário, relevante.

Como quanto maior o valor da frequência do termo, mais relevante, e vice-versa. Para garantir a consistência com a lógica da frequência do termo e que quanto maior a pontuação de relevância final, mais relevante, o algoritmo da taxa de documento foi ajustado, trocando o numerador e o denominador: `Número total de documentos / (Número de documentos contendo um determinado termo + 1)`, adicionando 1 para garantir que o denominador não seja zero, esta é a `Frequência Inversa do Documento`.

Frequência Inversa do Documento = `Número total de documentos / (Número de documentos contendo um determinado termo + 1)`.

No entanto, como o número de documentos é frequentemente muito grande, o valor da `Frequência Inversa do Documento` obtido acima será enorme, então a fórmula é ajustada introduzindo logaritmo para reduzir o tamanho do valor e torná-lo suave:

`Frequência Inversa do Documento = log(Número total de documentos / (Número de documentos contendo um determinado termo + 1))`


Termo (Term)|Frequência Inversa do Documento
------|------
E-commerce|log(3/(3+1))
Seckill|log(3/(1+1))

Finalmente, calcula-se a pontuação de relevância score (tf/idf) de cada documento correspondente a cada termo da Query: score de relevância = taxa de documento * taxa inversa de documento.

##### Ordenação Bruta/Ordenação Fina

O resultado da ordenação usando a pontuação tf/idf (`score de relevância = taxa de documento * taxa inversa de documento`) acima é apenas uma ordenação preliminar dos documentos recuperados (Recall), chamada de `Ordenação Bruta`.

Após obter o resultado da `Ordenação Bruta`, geralmente os documentos são ordenados com mais precisão de acordo com os requisitos reais do negócio, por exemplo, aumentando o peso de alguns documentos por meio de `intervenção manual`, fazendo com que fiquem melhor classificados, este processo é a `Ordenação Fina`.

Conceito de Termo de Busca|Descrição
------|------
Ordenação Bruta|O processo de ordenar documentos recuperados usando pontuações tf/idf
Ordenação Fina|O processo de ordenar os resultados da ordenação bruta com mais precisão de acordo com os requisitos reais do negócio, etc.

### Resumo do processo de busca

1. Processo de indexação: Documento (Doc) -> Análise -> Índice Invertido.

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220306223101.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220306223101.png" style="width:50%">
    </a>
</p>

2. Processo de consulta: Palavra-chave (Query) -> Análise -> Recall -> Ordenação Bruta -> Ordenação Fina.

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220309203114.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220309203114.png" style="width:30%">
    </a>
</p>

## Mecanismo de busca ES avançado

### Estrutura básica do índice (substantivo)

- Índice (Index)
    + Tipo (Type): Distinguir diferentes tipos de estrutura de dados de documentos
        * Mapeamento (Mapping): Gerenciar as propriedades do índice, como o analisador usado, etc.
        * Documento (Doc): O documento específico a ser pesquisado

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220308195018.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220308195018.png" style="width:60%">
    </a>
</p>

Aperfeiçoando ainda mais o processo de busca: adicionando uma estrutura de índice (substantivo) mais detalhada

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220308194814.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220308194814.png" style="width:80%">
    </a>
</p>

### Estrutura lógica do mecanismo de busca ES

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220129191435.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20220129191435.png" style="width:90%">
    </a>
</p>