# Ferramenta de Sorteio Universal (Glue/Cola Tudo)

## Análise de Requisitos de Sorteio

Primeiro, vamos revisar a composição do **Sistema de Marketing**:

|Sistema de Marketing|
|---|
|Sistema de Marketing de Atividades|
|Sistema de Marketing de Vendas|

Hoje trazemos a introdução do primeiro subsistema independente sob o **Sistema de Marketing de Atividades**, a **Ferramenta de Sorteio Universal**. Este artigo é dividido principalmente nas 4 partes a seguir:

- Cenários de sorteio comuns e classificação
- Configuração de requisitos de sorteio
- Tipos de prêmios comuns
- Os cinco elementos do sorteio

## Cenários de sorteio comuns e classificação

Abaixo estão alguns cenários de sorteio comuns que listei, como chuva de envelopes vermelhos, chuva de doces, Whac-A-Mole (bater na toupeira), roleta da sorte (jogo da velha), teste de visão, responder perguntas para passar de nível, jogo para passar de nível, raspadinha de pagamento, raspadinha de pontos, e outros cenários de marketing de atividades.

|Nome da Atividade|Descrição|
|------|------|
|Chuva de Envelopes Vermelhos|Sorteio de envelopes vermelhos 🧧 a cada hora cheia diariamente, geralmente pode participar uma vez a cada hora cheia|
|Chuva de Doces|Sorteio de doces 🍬 a cada hora cheia diariamente, geralmente pode participar uma vez a cada hora cheia|
|Whac-A-Mole|Sorteio de bater na toupeira a cada hora cheia diariamente, geralmente pode participar uma vez a cada hora cheia|
|Roleta da Sorte (Jogo da Velha)|Sorteio de roleta em um determinado período de tempo, geralmente pode participar N vezes por sessão|
|Teste de Visão|Em um determinado período de tempo, adivinhar em qual copo a bola está girando, se acertar pode sortear, geralmente pode participar N vezes por dia|
|Responder Perguntas|A cada nível passado, pode participar do sorteio, quanto mais longe, mais valiosos os prêmios|
|Jogo para Passar de Nível|A cada nível passado, pode participar do sorteio, quanto mais longe, mais valiosos os prêmios|
|Raspadinha de Pagamento|Pode raspar após pagar o pedido, quanto maior o valor do pagamento, mais valioso o prêmio|
|Raspadinha de Pontos|Raspar com pontos, quanto maior o valor de pontos consumidos, mais valioso o prêmio|

Através da descrição da atividade acima, classificamos todos os cenários de sorteio nas três categorias a seguir:

|Tipo|Nome da Atividade|Dimensão|
|-|-|-|
|Sorteio por Tempo|Chuva de Envelopes Vermelhos, Chuva de Doces, Whac-A-Mole, Roleta da Sorte, Teste de Visão|Dimensão de Tempo|
|Sorteio por Número de Sorteios|Responder Perguntas, Jogo para Passar de Nível|Dimensão de número de participações na atividade atual|
|Sorteio por Intervalo de Valor|Raspadinha de Pagamento, Raspadinha de Pontos|Dimensão de Intervalo de Valor|

Em seguida, vamos ver a configuração de requisitos de sorteio específica para cada tipo de atividade de sorteio.

## Configuração de requisitos de sorteio

A configuração de requisitos para cada tipo de atividade de sorteio nesta seção é dividida nas três partes a seguir:

- Configuração da Atividade
- Configuração da Sessão
- Configuração do Prêmio

### Primeiro, a primeira categoria: Configuração de requisitos de `Sorteio por Tempo`

|Tipo|Nome da Atividade|Características|
|-|-|-|
|Sorteio por Tempo|Chuva de Envelopes Vermelhos, Chuva de Doces, Whac-A-Mole, Roleta da Sorte, Teste de Visão|Dimensão de Tempo|

|Sorteio por Tempo|Múltiplas Sessões?|Limite de vezes por sessão única (vezes)|Limite total de vezes da sessão (vezes)|
|-|-|-|-|
|Chuva de Envelopes Vermelhos|Sim|1|N|
|Chuva de Doces|Sim|1|N|
|Whac-A-Mole|Sim|N|N|
|Roleta da Sorte|Não|N|N|
|Teste de Visão|Não|N|N|

Através da análise acima, obtivemos os conceitos de **Atividade** e **Sessão**: uma atividade precisa suportar a configuração de múltiplas sessões.

- Atividade (activity): Configurar o intervalo de datas da atividade
- Sessão (session): Configurar o intervalo de tempo específico de cada sessão

**Exemplo de configuração de requisitos para Chuva de Envelopes Vermelhos:**

> Características da Atividade: Chuva de Envelopes Vermelhos precisa suportar múltiplas sessões.

Por exemplo, durante o Double 12, três dias, três sessões de chuva de envelopes vermelhos em hora cheia por dia são configuradas da seguinte forma:

Configuração de Atividade e Sessão:

|Chuva de Envelopes Vermelhos Double 12|
|------|
|Configuração da Atividade:|
|2019-12-10 a 2019-12-12|
|Configuração da Sessão:|
|10:00:00 a 10:01:00|
|12:00:00 a 12:01:00|
|18:00:00 a 18:01:00|

Configuração de Prêmio:

|Sessão|Prêmio 1|Prêmio 2|---|Prêmio N|
|------|------|------|---|------|
|Sessão 10:00:00 a 10:01:00|Cupom de 2 yuans|Prêmio Vazio|---|Nenhum|
|Sessão 12:00:00 a 12:01:00|Cupom de 5 yuans|Prêmio Vazio|---|Nenhum|
|Sessão 18:00:00 a 18:01:00|Cupom de 10 yuans|Cupom de 20 yuans|---|Prêmio Vazio|

```md
Os resultados da configuração acima são os seguintes:

Três sessões de chuva de envelopes vermelhos em hora cheia em 2019-12-10:
2019-12-10 10:00:00 ~ 10:01:00
2019-12-10 12:00:00 ~ 12:01:00
2019-12-10 18:00:00 ~ 18:01:00

Três sessões de chuva de envelopes vermelhos em hora cheia em 2019-12-11:
2019-12-11 10:00:00 ~ 10:01:00
2019-12-11 12:00:00 ~ 12:01:00
2019-12-11 18:00:00 ~ 18:01:00

Três sessões de chuva de envelopes vermelhos em hora cheia em 2019-12-12:
2019-12-12 10:00:00 ~ 10:01:00
2019-12-12 12:00:00 ~ 12:01:00
2019-12-12 18:00:00 ~ 18:01:00
```

**Exemplo de configuração de requisitos para Roleta da Sorte:**

> Características da Atividade: Roleta da Sorte não precisa de múltiplas sessões.

Por exemplo, durante o Festival de Ano Novo de 2020-01-20 a 2020-02-10, a Roleta da Sorte é configurada da seguinte forma:

Configuração de Atividade e Sessão:

|Roleta da Sorte Double 12|
|------|
|Configuração da Atividade:|
|2019-12-10 a 2019-12-12|
|Configuração da Sessão:|
|00:00:00 a 23:59:59|

Configuração de Prêmio:

|Sessão|Prêmio 1|Prêmio 2|---|Prêmio N|
|------|------|------|---|------|
|Sessão 00:00:00 a 23:59:59|Cupom de 2 yuans|Prêmio Vazio|---|Nenhum|

```md
Os resultados da configuração acima são os seguintes:

A atividade de sorteio da Roleta da Sorte ocorrerá de 2019-12-10 00:00:00 ~ 2019-12-12 23:59:59
```

Atenção e Reflexão: A Roleta da Sorte Double 12 não precisa de várias sessões, apenas uma sessão precisa ser configurada, reutilizando completamente o modelo de sessão de atividade.

### Em seguida, a segunda categoria: Configuração de requisitos de `Sorteio por Número de Sorteios`

|Tipo|Nome da Atividade|Características|
|-|-|-|
|Sorteio por Número de Sorteios|Responder Perguntas, Jogo para Passar de Nível|(Participação bem-sucedida) Dimensão de número de participações na atividade atual|

**Exemplo de configuração de requisitos para Responder Perguntas:**

> Características da Atividade: Os prêmios em cada nível são diferentes, geralmente quanto mais longe, maior a probabilidade de ganhar um grande prêmio.

Configuração de Atividade e Sessão:

|Responder Perguntas Double 12|
|------|
|Configuração da Atividade:|
|2019-12-10 a 2019-12-12|
|Configuração da Sessão:|
|00:00:00 a 23:59:59|

Configuração de Prêmio:

|Responder Perguntas Double 12|Prêmio|
|------|------|
|Nível 1|Cupom de 2 yuans|
|Nível 2|Cupom de 5 yuans|
|Nível 3|Cupom de 10 yuans|
|Nível 4|Cupom de 20 yuans|
|Nível 5|Cupom de 50 yuans|
|Nível 6|Cupom de 100 yuans|

Atenção e Reflexão: Da mesma forma, a configuração de atividade e sessão é completamente reutilizada, igual à configuração da Roleta da Sorte (não precisa suportar múltiplas sessões).

### Finalmente, a terceira categoria: Configuração de requisitos de `Sorteio por Intervalo de Valor`:

|Tipo|Nome da Atividade|Características|
|-|-|-|
|Sorteio por Intervalo de Valor|Raspadinha de Pagamento, Raspadinha de Pontos|Dimensão de Intervalo de Valor|

**Exemplo de configuração de requisitos para Raspadinha de Pagamento:**

> Características da Atividade: Diferentes valores de pedidos, geralmente quanto maior o valor, maior a probabilidade de ganhar um grande prêmio.

Configuração de Atividade e Sessão:

|Responder Perguntas Double 12|
|------|
|Configuração da Atividade:|
|2019-12-10 a 2019-12-12|
|Configuração da Sessão:|
|00:00:00 a 23:59:59|

Configuração de Prêmio:

|Valor do Pedido|Prêmio 1|Prêmio 2|---|Prêmio N|
|------|------|------|---|------|
|0~100|Cupom de 2 yuans|Prêmio Vazio|---|Nenhum|
|100~200|Cupom de 5 yuans|Prêmio Vazio|---|Nenhum|
|200~1000|Cupom de 10 yuans|Cupom de 20 yuans|---|Prêmio Vazio|
|Acima de 1000|Cupom de 50 yuans|Notebook|---|Prêmio Vazio|

Atenção e Reflexão: Da mesma forma, a configuração de atividade e sessão é completamente reutilizada, igual à configuração da Roleta da Sorte (não precisa suportar múltiplas sessões).

> Resumo: Através da análise acima, obtivemos dois elementos da ferramenta de sorteio: **Atividade** e **Sessão**.


## Tipos de prêmios comuns

> O que sortear no sorteio?

|Tipos de prêmios comuns|
|-|
|Cupom|
|Pontos|
|Objeto físico|
|Prêmio Vazio|

> Resumo: Obtivemos outro elemento da ferramenta de sorteio: **Prêmio**.

## Os cinco elementos do sorteio

Através da análise acima, já obtivemos os **três elementos** do sorteio

- Atividade
- Sessão
- Prêmio

> Então, que outros elementos ainda não discutimos? Vamos ver a seguir.

### Quarto elemento: Probabilidade de ganhar

O sorteio naturalmente não pode ser separado da configuração da probabilidade de ganhar o prêmio. Sobre a probabilidade de ganhar, suportamos as seguintes configurações flexíveis:

1. Definir manualmente a probabilidade de ganhar o prêmio
2. Probabilidade automática, obter a probabilidade de ganhar com base na quantidade atual de prêmios e no peso dos prêmios

Por exemplo, a configuração da Chuva de Envelopes Vermelhos para uma grande promoção é a seguinte:

Configuração da Atividade|Descrição
------|------
Tempo da Atividade|2019-12-10 a 2019-12-12
Nome da Atividade|Chuva de Envelopes Vermelhos em Hora Cheia da Grande Promoção Double 12 2019
Descrição da Atividade|Atividade de Chuva de Envelopes Vermelhos em Hora Cheia em todos os terminais da Grande Promoção Double 12 2019
Definir probabilidade de prêmio manualmente|Sim

|Sessão|Tipo de Prêmio|Prêmio Específico|Quantidade de Prêmios|Probabilidade de Ganhar
|-|-|-|-|-|
|10:00:00 ~ 10:01:00|Cupom|Cupom de 2 yuans|2000|50%|
|-|Cupom|Cupom de 5 yuans|1000|20%|
|-|Prêmio Vazio|-|5000|30%|
|12:00:00 ~ 12:01:00|Cupom|Cupom de 2 yuans|2000|50%|
|-|Cupom|Cupom de 5 yuans|1000|20%|
|-|Prêmio Vazio|-|5000|30%|
|18:00:00 ~ 18:01:00|Cupom|Cupom de 2 yuans|2000|50%|
|-|Cupom|Cupom de 5 yuans|1000|20%|
|-|Prêmio Vazio|-|5000|30%|

Nota: A soma das probabilidades de ganhar em cada sessão deve ser 100%, caso contrário, a parte restante será adicionada como probabilidade de ganhar prêmio vazio por padrão.

### Quinto elemento: Distribuição Uniforme de Prêmios

> Como sortear os prêmios uniformemente?

Resposta: Distribuição uniforme de prêmios.

O método específico é dividir o número total de prêmios em períodos de tempo específicos e detalhados. Tomando a Roleta da Sorte Double 12 como exemplo:

|Sessão|Tipo de Prêmio|Prêmio Específico|Quantidade de Prêmios|Probabilidade de Ganhar|Tempo de Distribuição (Padrão 5 minutos antes da distribuição)|Quantidade de Distribuição
|-|-|-|-|-|-|-|
|00:00:00 a 23:59:59|Cupom|Cupom de 2 yuans|2000|50%|-|-|
|-|-|-|-|-|00:00:00|2000|
|-|-|-|-|-|06:00:00|2000|
|-|-|-|-|-|12:00:00|2000|
|-|-|-|-|-|18:00:00|2000|

Aqui obtivemos o **quinto elemento do sorteio: Distribuição Uniforme de Prêmios**.

## Resumo dos Requisitos

Através da análise acima, obtemos os cinco elementos do sorteio da seguinte forma:

Cinco Elementos do Sorteio|Nome do Elemento
------|------
Primeiro Elemento|Atividade
Segundo Elemento|Sessão
Terceiro Elemento|Prêmio
Quarto Elemento|Probabilidade de Ganhar
Quinto Elemento|Distribuição Uniforme de Prêmios

Ao mesmo tempo, através dos **Cinco Elementos do Sorteio**, também obtivemos os 5 passos básicos para configurar uma atividade de sorteio na **Ferramenta de Sorteio Universal**:

1. Configuração da Atividade
2. Configuração da Sessão
3. Configuração do Prêmio
4. Configuração da Probabilidade de Ganhar o Prêmio
5. Configuração da Distribuição de Prêmios

## Design do Sistema da Ferramenta de Sorteio Universal

Os requisitos já foram analisados, hoje vamos ver o design específico desta ferramenta de sorteio universal, dividido nas três partes a seguir:

- Design de DB
- Design de Backend de Configuração
- Design de Interface

## Design de DB

Primeiro elemento `Configuração da Atividade` tabela `Tabela de Atividades de Sorteio`:

```sql
-- Ferramenta de Sorteio Universal (Cola Tudo Glue) glue_activity Tabela de Atividades de Sorteio
CREATE TABLE `glue_activity` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID da Atividade',
    `serial_no` char(16) unsigned NOT NULL DEFAULT '' COMMENT 'Número da atividade (16 dígitos do meio do valor md5)',
    `name` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Nome da atividade',
    `description` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Descrição da atividade',
    `activity_type` tinyint(1) unsigned NOT NULL DEFAULT '1' COMMENT 'Tipo de sorteio da atividade 1: Sorteio por tempo 2: Sorteio por número de sorteios 3: Sorteio por intervalo de valor',
    `probability_type` tinyint(1) unsigned NOT NULL DEFAULT '1' COMMENT 'Tipo de probabilidade de ganhar 1: estática 2: dinâmica',
    `times_limit` tinyint(3) unsigned NOT NULL DEFAULT '0' COMMENT 'Limite de vezes de sorteio, 0 padrão sem limite',
    `start_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de início da atividade',
    `end_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de término da atividade',
    `create_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de criação',
    `create_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de atualização',
    `update_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1)  NOT NULL DEFAULT '0' COMMENT 'Status -1: deletado, 0: desativado, 1: ativado',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Tabela de Atividades de Sorteio';
```

Segundo elemento `Configuração da Sessão` tabela `Tabela de Sessões de Sorteio`:

```sql
-- Ferramenta de Sorteio Universal (Cola Tudo Glue) glue_session Tabela de Sessões de Sorteio
CREATE TABLE `glue_session` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID da Sessão',
    `activity_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID da Atividade',
    `times_limit` tinyint(3) unsigned NOT NULL DEFAULT '0' COMMENT 'Limite de vezes de sorteio, 0 padrão sem limite',
    `start_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de início da sessão',
    `end_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de término da sessão',
    `create_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de criação',
    `create_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de atualização',
    `update_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1)  NOT NULL DEFAULT '0' COMMENT 'Status -1: deletado, 0: desativado, 1: ativado',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Tabela de Sessões de Sorteio';
```

Terceiro e quarto elementos `Configuração do Prêmio` tabela `Tabela de Prêmios da Sessão de Sorteio`:

```sql
-- Ferramenta de Sorteio Universal (Cola Tudo Glue) glue_session_prizes Tabela de Prêmios da Sessão de Sorteio
CREATE TABLE `glue_session_prizes` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID Auto-incremental',
    `session_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID da Sessão',
    `node` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Identificador de nó Sorteio por tempo: Vazio, Sorteio por número de sorteios: Qual participação, Sorteio por intervalo de valor: Valor limite superior do intervalo',
    `prize_type` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tipo de prêmio 1: Cupom, 2: Pontos, 3: Objeto físico, 4: Prêmio vazio ...',
    `name` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Nome do prêmio',
    `pic_url` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Imagem do prêmio',
    `value` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Valor abstrato do prêmio Cupom: ID do cupom, Pontos: Valor dos pontos, Objeto físico: ID do sku',
    `probability` tinyint(3) unsigned NOT NULL DEFAULT '0' COMMENT 'Probabilidade de ganhar 1~100',
    `create_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de criação',
    `create_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de atualização',
    `update_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1)  NOT NULL DEFAULT '0' COMMENT 'Status -1: deletado, 0: desativado, 1: ativado',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Tabela de Prêmios da Sessão de Sorteio';

```

Quinto elemento `Distribuição Uniforme de Prêmios` tabela `Tabela de Temporizador de Distribuição de Prêmios da Sessão de Sorteio`:

```sql
-- Ferramenta de Sorteio Universal (Cola Tudo Glue) glue_session_prizes_timer Tabela de Temporizador de Distribuição de Prêmios da Sessão de Sorteio
CREATE TABLE `glue_session_prizes_timer` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID Auto-incremental',
    `session_prizes_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID do Prêmio da Sessão de Sorteio',
    `delivery_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de distribuição da quantidade de prêmios',
    `prize_quantity` tinyint(3) unsigned NOT NULL DEFAULT '0' COMMENT 'Quantidade de prêmios',
    `create_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de criação',
    `create_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de atualização',
    `update_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1)  NOT NULL DEFAULT '0' COMMENT 'Status -1: deletado, 0: espera, 1: sucesso',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Tabela de Temporizador de Distribuição de Prêmios da Sessão de Sorteio';

```

Outras tabelas, tabela de registro de sorteio e registro de emissão de prêmios:

```sql
-- Ferramenta de Sorteio Universal (Cola Tudo Glue) glue_user_draw_record Tabela de Registro de Sorteio do Usuário
CREATE TABLE `glue_user_draw_record` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID Auto-incremental',
    `activity_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID da Atividade',
    `session_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID da Sessão',
    `prize_type_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID do Tipo de Prêmio',
    `user_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'user_id do criador',
    `create_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de criação',
    `update_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de atualização',
    `status` tinyint(1)  NOT NULL DEFAULT '0' COMMENT 'Status -1: não ganhou, 1: ganhou , 2: falha na emissão , 3: emitido',
    `log` text COMMENT 'Registros de informações de operação, etc.',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Tabela de Registro de Sorteio do Usuário';
```

## Design de Backend de Configuração

### Criar Atividade

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20191229224816.png?imageMogr2/thumbnail/1934x1567!/format/webp/blur/1x0/quality/75|imageslim" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20191229224816.png?imageMogr2/thumbnail/1934x1567!/format/webp/blur/1x0/quality/75|imageslim" width="66%">
    </a>
</p>

### Criar Sessão de Atividade

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20191230081157.png?imageMogr2/thumbnail/971x2069!/format/webp/blur/1x0/quality/75%7Cimageslim" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20191230081157.png?imageMogr2/thumbnail/971x2069!/format/webp/blur/1x0/quality/75%7Cimageslim" width="66%">
    </a>
</p>

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20191229224543.png?imageMogr2/thumbnail/971x2214!/format/webp/blur/1x0/quality/75%7Cimageslim" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20191229224543.png?imageMogr2/thumbnail/971x2214!/format/webp/blur/1x0/quality/75%7Cimageslim" width="66%">
    </a>
</p>

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20191229224834.png?imageMogr2/thumbnail/971x1693!/format/webp/blur/1x0/quality/75%7Cimageslim" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20191229224834.png?imageMogr2/thumbnail/971x1693!/format/webp/blur/1x0/quality/75%7Cimageslim" width="66%">
    </a>
</p>

### Lista de Atividades

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20191229223706.png?imageMogr2/thumbnail/1338x761!/format/webp/blur/1x0/quality/75%7Cimageslim" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20191229223706.png?imageMogr2/thumbnail/1338x761!/format/webp/blur/1x0/quality/75%7Cimageslim" width="66%">
    </a>
</p>


## Design de Interface

1. Obter informações da atividade GET {version}/glue/activity

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
serial_no|string|Y|Número da atividade

Conteúdo da resposta:
```json
{
    "code": "200",
    "msg": "OK",
    "result": {
        "serial_no": "string, Número da atividade",
        "type": "number, Tipo de sorteio da atividade 1: Sorteio por tempo 2: Sorteio por número de sorteios 3: Sorteio por intervalo de valor",
        "name": "string, Nome da atividade",
        "description": "string, Descrição da atividade",
        "start_time": "number, Tempo de início da atividade",
        "end_time": "number, Tempo de término da atividade",
        "remaining_times": "number, Limite de vezes de sorteio da atividade, 0 sem limite",
        "sessions_list":[
            {
                "start_time": "number, Tempo de início da sessão",
                "end_time": "number, Tempo de término da sessão",
                "remaining_times": "number, Limite de vezes de sorteio da sessão, 0 sem limite",
                "prizes_list": [
                    {
                        "name": "string, Nome do prêmio",
                        "pic_url": "string, Imagem do prêmio"
                    }
                ]
            }
        ]
    }
}
```

2. Sorteio POST {version}/glue/activity/draw

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
serial_no|string|Y|Número da atividade
uid|number|Y|ID do usuário

Conteúdo da resposta:
```json
// Ganhou
{
    "code": "200",
    "msg": "OK",
    "result": {
        "serial_no": "string, spu id",
        "act_remaining_times": "number, Vezes restantes de sorteio nesta atividade, 0 sem limite",
        "session_remaining_times": "number, Vezes restantes de sorteio nesta sessão, 0 sem limite",
        "prizes_info": 
        {
            "name": "string, Nome do prêmio",
            "pic_url": "string, Imagem do prêmio"
        }
    }
}

// Não ganhou
{
    "code": "401",
    "msg": "",
    "result": {
        
    }
}
```

## Conclusão

O primeiro subsistema no sistema de marketing de atividades, a **Ferramenta de Sorteio Universal**, foi abordado hoje. Espero que seja útil ou inspirador para todos.
