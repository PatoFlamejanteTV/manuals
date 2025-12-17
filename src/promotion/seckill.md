# Serviço de Seckill (Oferta Relâmpago)

## Prefácio

Quem trabalha com negócios sabe:

> O conceito do sistema é muito importante

Porque na comunicação entre colegas, esses conceitos podem dizer com precisão aos outros o que você quer expressar. Especialmente no sistema de e-commerce, como todos sabem, existem muitos conceitos no e-commerce, como Sku, Spu, etc.

Então, primeiro precisamos entender **O que é Seckill (Oferta Relâmpago)?**


## O que é Seckill?

Vamos dar uma olhada nas capturas de tela das páginas de seckill do JD, Youpin e Pinduoduo abaixo.

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200712224532.jpeg">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200712224532.jpeg" width="30%">
    </a>
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200712224556.jpeg">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200712224556.jpeg" width="30%">
    </a>
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200712224606.jpeg">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200712224606.jpeg" width="30%">
    </a>
</p>

Através das informações na página, podemos obter as seguintes informações úteis:

|Conceito|Descrição|
|-------|-------|
|Conceito 1|Atividade|
|Conceito 2|Conceito de Sessão, a sessão é um subconjunto da atividade|

|Informações de dados na página|Descrição|
|-------|-------|
|Informações da atividade|Informações da atividade e da sessão|
|Informações do produto de seckill|Imagem do produto, nome do produto, preço de adição ao carrinho, preço de venda, outras informações descritivas|
|Progresso do Seckill|Progresso do estoque|

Definição de Seckill:

> Seckill é um meio de marketing de e-commerce, sendo comum o seckill de um yuan, etc.

## Quais são as dimensões de marketing das atividades de Seckill?

|Dimensão de Marketing|
|-------|
|Dimensão de Preço|
|Dimensão de Quantidade|
|Dimensão do Produto|
|Dimensão de Tempo|

|Dimensão de Preço|
|-------|
|Preço de banana (muito barato)|
|Preço não banana|

|Dimensão de Quantidade|
|-------|
|Extremamente pequena (por exemplo, alguns)|
|Não extremamente pequena|

|Dimensão do Produto|
|-------|
|Produto popular (爆品)|
|Não popular|

|Dimensão de Tempo|
|-------|
|Tempo limitado|

Combinando as dimensões acima de acordo com as necessidades operacionais, obtemos diferentes tipos de atividades de seckill, como segue:

### Primeiro, Seckill de um yuan e afins: Preço de banana + Extremamente pequeno + (Produto popular ou não) + Tempo limitado

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200718215320.png">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200718215320.png" width="80%">
    </a>
</p>

### Segundo, Compra por Tempo Limitado (também conhecida como Seckill Convencional): Preço não banana + (Extremamente pequeno ou não) + (Produto popular ou não) + Tempo limitado

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200718215411.png">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200718215411.png" width="80%">
    </a>
</p>

### Em seguida, Seckill de Produto Popular: Preço não banana + (Extremamente pequeno ou não) + Produto popular + Tempo limitado

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200718215425.png">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200718215425.png" width="80%">
    </a>
</p>

|Tipo de Atividade de Seckill|Dimensão de Marketing|
|-------|-------|
|Seckill de um yuan e afins|Preço de banana + Extremamente pequeno + (Produto popular ou não) + Tempo limitado|
|Compra por Tempo Limitado (Seckill Convencional) |Preço não banana + (Extremamente pequeno ou não) + (Produto popular ou não) + Tempo limitado -> |
|Seckill de Produto Popular|Preço não banana + (Extremamente pequeno ou não) + Produto popular + Tempo limitado|

## Um Sistema de Seckill Simples

**Princípio de implementação:** Redução de estoque através de operação atômica do Redis

**Figura 1**
<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200501175532.png">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200501175532.png" width="100%">
    </a>
</p>

Vantagens|Desvantagens
------------|------------
Simples e fácil de usar|Testa a capacidade do serviço Redis

|É justo?|
|-------|
|Justo|
|Primeiro a chegar, primeiro a ser servido|

Chamamos este tipo de sistema de seckill de:

> Sistema de Seckill Simples

Se o QPS não for alto no início e o Redis puder suportar totalmente, você pode depender completamente deste "Sistema de Seckill Simples".

## Um Sistema de Seckill Suficiente

**Princípio de implementação:** Algoritmo de limitação de fluxo na memória do serviço + redução de estoque por operação atômica do Redis

**Figura 2**
<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200501183037.png">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200501183037.png">
    </a>
</p>

Vantagens|Desvantagens
------------|------------
Simples e fácil de usar|-

|É justo?|
|-------|
|Não muito justo|
|Relativamente primeiro a chegar, primeiro a ser servido|

Chamamos este tipo de sistema de seckill de:

> Sistema de Seckill Suficiente

## Um Sistema de Seckill com Melhor Desempenho

**Princípio de implementação:** Redução de estoque por operação atômica na memória local do serviço

> De onde vem o estoque na memória local do serviço?

Aloque o estoque de cada máquina antes do início da atividade e envie para a máquina.

**Figura 3**
<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200501200309.png">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200501200309.png">
    </a>
</p>

Vantagens|Desvantagens
------------|------------
Alto desempenho|Não suporta escalonamento dinâmico (durante a atividade), porque o estoque é alocado antes do início da atividade
Libera pressão do Redis|-

|É justo?|
|-------|
|Não muito justo|
|Não é absolutamente primeiro a chegar, primeiro a ser servido|


Chamamos este tipo de sistema de seckill de:

> Sistema de Seckill de Estoque Preparado

## Sistema de Seckill com Suporte a Escalonamento Dinâmico

**Princípio de implementação:** Corrotina do serviço local **reduz parte do estoque por operação atômica do Redis periodicamente** para a memória local + redução de estoque por operação atômica na memória local do serviço

**Figura 4**
<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200501200846.png">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200501200846.png">
    </a>
</p>

Vantagens|Desvantagens
------------|------------
Alto desempenho|Suporta escalonamento dinâmico (durante a atividade)
Libera pressão do Redis|-
**Possui generalidade**|-

|É justo?|
|-------|
|Não muito justo, mas um pouco melhor|
|Quase primeiro a chegar, primeiro a ser servido|

Chamamos este tipo de sistema de seckill de:

> Sistema de Seckill de Estoque Preparado em Tempo Real

## Sistema de Seckill Justo

**Princípio de implementação:** Goroutine do serviço local **sincroniza periodicamente se está esgotado** para a memória local + fila + polling (ou Push ativo) do resultado do sucesso na fila

**Figura 5**
<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200502195413.png">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200502195413.png">
    </a>
</p>

Vantagens|Desvantagens
------------|------------
Alto desempenho|Alto custo de desenvolvimento (precisa de notificação ativa ou polling do resultado da fila)
Verdadeiramente justo|-
**Possui generalidade**|-

|É justo?|
|-------|
|Muito justo|
|Absolutamente primeiro a chegar, primeiro a ser servido|

Chamamos este tipo de sistema de seckill de:

> Sistema de Seckill de Fila Justa

## Operação "Sāo" (Astuta/Incomum)

> O sistema de seckill acima não é perfeito o suficiente?

Resposta: Sim.

> Há mais espaço para otimização?

Resposta: Estática da interface de obtenção de informações da atividade de seckill.

> O que significa estática?

Resposta: Por exemplo, obter informações da atividade de seckill é feito através da interface `https://seckill.skrshop.tech/v1/acticity/get`. Agora, precisamos obter através da interface `https://static-api.skrshop.tech/seckill/v1/acticity/get`. Qual é a diferença? Veja abaixo:

Nome do Serviço|Interface|Local de Armazenamento de Dados
------|------|------
Serviço de Seckill|https://seckill.skrshop.tech/v1/acticity/get|Memória do serviço de seckill ou Redis, etc.
Serviço de Estática de Interface|https://static-api.skrshop.tech/seckill/v1/acticity/get|CDN, arquivo local

**Antes era assim**
<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200502195950.png">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200502195950.png" width="66%">
    </a>
</p>

**Tornou-se assim**
<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200502200723.png">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200502200723.png" width="100%">
    </a>
</p>

Resultado: As informações da atividade de seckill podem ser obtidas através da interface `https://static-api.skrshop.tech/seckill/v1/acticity/get`, o tráfego é distribuído para o CDN, e o próprio serviço de seckill não tem essa carga.

## Esquema de Processo Completo ①

O processo completo envolve principalmente três interfaces:

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200718215732.png">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200718215732.png" width="80%">
    </a>
</p>

|Interface de Serviço de Seckill|Interno ou Externo|Descrição|
|-------|-------|-------|
|Interface de Obtenção de Informações de Seckill|Externo|Requisito de QPS alto, portanto pode ser diretamente externo
|Obter Qualificação de Seckill|Externo|O usuário obtém a qualificação para adicionar este produto ao carrinho|
|Verificar e Obter Preço de Seckill|Interno|A interface do carrinho verifica a qualificação e retorna a qualificação de seckill da atividade atual para este produto|

## Esquema de Processo Completo ②

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200718215817.png">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200718215817.png" width="80%">
    </a>
</p>

Qual é a diferença entre este esquema e o anterior? Resposta: A interface para obter informações da atividade de seckill foi unificada no "Centro de Marketing", com o objetivo de:

> Abstrair todas as atividades de marketing em uma interface de "Informações de Atividade do Produto"

Desta forma, a lógica de leitura da nossa página de detalhes do produto fica muito clara, da seguinte forma:

- Tipo 1: Interface de informações básicas do produto, obtém informações básicas do produto (imagem, nome, descrição, preço, estoque, etc.)
- Tipo 2: Interface de informações de atividade do produto, obtém todas as informações de atividades de marketing em que o produto participa (desconto por valor, brinde por valor, compra e ganhe, seckill, etc.)

Ilustração:

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200802220914.png">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200802220914.png" width="50%">
    </a>
</p>

## Resumo

Acima, obtivemos os seguintes tipos de `Sistemas de Seckill`

|Sistema de Seckill|
------------|
|Sistema de Seckill Simples|
|Sistema de Seckill Suficiente|
|Sistema de Seckill de Estoque Preparado|
|Sistema de Seckill de Estoque Preparado em Tempo Real|
|Sistema de Seckill de Fila Justa|

O que quero dizer é que não existe o melhor esquema, nem o pior esquema, apenas o **adequado para você**.

Tomando `primeiro a chegar, primeiro a ser servido` como exemplo, você deve olhar para a propaganda externa do seu produto, não busque o absoluto primeiro a chegar, primeiro a ser servido logo de cara. Na verdade, se você olhar para todos os esquemas, relativamente falando, são todos "primeiro a chegar, primeiro a ser servido". Por exemplo, se a atividade começou há uma hora e você vem tentar pegar, naturalmente não conseguirá pegar contra usuários pontuais, certo?

Outro exemplo é o `Sistema de Seckill de Estoque Preparado`, embora não suporte escalonamento dinâmico. Mas se o seu ambiente atender a qualquer uma das seguintes condições, é completamente suficiente.

- O tempo de término do cenário de seckill é rápido, geralmente termina em alguns segundos, e a atividade real pode ter as seguintes situações:
    + A pressão do serviço é alta, mas não caiu: simplesmente não há tempo para escalonamento dinâmico
    + A pressão do serviço é alta e caiu: pode pausar a atividade primeiro, serviço sobe & expansão termina, empurrar novamente com o estoque restante
- A própria operação e manutenção não têm a capacidade de escalonamento dinâmico

Portanto:

> Se for adequado e fácil de usar, está bom, não faça over-design.