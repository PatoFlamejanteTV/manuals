<h1 align="center">《Manual de Design de E-commerce | SkrShop》</h1>

<p align="center">Do design No code | Apenas design, sem código</p>

<p align="center">
    <img src="https://img.shields.io/badge/License-CC%20BY--NC--ND%204.0-red" alt="Lisense">
</p>

<p align="center"><a href="http://skrshop.tigerb.cn/">skrshop.tigerb.cn</a></p>

<p align="center">
    <img style="vertical-align:middle" width="60%" src="https://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20191222164829.jpg?imageMogr2/thumbnail/1280x720!/format/webp/blur/1x0/quality/90|imageslim">
<p>

# Declaração de Direitos Autorais
- Sem a autorização expressa do detentor dos direitos autorais, é proibida a distribuição deste manual e suas versões substancialmente modificadas (incluindo artigos, vídeos, podcasts e qualquer outra mídia).
- Sem a autorização prévia do detentor dos direitos autorais, é proibida a distribuição desta obra e de suas obras derivadas na forma de livro padrão (papel).
- Não há cooperação com terceiros de nenhuma forma.

<p align="center">
    <img style="vertical-align:middle" width="25%" src="https://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/wechat-blog-qrcode.jpg?imageMogr2/thumbnail/260x260!/format/webp/blur/1x0/quality/90|imageslim">
    <img style="vertical-align:middle" width="45%" src="https://dayutalk.cn/img/pub-qr.jpeg">
    <i style="display:inline-block; height:100%; vertical-align:middle; width:0;"></i>
<p>


# Tendência de Stars

[![Stargazers over time](https://starchart.cc/skr-shop/manuals.svg?background=%23000000&axis=%23ffffff&line=%232ff76e)](https://starchart.cc/skr-shop/manuals)

# Arquitetura

<p align="center">
    <a href="https://blog-1251019962.cos.ap-beijing.myqcloud.com/skrshop%2Fskrshop.png">
        <img src="https://blog-1251019962.cos.ap-beijing.myqcloud.com/skrshop%2Fskrshop.png" width="100%">
    </a>
</p>

# Índice

- [Prefácio](http://skrshop.tech/#/)
- [Índice](http://skrshop.tech/#/guide)
- [Seleção de Stack Tecnológico](http://skrshop.tech/#/?id=seleção-de-stack-tecnológico)
- [Repositório de Código](http://skrshop.tech/#/?id=repositório-de-código)
- [Sistema de Usuários](/src/account/?id=sistema-de-usuários)
    + [Serviço de Contas](/src/account/?id=design-de-arquitetura)
    + [Serviço de Permissões](/src/account/?id=gestão-de-permissões-de-backend)
- [Sistema de Compras](/src/shopping/cart?id=sistema-de-compras)
    + [Serviço de Carrinho de Compras](/src/shopping/cart?id=serviço-de-carrinho-de-compras)
    + [Arquitetura do Carrinho de Compras](/src/shopping/cart?id=arquitetura-do-carrinho-de-compras)
    + [Módulo de Pedidos](/src/order/)
        * [Página de Checkout de Pedidos](/src/order/checkout)
- [Sistema de Marketing](/src/promotion/?id=sistema-de-marketing)
    + Sistema de Atividades
        * [Ferramenta de Sorteio Universal (Glue/Cola Tudo)](/src/promotion/glue)
    + Sistema de Vendas
        * [Serviço de Seckill (Oferta Relâmpago)](/src/promotion/seckill?id=serviço-de-seckill-oferta-relâmpago)
        * [Serviço de Cupons](/src/promotion/coupon?id=serviço-de-cupons)
        * Serviço de Pontos
- [Serviços Básicos](/src/base/)
    + [Sistema de Produtos (Temporal/Tudo)](/src/shopping/product?id=sistema-de-produtos)
    + [Sistema de Pagamentos](/src/trade/)
        * [Fluxo Comum de Pagamento de Terceiros](/src/trade/?id=fluxo-comum-de-pagamento-de-terceiros)
        * [Design do Sistema de Pagamento](/src/trade/?id=design-do-sistema-de-pagamento)
        * Caixa (Cashier)
    + Serviço de Busca
        * [Introdução ao Negócio de Busca em E-commerce](/src/base/search/business)
        * [Do Raso ao Fundo, Introdução aos Princípios de Busca](/src/base/search/tech)
    + Serviço de Estática de Interface
    + Serviço de Upload
    + Serviço de Mensagens
        * SMS
        * E-mail
        * Mensagens de Modelo WeChat
        * Mensagens Internas do Site
- [Sistema de Armazenamento](http://skrshop.tech/#/src/warehouse/)
    + Serviço de Endereços
- [Sistema de Logística](http://skrshop.tech/#/src/express/)
- [Serviço de Pós-venda](http://skrshop.tech/#/src/aftersale/)

# Prefácio

Trabalho com desenvolvimento de e-commerce há mais de três anos. Olhando para trás, percebo que não entendo muito bem todo o fluxo de negócios, o que é vergonhoso de admitir. Mas, estando no ambiente de e-commerce, tive contato com vários negócios em maior ou menor grau. Além disso, tenho muitos colegas e amigos que trabalham com e-commerce, e isso são recursos valiosos. Então, decidi organizar e compartilhar todo o processo com meus colegas, amigos e até com vocês. Não prometo que o resultado será perfeito, mas garanto que seremos meticulosos em cada parte que tivermos capacidade de fazer bem.

Além disso, para satisfazer nossa realização técnica que talvez não consigamos no trabalho, usaremos a stack tecnológica que mais desejamos durante o design de todo o sistema. Realizaremos essa stack tecnológica com a ajuda do Docker. O resultado final: por um lado, dominamos o negócio e, por outro, obtemos satisfação técnica. O melhor dos dois mundos.

Por fim, considerando o tempo, propusemos uma ideia: **Do design No code**. **【Apenas design, sem código】** O que isso significa: no final, projetaremos o modelo de dados de todo o sistema, a documentação da API, os processos de interação e até a implantação do ambiente, mas não escreveremos o código. Certo? Se fizéssemos isso, qual seria o sentido de escrever código? Claro, não é exatamente assim. Considerando o tempo, é claro que também implementaremos em código. Quem sabe, talvez seja você aí do outro lado quem vai implementar.

Além disso, este conteúdo certamente pode não ser abrangente ou pode haver complexidades maiores em negócios de grande escala. Sinta-se à vontade para apontar falhas; também queremos aprender e compartilhar suas experiências.

# Seleção de Stack Tecnológico

```
- Ambiente Básico
    + k8s
    + docker
- Armazenamento
    + mysql
    + redis
        * codis
        * redis master-slave
- queue (filas)
    + kafka
    + rocketmq
    + rabbitmq
- gw (gateway)
    + kong
    + zuul
- webserver
    + nginx/openresty
    + envoy
- server
    + go
    + php
- frontend
    + vue
- rpc
    + grpc
    + thrift
- Capacidades Básicas
    + Monitoramento
        * zipkin
        * elk
        * falcon
    + Descoberta de Serviço
        * zookeeper
        * etcd
    + Integração Contínua
        * ci/cd
- Busca
    + es
    + solr
```

# Repositório de Código

Por favor, aguarde pacientemente...

# Introdução aos Membros do Projeto Skr Shop

A ordem não implica classificação, ordem de dicionário.

Apelido|Introdução|Blog Pessoal
--------|--------|--------
AStraw|Empreendedor e pós-graduando|Conta pública "Vida de Palha"
Dayu|Autor do projeto de agregação de pagamentos de terceiros em PHP [Payment](https://github.com/helei112g/payment) mais usado na China, empreendedor|[Dayu Talk](http://dayutalk.cn/)
lwhcv|Trabalhou no Baidu/Rong360|--------
TIGERB|Autor do framework PHP [EasyPHP](http://easy-php.tigerb.cn/#/)| [Blog Técnico do TIGERB](http://tigerb.cn)
Veaer|Engenheiro Full-stack Invencível do Universo| [Veaer](http://veaer.com)
