## Prefácio

Até agora, o conteúdo organizado pela série SkrShop "Manual de Design de E-commerce" já abrangeu os seguintes grandes blocos:

- Usuário
- Produto
- Carrinho de Compras
- Marketing
- Pagamento
- Serviços Básicos

Hoje estamos prontos para iniciar um novo capítulo: **Centro de Pedidos**.

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201026131854.jpg" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201026131854.jpg" width="100%">
    </a>
</p>

O conteúdo principal da série Centro de Pedidos é o seguinte:

|Ponto de Conhecimento|
|-------|
|Página de Checkout do Pedido|
|Criar Pedido|
|Cumprimento do Pedido|
|Status do Pedido|
|Detalhes do Pedido|
|Operação Reversa do Pedido|
|...|

Primeiro, vamos revisar um processo simples de compra de usuários em plataformas de e-commerce, conforme mostrado na figura abaixo:

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201015193036.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201015193036.png" width="80%">
    </a>
</p>

> Então, sobre o que vamos conversar hoje?

```
Resposta: No artigo de hoje, vamos falar principalmente sobre o design e a implementação da "Página de Checkout do Pedido" no fluxo acima.
```


## Como é a Página de Checkout do Pedido?

Vamos ver a página de checkout de pedidos do JD:

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200331124724.jpeg" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200331124724.jpeg" width="36%">
    </a>
</p>

Vamos ver a página de checkout de pedidos do Taobao:

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200929124345.jpeg" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20200929124345.jpeg" width="36%">
    </a>
</p>

Através das capturas de tela acima, podemos obter aproximadamente o conteúdo principal da **página de checkout de pedido**:

- Informações de endereço de entrega padrão do usuário
- Seleção de método de pagamento
- Informações da loja e do produto
- Métodos de entrega selecionáveis para o produto
- Seleção de tipo de fatura
- Informações de desconto
- Valor relacionado ao pedido
- E assim por diante

## Composição da Página de Checkout do Pedido

> Sempre estive pensando, se o frontend pode ser modular, os dados da interface de backend não podem ser modulares?

```
Minha resposta: Sim, podem.
```

Com base no conteúdo organizado acima, e através da experiência anterior, realizamos a divisão modular e combinação da **página de checkout de pedido**, obtendo a seguinte **composição modular** da página de checkout de pedido:

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201026165711.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201026165711.png" width="38%">
    </a>
</p>

Sobre como projetar este código, você pode consultar meu artigo [《Componente de Código | Meu código não tem else》](http://tigerb.cn/go-patterns/#/?id=%e7%bb%84%e5%90%88%e6%a8%a1%e5%bc%8f)

## Análise de cada módulo da página de checkout de pedido

Número do Módulo|Nome do Módulo|Número do Submódulo|Nome do Submódulo|Descrição do Módulo
------------|------------|------------|------------|------------
1|Módulo de Endereço|-|-|Exibir o endereço ideal do usuário
2|Módulo de Método de Pagamento|-|-|Métodos de pagamento suportados por este pedido
3|Módulo de Loja|-|-|Inclui informações da loja, informações do produto, informações de desconto participantes, métodos de logística opcionais, informações de pós-venda do produto, etc.
3|-|3.1|Módulo de Produto|Inclui submódulos: módulo de informações básicas do produto, módulo de informações de desconto do produto, módulo de pós-venda
3|-|3.2.1|Módulo de Informações Básicas do Produto|Informações do produto, nome, imagem, preço, estoque, etc.
3|-|3.2.2|Módulo de Informações de Desconto do Produto|Opções de desconto de atividade de vendas selecionadas
3|-|3.2.3|Módulo de Pós-venda|Informações de direitos pós-venda que o produto desfruta
3|-|3.3|Módulo de Logística|Métodos de entrega selecionáveis
3|-|3.4|Módulo de Informações de Valor do Produto da Loja|-
4|Módulo de Fatura|-|-|Selecionar o tipo de fatura, complementar informações da fatura
5|Módulo de Cupom|-|-|Exibir a lista de cupons que podem ser usados neste pedido
6|Módulo de Cartão Presente|-|-|Exibir a lista de cartões presente que podem ser selecionados para uso
7|Módulo de Pontos da Plataforma|-|-|Usuários podem usar pontos para abater parte do dinheiro
8|Módulo de Informações de Valor do Pedido|-|-|Inclui detalhes do valor deste pedido

## Módulo de Endereço

> Exibir o endereço ideal do usuário

Lógica de endereço ideal:

- Primeiro, o endereço padrão definido pelo usuário
- Se não houver endereço padrão, retornar o endereço do pedido mais recente

Nome do Campo|Tipo|Nome do Campo Inferior|Tipo|Significado do Campo
------|------|------|------|------
consignee|string|-|-|Nome do destinatário
email|string|-|-|E-mail do destinatário (parte do nome de usuário mascarada no valor de retorno)
mobile|string|-|-|Número de celular do destinatário (quatro dígitos do meio mascarados no valor de retorno)
country|object|id|int64|ID do País
country|object|name|string|Nome do País
province|object|id|int64|ID da Província
province|object|name|string|Nome da Província
city|object|id|int64|ID da Cidade
city|object|name|string|Nome da Cidade
county|object|id|int64|ID do Distrito/Condado
county|object|name|string|Nome do Distrito/Condado
street|object|id|int64|ID da Rua/Vila
street|object|name|string|Nome da Rua/Vila
detailed_address|string|-|-|Endereço detalhado (preenchido manualmente pelo usuário)
postal_code|string|-|-|CEP
address_id|int64|-|-|ID do Endereço
is_default|bool|-|-|Se é o endereço padrão
label|string|-|-|Rótulo do tipo de endereço, casa, empresa, etc.
longitude|string|-|-|Longitude
latitude|string|-|-|Latitude

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201010203421.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201010203421.png" width="100%">
    </a>
</p>

Demo de dados do módulo:
```json
{
    "address_module": {
        "consignee": "Nome do Destinatário",
        "email": "E-mail do destinatário (parte do nome de usuário mascarada)",
        "mobile": "Celular do destinatário (quatro dígitos do meio mascarados)",
        "country": {
            "id": 666,
            "name": "Nome do País"
        },
        "province": {
            "id": 12123,
            "name": "Nome da Província"
        },
        "city": {
            "id": 212333,
            "name": "Nome da Cidade"
        },
        "county": {
            "id": 1233222,
            "name": "Nome do Distrito"
        },
        "street": {
            "id": 9989999,
            "name": "Nome da Rua"
        },
        "detailed_address": "Endereço detalhado (preenchido manualmente)",
        "postal_code": "CEP",
        "address_id": 212399999393,
        "is_default": false,
        "label": "Rótulo do tipo de endereço, casa, empresa, etc.",
        "longitude": "Longitude",
        "latitude": "Latitude"
    }
}
```

## Módulo de Método de Pagamento

> Métodos de pagamento suportados por este pedido

Opções de método de pagamento:

- Pagamento Online
- Pagamento na Entrega

Nome do Campo|Tipo|Nome do Campo Inferior|Tipo|Significado do Campo
------|------|------|------|------
pay_method_list|array|id|int|ID do método de pagamento
pay_method_list|array|name|string|Nome do método de pagamento
pay_method_list|array|desc|string|Descrição do método de pagamento


<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201010203818.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201010203818.png" width="100%">
    </a>
</p>

Demo de dados do módulo:
```json
{
    "pay_method_module": {
        "pay_method_list": [
            {
                "id": 1,
                "name": "Pagamento Online",
                "desc": "Descrição do pagamento online"
            },
            {
                "id": 2,
                "name": "Pagamento na Entrega",
                "desc": "Descrição do pagamento na entrega"
            }
        ]
    }   
}
```

## Módulo de Loja

> Inclui informações da loja, informações do produto, informações de desconto participantes, métodos de logística opcionais, informações de pós-venda do produto, etc.

O módulo de loja é composto pelos seguintes submódulos:

- Módulo de Produto
    + Módulo de informações básicas do produto
    + Módulo de informações de desconto do produto
    + Módulo de pós-venda
- Módulo de Logística do Produto
- Módulo de Informações de Valor Total do Produto da Loja

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201014203138.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201014203138.png" width="36%">
    </a>
</p>

Devido à grande quantidade de conteúdo aqui, analisaremos separadamente mais tarde.

## Módulo de Fatura

> O usuário seleciona o tipo de fatura e complementa as informações da fatura

Selecionar tipo de fatura:

- Pessoal
- Unidade (Empresa)

Nome do Campo|Tipo|Nome do Campo Inferior|Tipo|Significado do Campo
------|------|------|------|------
type_id|int|-|-|Tipo de fatura: Pessoal; Unidade
type_name|string|-|-|Nome do tipo de fatura
type_desc|string|-|-|Descrição do tipo de fatura


<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201015195856.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201015195856.png" width="100%">
    </a>
</p>

Demo de dados do módulo:
```json
{
    "invoice_module": {
        "type_list": [
            {
                "type_id": 1,
                "type_name": "Pessoal",
                "type_desc": "Descrição"
            },
            {
                "type_id": 2,
                "type_name": "Empresa",
                "type_desc": "Descrição"
            }
        ]
    }
}
```

## Módulo de Cupom

> Retorna a lista de cupons que podem ser usados neste pedido, bem como o cupom ideal selecionado por padrão para o pedido atual

- Exibir lista de cupons do usuário: disponíveis para o pedido atual no topo, outros no final
- Seleção padrão do cupom ideal: o cupom com o maior desconto para o pedido atual

Para outros conteúdos sobre cupons, você pode ler o capítulo sobre Cupons.

## Módulo de Cartão Presente

> Exibir a lista de cartões presente que podem ser selecionados para uso

Nome do Campo|Tipo|Nome do Campo Inferior|Tipo|Significado do Campo
------|------|------|------|------
giftcard_list|array|id|int64|id do cartão presente
giftcard_list|array|name|string|Nome do cartão presente
giftcard_list|array|desc|string|Descrição do cartão presente
giftcard_list|array|pic_url|string|Imagem do cartão presente
giftcard_list|array|total_amount|float64|Valor total inicial do cartão presente
giftcard_list|array|total_amount_txt|string|Valor total inicial do cartão presente - formatado
giftcard_list|array|remaining_amount|float64|Valor restante do cartão presente
giftcard_list|array|remaining_amount_txt|string|Valor restante do cartão presente - formatado


<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201026165855.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201026165855.png" width="100%">
    </a>
</p>

Demo de dados do módulo:
```json
{
    "giftcard_module": {
        "giftcard_list": [
            {
                "id": 341313121,
                "name": "Nome do Cartão Presente",
                "desc": "Descrição do Cartão Presente",
                "pic_url": "Imagem do Cartão Presente",
                "total_amount": 100.00,
                "total_amount_txt": "100.00",
                "remaining_amount": 21.00,
                "remaining_amount_txt": "21.00"
            }
        ]
    }
}
```

## Módulo de Pontos da Plataforma

> Usuários podem usar pontos para abater dinheiro

Por exemplo, os Jingdou na página de checkout de pedidos do JD.

Nome do Campo|Tipo|Nome do Campo Inferior|Tipo|Significado do Campo
------|------|------|------|------
order_amount_min|float64|-|-|Limite inferior do valor do pedido para usar a função de abatimento por pontos
total_points|int64|-|-|Total de pontos do usuário
can_use_points|int64|-|-|Pontos utilizáveis (pode haver pontos congelados)
points2money_rate|int|-|-|Taxa de conversão de pontos para dinheiro, por exemplo, cada 100 pontos valem 1 yuan, mínimo de 1 ponto vale 0,01 yuan
points2money_min|int|-|-|Mínimo de pontos que o usuário deve ter para usar o abatimento por pontos
points2money_max|int|-|-|Limite máximo de pontos que podem ser usados em um único pedido
points_amount|float64|-|-|Valor que pode ser abatido por pontos neste pedido
points_amount_txt|string|-|-|Valor que pode ser abatido por pontos neste pedido - formatado


<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201015193559.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201015193559.png" width="100%">
    </a>
</p>

Demo de dados do módulo:
```json
{
    "points_module": {
        "order_amount_min": 100.00,
        "total_points": 9999,
        "can_use_points": 9999,
        "points2money_rate": 100,
        "points2money_min": 1000,
        "points2money_max": 9999,
        "points_amount": 99.99,
        "points_amount_txt": "99.99"
    }
}
```

## Módulo de Informações de Valor do Pedido

> Inclui detalhes do valor deste pedido

Nome do Campo|Tipo|Nome do Campo Inferior|Tipo|Significado do Campo
------|------|------|------|------
skus_amount|float64|-|-|Valor total dos produtos
promotion_amount|float64|-|-|Valor total de descontos
freight|float64|-|-|Frete
final_amount|float64|-|-|Valor a pagar
promotion_detail|object|coupon_amount|float64|Valor de desconto do cupom
promotion_detail|object|sales_activity_amount|float64|Valor de desconto da atividade de vendas
promotion_detail|object|giftcard_amount|float64|Valor usado do cartão presente
promotion_detail|object|points_amount|float64|Valor abatido por pontos neste pedido

```
Campo _txt omitido
```

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201015200913.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20201015200913.png" width="100%">
    </a>
</p>

Demo de dados do módulo:
```json
{
    "order_amount_module": {
        "skus_amount": 99.99,
        "skus_amount_txt": "99.99",
        "promotion_amount_total": 10.00,
        "promotion_amount_total_txt": "10.00",
        "freight_total": 8.00,
        "freight_total_txt": "8.00",
        "final_amount": 97.99,
        "final_amount_txt": "97.99",
        "promotion_detail": {
            "coupon_amount": 5.00,
            "coupon_amount_txt": "5.00",
            "sales_activity_amount": 5.00,
            "sales_activity_amount_txt": "5.00",
            "giftcard_amount": 0,
            "giftcard_activity_amount_txt": "0",
            "points_amount": 0,
            "points_amount_txt": "0"
        }
    }
}
```

## Conclusão

Acima, a introdução básica do conteúdo da página de checkout do pedido foi concluída. Se você tiver alguma dúvida, sinta-se à vontade para deixar uma mensagem em nosso projeto no github <https://github.com/skr-shop/manuals/issues>.
