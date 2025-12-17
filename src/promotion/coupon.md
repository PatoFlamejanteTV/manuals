# Serviço de Cupons

## Prefácio

Entrando no tópico principal, o serviço básico do sistema de marketing: "Serviço de Cupons". Apresentamos os cupons através das seguintes perguntas:

- Quais são os **tipos** de cupons?
- Quais são os **escopos de aplicação** dos cupons?
- Quais são os **cenários comuns** para cupons?
- Quais **status** o próprio cupom deve ter?
- Quais **capacidades de serviço** o serviço de cupons deve ter?
- Como fazer o **controle de risco** do serviço de cupons?

## Quais são os tipos de cupons?

Para os usuários que obtêm cupons, o foco está na capacidade de desconto do cupom, portanto, de acordo com a dimensão da capacidade de desconto, os cupons são divididos principalmente nas três categorias a seguir:

Dimensão da capacidade de desconto|Descrição
------------|------------
Cupom de desconto por valor (Man Jian)|Quanto desconto em valor pode ser obtido ao atingir um certo montante (excluindo frete)
Cupom de dinheiro (Cash)|Quanto dinheiro pode ser deduzido (sem limite mínimo)
Cupom de dedução|Deduz o valor total de um determinado Sku (uma quantidade)
Cupom de desconto percentual|Desconto percentual

Para o pessoal de operação que emite cupons:

Um tipo é "**Validade fixa**", os carimbos de data/hora de início e expiração do cupom já estão determinados quando o cupom é criado. Independentemente de quando o usuário retira o cupom, o tempo de validade do cupom é o tempo de início e fim da validade definido anteriormente.

O outro tipo é "**Validade dinâmica**", ao criar o cupom, define-se um período de validade, como 7 dias de validade, 12 horas de validade, etc. Para esses cupons, o tempo de início da validade é o momento em que o usuário retira o cupom, e o tempo de término da validade é o momento em que o usuário retira o cupom + o tempo de validade.

Dimensão da validade|Tipo de cupom|Tempo de início do cupom|Tempo de expiração do cupom|Descrição
------------|------------|------------|------------|------------
Fixa|Validade fixa|Determinado quando o tipo de cupom é criado|Determinado quando o tipo de cupom é criado|Independentemente de quando o usuário retira o cupom, o tempo de validade do cupom é o tempo unificado definido
Dinâmica|Validade dinâmica|Quando o usuário retira o cupom, timestamp atual|Quando o usuário retira o cupom, timestamp atual + N\*24\*60\*60|Quando o tipo de cupom é criado, apenas a validade do cupom é determinada, por exemplo, 6 horas, 7 dias, um mês

O resumo é o seguinte:

<p align="center">
    <a href="http://cdn.veaer.com/coupon-type.png" target="_blank">
        <img src="http://cdn.veaer.com/coupon-type.png" width="100%">
    </a>
</p>


## Quais são os escopos de aplicação dos cupons?

### Estratégia de Operação

Dimensão de Restrição|Detalhes da Restrição|Descrição
------------|------------|------------
Dimensão do Produto||
|(Não) Sku Especificado|Cupom de Sku
|(Não) Spu Especificado|Cupom de Spu
|(Não) Categoria Especificada|Cupom de Categoria
Dimensão da Plataforma||
|Loja Especificada|Cupom de Loja
|Uso Geral em Todo o Site|Cupom de Plataforma

### Terminais Aplicáveis

Terminais Aplicáveis (Checkbox)|Descrição
------------|------------
Android|Terminal Android
iOS|Terminal iOS
PC|Terminal Web PC
Mobile|Terminal Web Mobile
Wechat|Terminal WeChat
Mini Programa WeChat|Mini Programa WeChat
All|Todos os acima

### Métodos de Pagamento

Tipo de Pagamento|Descrição
------------|------------
Pagamento na Entrega|
Pagamento Online|Alipay/WeChat/UnionPay
All|

### Público Aplicável

Público Aplicável|Descrição
------------|------------
Whitelist|Usuários de Teste
Membro|Exclusivo para Membros

O resumo é o seguinte:

<p align="center">
    <a href="http://cdn.veaer.com/coupon-scope.jpg" target="_blank">
        <img src="http://cdn.veaer.com/coupon-scope.jpg" width="80%">
    </a>
</p>




## Quais são os cenários comuns para cupons?

### Cenários de Retirada de Cupons

Cenário de Retirada de Cupons|Descrição
------------|------------
Página de Atividade|Botão para obter cupons exibido em páginas de grandes promoções e feriados
Página de Jogo|Obter cupons através de jogos
Página Inicial da Loja|Entrada para retirada de cupons exibida na página inicial da loja
Detalhes do Produto|Entrada para retirada de cupons exibida na página de detalhes do produto
Centro de Pontos|Troca de pontos por cupons

### Cenários de Exibição de Cupons

Cenário de Exibição de Cupons|Descrição
------------|------------
Página de Atividade|Exibição de cupons que podem ser retirados em páginas de grandes promoções e feriados
Detalhes do Produto|Lista de cupons que podem ser retirados e usados exibida na página de detalhes do produto
Centro Pessoal - Meus Cupons|Minha lista de cupons
Página de Checkout do Pedido|Lista de cupons aplicáveis a este pedido e recomendações na página de checkout
Centro de Pontos|Exibição de detalhes de cupons que podem ser trocados


### Cenários de Seleção de Cupons

Cenário de Seleção de Cupons|Descrição
------------|------------
Detalhes do Produto|Exibição de cupons que o usuário já possui e são aplicáveis a este produto na página de detalhes do produto
Página de Checkout do Pedido - Lista de Cupons|Seleção de cupom disponível para checkout
Página de Checkout do Pedido - Inserir Código de Cupom|Inserir código de cupom para checkout

### Cenários de Devolução de Cupons

Cenário de Devolução de Cupons|Descrição
------------|------------
Cancelamento de Pedido Não Pago|Pedidos não pagos, usuário cancela ativamente para devolver o cupom, ou fechamento de pedido por tempo limite devolve o cupom
Cancelamento de Pedido Pago Integralmente|Pedidos pagos (não recebidos), reembolso parcial do pedido não devolve, quando todo o pedido é reembolsado, o cupom é devolvido

### Exemplos de Cenários de Obtenção de Cupons

Exemplo de Cenário|Descrição
------------|------------
Retirada de Cupom na Página de Atividade|Botão para obter cupons exibido em páginas de grandes promoções e feriados
Emissão de Cupom em Jogo|Recompensa de jogo
Retirada de Cupom na Página do Produto|-
Retirada de Cupom na Página da Loja|-
Retorno de Cupom na Compra|Após a compra de um determinado Sku e a entrega do pedido, um cupom é emitido
Emissão de Cupom para Novo Usuário|Emissão de cupom ao registrar novo usuário
Troca de Pontos por Cupom|Troca de pontos por cupons
Retorno de Cupom por Comentário|Após o usuário receber o pedido e adicionar um comentário (após revisão), um cupom especificado é devolvido ao usuário

O resumo é o seguinte:

<p align="center">
    <a href="http://cdn.veaer.com/coupon-scene.jpg" target="_blank">
        <img src="http://cdn.veaer.com/coupon-scene.jpg" width="80%">
    </a>
</p>


## Quais status o próprio cupom deve ter?

<p align="center">
    <a href="http://cdn.veaer.com/coupon-status.jpg" target="_blank">
        <img src="http://cdn.veaer.com/coupon-status.jpg" width="60%">
    </a>
</p>


## Quais capacidades de serviço o serviço de cupons deve ter?

### Capacidade de Serviço 1: Emissão de Cupons

Método de Emissão|Descrição
------------|------------
Emissão Síncrona|Aplicável a cenários de obtenção de cupons com altos requisitos de tempo real, como o usuário clicando para retirar cupons
Emissão Assíncrona|Aplicável a cenários de emissão de cupons com baixos requisitos de tempo real, como emissão de cupons para registro de novos usuários

Capacidade de Emissão|Descrição
------------|------------
Emissão Única|Especificar um ID de tipo de cupom e um UID para emitir apenas um cupom
Emissão em Lote|Especificar um ID de tipo de cupom e um lote de UIDs, cada UID recebe apenas um cupom

Tipo de Emissão|Descrição
------------|------------
Identificador de Tipo de Cupom|Emitido através do identificador de identidade do tipo de cupom, por exemplo, ao criar um tipo de cupom, um código de identificação de 16 dígitos será gerado, e o usuário retira o cupom através do `código de identificação de 16 dígitos`; ID auto-incremental não é usado aqui (para evitar vazar o número de cupons criados historicamente para o exterior)
Código Promocional (Code)|Ao criar um tipo de cupom, a equipe de operação preencherá um código Ascii de cerca de 6 dígitos para o cupom, como `SKR6a6`, e o usuário retira o cupom através desse código

### Capacidade de Serviço 2: Revogação de Cupons no Backend

Capacidade de Revogação|Descrição
------------|------------
Revogação Única|Especificar um ID de cupom e um UID, verificar se a propriedade é legal e depois revogar um cupom
Revogação em Lote|Especificar um ID de tipo de cupom e um lote de UIDs, revogar um cupom para cada UID

### Capacidade de Serviço 3: Mudança de Status Acompanhando o Sistema de Pedidos

| Tipo de Mudança | Descrição                                 |
| -------- | ------------------------------------ |
| Congelar     | Vincular e ocupar o cupom ao fazer o pedido       |
| Descongelar     | Cancelar o pedido antes do pagamento do usuário, descongelar e devolver o cupom |
| Baixar (Write-off)     | Marcar o cupom congelado como usado       |

| Capacidade de Mudança           | Descrição                                                 |
| ------------------ | ---------------------------------------------------- |
| Congelar/Descongelar/Baixar Único | Especificar um ID de cupom, um ID de pedido, alterar o status do cupom        |
| Congelar/Descongelar/Baixar em Lote | Especificar um mapeamento de IDs de cupons e IDs de pedidos, realizar alteração de status do cupom |

### Capacidade de Serviço 4: Consulta de Lista de Cupons

Lista de Cupons do Usuário|Subclasse|Descrição
------------|------------|------------
Todos|-|Consultar todos os cupons deste usuário
Utilizáveis|Todos|Consultar todos os cupons que este usuário pode usar
-|Aplicável a um determinado spu ou sku|Consultar cupons utilizáveis deste usuário aplicáveis a um determinado spu ou sku
-|Aplicável a uma determinada categoria|Consultar cupons utilizáveis deste usuário aplicáveis a uma determinada categoria
-|Aplicável a uma determinada loja|Consultar cupons utilizáveis deste usuário aplicáveis a uma determinada loja
Usados|Todos|Consultar todos os cupons que este usuário já usou
-|Uso Normal|Consultar cupons usados em todos os pedidos concluídos deste usuário
-|Congelado em Pedido|Consultar cupons congelados em todos os pedidos não fechados deste usuário
Inválidos|Todos|Consultar cupons que este usuário possui, mas não estão disponíveis
-|Expirado|Consultar todos os cupons expirados deste usuário
-|Invalidado|Consultar todos os cupons deste usuário congelados pelo backend

### Capacidade de Serviço 5: Consulta Condicional de Cupons

Existem dois cenários de consulta condicional

1. A página de exibição de produto/loja pode exibir a coleção de cupons já retirados/que podem ser retirados para este produto/loja
2. Filtrar a coleção de cupons disponíveis atualmente com base no produto/loja/método de pagamento ao fazer o pedido

Nota: A coleção de cupons da página de atividade geralmente é personalizada com o sistema de operação correspondente e não está incluída aqui

### Capacidade de Serviço 6: Recomendação de Cupom na Página de Checkout

Recomendar o cupom mais adequado (valor) para o pedido na página de checkout do pedido

O resumo é o seguinte:

<p align="center">
    <a href="http://cdn.veaer.com/coupon-serve.jpg" target="_blank">
        <img src="http://cdn.veaer.com/coupon-serve.jpg" width="100%">
    </a>
</p>


## Como fazer o controle de risco do serviço de cupons?

Uma vez que haja a possibilidade de risco, o controle de risco é acionado:

- Para o usuário, sugerir tentar novamente mais tarde ou entrar em contato com o atendimento ao cliente
- Para o interno, alerta de alarme, verificar se há problema com o alarme

### Limite de Frequência

Retirada|Descrição
------------|------------
ID do Dispositivo|Limite de número de retiradas de um determinado cupom por dia
UID|Limite de número de retiradas de um determinado cupom por dia
IP|Limite de número de retiradas de um determinado cupom por dia

Uso|Descrição
------------|------------
ID do Dispositivo|Limite de número de uso de um determinado cupom por dia
UID|Limite de número de uso de um determinado cupom por dia
IP|Limite de número de uso de um determinado cupom por dia
Número de Celular|Limite de número de uso de um determinado cupom por dia
Código Postal|Por exemplo, em regiões ultramarinas que focam em códigos postais, limite de número de uso de um determinado cupom por dia

### Nível de Risco do Usuário

Com base nos dados históricos de pedidos do usuário, obter a taxa de conclusão bem-sucedida de transações do usuário (por exemplo, entrega bem-sucedida 15 dias+). Classificar os usuários com base nessa taxa, níveis altos entram na lista Unblock, níveis baixos entram na lista Block, e definir estratégias de restrição de acordo com diferentes níveis de usuário. E outros meios de análise de big data.

### Limiar

- Orçamento de emissão de cupons
- Orçamento de uso real de cupons

Definir o limiar total de emissão de cupons com base no valor do orçamento. Quando o limiar for acionado, bloquear e alarmar.

### Cupons não devem suportar produtos virtuais

Cupons devem tentar não suportar produtos virtuais para evitar atividades ilegais que possam ser exploradas.

<p align="center">
    <a href="http://cdn.veaer.com/coupon-control.png" target="_blank">
        <img src="http://cdn.veaer.com/coupon-control.png" width="80%">
    </a>
</p>
