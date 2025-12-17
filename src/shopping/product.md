# Sistema de Produtos

## Sistema de Produtos (Temporal/Tudo)

Hoje começamos o capítulo "Sistema de Produtos". Este artigo está dividido nos seguintes cinco módulos principais:

- Análise de Requisitos
- Design de Arquitetura
- A História de Spu e Sku
- Design do Modelo de Dados
- Design de Interface

No primeiro artigo, vamos ver principalmente como uma plataforma de e-commerce introdutória (**B2C**) constrói suas **informações básicas de produtos**. Na verdade, isso é muito simples. Pense em nossa vida real: o comerciante **coloca** os produtos na prateleira, o cliente **escolhe** os produtos da prateleira, o cliente **coloca** os produtos escolhidos no carrinho (cesta) de compras e, finalmente, o cliente vai ao caixa para **pagar**.

### Análise de Requisitos

Para uma plataforma de e-commerce, como entendemos o exemplo simples acima? Em seguida, vamos dividir essa coisa simples acima:

> O comerciante **coloca** os produtos na prateleira, o cliente **escolhe** os produtos da prateleira, o cliente **coloca** os produtos escolhidos no carrinho (cesta) de compras e, finalmente, o cliente vai ao caixa para **pagar**.

1. Quem é o comerciante: Plataforma de E-commerce
2. O que significa colocar: Listar (Up)
3. Onde está a prateleira: Sistema Frontend (web/app/...)
4. Escolher: Navegar no Sistema Frontend
5. Colocar: Clicar no botão "Adicionar ao Carrinho" no Sistema Frontend
6. ... (não vamos falar muito por enquanto)

**Nota**: Este artigo analisará principalmente como projetar os passos 1, 2, 3 e 4.

Através da análise acima, podemos obter as seguintes informações:

1. Precisamos de uma "Plataforma de E-commerce", e dentro da plataforma de e-commerce precisamos de um **Sistema de Backend de Produtos**.
2. O que listamos? Produtos! Portanto, o **Sistema de Backend de Produtos** precisa ter as funções de **Criar** e **Publicar** produtos para o **Sistema Frontend**.
3. Precisamos de um **Sistema Frontend** (como uma página da web). O sistema frontend possui páginas de lista de produtos e detalhes de produtos para os usuários **navegarem**.
4. De onde vêm os dados do sistema frontend? Então precisamos de um **Gateway de Interface** (fornecendo capacidades de serviço externamente unificadas, barramento corporativo) e **Serviço de Produto**.

Após a organização, obtemos os seguintes pontos de requisitos:

Ponto de Requisito|Ponto de Função|Nome do Projeto|Stack Tecnológico
-----|-----|-----|-----
Sistema de Backend de Produtos|1. Criar produto 2. Publicar produto no sistema frontend|Temporal Backend|PHP
Sistema Frontend|1. Lista de produtos 2. Detalhes do produto| Skr Frontend|Vue
Gateway de Interface|Barramento Corporativo| Skr Gateway|kong
Serviço de Produto|1. Interface de criação de produto 2. Interface de alteração de status do produto 2. Interface de lista de produtos 3. Interface de detalhes do produto|Temporal Service|Golang

### Design de Arquitetura

Através da análise de requisitos acima, mais o sistema de usuários no "Manual de Design de E-commerce - Sistema de Usuários" anterior e o serviço de pagamento em "Desenvolvimento de Pagamentos, Processos de Pagamento de Terceiros Nacionais e Internacionais que Você Precisa Conhecer", planejamos o seguinte diagrama de arquitetura.

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-product-service.jpg" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-product-service.jpg">
    </a>
</p>

### A História de Spu e Sku

Para nós programadores, a aparência inicial do "Sistema de Produtos" é a seguinte:

1. Função de criação de produto: Primeiro teremos uma tabela de produtos. Cada vez que criamos um produto, obtemos um goods_id. Se houver uma relação pai-filho entre produtos, adicionar um campo parent_id resolve.
2. Interface de lista de produtos: Consulta paginada da tabela de produtos.
3. Interface de detalhes do produto: Consulta de informações do produto indexada por goods_id na tabela de produtos.

Muito simples, certo? Basicamente uma tabela resolve, parece não haver problema. Mas, a engenhosidade do design de programas está na **capacidade de abstração**. A indústria de e-commerce abstraiu ainda mais o `goods_id`, gerando os conceitos de Spu e Sku. Antes de entender as definições de Spu e Sku, precisamos entender o significado de **Atributo de Venda**. Vamos dar um exemplo para facilitar o entendimento:

Pense em nossa vida real. Suponha que vamos ao mercado de atacado comprar um lote de tênis AJ1. O atacadista nos dará tênis AJ1 de diferentes **cores** e **tamanhos**. Quando vendemos esses produtos na loja, perguntamos aos clientes: "Que **cor** e **tamanho** de tênis AJ1 você precisa?". A **cor** e o **tamanho** aqui são os chamados **Atributos de Venda**, porque tênis AJ1 de diferentes **cores** e **tamanhos** podem ter preços diferentes e quantidades de estoque diferentes. Na vida real não é assim? AJ1 de cores ou tamanhos diferentes têm preços muito diferentes.

Em seguida, vamos ver as definições de Spu e Sku:

Nome|Conceito|Explicação
---|---|---
Spu|standard product unit (unidade de produto padrão)|A parte do goods_id sem atributos de venda, por exemplo: Xiaomi 8. Na lista de produtos, exibimos a lista de Spu.
Sku|stock keeping unit (unidade de manutenção de estoque)|É o número real do produto que você deseja comprar. O estoque correspondente a este número é a quantidade de estoque do produto que você deseja comprar. Spu + um ou mais atributos de venda correspondem a um Sku, por exemplo: Xiaomi 8 Preto 128G, onde Preto e 128G são atributos de venda, e Xiaomi 8 é um Spu.

Entendeu?

### Design do Modelo de Dados

Então, no final, a **tabela de produtos** simples foi dividida em **tabela spu** e **tabela sku**. Em seguida, também abstraímos a **tabela de atributos de venda** e a **tabela de valores de atributos de venda** reutilizáveis. Além disso, devemos ter também **tabela de marcas**, **tabela de categorias** e **tabela de estoque sku** simples (atualmente projetamos esta tabela de forma simples, e reconstruiremos esta tabela com negócios específicos posteriormente). Em seguida, listamos os detalhes dessas tabelas:

Nome da Tabela|Nome da Tabela
---|---
Tabela de Marcas|product_brands
Tabela de Categorias|product_category
Tabela Spu|product_spu
Tabela Sku|product_sku
Tabela de Atributos de Venda|product_attr
Tabela de Valores de Atributos de Venda|product_attr_value
Tabela de Estoque Sku|product_sku_stock

Além das tabelas acima, adicionei outra tabela `Tabela de redundância de relacionamento product_spu_sku_attr_map`. Por quê? Como o nome sugere, é para redundância. Com essa tabela, podemos obter de forma muito eficiente:

1. Quais skus existem sob o spu
2. Quais atributos de venda existem sob o spu
3. Valores de atributos de venda correspondentes a cada atributo de venda sob o spu (um para muitos)
4. Sku correspondente a cada valor de atributo de venda sob o spu (um para muitos)

A estrutura específica da tabela é mostrada abaixo:

```sql

-- Tabela de Marcas product_brands
CREATE TABLE `product_brands` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID da Marca',
    `name` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Nome da marca',
    `desc` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Descrição da marca',
    `logo_url` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Logo da marca',
    `create_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de criação',
    `create_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de atualização',
    `update_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT 'Status 1:ativado, 0:desativado, -1:deletado',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Tabela de Marcas';

-- Tabela de Categorias product_category
CREATE TABLE `product_category` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID da Categoria',
    `pid` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID Pai',
    `name` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Nome da categoria',
    `desc` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Descrição da categoria',
    `pic_url` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Imagem da categoria',
    `path` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Endereço da categoria {pid}-{child_id}-...',
    `create_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de criação',
    `create_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de atualização',
    `update_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT 'Status 1:ativado, 0:desativado, -1:deletado',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Tabela de Categorias';

-- Tabela Spu product_spu
-- spu: standard product unit (unidade de produto padrão)
CREATE TABLE `product_spu` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'SPU ID',
    `brand_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID da Marca',
    `category_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID da Categoria',
    `name` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Nome do spu',
    `desc` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Descrição do spu',
    `selling_point` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Ponto de venda',
    `unit` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Unidade do spu',
    `banner_url` text COMMENT 'Imagem do banner, múltiplas imagens separadas por vírgula',
    `main_url` text COMMENT 'Imagem principal de introdução do produto, múltiplas imagens separadas por vírgula',
    `price_fee` int unsigned NOT NULL DEFAULT 0 COMMENT 'Preço de venda, salvo como inteiro',
    `price_scale` tinyint unsigned NOT NULL DEFAULT 0 COMMENT 'Preço de venda, casas decimais correspondentes ao valor',
    `market_price_fee` int unsigned NOT NULL DEFAULT 0 COMMENT 'Preço de mercado, salvo como inteiro',
    `market_price_scale` tinyint unsigned NOT NULL DEFAULT 0 COMMENT 'Preço de mercado, casas decimais correspondentes ao valor',
    `create_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de criação',
    `create_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de atualização',
    `update_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT 'Status 1:ativado, 0:desativado, -1:deletado',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT AUTO_INCREMENT=666666 CHARSET=utf8mb4 COMMENT='Tabela Spu';

-- Tabela Sku product_sku
-- sku: stock keeping unit (unidade de manutenção de estoque)
CREATE TABLE `product_sku` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'SKU ID',
    `spu_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'SPU ID',
    `attrs` text COMMENT 'Valor do atributo de venda {attr_value_id}-{attr_value_id}, múltiplos valores separados por vírgula',
    `banner_url` text COMMENT 'Imagem do banner, múltiplas imagens separadas por vírgula',
    `main_url` text COMMENT 'Imagem principal de introdução do produto, múltiplas imagens separadas por vírgula',
    `price_fee` int unsigned NOT NULL DEFAULT 0 COMMENT 'Preço de venda, salvo como inteiro',
    `price_scale` tinyint unsigned NOT NULL DEFAULT 0 COMMENT 'Preço de venda, casas decimais correspondentes ao valor',
    `market_price_fee` int unsigned NOT NULL DEFAULT 0 COMMENT 'Preço de mercado, salvo como inteiro',
    `market_price_scale` tinyint unsigned NOT NULL DEFAULT 0 COMMENT 'Preço de mercado, casas decimais correspondentes ao valor',
    `create_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de criação',
    `create_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de atualização',
    `update_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT 'Status 1:ativado, 0:desativado, -1:deletado',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT AUTO_INCREMENT=666666 CHARSET=utf8mb4 COMMENT='Tabela Sku';

-- Tabela de Atributos de Venda product_attr
CREATE TABLE `product_attr` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID do Atributo de Venda',
    `name` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Nome do atributo de venda',
    `desc` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Descrição do atributo de venda',
    `create_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de criação',
    `create_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de atualização',
    `update_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT 'Status 1:ativado, 0:desativado, -1:deletado',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Tabela de Atributos de Venda';

-- Valores de Atributos de Venda product_attr_value
CREATE TABLE `product_attr_value` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID do Valor do Atributo de Venda',
    `attr_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID do Atributo de Venda',
    `value` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Valor do atributo de venda',
    `desc` varchar(255)  NOT NULL DEFAULT '' COMMENT 'Descrição do valor do atributo de venda',
    `create_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de criação',
    `create_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de atualização',
    `update_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT 'Status 1:ativado, 0:desativado, -1:deletado',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Valores de Atributos de Venda';

-- Tabela de redundância de relacionamento product_spu_sku_attr_map
-- 1. Quais skus existem sob o spu
-- 2. Quais atributos de venda existem sob o spu
-- 3. Valores de atributos de venda correspondentes a cada atributo de venda sob o spu (um para muitos)
-- 4. Sku correspondente a cada valor de atributo de venda sob o spu (um para muitos)
CREATE TABLE `product_spu_sku_attr_map` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID Auto-incremental',
    `spu_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'SPU ID',
    `sku_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'SKU ID',
    `attr_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID do Atributo de Venda',
    `attr_name` varchar(255) NOT NULL DEFAULT '0' COMMENT 'Nome do atributo de venda',
    `attr_value_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID do Valor do Atributo de Venda',
    `attr_value_name` varchar(255) NOT NULL DEFAULT '0' COMMENT 'Valor do atributo de venda',
    `create_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de criação',
    `create_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `status` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT 'Status 1:ativado, 0:desativado, -1:deletado',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Tabela de redundância de relacionamento';

-- Tabela de Estoque Sku product_sku_stock
CREATE TABLE `product_sku_stock` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID Auto-incremental',
    `sku_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'SKU ID',
    `quantity` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Estoque',
    `create_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de criação',
    `create_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'Tempo de atualização',
    `update_by` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT 'Status 1:ativado, 0:desativado, -1:deletado',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Tabela de Estoque Sku';

```

### Design de Interface

Sobre o design da interface, atualmente é muito simples, nada mais que listas e detalhes. Mas aqui fiz um design muito bom de **Separação Dinâmica e Estática**. Por exemplo, os dados dinâmicos de estoque fornecem interfaces separadas, e outros dados de lista e detalhes são completamente estáticos, enviando o tráfego para a CDN. Aqui falaremos sobre o "Serviço de Recursos Estáticos" em nosso sistema de serviços básicos planejado para a próxima etapa. A principal função deste serviço é tornar os dados de nossa interface estáticos. O design da interface da versão **V1.0** específica é o seguinte:

1. Detalhes do Spu GET {version}/product/spu/{spu_id}

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
spu_id|number|yes|spu ID

Conteúdo da resposta:
```json
{
    "code": "200",
    "msg": "OK",
    "result": {
        "brand_info": {
            "id": "number, ID da Marca",
            "name": "string, Nome da marca",
            "desc": "string, Descrição da marca",
            "logo_url": "string, Logo da marca",
        },
        "category_info": {
            "id": "number, ID da Categoria",
            "name": "string, Nome da marca",
            "desc": "string, Descrição da marca",
            "pic_url": "string, Imagem da categoria",
            "path": "string, Endereço da categoria {pid}-{child_id}-...",
        },
        "spu_info": {
            "id": "number, spu id",
            "name": "string, nome do spu",
            "desc": "string, descrição do spu",
            "selling_point": "string, ponto de venda",
            "unit": "string, unidade do spu",
            "banner_url": [
                "string, url da imagem do banner",
                "string, url da imagem do banner",
            ],
            "main_url": [
                "string, url da imagem principal de introdução do produto",
                "string, url da imagem principal de introdução do produto",
            ],
            "price": "string, preço de venda",
            "market_price": "string, preço de mercado",
            "attrs": [ // Quais atributos de venda existem
                {
                    "id": "ID do Atributo de Venda",
                    "name": "string, Nome do atributo de venda",
                    "desc": "string, Descrição do atributo de venda",
                    "values": [ // Valores de atributos de venda correspondentes a cada atributo de venda (um para muitos)
                        {
                            "id": "ID do Valor do Atributo de Venda",
                            "name": "string, Valor do atributo de venda",
                            "desc": "string, Descrição do valor do atributo de venda",
                            // Sku correspondente a cada valor de atributo de venda (um para muitos)
                            // Ao inicializar a página, julgamento lógico de botão não clicável: Se todos os skus sob este valor de atributo de venda não tiverem estoque, o botão deste atributo de venda não é clicável
                            // Ao selecionar um valor de atributo de venda, julgamento lógico de botão não clicável: Atributos de venda formam uma lista duplamente encadeada, e cada atributo de venda é uma lista encadeada simples armazenando todos os valores de atributo de venda correspondentes. Sempre que um valor de atributo de venda é selecionado, percorre os atributos de venda anteriores e posteriores, botões de todos os skus esgotados sob o valor de atributo de venda não são clicáveis, e o mapa do valor de atributo de venda atual registra a chave como o ID do valor de atributo de venda clicado atualmente, o valor apenas identifica unificadamente, o objetivo é registrar qual valor de atributo de venda selecionado causou o status de esgotado do valor de atributo de venda atual
                            // Ao cancelar a seleção do valor do atributo de venda, julgamento de recuperação lógica de botão não clicável: Estrutura de dados igual acima, percorre, o mapa registrado exclui a chave do valor de atributo de venda cancelado atualmente e julga se há outras chaves que tornam o valor de atributo de venda esgotado, se não houver, recupera o status não esgotado
                            "skus": [
                                "number, sku id",
                                "number, sku id",
                            ],
                        }
                    ],
                }
            ],
            "skus": [ // Quais skus existem
                "number, sku id",
                "number, sku id",
            ],
            "skus_map": {
                "{attr_value_id}-{attr_value_id}-...": "number, sku id",
                "{attr_value_id}-{attr_value_id}-...": "number, sku id",
                "{attr_value_id}-{attr_value_id}-...": "number, sku id",
                "{attr_value_id}-{attr_value_id}-...": "number, sku id",
                "{attr_value_id}-{attr_value_id}-...": "number, sku id",
                "{attr_value_id}-{attr_value_id}-...": "number, sku id",
            }
        }
    }
}
```

2. Obter estoque de todos os skus sob spu GET {version}/stock/spu/{spu_id}

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
spu_id|number|yes|spu ID

Conteúdo da resposta:
```json
{
    "code": "200",
    "msg": "OK",
    "result": {
            "skus_stock": {
                "int, sku id": {
                    "quantity": "int, quantidade de estoque restante"
                }
            }
        }
    }
}
```

3. Detalhes do sku GET {version}/product/sku/{sku_id}

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
sku|number|yes|sku ID

Conteúdo da resposta:
```json
{
    "code": "200",
    "msg": "OK",
    "result": {
        "id": "number, sku id",
        "name": "string, nome do sku",
        "desc": "string, descrição do sku",
        "unit": "string, unidade do sku",
        "banner_url": [
            "string, url da imagem do banner",
            "string, url da imagem do banner",
        ],
        "main_url": [
            "string, url da imagem principal de introdução do produto",
            "string, url da imagem principal de introdução do produto",
        ],
        "price": "string, preço de venda",
        "market_price": "string, preço de mercado",
    }
}
```

4. Lista de spu GET {version}/product/spu/list

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
-|-|-|-

Conteúdo da resposta:
```json
{
    "code": "200",
    "msg": "OK",
    "result": {
        "list": [
            {
                "id": "number, spu id",
                "name": "string, nome do spu",
                "desc": "string, descrição do spu",
                "unit": "string, unidade do spu",
                "banner_url": [
                    "string, url da imagem do banner",
                    "string, url da imagem do banner",
                ],
                "price": "string, preço de venda",
                "market_price": "string, preço de mercado",
            }
        ]
    }
}
```