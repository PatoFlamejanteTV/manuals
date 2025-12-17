# Centro de Transações

## Fluxo Comum de Pagamento de Terceiros
Nos últimos anos de trabalho, tenho lidado constantemente com pagamentos. Aproveitando o projeto [skr-shop](https://github.com/skr-shop/manuals), gostaria de compartilhar e explorar com vocês como projetar e fazer um sistema de pagamento. Vamos começar analisando alguns fluxos de pagamento comuns, identificar os pontos em comum desses pagamentos e, após a abstração, discutir o design específico do banco de dados e o design da estrutura do código.

Projetos relacionados:

- [SDK de Pagamento versão PHP](https://github.com/helei112g/payment)
- [SDK de Pagamento versão Go - Em desenvolvimento](https://github.com/skr-shop/fool-pay)



> O fluxo geral de pagamento é: iniciar uma transação para um terceiro, o usuário conclui o pagamento através do terceiro, o terceiro me informa que o pagamento foi bem-sucedido e eu entrego o produto comprado ao usuário.

![pay-1](https://dayutalk.cn/img/pay-1.jpg)

O fluxo parece simples, mas diferentes instituições de pagamento têm diferentes tratamentos. Abaixo, resumirei alguns pagamentos com os quais tive contato.

### Pagamento Doméstico (China)

Os representantes típicos de pagamento doméstico são: **Alipay**, **WeChat**, **Banco** (tome o China Merchants Bank como exemplo). Como os pagamentos domésticos suportam vários canais de pagamento, para simplificar a descrição, explicaremos usando o pagamento no PC como exemplo.

#### Alipay

A integração do Alipay é a que considero mais simples. Para a capacidade de pagamento no PC, o Alipay fornece [Pagamento no Computador]. Após o usuário fazer o pedido, o sistema do comerciante constrói uma url de acordo com as regras do Alipay. O usuário salta para essa url e entra na página de pagamento do Alipay, completando o fluxo de pagamento.

Após o sucesso do pagamento, o Alipay informará o sistema do comerciante sobre o sucesso do pagamento através de dois métodos: **notificação síncrona** e **notificação assíncrona**. Os resultados de ambos os métodos de notificação são confiáveis, e o atraso da mensagem de notificação assíncrona também é muito curto.

Para o processo de reembolso, o Alipay suporta reembolso total e parcial. E pode distinguir se é o mesmo reembolso com base no número do pedido de reembolso do comerciante, evitando a possibilidade de reembolsos duplicados. O reembolso do pagamento retorna o resultado de forma síncrona após a chamada e não haverá notificação assíncrona.

#### WeChat Pay

O WeChat não fornece capacidade real de pagamento no PC, mas podemos usar o [Pagamento via Scan QR Code] para atingir o objetivo de pagamento no computador. Existem dois modos de pagamento via scan, aqui tomamos o modo dois como exemplo.

O WeChat chama a interface de pedido para obter este link de código QR, e então o usuário escaneia o código para entrar no processo de pagamento. Após completar o pagamento, o WeChat enviará uma **notificação assíncrona**, mas não há **notificação síncrona** aqui. Portanto, a página frontend só pode verificar se a transação foi paga por meio de polling regular até que o sucesso seja consultado ou o usuário feche a página ativamente.

A maior diferença entre o processo de reembolso e o Alipay é que há uma **notificação assíncrona** que precisa ser processada pelo sistema do comerciante.

> Primeira diferença:
>
> 1. A interface de notificação assíncrona precisa lidar com vários tipos diferentes de mensagens assíncronas

#### China Merchants Bank

Com o desenvolvimento vigoroso do pagamento online na China, vários bancos também estão lançando constantemente suas próprias capacidades de pagamento online. O destaque entre eles é o **China Merchants Bank**. O Didi, que todos usam frequentemente, tem esse método de pagamento, você pode experimentar.

O pagamento do China Merchants usa cartões bancários, portanto, o usuário deve vincular o cartão pela primeira vez. Portanto, pode haver um processo extra aqui. Primeiro, deve-se registrar se o usuário vinculou o cartão e, em seguida, a chave pública usada para assinatura mudará e precisará ser atualizada regularmente.

A experiência de pagamento em todas as plataformas do China Merchants é consistente. Ele saltará para a página H5 do China Merchants Bank para completar a lógica. Após o sucesso do pagamento, ele não saltará automaticamente de volta para o comerciante, ou seja, não há **notificação síncrona**. Seu resultado de pagamento seguirá apenas o fluxo de notificação assíncrona, e o atraso é muito curto.

O processo de reembolso é o mesmo do Alipay, retornando o resultado do reembolso de forma síncrona, sem notificação assíncrona.

> Segunda diferença:
>
> 1. Antes do pagamento, é necessário verificar se o usuário assinou o contrato, há um processo de assinatura

#### Resumo

O processo de pagamento online doméstico é relativamente completo e a integração é muito fácil. Um ponto a notar é: após o reembolso, o pedido pago anteriormente ainda está no status de pagamento bem-sucedido e não mudará para o status de reembolso. Porque reembolso e pagamento são transações diferentes.

Este ponto é basicamente uma prática comum de pagamento online doméstico.

### Pagamento Internacional

Existem muitas plataformas de pagamento internacional, incluindo Alipay e WeChat que também estão expandindo esse mercado. Farei um breve resumo com base em algumas empresas de pagamento com as quais tive contato.

#### WorldPay

Esta é uma empresa de pagamento internacional bastante famosa, que faz principalmente pagamentos com cartão bancário, e a empresa fica no Reino Unido.

No processo de pagamento, também é construída a url de solicitação de acordo com as regras, e então salta diretamente para a página do **WorldPay** para completar o pagamento com cartão de crédito. O mecanismo de processamento mais problemático aqui é: após o sucesso do pagamento, a primeira notificação de mensagem assíncrona/síncrona que ele fornece não pode ser usada como base para o sucesso do pagamento. Somente após a confirmação real da transferência do banco, a notificação de sucesso real do pagamento será dada. Nesse meio tempo, pode haver uma notificação assíncrona informando que a solicitação de pagamento foi recusada. A maior dor de cabeça é que o intervalo de tempo das mensagens assíncronas de diferentes status é calculado com base em atrasos de nível de minutos ou mais.

No processo de reembolso, o status é o mesmo do WeChat, e o status do reembolso precisa ser confirmado por meio de mensagem assíncrona. Em segundo lugar, sua diferença é que não é possível confirmar se um reembolso já foi iniciado com base no número do pedido de reembolso do comerciante. Portanto, para ele, desde que a interface de reembolso seja solicitada uma vez, ele assume por padrão que um reembolso foi iniciado.

> Terceira e quarta diferenças:
>
> 1. Existem vários status de notificação após o sucesso do pagamento, envolvendo tratamento especial do processo de negócios do sistema do comerciante.
>
> 2. O reembolso não suporta o número do pedido de reembolso do comerciante e não suporta a prevenção de reembolsos duplicados, o que requer que o comerciante trate por conta própria.

#### Assist

Esta é uma empresa de pagamento russa, e também é uma empresa que pode te matar de raiva, veja a introdução abaixo.

A iniciação do pagamento requer a construção de um formulário form para postar os dados relacionados ao pagamento para ele. Após o sucesso, ele saltará para sua página de pagamento e o usuário poderá completar o pagamento. Para **notificação síncrona**, requer que o usuário acione manualmente o salto de volta para o comerciante, muito semelhante à lógica do China Merchants, a sincronização é apenas para retorno e não informará realmente o resultado do pagamento. **Notificação assíncrona** é a que realmente informa o status do pagamento. O que é mais nojento é que, ao pagar, deve-se passar as informações do produto em um formato especificado, o que será usado no reembolso parcial.

Agora falando sobre reembolso, o reembolso também é o mesmo do **WorldPay**, não suporta o número do pedido de reembolso do comerciante, portanto, a prevenção de duplicidade talvez precise ser projetada pelo seu próprio sistema. E se for um reembolso parcial, os produtos de reembolso especificados devem ser passados. Isso criará uma situação muito embaraçosa: o valor do reembolso parcial não corresponde ao valor de nenhum produto, e o reembolso falhará.

> Quinta diferença:
>
> 1. No reembolso parcial, as informações do produto do reembolso parcial devem ser passadas e o valor deve ser consistente.

#### Doku

Vamos falar sobre a instituição de pagamento **doku** na Indonésia. Como a penetração de cartões de crédito neste país não é alta, seu pagamento online oferece um método de pagamento em lojas de conveniência (Supermarket/Convenience Store Payment).

O que é pagamento em lojas de conveniência? Ou seja, após o usuário completar o pedido online, ele receberá um código QR ou código de barras. O usuário leva este código de barras para a loja de conveniência (como 7-11, FamilyMart), o caixa escaneia o código, paga em dinheiro para a loja de conveniência e completa o fluxo de pagamento.

O problema trazido por este método é que o usuário demora muito para pagar, fazendo com que o pedido expire e seja fechado antes do pagamento. Isso traz muitos danos a todo o processo de negócios e à experiência do usuário.

Falando em reembolso, devido à existência deste método de pagamento em lojas de conveniência, este pagamento não suporta reembolso automático online. É necessário coletar manualmente as informações do cartão bancário do usuário e depois completar a operação de transferência. Muito doloroso.

> Sexta diferença:
>
> 1. Não há pagamento online, apenas obtenção do código de pagamento, o reembolso requer operação manual.

#### AmazonPay

Produzido pela Amazon, muito semelhante ao Alipay. Fornece um processo de carteira integrado.

Ao pagar, construa diretamente uma url e pule para a Amazon para concluir o pagamento. Também fornece um modo de autorização, que permite concluir o pagamento no lado do comerciante sem pular para a amazon.

Após o sucesso do pagamento, também haverá um salto síncrono. O conteúdo da **notificação síncrona** pode ser usado como base para julgar se o pagamento foi bem-sucedido. Após verificação real, a chegada da **notificação assíncrona** terá um pequeno atraso, cerca de 10s.

O reembolso também suporta o número do pedido de reembolso do comerciante, que pode ser usado para prevenção de duplicidade. Mas o status do reembolso também é baseado em assíncrono.

### Resumo

Existem outros pagamentos internacionais, como: **PayPal**, **GooglePay**, **PayTM** e outras instituições de pagamento conhecidas que não foram apresentadas, porque basicamente seus processos também estão dentro dos modelos acima. Nosso design subsequente da estrutura de código e design de banco de dados serão baseados no cumprimento dos vários modelos de pagamento acima.

Por fim, dou a todos um mapa mental, esta é uma lista de verificação de problemas que devem ser esclarecidos ao integrar um pagamento.

![pay-2](https://dayutalk.cn/img/pay-2.png)

## Design do Sistema de Pagamento

No texto, conversamos passo a passo de uma perspectiva rigorosa sobre como o pagamento evoluiu para um sistema independente. O conteúdo inclui: processo de evolução do sistema, design de interface, design de banco de dados e exemplos de como o código é organizado. Se houver deficiências, discussões são bem-vindas para aprendermos juntos.

### De Módulo para Serviço

Lembro que quando comecei a trabalhar, todas as funções: adicionar ao carrinho/fazer pedido/pagar e outras lógicas eram colocadas em um único projeto. Se um novo projeto precisasse de uma determinada função, o pacote de função dessa parte era copiado para o novo projeto. O banco de dados também era copiado intacto e ligeiramente modificado de acordo com as necessidades.

Esta é a chamada era da **aplicação monolítica**. À medida que a linha de produtos da empresa começou a se diversificar, cada linha de produtos precisava usar serviços de pagamento. Se o módulo de pagamento ajustasse o código, haveria mudanças em todos os lugares e testes em todos os lugares. Por outro lado, os dados de transação da empresa estavam fragmentados em sistemas diferentes, impossibilitando a agregação e análise unificada eficaz.

Chegou a hora da evolução do sistema. Separamos o módulo de pagamento de cada linha de produtos em um serviço unificado. Fornecemos uma API unificada para uso interno da empresa e podemos empacotar ainda mais essas APIs em SDKs correspondentes para acesso rápido pelas linhas de negócios internas. Aqui, o serviço pode usar o protocolo HTTP ou RPC, dependendo da situação real da empresa. No entanto, se considerarmos o uso futuro por terceiros, é recomendável usar o protocolo HTTP.

**Processo de evolução do sistema:**

![image-20190309104541749](https://dayutalk.cn/img/image-20190309104541749.png)

Resumindo, separar o pagamento em um serviço independente traz os seguintes benefícios:

1. Evitar o desenvolvimento repetido e o fenômeno de isolamento de dados;
2. A evolução das funções periféricas do sistema de pagamento é mais fácil, e todo o sistema é mais completo e rico. Por exemplo: sistema de reconciliação, exibição de dados de transação em tempo real;
3. Pode ser desenvolvido para o exterior a qualquer momento, exportando capacidade Paas e tornando-se um projeto gerador de receita;
4. Equipe dedicada para manutenção, o sistema tem mais chance de evoluir para um sistema de alto nível;
5. Informações importantes da conta da empresa são salvas em um só lugar, com menor risco.

### Capacidade do Sistema

Se assumirmos essa demanda, precisamos construir um sistema de pagamento do zero para a empresa. Por onde devemos começar? Que tipo de capacidades tal sistema precisa ter?

Primeiro, podemos entender o sistema de pagamento como um adaptador. Ele precisa integrar e encapsular muitas interfaces de terceiros e fornecer interfaces unificadas internamente, reduzindo o custo de acesso interno. Como um sistema de pagamento básico, ele precisa fornecer as seguintes interfaces internamente:

1. Iniciar pagamento, nomeamos: `/gopay`
2. Iniciar reembolso, nomeamos: `/refund`
3. Notificação assíncrona da interface, nomeamos: `/notify/canal_de_pagamento/numero_transacao_comerciante`
4. Notificação síncrona da interface, nomeamos: `/return/canal_de_pagamento/numero_transacao_comerciante`
5. Consulta de transação, nomeamos: `/query/trade`
6. Consulta de reembolso, nomeamos: `/query/refund`
7. Obtenção de fatura (Bill), nomeamos: `/query/bill`
8. Detalhes de liquidação, nomeamos: `/query/settle`

Um sistema de pagamento básico certamente precisa fornecer as 8 interfaces acima (aqui ignoramos interfaces como transferência e vinculação de cartão em alguns pagamentos). Agora vamos ver quais sistemas usarão essas interfaces.

![image-20190309111001880](https://dayutalk.cn/img/image-20190309111001880.png)

Abaixo, apresentaremos como usar essas interfaces e algumas lógicas internas de acordo com a dimensão do sistema.

#### Sistema de Aplicação

Geralmente, o gateway de pagamento fornecerá duas maneiras para o sistema de aplicação acessar:

1. Modo Gateway, ou seja, o sistema de aplicação precisa desenvolver seu próprio caixa; (adequado para fornecimento a terceiros)
2. Modo Caixa (Cashier), o sistema de aplicação abre diretamente o caixa unificado do gateway de pagamento. (Negócio interno)

Para explicar claramente a ideia de design, explicaremos de acordo com o **Modo Gateway**.

Para o sistema de aplicação, ele precisa ser capaz de solicitar pagamento, ou seja, chamar a interface `gopay`. Esta interface processará os dados do comerciante, chamará a interface do gateway de terceiros após a conclusão e retornará o resultado unificado para a parte da aplicação.

Aqui deve-se notar que, de acordo com minha experiência, a interface de pagamento de terceiros geralmente tem as seguintes situações:

1. Ao pagar, não há necessidade de chamar o terceiro, basta gerar dados de acordo com as regras;
2. Ao pagar, é necessário chamar várias interfaces de terceiros para completar a lógica (isso pode ser lento, e limitação de fluxo/downgrade deve ser considerado durante grandes eventos);
3. Os dados retornados são uma url, que pode saltar diretamente para o terceiro para completar o pagamento (site wap/pc);
4. Os dados retornados são uma estrutura xml/json, que precisa ser montada ou passada como parâmetro para seu sdk (app).

Devido à inconsistência da estrutura de retorno de terceiros, precisamos processá-la uniformemente em um formato unificado e retorná-la ao lado do comerciante. Recomendo usar o formato json.

```json
{
    "errno":0,
    "msg":"ok",
    "data":{

    }
}
```

Encapsulamos todas as mudanças na estrutura **data**. Por exemplo, se uma url for retornada. O aplicativo só precisa iniciar uma solicitação **GET**. Podemos retornar assim:

```json
{
    "errno":0,
    "msg":"ok",
    "data":{
        "url":"xxxxx",
        "method":"GET"
    }
}
```

Se for uma estrutura retornada, o aplicativo precisa iniciar uma solicitação **POST** diretamente. Podemos retornar assim:

```json
{
    "errno":1,
    "msg":"ok",
    "data":{
        "from":"<form action="xxx" method="POST">xxxxx</form>",
        "method":"POST"
    }
}
```

O campo **form** aqui gera um formulário form, que o aplicativo pode exibir diretamente e enviar automaticamente após obtê-lo. Claro, a etapa de encapsulamento em formulário também pode ser feita no lado do comerciante.

O formato de dados acima é apenas uma referência. Todos podem ajustar de acordo com suas próprias necessidades.

Geralmente, além de chamar a interface para iniciar o pagamento, o sistema de aplicação também pode precisar chamar a **interface de consulta de resultado de pagamento**. Claro, na maioria dos casos não é necessário chamar, o sistema de aplicação deve depender apenas do status do seu próprio sistema para o status da transação.

#### Sistema de Reconciliação

Para reconciliação, geralmente é dividido em dois tipos: **Reconciliação de Transação** e **Reconciliação de Liquidação**

##### Reconciliação de Transação

O ponto central da reconciliação de transação é: **Verificar se cada transação está correta**. Seu objetivo principal é ver se cada transação em nosso sistema é consistente com cada transação do terceiro.

Essa lógica de verificação é muito simples, comparando dois conjuntos de dados de faturamento. Ele usa principalmente a interface `/query/bill` para obter os dados da transação concluída no lado do terceiro. Em seguida, compare com nossos dados de sucesso da transação. Verifique se há erros.

Essa lógica é muito simples, mas há alguns pontos que todos precisam prestar atenção:

1. Nossos dados precisam ser a soma dos dados de pagamento normais + dados de pagamento duplicados;
2. Falha na verificação de reconciliação inclui principalmente: **valor incorreto**, **o terceiro não encontrou os dados de transação correspondentes**, **nosso lado não tem os dados de transação correspondentes**.

Para essas situações, deve haver meios de processamento correspondentes. Na minha experiência, encontrei todas as situações acima.

**Valor incorreto**: Principalmente devido a problemas de terceiros, pode ser falha na atualização do sistema, pode ser erro no valor da interface da fatura;

**Terceiro sem dados de transação:** Pode ser um problema na dimensão de tempo da fatura obtida (por exemplo, diferença de fuso horário), esse problema de fuso horário precisa ser confirmado com o terceiro para encontrar a diferença de tempo correspondente. Também pode ser um ataque, alguém fingindo ser uma notificação assíncrona de terceiro (indicando que o mecanismo de verificação do sistema tem problemas ou a chave vazou).

**Próprio sistema sem dados de transação:** Esse motivo pode ser causado pelo fato de a notificação de terceiro não ter sido enviada ou não ter sido processada corretamente.

O tratamento da grande maioria desses problemas pode depender de `query/trade` `query/refund` para completar o processamento automatizado.

##### Reconciliação de Liquidação

Então, com a **Reconciliação de Transação** acima, por que precisamos da **Reconciliação de Liquidação**? O que este sistema faz? Vamos primeiro ver o significado de liquidação.

> Liquidação é quando o gateway de terceiros transfere o valor de T+x ou outro tempo acordado para a conta da empresa em um ponto fixo no tempo.

Vamos supor que o ciclo de liquidação seja: **T+1**. A interface usada principalmente para reconciliação de liquidação é `/query/settle`. O conteúdo principal obtido por esta interface é: de quais transações (sucesso de transação e dados de reembolso) cada item de liquidação é composto. E quanto de taxa de manuseio foi deduzido nesta liquidação.

Sua lógica é realmente muito simples. Primeiro calculamos quanto a outra parte deve nos transferir de acordo com o ciclo de liquidação de **T+1** do nosso próprio sistema. Em seguida, compare com o valor dos dados obtidos na interface agora:

> Valor recebido pelo banco + Taxa de manuseio = Valor calculado pelo nosso sistema

Após passar nesta verificação, significa que não há problema com o valor. Em seguida, é necessário verificar se cada pedido sob esta liquidação é consistente.

O sistema de liquidação é **fortemente dependente** do sistema de reconciliação. Se a reconciliação encontrar uma anomalia, o valor da liquidação certamente terá uma anomalia. Além disso, alguns problemas que precisam ser observados na liquidação são:

- O banco pode reembolsar o usuário por conta própria, pois o usuário pode solicitar reembolso diretamente ao banco emissor do cartão;
- A liquidação também tem problemas de diferença de fuso horário;
- O status detalhado da transação na interface de liquidação não é totalmente consistente com o nosso. Por exemplo: o banco descobre que um determinado reembolso foi concluído durante a liquidação, mas nosso sistema está processando de acordo com a lógica de reembolso não concluído ao comparar.

Para os problemas acima, todos precisam fazer alguns planos para processamento automatizado de acordo com suas necessidades de negócios.

#### Sistema Financeiro

O sistema financeiro tem muitos negócios internos, aqui falarei apenas sobre os relacionados ao sistema de pagamento. (Claro, o sistema de reconciliação acima também pode ser considerado parte da categoria financeira).

Um ponto principal de relacionamento entre o sistema financeiro e o pagamento está na verificação de transações e reembolsos. Aqui, a verificação da transação pode usar as interfaces `query/trade` `query/refund` para completar. Este processo lógico não precisa ser dito. Vamos nos concentrar no reembolso.

Vejo que o reembolso em muitos sistemas é colocado diretamente na aplicação, e o usuário solicita o reembolso e chama diretamente a interface de reembolso para reembolsar. O risco disso é muito alto. As interfaces do sistema de pagamento relacionadas ao fluxo de fundos devem ser tratadas com cautela e não podem ser excessivamente expostas diretamente ao exterior, trazendo riscos.

A função de reembolso deve ser colocada no sistema financeiro. Desta forma, pode seguir o processo de aprovação interno (se necessário de acordo com o negócio), e mais verificações podem ser realizadas no sistema financeiro para decidir se deve reembolsar imediatamente, ou entrar em processos de espera, rejeição, etc.

#### Gateway de Terceiros

Para terceiros, o que é usado principalmente são as duas interfaces: notificação assíncrona e notificação síncrona. A lógica desta parte é realmente muito simples. É completar a mudança de status da transação de acordo com a notificação do terceiro. E notificar seu sistema de aplicação correspondente.

A parte mais complicada aqui é que a estrutura de dados de notificação de terceiros não é unificada e o tipo de notificação não é unificado. Por exemplo: alguns reembolsos retornam resultados de forma síncrona, alguns retornam resultados de forma assíncrona. Como projetar isso será respondido no **Design do Sistema** posterior.

### Design de Banco de Dados

O design dos dados é baseado em: Transação, Reembolso, Log. Para funções como a reconciliação mencionada acima, não há design aqui. Esta parte não é difícil, todos podem projetar por si mesmos, seguindo as ideias mencionadas acima. As principais tabelas são apresentadas a seguir:

- `pay_transaction` Registra todos os dados de transação.
- `pay_transaction_extension` Registra o número da transação gerado a cada vez que uma transação é iniciada com um terceiro.
- `pay_log_data` Todos os dados de log, como: solicitação de pagamento, solicitação de reembolso, notificação assíncrona, etc.
- `pay_repeat_transaction` Dados de pagamento duplicados.
- `pay_notify_app_log` Log de notificação para o aplicativo.
- `pay_refund` Registra todos os dados de reembolso.

**Estrutura específica das tabelas:**

```sql
-- -----------------------------------------------------
-- Table Criar tabela de fluxo de pagamento
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `pay_transaction` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  `app_id` VARCHAR(32) NOT NULL COMMENT 'id da aplicação',
  `pay_method_id` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'id do método de pagamento, pode ser usado para identificar pagamento, como: Alipay, WeChat, Paypal, etc.',
  `app_order_id` VARCHAR(64) NOT NULL COMMENT 'número do pedido do lado da aplicação',
  `transaction_id` VARCHAR(64) NOT NULL COMMENT 'id único desta transação, único em todo o sistema de pagamento, a razão para gerá-lo é principalmente que order_id pode ser repetido para outras aplicações',
  `total_fee` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'valor do pagamento, salvo como inteiro',
  `scale` TINYINT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'casas decimais correspondentes ao valor',
  `currency_code` CHAR(3) NOT NULL DEFAULT 'CNY' COMMENT 'moeda da transação',
  `pay_channel` VARCHAR(64) NOT NULL COMMENT 'canal de pagamento selecionado, por exemplo: Huabei no Alipay, cartão de crédito, etc.',
  `expire_time` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'tempo de expiração do pedido',
  `return_url` VARCHAR(255) NOT NULL COMMENT 'url de salto após pagamento',
  `notify_url` VARCHAR(255) NOT NULL COMMENT 'url de notificação assíncrona após pagamento',
  `email` VARCHAR(64) NOT NULL COMMENT 'e-mail do usuário',
  `sign_type` VARCHAR(10) NOT NULL DEFAULT 'RSA' COMMENT 'método de assinatura adotado: MD5 RSA RSA2 HASH-MAC etc.',
  `intput_charset` CHAR(5) NOT NULL DEFAULT 'UTF-8' COMMENT 'codificação do conjunto de caracteres',
  `payment_time` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'tempo de sucesso do pagamento de terceiros',
  `notify_time` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'tempo de recebimento da notificação assíncrona',
  `finish_time` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'tempo de notificação ao sistema upstream',
  `trade_no` VARCHAR(64) NOT NULL COMMENT 'número de fluxo do terceiro',
  `transaction_code` VARCHAR(64) NOT NULL COMMENT 'código de transação real para o terceiro, atualizado na notificação assíncrona',
  `order_status` TINYINT NOT NULL DEFAULT 0 COMMENT '0:aguardando pagamento, 1:pagamento pendente conclusão, 2:pagamento concluído, 3:transação fechada, -1:falha no pagamento',
  `create_at` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'tempo de criação',
  `update_at` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'tempo de atualização',
  `create_ip` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'ip de criação, pode ser o ip do próprio serviço',
  `update_ip` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'ip de atualização',
  PRIMARY KEY (`id`),
  UNIQUE INDEX `uniq_tradid` (`transaction_id`),
  INDEX `idx_trade_no` (`trade_no`),
  INDEX `idx_ctime` (`create_at`))
ENGINE = InnoDB
DEFAULT CHARACTER SET = utf8mb4
COMMENT = 'Dados de início de pagamento';

-- -----------------------------------------------------
-- Table Tabela de extensão de transação
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `pay_transaction_extension` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  `transaction_id` VARCHAR(64) NOT NULL COMMENT 'id de transação único do sistema',
  `pay_method_id` INT UNSIGNED NOT NULL DEFAULT 0,
  `transaction_code` VARCHAR(64) NOT NULL COMMENT 'número do pedido gerado para transmissão ao terceiro',
  `call_num` TINYINT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'número de chamadas iniciadas',
  `extension_data` TEXT NOT NULL COMMENT 'conteúdo estendido, precisa salvar: mapeamento entre transaction_code e trade no, preenchido na notificação assíncrona',
  `create_at` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'tempo de criação',
  `create_ip` INT UNSIGNED NOT NULL COMMENT 'ip de criação',
  PRIMARY KEY (`id`),
  INDEX `idx_trads` (`transaction_id`),
  UNIQUE INDEX `uniq_code` (`transaction_code`))
ENGINE = InnoDB
DEFAULT CHARACTER SET = utf8mb4
COMMENT = 'Tabela de extensão de transação';

-- -----------------------------------------------------
-- Table Todos os logs do sistema de transação
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `pay_log_data` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  `app_id` VARCHAR(32) NOT NULL COMMENT 'id da aplicação',
  `app_order_id` VARCHAR(64) NOT NULL COMMENT 'número do pedido do lado da aplicação',
  `transaction_id` VARCHAR(64) NOT NULL COMMENT 'id único desta transação, único em todo o sistema de pagamento, a razão para gerá-lo é principalmente que order_id pode ser repetido para outras aplicações',
  `request_header` TEXT NOT NULL COMMENT 'header da requisição',
  `request_params` TEXT NOT NULL COMMENT 'parâmetros da requisição de pagamento',
  `log_type` VARCHAR(10) NOT NULL COMMENT 'tipo de log, payment:pagamento; refund:reembolso; notify:notificação assíncrona; return:notificação síncrona; query:consulta',
  `create_at` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'tempo de criação',
  `create_ip` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'ip de criação',
  PRIMARY KEY (`id`),
  INDEX `idx_tradt` (`transaction_id`, `log_type`))
ENGINE = InnoDB
DEFAULT CHARACTER SET = utf8mb4
COMMENT = 'Tabela de log de transação';


-- -----------------------------------------------------
-- Table Transações de pagamento duplicado
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `pay_repeat_transaction` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  `app_id` VARCHAR(32) NOT NULL COMMENT 'id da aplicação',
  `transaction_id` VARCHAR(64) NOT NULL COMMENT 'número de identificação única da transação do sistema',
  `transaction_code` VARCHAR(64) NOT NULL COMMENT 'código desta transação quando o pagamento for bem-sucedido',
  `trade_no` VARCHAR(64) NOT NULL COMMENT 'número da transação correspondente do terceiro',
  `pay_method_id` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'método de pagamento',
  `total_fee` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'valor da transação',
  `scale` TINYINT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'casas decimais',
  `currency_code` CHAR(3) NOT NULL DEFAULT 'CNY' COMMENT 'moeda selecionada para pagamento, CNY, HKD, USD, etc.',
  `payment_time` INT NOT NULL COMMENT 'tempo da transação de terceiro',
  `repeat_type` TINYINT UNSIGNED NOT NULL DEFAULT 1 COMMENT 'tipo de duplicidade: 1 pagamento no mesmo canal, 2 pagamento em canais diferentes',
  `repeat_status` TINYINT UNSIGNED DEFAULT 0 COMMENT 'status de processamento, 0: não processado; 1: processado',
  `create_at` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'tempo de criação',
  `update_at` INT UNSIGNED NOT NULL COMMENT 'tempo de atualização',
  PRIMARY KEY (`id`),
  INDEX `idx_trad` ( `transaction_id`),
  INDEX `idx_method` (`pay_method_id`),
  INDEX `idx_time` (`create_at`))
ENGINE = InnoDB
DEFAULT CHARACTER SET = utf8mb4
COMMENT = 'Registrar pagamentos duplicados';


-- -----------------------------------------------------
-- Table Log de notificação para aplicação upstream
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `pay_notify_app_log` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  `app_id` VARCHAR(32) NOT NULL COMMENT 'id da aplicação',
  `pay_method_id` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'método de pagamento',
  `transaction_id` VARCHAR(64) NOT NULL COMMENT 'número da transação',
  `transaction_code` VARCHAR(64) NOT NULL COMMENT 'código desta transação quando o pagamento for bem-sucedido',
  `sign_type` VARCHAR(10) NOT NULL DEFAULT 'RSA' COMMENT 'método de assinatura adotado: MD5 RSA RSA2 HASH-MAC etc.',
  `input_charset` CHAR(5) NOT NULL DEFAULT 'UTF-8',
  `total_fee` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'valor envolvido, sem decimais',
  `scale` TINYINT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'casas decimais',
  `pay_channel` VARCHAR(64) NOT NULL COMMENT 'canal de pagamento',
  `trade_no` VARCHAR(64) NOT NULL COMMENT 'número da transação de terceiro',
  `payment_time` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'tempo de pagamento',
  `notify_type` VARCHAR(10) NOT NULL DEFAULT 'paid' COMMENT 'tipo de notificação, paid/refund/canceled',
  `notify_status` VARCHAR(7) NOT NULL DEFAULT 'INIT' COMMENT 'resultado da notificação ao chamador do pagamento; INIT: inicialização, PENDING: em andamento; SUCCESS: sucesso; FAILED: falha',
  `create_at` INT UNSIGNED NOT NULL DEFAULT 0,
  `update_at` INT UNSIGNED NOT NULL DEFAULT 0,
  PRIMARY KEY (`id`),
  INDEX `idx_trad` (`transaction_id`),
  INDEX `idx_app` (`app_id`, `notify_status`),
  INDEX `idx_time` (`create_at`))
ENGINE = InnoDB
DEFAULT CHARACTER SET = utf8mb4
COMMENT = 'Registro do chamador de pagamento';


-- -----------------------------------------------------
-- Table Reembolso
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `pay_refund` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  `app_id` VARCHAR(64) NOT NULL COMMENT 'id da aplicação',
  `app_refund_no` VARCHAR(64) NOT NULL COMMENT 'id de reembolso do upstream',
  `transaction_id` VARCHAR(64) NOT NULL COMMENT 'número da transação',
  `trade_no` VARCHAR(64) NOT NULL COMMENT 'número da transação de terceiro',
  `refund_no` VARCHAR(64) NOT NULL COMMENT 'número único do pedido de reembolso gerado pela plataforma de pagamento',
  `pay_method_id` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'método de pagamento',
  `pay_channel` VARCHAR(64) NOT NULL COMMENT 'canal de pagamento selecionado, por exemplo: Huabei no Alipay, cartão de crédito, etc.',
  `refund_fee` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'valor do reembolso',
  `scale` TINYINT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'casas decimais',
  `refund_reason` VARCHAR(128) NOT NULL COMMENT 'motivo do reembolso',
  `currency_code` CHAR(3) NOT NULL DEFAULT 'CNY' COMMENT 'moeda, CNY USD HKD',
  `refund_type` TINYINT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'tipo de reembolso; 0: reembolso de negócio; 1: reembolso duplicado',
  `refund_method` TINYINT UNSIGNED NOT NULL DEFAULT 1 COMMENT 'método de reembolso: 1 retorno automático pelo caminho original; 2 transferência manual',
  `refund_status` TINYINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '0 não reembolsado; 1 reembolso em processamento; 2 reembolso bem-sucedido; 3 reembolso mal-sucedido',
  `create_at` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'tempo de criação',
  `update_at` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'tempo de atualização',
  `create_ip` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'ip de origem da requisição',
  `update_ip` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'ip de origem da requisição',
  PRIMARY KEY (`id`),
  UNIQUE INDEX `uniq_refno` (`refund_no`),
  INDEX `idx_trad` (`transaction_id`),
  INDEX `idx_status` (`refund_status`),
  INDEX `idx_ctime` (`create_at`))
ENGINE = InnoDB
DEFAULT CHARACTER SET = utf8mb4
COMMENT = 'Registro de reembolso';
```



A lógica de uso da tabela é descrita brevemente abaixo:

**Pagamento**, primeiro precisa registrar o log de requisição em `pay_log_data`, e então gerar dados de transação para registrar em `pay_transaction` e `pay_transaction_extension`.

**Recebimento de notificação**, registrar dados em `pay_log_data`, e então atualizar o status de `pay_transaction` e `pay_refund` dependendo se é uma notificação de pagamento ou notificação de reembolso. Se for pagamento duplicado, precisa registrar dados em `pay_repeat_transaction`. E registrar os dados que precisam notificar a aplicação em `pay_notify_app_log`, esta tabela é equivalente a uma tabela de mensagens, e haverá consumidores que consumirão seu conteúdo.

**Reembolso**, registrar log em `pay_log_data`, e então registrar dados na tabela de reembolso `pay_refund`.

Claro, há alguns detalhes nisso, que exigem que todos vejam a estrutura da tabela e pensem em como usá-la na prática. Se tiver alguma dúvida, deixe uma mensagem em nosso projeto GitHub (clique em Leia o Artigo Original), e responderemos uma a uma.

> Essas tabelas podem atender às necessidades mais básicas, outros conteúdos podem ser expandidos de acordo com suas próprias necessidades, como: suporte à lista de cartões do usuário, reembolso via cartão bancário, etc.

### Design do Sistema

Esta parte fala principalmente sobre como construir o sistema e sugestões sobre a organização do código.

#### Arquitetura do Sistema

Como a segurança do sistema de pagamento é muito alta, não é recomendável expor a entrada correspondente diretamente visível ao usuário. Deveria ser o seu próprio sistema de aplicação chamando a interface do sistema de pagamento para completar o negócio. Além disso, o requisito de dados do sistema é: consistência forte. Portanto, não há intervenção de cache (cache pode ser usado para alarmes, o que não está no escopo deste artigo).

![image-20190309135800643](https://dayutalk.cn/img/image-20190309135800643.png)

Na implementação específica, o sistema usará dois nomes de domínio, um para uso interno, onde apenas ips de origem especificados podem acessar funções fixas (acessar outras funções exceto notificação). O outro nome de domínio só pode acessar as duas rotas `notify` e `return`. Desta forma, a segurança do sistema pode ser garantida.

No uso do banco de dados, independentemente da solicitação, vá diretamente para o banco **Master**. Isso garante forte consistência dos dados. Claro, bancos escravos também são necessários. Por exemplo: lógica relacionada a faturas e reconciliação pode ser completada usando bancos escravos.

#### Design de Código

Não importa o que você queira fazer, no final deve ser implementado com código. Todos nós sabemos que precisamos de código sustentável e extensível. Então, especificamente para o sistema de pagamento, o que você faria? Usarei o pagamento como exemplo para explicar minha ideia de design de estrutura de código. Apenas para referência. Por exemplo, quero integrar: WeChat, Alipay e China Merchants Bank, três pagamentos. Meu diagrama de estrutura de código é o seguinte:

![image-20190309142925499](https://dayutalk.cn/img/image-20190309142925499.png)

Vou apresentar brevemente com texto. Vou encapsular cada terceiro em: classe `XXXGateway`, que encapsula puramente a interface de terceiros, independentemente de a outra parte ser uma solicitação HTTP ou solicitação SOAP, tudo é processado uniformemente internamente.

Além disso, há uma camada `XXXProxy` para encapsular as capacidades fornecidas por esses terceiros. Esta camada faz principalmente duas coisas: processamento personalizado dos dados de solicitação de pagamento recebidos. Processamento unificado da estrutura retornada para retornar a estrutura unificada da camada superior. Claro, dependendo de circunstâncias especiais, todo o processamento de negócios pode ser realizado aqui;

Através das operações acima, as mudanças foram basicamente completamente encapsuladas. Se adicionar um novo canal de pagamento. Só precisa adicionar: `XXXGateway` e `XXXProxy`.

Então, para que servem `Context` e `Server`? `Server` encapsula toda a lógica de negócios internamente e fornece interfaces para action ou outros servers chamarem. E o valor da existência da camada `Context` é lidar com os erros retornados pela camada `Proxy`. E realizar o processamento relacionado a alarmes aqui.

A estrutura acima é apenas uma prática minha, discussões são bem-vindas.

O sistema descrito neste artigo atende apenas aos requisitos de pagamento mais básicos. Faltam monitoramento e alarme relacionados. Se você projetar seu próprio sistema de acordo com o texto acima, o risco é seu e não tem nada a ver comigo.
