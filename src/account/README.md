# Sistema de Usuários

Hoje, começamos o design da primeira parte: **Sistema de Usuários**. Este artigo é dividido nos seguintes quatro módulos:

- Design de Arquitetura
- Design do Modelo de Dados
- Design de Interação
- Design de Interface

## Design de Arquitetura

### Uma visão simples do sistema de usuários

Quando você entra em contato pela primeira vez com produtos de internet relacionados a usuários, ou como eu via antigamente, o **Sistema de Usuários** nada mais é do que "Login", "Cadastro", "Modificar informações do usuário", etc. Simplificando, precisamos apenas de uma tabela para registrar as informações de identidade do usuário: ao registrar (operação insert), inserimos um dado na tabela; ao logar (operação select & update), verificamos se a senha do usuário está correta através do identificador do usuário (número de celular, e-mail, etc.); ao modificar as informações do usuário (operação select & update), atualizamos diretamente as informações desse uid (avatar, apelido, etc.).

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-smaple-structure.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-smaple-structure.png">
    </a>
</p>

Não há nada de errado com esse design, é bem simples, não é? Mas com o desenvolvimento do negócio, por um lado precisamos fornecer um gerenciamento unificado de usuários (alta coesão), e por outro lado queremos melhorar a escalabilidade do sistema. Então, o que quero apresentar é o que eu entendo que **um sistema básico de usuários deve ter**.

### O que um sistema básico de usuários deve ter

Primeiro, abstraímos a tabela de usuários original mais uma vez (separando campos dependentes de registro e login, login de terceiros) -> **Tabela de Contas**. Por que fazer isso? Com o desenvolvimento do negócio, antes mantínhamos apenas um produto, talvez um dia desenvolvamos novos produtos. Assim, podemos manter a lógica de registro e login de todos os produtos da nossa empresa de forma unificada, e produtos diferentes mantêm apenas as informações relacionadas ao usuário desse produto (dependendo da forma do produto). Como mostrado na figura abaixo:

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-user-system-2.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-user-system-2.png">
    </a>
</p>

Na figura acima, também são mencionados Login de Terceiros/Tabela de Funcionários/Gestão de Permissões de Backend, que são estruturas básicas essenciais para um sistema de usuários.

Login de Terceiros: O login de terceiros também é um tipo de método de login, e também o abstraímos como parte da conta, como mostrado na figura acima. Além disso, existe um problema de design de interação sobre o login de terceiros aqui, que será mencionado no design de interação posteriormente.

Funcionários: Como separamos a tabela de contas acima, o backend do sistema de gerenciamento interno também pode usar a lógica de login da tabela de contas de forma unificada, alcançando assim uma verdadeira alta coesão na questão de contas em toda a empresa.

Falando em funcionários, nossos vários backends de sistemas internos certamente envolvem vários gerenciamentos de permissões, então aqui mencionamos um RBAC simples (Controle de Acesso Baseado em Papel). O design lógico detalhado do modelo de dados será mencionado.

### A Arquitetura Final

À medida que a forma dos produtos de negócios se torna cada vez mais complexa, ao projetar a arquitetura, precisamos analisar o que **muda e o que não muda**:

- Muda: Cada vez mais demandas personalizadas de usuários dos produtos
- Não muda: A lógica de registro e login

O resultado final é que dividimos o usuário original em **Conta** e **Usuário**, e também precisamos esclarecer a diferença entre esses dois conceitos aqui:

- Conta: O único lugar em todo o sistema que produz o uid, coeso com a lógica de registro e login, não envolvendo requisitos de negócios do produto
- Usuário: Informações de demanda do usuário personalizadas para diferentes produtos

O diagrama de arquitetura final é o seguinte:

- Primeira parte: Conta (**Camada de Serviço**)
- Segunda parte: Usuário (**Camada de Aplicação**, expansão horizontal infinita)
- Terceira parte: Funcionário (**Camada de Aplicação**, sistema de permissões de funcionários)

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-structure.jpg" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-structure.jpg">
    </a>
</p>


## Design do Modelo de Dados

Correspondendo à arquitetura acima, podemos facilmente projetar nosso modelo de dados (assumindo que temos apenas uma aplicação para o consumidor final (C-end) no momento):

```
Conta -> 1. Tabela de Contas
Usuário -> 2. Tabela de Usuários
Funcionário -> 3. Tabela de Funcionários
```

Além das três tabelas acima, também precisamos do nosso gerenciamento de permissões R(role)B(base)A(access)C(control). Todos devem estar familiarizados com o gerenciamento de permissões baseado em papéis RBAC, então não vou entrar em detalhes aqui. Um RBAC simples precisa primeiro de:

```
4. Tabela de Menus do Sistema (menu é permissão), o caminho uri do sistema
5. Tabela de Permissões (menu é permissão), a permissão específica é acessar o menu do sistema
6. Tabela de Papéis, quais permissões um papel possui
7. Tabela de Associação de Funcionários e Papéis, a qual papel um funcionário pertence
```

Ok, as tabelas envolvidas em um RBAC simples foram basicamente listadas. Mas na minha experiência de trabalho, o gerenciamento de permissões que todos implementam geralmente visa apenas um determinado sistema. Para muitos backends de sistemas, isso é confuso, reinvenção da roda e baixa eficiência no gerenciamento de permissões. Portanto, no design de arquitetura acima, coloquei as permissões como um serviço para fornecer capacidades de serviço básico para todo o sistema. E para alcançar esse resultado, só preciso adicionar mais uma tabela:

```
8. Tabela de Sistemas de Gestão de Backend, registra todos os sistemas de gestão de backend (assim, através do id do sistema e do id do uri do recurso do sistema, pode-se constituir uma unicidade global. O uri puro tem a possibilidade de duplicação. A razão para usar uri e não url é a possibilidade de mudança de domínio)
```

Finalmente, nosso sistema de usuários deve ter basicamente as 8 tabelas acima. Opa, parece que esqueci o login de terceiros, vamos adicionar, é bem simples:

```
9. Tabela de Login de Usuários de Terceiros, registra os identificadores de usuários de diferentes terceiros
```

Por fim, são as 9 tabelas acima. A estrutura específica das tabelas e o sql são os seguintes:

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-model.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-model-2.png">
    </a>
</p>


### SQL das Tabelas

**Modelo de Conta**

```sql

-- Modelo de Conta
CREATE TABLE `account_user` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id da conta',
  `email` varchar(30) NOT NULL DEFAULT '' COMMENT 'e-mail',
  `phone` varchar(15) NOT NULL DEFAULT '' COMMENT 'celular',
  `username` varchar(30) NOT NULL DEFAULT '' COMMENT 'nome de usuário',
  `password` varchar(32) NOT NULL DEFAULT '' COMMENT 'senha',
  `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de criação',
  `create_ip_at` varchar(12) NOT NULL DEFAULT '' COMMENT 'ip de criação',
  `last_login_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo do último login',
  `last_login_ip_at` varchar(12) NOT NULL DEFAULT '' COMMENT 'ip do último login',
  `login_times` int(11) NOT NULL DEFAULT '0' COMMENT 'número de logins',
  `status` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'status 1:ativado, 0:desativado, -1:deletado',
  PRIMARY KEY (`id`),
  KEY `idx_email` (`email`),
  KEY `idx_phone` (`phone`),
  KEY `idx_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Conta';

-- Conta de Terceiros
CREATE TABLE `account_platform` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id auto-incremental',
  `uid` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'id da conta',
  `platform_id` varchar(60) NOT NULL DEFAULT '' COMMENT 'id da plataforma',
  `platform_token` varchar(60) NOT NULL DEFAULT '' COMMENT 'access_token da plataforma',
  `type` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'tipo de plataforma 0:desconhecido,1:facebook,2:google,3:wechat,4:qq,5:weibo,6:twitter',
  `nickname` varchar(60) NOT NULL DEFAULT '' COMMENT 'apelido',
  `avatar` varchar(255) NOT NULL DEFAULT '' COMMENT 'avatar',
  `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de criação',
  `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de atualização',
  PRIMARY KEY (`id`),
  KEY `idx_uid` (`uid`),
  KEY `idx_platform_id` (`platform_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Informações de usuário de terceiros';
```

**Modelo de Usuário**

```sql

-- Modelo de Usuário
CREATE TABLE `skr_member` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id do usuário',
  `uid` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'id da conta',
  `nickname` varchar(30) NOT NULL DEFAULT '' COMMENT 'apelido',
  `avatar` varchar(255) NOT NULL DEFAULT '' COMMENT 'avatar (caminho relativo)',
  `gender` enum('male','female','unknow') NOT NULL DEFAULT 'unknow' COMMENT 'gênero',
  `role` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT 'papel 0:usuário comum 1:vip',
  `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de criação',
  `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de atualização',
  PRIMARY KEY (`id`),
  KEY `idx_uid` (`uid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Informações da conta';
```

**Modelo de Funcionário**

```sql

-- Tabela de Funcionários
CREATE TABLE `staff_info` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id do funcionário',
    `uid` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'id da conta',
    `email` varchar(30) NOT NULL DEFAULT '' COMMENT 'e-mail do funcionário',
    `phone` varchar(15) NOT NULL DEFAULT '' COMMENT 'celular do funcionário',
    `name` varchar(30) NOT NULL DEFAULT '' COMMENT 'nome do funcionário',
    `nickname` varchar(30) NOT NULL DEFAULT '' COMMENT 'apelido do funcionário',
    `avatar` varchar(255) NOT NULL DEFAULT '' COMMENT 'avatar do funcionário (caminho relativo)',
    `gender` enum('male','female','unknow') NOT NULL DEFAULT 'unknow' COMMENT 'gênero do funcionário',
    `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de criação',
    `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de atualização',
    PRIMARY KEY (`id`),
    KEY `idx_uid` (`uid`),
    KEY `idx_email` (`email`),
    KEY `idx_phone` (`phone`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Informações do funcionário (informações aproximadas listadas aqui, mais podem ser divididas verticalmente)';

```

**Modelo de Gestão de Permissões do Sistema**

```sql

-- Gestão de Permissões: Mapa do Sistema
CREATE TABLE `auth_ms` (
    `id` smallint(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id auto-incremental',
    `ms_name` varchar(255) NOT NULL DEFAULT '0' COMMENT 'nome do sistema',
    `ms_desc` varchar(255) NOT NULL DEFAULT '0' COMMENT 'descrição do sistema',
    `ms_domain` varchar(255) NOT NULL DEFAULT '0' COMMENT 'domínio do sistema',
    `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de criação',
    `create_by` int(11) NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de atualização',
    `update_by` int(11) NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'status 1:ativado, 0:desativado, -1:deletado',
    PRIMARY KEY (`id`),
    KEY `idx_domain` (`ms_domain`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Mapa do sistema (registra informações dos sistemas de backend existentes atualmente)';

-- Gestão de Permissões: Menu do Sistema
CREATE TABLE `auth_ms_menu` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id auto-incremental',
    `ms_id` smallint(11) unsigned NOT NULL DEFAULT '0' COMMENT 'id do sistema',
    `parent_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'id do menu pai',
    `menu_name` varchar(255) NOT NULL DEFAULT '0' COMMENT 'nome do menu',
    `menu_desc` varchar(255) NOT NULL DEFAULT '0' COMMENT 'descrição do menu',
    `menu_uri` varchar(255) NOT NULL DEFAULT '0' COMMENT 'uri do menu',
    `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de criação',
    `is_show` enum('yes','no') NOT NULL DEFAULT 'no' COMMENT 'se exibe o menu',
    `create_by` int(11) NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de atualização',
    `update_by` int(11) NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'status 1:ativado, 0:desativado, -1:deletado',
    PRIMARY KEY (`id`),
    KEY `idx_ms_id` (`ms_id`),
    KEY `idx_parent_id` (`parent_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Menu do sistema';

-- Gestão de Permissões: Permissão do Sistema
CREATE TABLE `auth_item` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id auto-incremental',
    `ms_id` tinyint(11) unsigned NOT NULL DEFAULT '0' COMMENT 'id do sistema',
    `menu_id` varchar(255) NOT NULL DEFAULT '0' COMMENT 'uri da página/interface',
    `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de criação',
    `create_by` int(11) NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de atualização',
    `update_by` int(11) NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'status 1:ativado, 0:desativado, -1:deletado',
    PRIMARY KEY (`id`),
    KEY `idx_ms_menu` (`ms_id`, `menu_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Permissão do sistema';

-- Gestão de Permissões: Permissão do Sistema (Coleções de permissões)
CREATE TABLE `auth_role` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id auto-incremental',
    `name` varchar(255) NOT NULL DEFAULT '0' COMMENT 'nome do papel',
    `desc` varchar(255) NOT NULL DEFAULT '0' COMMENT 'descrição do papel',
    `auth_item_set` text COMMENT 'conjunto de permissões, múltiplos valores separados por vírgula',
    `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de criação',
    `create_by` int(11) NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de atualização',
    `update_by` int(11) NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'status 1:ativado, 0:desativado, -1:deletado',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Papel do funcionário';

-- Gestão de Permissões: Relação Papel e Funcionário
CREATE TABLE `auth_role_staff` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id auto-incremental',
    `staff_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'id do funcionário',
    `role_set` text COMMENT 'conjunto de papéis, múltiplos valores separados por vírgula',
    `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de criação',
    `create_by` int(11) NOT NULL DEFAULT '0' COMMENT 'staff_id do criador',
    `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'tempo de atualização',
    `update_by` int(11) NOT NULL DEFAULT '0' COMMENT 'staff_id do modificador',
    `status` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'status 1:ativado, 0:desativado, -1:deletado',
    PRIMARY KEY (`id`),
    KEY `idx_staff_id` (`staff_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Relação entre papel de permissão e funcionário';

```


## Design de Interação

> Dica amigável: Uma grande onda de imagens está chegando, se não estiver claro, clique na imagem grande para ver

### Cadastro

Existem pelo menos dois tipos de interação após o cadastro bem-sucedido:

1. Cadastro com sucesso -> Redirecionar para a página de login
2. Cadastro com sucesso -> Login automático -> Redirecionar para a página inicial do aplicativo (ou outra página)

O fluxo de interação específico é o seguinte:

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-register-bmpr.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-register-bmpr.png" width="39%">
    </a>
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-register.jpg" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-register.jpg" width="90%">
    </a>
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-register-result-2.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-register-result-2.png" width="90%">
    </a>
</p>

--- 

### Login

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-login-page.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-login-page.png" width="30%">
    </a>
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-login-logic.jpg" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-login-logic.jpg" width="90%">
    </a>
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-home-page.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-home-page.png" width="30%">
    </a>
</p>

##### Login Rápido

O fluxo de login rápido é basicamente o mesmo que o acima, apenas a verificação de senha é trocada por verificação de código de verificação.

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-simple-login-page.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-simple-login-page.png" width="39%">
    </a>
</p>

---

### Login de Terceiros

Na verdade, existe esse problema na interação do login de terceiros:

1. Após o sucesso do login da conta de terceiros, ainda é necessário vincular o número de celular/E-mail?

Descobri que alguns PMs, para melhorar a simplicidade e rapidez do uso pelo usuário, geralmente geram um uid diretamente após o sucesso do login de terceiros, sem vincular a conta. Isso envolve o problema de fusão de contas posteriormente ao vincular a conta, o que é muito problemático (se houver carteira, etc.). Se fizermos a operação de vinculação desde o início, a relação da conta será clara e fácil de manter no futuro, o login de terceiros é na verdade equivalente a um alias da conta comum.
O resultado final de fazer isso ou não é se a relação entre a tabela de contas account_user e a tabela de informações de usuários de terceiros account_platform é **um para muitos** ou **um para um**.

2. Se vincular, o número de celular/E-mail já registrado pode ser vinculado?

Isso é fácil de dizer, geralmente a escolha de vincular é basicamente correta. O fluxograma final específico é o seguinte:


<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-platform-login.jpg" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-platform-login.jpg" width="100%">
    </a>
</p>


A interface de interação é a seguinte:

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-platform-login-page.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-platform-login-page.png" width="39%">
    </a>
</p>

---

### Gestão de Permissões de Backend

Primeiro, nosso sistema de gerenciamento de backend precisa de um nome sonoro. Pensei um pouco, a empresa anterior usava apollo, então eu estava pronto para usar mars, mas de repente surgiu **earth**, a raiz de todas as coisas na terra, e como este é um sistema de gerenciamento de serviços básicos para todos os negócios, haha, que seja assim ~ **Earth System**

A função de gerenciamento de permissões do **Earth System** é dividida principalmente nas quatro partes a seguir:

- Gestão do Sistema (The manage system page)
    + Página de edição
    + Página de lista
- Gestão de Menu (The menu page)
    + Página de edição
    + Página de lista
- Gestão de Papel (The role page)
    + Página de edição
    + Página de lista
- Gestão de Associação de Funcionários e Papéis (The role staff map page)
    + Página de edição
    + Página de lista

A interação específica é a seguinte:

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-earth-2.jpg" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-earth-2.jpg" width="100%">
    </a>
</p>

---

## Design de Interface

### Interface da Camada de Aplicação (Externa)

1. Interface de Cadastro

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
username|string|Não|Conta do usuário
email|string|um dos dois email/phone|E-mail do usuário
phone|string|um dos dois email/phone|Celular do usuário
code|int|Sim|Código de verificação

Conteúdo da resposta do modo de interação um (redirecionar para página de login):
```json
{
    "code": "200",
    "msg": "OK",
    "result": []
}
```

Conteúdo da resposta do modo de interação dois (redirecionar para página inicial):
```json
{
    "code": "200",
    "msg": "OK",
    "result": {
        "s_token": "string, token de sessão do usuário",
        "s_token_expire": "string, tempo de expiração do token de sessão do usuário, 0 não expira",
        "username": "string, nome de usuário",
        "nickname": "string, apelido do usuário",
        "avatar": "string, avatar do usuário",
        "gender": "string, gênero do usuário, male:homem, female:mulher, other:desconhecido",
    }
}
```

2. Interface de Login

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
username|string|um dos três username/email/phone|Conta do usuário
email|string|um dos três username/email/phone|E-mail do usuário
phone|string|um dos três username/email/phone|Celular do usuário
password|string|Sim|Senha

Conteúdo da resposta:
```json
{
    "code": "200",
    "result": {
        "s_token": "string, token de sessão do usuário",
        "s_token_expire": "string, tempo de expiração do token de sessão do usuário, 0 não expira",
        "nickname": "string, apelido do usuário",
        "username": "string, nome de usuário",
        "avatar": "string, avatar do usuário",
        "gender": "string, gênero do usuário, male:homem, female:mulher, other:desconhecido",
    }
}
```

3. Interface de Login Rápido

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
email|string|um dos dois email/phone|E-mail do usuário
phone|string|um dos dois email/phone|Celular do usuário
code|int|Sim|Código de verificação

Conteúdo da resposta:
```json
{
    "code": "200",
    "result": {
        "s_token": "string, token de sessão do usuário",
        "s_token_expire": "string, tempo de expiração do token de sessão do usuário, 0 não expira",
        "nickname": "string, apelido do usuário",
        "username": "string, nome de usuário",
        "avatar": "string, avatar do usuário",
        "gender": "string, gênero do usuário, male:homem, female:mulher, other:desconhecido",
    }
}
```

4. Interface de Login de Terceiros

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
type|string|Sim|Tipo de plataforma 1:facebook,2:google,3:wechat,4:qq,5:weibo,6:twitter
platform_id|string|Sim|ID do usuário da plataforma de terceiros
platform_token|string|Sim|Token da plataforma de terceiros

Conteúdo da resposta:
```json
{
    "code": "200",
    "result": {
        "s_token": "string, token de sessão do usuário",
        "s_token_expire": "string, tempo de expiração do token de sessão do usuário, 0 não expira",
        "username": "string, nome de usuário",
        "nickname": "string, apelido do usuário",
        "avatar": "string, avatar do usuário",
        "gender": "string, gênero do usuário, male:homem, female:mulher, other:desconhecido",
    }
}
```

5. Interface de Modificação de Informações do Usuário

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
username|string|Não|Conta do usuário
nickname|string|Não|Apelido
avatar|string|Não|Url do avatar
gender|string|Não|Gênero do usuário, male:homem, female:mulher, other:desconhecido

Conteúdo da resposta:
```json
{
    "code": "200",
    "result": {
        "username": "string, nome de usuário",
        "nickname": "string, apelido do usuário",
        "avatar": "string, avatar do usuário",
        "gender": "string, gênero do usuário, male:homem, female:mulher, other:desconhecido",
    }
}
```

6. Verificação de Status de Login do Usuário

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
s_token|string|Sim|Token de sessão do usuário

Conteúdo da resposta:
```json
{
    "code": "200",
    "result": {
        "s_token_expire": "string, tempo de expiração do token de sessão do usuário, 0 não expira, -1 login inválido",
    }
}
```

### Interface de Serviço (Serviço Básico, Interno)

**Serviço de Conta:**

1. Cadastro

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
username|string|Não|Conta do usuário
email|string|um dos dois email/phone|E-mail do usuário
phone|string|um dos dois email/phone|Celular do usuário

Conteúdo da resposta do modo de interação um (redirecionar para página de login):
```json
{
    "code": "200",
    "msg": "OK",
    "result": {
        "uid": "string, ID da conta"
    }
}
```

2. Login

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
username|string|Não|Conta do usuário
email|string|um dos dois email/phone|E-mail do usuário
phone|string|um dos dois email/phone|Celular do usuário
password|string|Sim|Senha

Conteúdo da resposta:
```json
{
    "code": "200",
    "msg": "OK",
    "result": {
        "uid": "string, ID da conta"
    }
}
```

2. Login de Terceiros

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
type|string|Sim|Tipo de plataforma 1:facebook,2:google,3:wechat,4:qq,5:weibo,6:twitter
platform_id|string|Sim|ID do usuário da plataforma de terceiros
platform_token|string|Sim|Token da plataforma de terceiros

Conteúdo da resposta:
```json
{
    "code": "200",
    "result": {
        "uid": "string, ID da conta",
        "nickname": "string, apelido do usuário",
        "avatar": "string, avatar do usuário",
    }
}
```

**Serviço de Permissões**

1. Obter Menu do Sistema

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
ms_id|string|Sim|ID do Sistema

Conteúdo da resposta:
```json
{
    "code": "200",
    "msg": "OK",
    "result": {
        "ms_name": "string, nome do sistema",
        "ms_desc": "string, descrição do sistema",
        "ms_domain": "string, domínio do sistema",
        "list": [
            {
                "parent_id": "string, ID do menu pai",
                "menu_id": "string, ID do menu",
                "menu_name": "string, nome do menu",
                "menu_desc": "string, descrição do menu",
                "menu_uri": "string, uri do menu",
                "child" : [
                    {
                        "parent_id": "string, ID do menu pai",
                        "menu_id": "string, ID do menu",
                        "menu_name": "string, nome do menu",
                        "menu_desc": "string, descrição do menu",
                        "menu_uri": "string, uri do menu",
                        "child" : []
                    }
                ]
            }
        ]
    }
}
```

2. Verificação de Permissão

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
menu_id|string|Sim|ID do menu

Conteúdo da resposta:
```json
{
    "code": "200",
    "msg": "OK",
    "result": []
}
```