<h1 align="center">《Manual de Design de E-commerce | SkrShop》</h1>

<p align="center">Do design No code | Apenas Design, Sem Codar</p>

<p align="center">
    <img src="https://img.shields.io/badge/License-CC%20BY--NC--ND%204.0-red" alt="Lisense">
</p>

<p align="center"><a href="http://skrshop.tigerb.cn/">skrshop.tigerb.cn</a></p>

<p align="center">
    <img style="vertical-align:middle" width="60%" src="https://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/20191222164829.jpg?imageMogr2/thumbnail/1280x720!/format/webp/blur/1x0/quality/90|imageslim">
<p>

# Declaração de Direitos Autorais
- Sem a autorização expressa do detentor dos direitos autorais, é proibida a distribuição deste manual e de suas versões substancialmente modificadas (incluindo artigos, vídeos, podcasts e qualquer outra mídia).
- Sem a autorização prévia do detentor dos direitos autorais, é proibido distribuir esta obra e suas obras derivadas em formato de livro padrão (impresso).
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
- [Seleção de Tech Stack](http://skrshop.tech/#/?id=seleção-de-tech-stack)
- [Repositório de Código](http://skrshop.tech/#/?id=repositório-de-código)
- [Sistema de Usuários](/src/account/?id=sistema-de-usuários)
    + [Serviço de Conta](/src/account/?id=arquitetura)
    + [Serviço de Permissões](/src/account/?id=gestão-de-permissões-de-backend)
- [Sistema de Comércio](/src/shopping/cart?id=交易体系)
    + [Serviço de Carrinho de Compras](/src/shopping/cart?id=购物车服务)
    + [Arquitetura do Carrinho](/src/shopping/cart?id=购物车架构)
    + [Módulo de Pedidos](/src/order/)
        * [Página de Checkout de Pedido](/src/order/checkout)
- [Sistema de Marketing](/src/promotion/?id=营销体系)
    + Sistema de Atividades
        * [Ferramenta de Sorteio Universal (Cola Universal Glue)](/src/promotion/glue)
    + Sistema de Marketing
        * [Serviço de Venda Relâmpago (Seckill)](/src/promotion/seckill?id=秒杀服务)
        * [Serviço de Cupons](/src/promotion/coupon?id=优惠券服务)
        * Serviço de Pontos
- [Serviços Básicos](/src/base/)
    + [Sistema de Produtos (Temporal Tudo)](/src/shopping/product?id=商品系统)
    + [Sistema de Pagamento](/src/trade/)
        * [Fluxo Comum de Pagamento de Terceiros](/src/trade/?id=常见第三方支付流程)
        * [Design do Sistema de Pagamento](/src/trade/?id=支付系统设计)
        * Checkout
    + Serviço de Busca
        * [Introdução ao Negócio de Busca em E-commerce](/src/base/search/business)
        * [Do Raso ao Fundo, Introdução aos Princípios de Busca](/src/base/search/tech)
    + Serviço de Interface Estática
    + Serviço de Upload
    + Serviço de Mensagens
        * SMS
        * E-mail
        * Mensagens de Modelo WeChat
        * Mensagens Internas do Site
- [Sistema de Armazenamento](http://skrshop.tech/#/src/warehouse/)
    + Serviço de Endereço
- [Sistema de Logística](http://skrshop.tech/#/src/express/)
- [Serviço Pós-Venda](http://skrshop.tech/#/src/aftersale/)

# Prefácio

Trabalho com desenvolvimento de e-commerce na internet há mais de três anos e, olhando para trás, percebo que não entendo muito bem todo o fluxo de negócios, o que é vergonhoso de admitir. Mas estando no ambiente de e-commerce, tive contato em maior ou menor grau com vários desses negócios e, além disso, tenho muitos colegas e amigos que trabalham com e-commerce, o que é um grande recurso. Então, decidi organizar todo o fluxo e compartilhá-lo com meus colegas, amigos e até mesmo com vocês. Não prometo que o resultado será perfeito, mas certamente seremos meticulosos em cada parte que tivermos capacidade de fazer bem.

Além disso, para satisfazer nossa própria realização técnica que talvez não consigamos no trabalho, usaremos a tech stack que mais queremos usar durante o processo de design de todo o sistema. Para a tech stack, usaremos o docker para implementar, então o resultado final: por um lado dominamos o negócio, e por outro lado obtemos satisfação técnica, matando dois coelhos com uma cajadada só.

Por fim, considerando o tempo, propusemos uma ideia **Do design No code**. **【Apenas Design, Sem Codar】** O significado desta frase: no final, projetamos o modelo de dados de todo o sistema, documentação da interface, e até processos de interação, implantação de ambiente, etc., mas no final não escrevemos código. É isso, né? Se for assim, qual o sentido de escrever código? Claro, não é totalmente assim, considerando o tempo, é claro que também será implementado em código, talvez seja você aí do outro lado que vai implementar.

Em segundo lugar, este conteúdo certamente tem considerações incompletas ou lugares mais complexos em negócios de grande escala, correções são bem-vindas, e também esperamos aprender e compartilhar sua experiência.

# Seleção de Tech Stack

```
- Ambiente Básico
    + k8s
    + docker
- Armazenamento
    + mysql
    + redis
        * codis
        * redis master-slave
- queue
    + kafka
    + rocketmq
    + rabbitmq
- gw
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

Sem ordem específica, ordem alfabética

Apelido|Introdução|Blog Pessoal
--------|--------|--------
AStraw|Empreendedor de pós-graduação|Conta Oficial "Rice Straw Life"
大愚Dayu|Autor do projeto de agregação de pagamentos de terceiros em PHP usado pela maioria das pessoas no país [Payment](https://github.com/helei112g/payment), já empreendeu|[Dayu Talk](http://dayutalk.cn/)
lwhcv|Trabalhou no Baidu/Rong360|--------
TIGERB|Autor do framework PHP [EasyPHP](http://easy-php.tigerb.cn/#/)| [Blog Técnico do TIGERB](http://tigerb.cn)
Veaer|Engenheiro Full-stack Invencível do Universo| [Veaer](http://veaer.com)
