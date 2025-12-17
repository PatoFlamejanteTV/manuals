# Sistema de Usuários

Hoje, vamos começar o design da primeira parte: o **Sistema de Usuários**. Este artigo é dividido em quatro módulos:

- Arquitetura
- Design do Modelo de Dados
- Design de Interação
- Design de Interface

## Arquitetura

### Uma Visão Simples do Sistema de Usuários

Quando você entra em contato com produtos de internet relacionados a usuários pela primeira vez, ou como eu costumava ver. O **Sistema de Usuários** nada mais é do que "Login", "Cadastro", "Alterar Informações do Usuário", etc. De forma simples, só precisamos de uma tabela para registrar as informações de identidade do usuário: ao registrar (operação insert), insira um dado na tabela; ao fazer login (operação select & update), julgue se a senha do usuário está correta através do identificador do usuário (número de celular, e-mail, etc.); ao modificar as informações do usuário (operação select & update), basta atualizar diretamente as informações do usuário desse uid (avatar, apelido, etc.).

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-smaple-structure.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-smaple-structure.png">
    </a>
</p>

Não há nada de errado com esse design, é muito simples, não é? Mas com o desenvolvimento do negócio, por um lado precisamos fornecer um gerenciamento de usuários unificado (alta coesão), e por outro lado precisamos melhorar a escalabilidade do sistema, então o que quero apresentar é o que entendo que **um sistema de usuários básico deve ter**.

### O Que Um Sistema de Usuários Básico Deve Ter

Primeiro, abstraímos a tabela de usuários original novamente (extraindo os campos dependentes de registro e login de usuário, login de terceiros) -> **Tabela de Contas**. Por que fazer isso? Com o desenvolvimento do negócio, costumávamos manter apenas um produto, talvez um dia desenvolvamos novos produtos, para que possamos manter unificadamente a lógica de registro e login de todos os produtos da nossa empresa, e diferentes produtos mantêm apenas as informações relacionadas a esse produto e ao usuário (dependendo da forma do produto). Como mostrado abaixo:

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-user-system-2.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-user-system-2.png">
    </a>
</p>

Na figura acima, também são mencionados login de terceiros / tabela de funcionários / gestão de permissões de backend, que são algumas estruturas básicas necessárias para o sistema de usuários.

Login de terceiros: O login de terceiros também é um tipo de método de login, e também o abstraímos como parte da conta, como mostrado acima. Em segundo lugar, há um problema no design do método de interação sobre o login de terceiros, que será mencionado no design de interação posterior.

Funcionários: Como extraímos a tabela de contas acima, o sistema de gerenciamento interno (backend) também pode usar unificadamente a lógica de login da tabela de contas, alcançando assim uma verdadeira alta coesão na questão de contas em toda a empresa.

Falando em funcionários, nossos vários sistemas de backend internos certamente envolvem vários gerenciamentos de permissões, então aqui mencionamos um RBAC simples (Controle de Acesso Baseado em Função), e o design específico do modelo de dados lógico será mencionado.

### A Arquitetura Final

À medida que a forma do produto de negócios se torna cada vez mais complexa, ao projetar a arquitetura, precisamos analisar o que **muda e o que não muda**:

- Muda: Cada vez mais necessidades personalizadas de usuários de produtos
- Não muda: A lógica de registro e login

O resultado final é que dividimos o usuário original em **Conta** e **Usuário**, e também precisamos esclarecer a diferença entre esses dois conceitos aqui:

- Conta: O único lugar em todo o sistema que gera uid, coeso com a lógica de registro e login, não envolvendo requisitos de negócios do produto
- Usuário: Informações de requisitos de usuário personalizadas de diferentes produtos

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

Correspondendo à arquitetura acima, podemos facilmente projetar nosso modelo de dados (assumindo que atualmente temos apenas um aplicativo para o consumidor final - C-end):

```
Conta -> 1. Tabela de Contas
Usuário -> 2. Tabela de Usuários
Funcionário -> 3. Tabela de Funcionários
```

Além das três tabelas acima, também precisamos do nosso gerenciamento de permissões R(role)B(base)A(access)C(control). Todos devem estar familiarizados com o gerenciamento de permissões baseado em funções RBAC, então não entrarei em detalhes aqui. Um RBAC simples precisa primeiro de:

```
4. Tabela de menus do sistema (menu é permissão), caminho uri do sistema
5. Tabela de permissões (menu é permissão), a permissão específica é acessar o menu do sistema
6. Tabela de funções, quais permissões uma função possui
7. Tabela de associação entre funcionários e funções, a qual função um funcionário pertence
```

Ok, as tabelas envolvidas em um RBAC simples estão basicamente listadas, mas na minha experiência de trabalho, o gerenciamento de permissões implementado por todos geralmente visa apenas um determinado sistema, o que é confuso, repetitivo ("reinventar a roda") e ineficiente para muitos sistemas de backend. Portanto, no design da arquitetura acima, fiz da permissão um serviço para fornecer recursos básicos de serviço para todo o sistema. E para atingir esse objetivo, só preciso adicionar mais uma tabela:

```
8. Tabela de sistemas de gerenciamento de backend, registra todos os sistemas de gerenciamento de backend (assim, através do id do sistema e do id do uri do recurso do sistema, a exclusividade global pode ser constituída. O uri simples tem a possibilidade de duplicação. A razão para usar uri em vez de url é a possibilidade de mudanças de domínio)
```

Finalmente, nosso sistema de usuários deve ter basicamente as 8 tabelas acima. Opa, parece que esqueci o login de terceiros, vamos adicionar, é muito simples:

```
9. Tabela de login de usuário de terceiros, registra diferentes identificadores de usuários de terceiros
```

E finalmente, são as 9 tabelas acima. A estrutura específica da tabela e o sql são os seguintes:

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
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID da conta',
  `email` varchar(30) NOT NULL DEFAULT '' COMMENT 'E-mail',
  `phone` varchar(15) NOT NULL DEFAULT '' COMMENT 'Número de celular',
  `username` varchar(30) NOT NULL DEFAULT '' COMMENT 'Nome de usuário',
  `password` varchar(32) NOT NULL DEFAULT '' COMMENT 'Senha',
  `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de criação',
  `create_ip_at` varchar(12) NOT NULL DEFAULT '' COMMENT 'IP de criação',
  `last_login_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data do último login',
  `last_login_ip_at` varchar(12) NOT NULL DEFAULT '' COMMENT 'IP do último login',
  `login_times` int(11) NOT NULL DEFAULT '0' COMMENT 'Número de logins',
  `status` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'Status 1:enable, 0:disable, -1:deleted',
  PRIMARY KEY (`id`),
  KEY `idx_email` (`email`),
  KEY `idx_phone` (`phone`),
  KEY `idx_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Conta';

-- Conta de Terceiros
CREATE TABLE `account_platform` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID incremental',
  `uid` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID da conta',
  `platform_id` varchar(60) NOT NULL DEFAULT '' COMMENT 'ID da plataforma',
  `platform_token` varchar(60) NOT NULL DEFAULT '' COMMENT 'Access token da plataforma',
  `type` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'Tipo de plataforma 0:desconhecido,1:facebook,2:google,3:wechat,4:qq,5:weibo,6:twitter',
  `nickname` varchar(60) NOT NULL DEFAULT '' COMMENT 'Apelido',
  `avatar` varchar(255) NOT NULL DEFAULT '' COMMENT 'Avatar',
  `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de criação',
  `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de atualização',
  PRIMARY KEY (`id`),
  KEY `idx_uid` (`uid`),
  KEY `idx_platform_id` (`platform_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Informações de usuário de terceiros';
```

**Modelo de Usuário**

```sql

-- Modelo de Usuário
CREATE TABLE `skr_member` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID do usuário',
  `uid` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID da conta',
  `nickname` varchar(30) NOT NULL DEFAULT '' COMMENT 'Apelido',
  `avatar` varchar(255) NOT NULL DEFAULT '' COMMENT 'Avatar (caminho relativo)',
  `gender` enum('male','female','unknow') NOT NULL DEFAULT 'unknow' COMMENT 'Gênero',
  `role` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT 'Papel 0:usuário comum 1:vip',
  `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de criação',
  `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de atualização',
  PRIMARY KEY (`id`),
  KEY `idx_uid` (`uid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Informações da conta';
```

**Modelo de Funcionário**

```sql

-- Tabela de Funcionários
CREATE TABLE `staff_info` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID do funcionário',
    `uid` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID da conta',
    `email` varchar(30) NOT NULL DEFAULT '' COMMENT 'E-mail do funcionário',
    `phone` varchar(15) NOT NULL DEFAULT '' COMMENT 'Celular do funcionário',
    `name` varchar(30) NOT NULL DEFAULT '' COMMENT 'Nome do funcionário',
    `nickname` varchar(30) NOT NULL DEFAULT '' COMMENT 'Apelido do funcionário',
    `avatar` varchar(255) NOT NULL DEFAULT '' COMMENT 'Avatar do funcionário (caminho relativo)',
    `gender` enum('male','female','unknow') NOT NULL DEFAULT 'unknow' COMMENT 'Gênero do funcionário',
    `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de criação',
    `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de atualização',
    PRIMARY KEY (`id`),
    KEY `idx_uid` (`uid`),
    KEY `idx_email` (`email`),
    KEY `idx_phone` (`phone`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Informações do funcionário (informações aproximadas listadas aqui, mais podem ser divididas verticalmente)';

```

**Modelo de Gerenciamento de Permissões do Sistema**

```sql

-- Gerenciamento de Permissões: Mapa do Sistema
CREATE TABLE `auth_ms` (
    `id` smallint(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID incremental',
    `ms_name` varchar(255) NOT NULL DEFAULT '0' COMMENT 'Nome do sistema',
    `ms_desc` varchar(255) NOT NULL DEFAULT '0' COMMENT 'Descrição do sistema',
    `ms_domain` varchar(255) NOT NULL DEFAULT '0' COMMENT 'Domínio do sistema',
    `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de criação',
    `create_by` int(11) NOT NULL DEFAULT '0' COMMENT 'ID do funcionário criador',
    `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de atualização',
    `update_by` int(11) NOT NULL DEFAULT '0' COMMENT 'ID do funcionário modificador',
    `status` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'Status 1:enable, 0:disable, -1:deleted',
    PRIMARY KEY (`id`),
    KEY `idx_domain` (`ms_domain`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Mapa do sistema (registra informações do sistema de backend existentes atualmente)';

-- Gerenciamento de Permissões: Menu do Sistema
CREATE TABLE `auth_ms_menu` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID incremental',
    `ms_id` smallint(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID do sistema',
    `parent_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID do menu pai',
    `menu_name` varchar(255) NOT NULL DEFAULT '0' COMMENT 'Nome do menu',
    `menu_desc` varchar(255) NOT NULL DEFAULT '0' COMMENT 'Descrição do menu',
    `menu_uri` varchar(255) NOT NULL DEFAULT '0' COMMENT 'URI do menu',
    `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de criação',
    `is_show` enum('yes','no') NOT NULL DEFAULT 'no' COMMENT 'Se exibe o menu',
    `create_by` int(11) NOT NULL DEFAULT '0' COMMENT 'ID do funcionário criador',
    `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de atualização',
    `update_by` int(11) NOT NULL DEFAULT '0' COMMENT 'ID do funcionário modificador',
    `status` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'Status 1:enable, 0:disable, -1:deleted',
    PRIMARY KEY (`id`),
    KEY `idx_ms_id` (`ms_id`),
    KEY `idx_parent_id` (`parent_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Menu do sistema';

-- Gerenciamento de Permissões: Permissões do Sistema
CREATE TABLE `auth_item` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID incremental',
    `ms_id` tinyint(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID do sistema',
    `menu_id` varchar(255) NOT NULL DEFAULT '0' COMMENT 'URI da página/interface',
    `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de criação',
    `create_by` int(11) NOT NULL DEFAULT '0' COMMENT 'ID do funcionário criador',
    `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de atualização',
    `update_by` int(11) NOT NULL DEFAULT '0' COMMENT 'ID do funcionário modificador',
    `status` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'Status 1:enable, 0:disable, -1:deleted',
    PRIMARY KEY (`id`),
    KEY `idx_ms_menu` (`ms_id`, `menu_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Permissões do sistema';

-- Gerenciamento de Permissões: Permissões do Sistema (Conjunto de Permissões)
CREATE TABLE `auth_role` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID incremental',
    `name` varchar(255) NOT NULL DEFAULT '0' COMMENT 'Nome da função',
    `desc` varchar(255) NOT NULL DEFAULT '0' COMMENT 'Descrição da função',
    `auth_item_set` text COMMENT 'Conjunto de permissões, múltiplos valores separados por vírgula',
    `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de criação',
    `create_by` int(11) NOT NULL DEFAULT '0' COMMENT 'ID do funcionário criador',
    `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de atualização',
    `update_by` int(11) NOT NULL DEFAULT '0' COMMENT 'ID do funcionário modificador',
    `status` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'Status 1:enable, 0:disable, -1:deleted',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Função do funcionário';

-- Gerenciamento de Permissões: Relação entre Função e Funcionário
CREATE TABLE `auth_role_staff` (
    `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID incremental',
    `staff_id` int(11) unsigned NOT NULL DEFAULT '0' COMMENT 'ID do funcionário',
    `role_set` text COMMENT 'Conjunto de funções, múltiplos valores separados por vírgula',
    `create_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de criação',
    `create_by` int(11) NOT NULL DEFAULT '0' COMMENT 'ID do funcionário criador',
    `update_at` int(11) NOT NULL DEFAULT '0' COMMENT 'Data de atualização',
    `update_by` int(11) NOT NULL DEFAULT '0' COMMENT 'ID do funcionário modificador',
    `status` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'Status 1:enable, 0:disable, -1:deleted',
    PRIMARY KEY (`id`),
    KEY `idx_staff_id` (`staff_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Relação entre função de permissão e funcionário';

```


## Design de Interação

> Dica amigável: Uma grande onda de imagens está chegando, se as imagens não estiverem claras, clique na imagem para ver ampliado

### Cadastro

Existem pelo menos dois métodos de interação após o registro bem-sucedido:

1. Cadastro com sucesso -> Ir para a página de login
2. Cadastro com sucesso -> Login automático -> Ir para a página inicial do aplicativo (ou outra página)

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

O processo de login rápido é basicamente o mesmo que o acima, apenas a verificação de senha é alterada para verificação de código.

<p align="center">
    <a href="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-simple-login-page.png" data-lightbox="roadtrip">
        <img src="http://blog-1251019962.cos.ap-beijing.myqcloud.com/qiniu_img_2022/skr-account-simple-login-page.png" width="39%">
    </a>
</p>

---

### Login de Terceiros

A interação de login de terceiros na verdade tem este problema:

1. Após o login bem-sucedido da conta de terceiros, ainda é necessário vincular o número de celular/e-mail?

Porque descobri que alguns PMs (Product Managers), para melhorar a simplicidade e rapidez de uso dos usuários, muitas vezes geram uid diretamente após o login de terceiros bem-sucedido, sem vincular a conta. Dessa forma, a vinculação de conta posterior envolve o problema de fusão de contas, o que é muito problemático (se houver carteiras, etc.). Se fizermos a operação de vinculação desde o início, a relação futura da conta será clara e fácil de manter. O login de terceiros é na verdade equivalente a um alias de uma conta comum.
O resultado final de fazer isso ou não é se a relação entre a tabela de contas `account_user` e a tabela de informações de usuários de terceiros `account_platform` é **um para muitos** ou **um para um**.

2. Se vincular, o celular/e-mail já registrado pode ser vinculado?

Isso é fácil de dizer, geralmente a escolha de vincular é basicamente correta. O fluxograma específico final é o seguinte:


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

Primeiro, nosso sistema de gerenciamento de backend precisa de um nome sonoro. Pensei um pouco, a empresa anterior usava apollo, então eu estava pronto para usar mars, mas de repente apareceu earth, a raiz de todas as coisas na terra. Acontece que este é um sistema de gerenciamento de serviço básico para todos os negócios, haha, então que seja assim ~ **Earth System**

A função de gerenciamento de permissões do **Earth System** é dividida principalmente nas quatro partes a seguir:

- Gerenciamento de Sistema (A página de gerenciamento do sistema)
    + Página de edição
    + Página de lista
- Gerenciamento de Menu (A página de menu)
    + Página de edição
    + Página de lista
- Gerenciamento de Função (A página de função)
    + Página de edição
    + Página de lista
- Gerenciamento de Associação entre Funcionário e Função (A página de mapa de função e funcionário)
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

### Interfaces da Camada de Aplicação (Externa)

1. Interface de Cadastro

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
username|string|Não obrigatório|Conta de usuário
email|string|Escolha um entre email/phone|E-mail do usuário
phone|string|Escolha um entre email/phone|Celular do usuário
code|int|Obrigatório|Código de verificação

Resposta do método de interação um (ir para a página de login):
```json
{
    "code": "200",
    "msg": "OK",
    "result": []
}
```

Resposta do método de interação dois (ir para a página inicial):
```json
{
    "code": "200",
    "msg": "OK",
    "result": {
        "s_token": "string, identificador de sessão do usuário",
        "s_token_expire": "string, tempo de expiração do identificador de sessão do usuário, 0 não expira",
        "username": "string, nome de usuário",
        "nickname": "string, apelido do usuário",
        "avatar": "string, avatar do usuário",
        "gender": "string, gênero do usuário, male:masculino, female:feminino, other:desconhecido",
    }
}
```

2. Interface de Login

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
username|string|Escolha um entre username/email/phone|Conta de usuário
email|string|Escolha um entre username/email/phone|E-mail do usuário
phone|string|Escolha um entre username/email/phone|Celular do usuário
password|string|Obrigatório|Senha

Conteúdo da resposta:
```json
{
    "code": "200",
    "result": {
        "s_token": "string, identificador de sessão do usuário",
        "s_token_expire": "string, tempo de expiração do identificador de sessão do usuário, 0 não expira",
        "nickname": "string, apelido do usuário",
        "username": "string, nome de usuário",
        "avatar": "string, avatar do usuário",
        "gender": "string, gênero do usuário, male:masculino, female:feminino, other:desconhecido",
    }
}
```

3. Interface de Login Rápido

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
email|string|Escolha um entre email/phone|E-mail do usuário
phone|string|Escolha um entre email/phone|Celular do usuário
code|int|Obrigatório|Código de verificação

Conteúdo da resposta:
```json
{
    "code": "200",
    "result": {
        "s_token": "string, identificador de sessão do usuário",
        "s_token_expire": "string, tempo de expiração do identificador de sessão do usuário, 0 não expira",
        "nickname": "string, apelido do usuário",
        "username": "string, nome de usuário",
        "avatar": "string, avatar do usuário",
        "gender": "string, gênero do usuário, male:masculino, female:feminino, other:desconhecido",
    }
}
```

4. Interface de Login de Terceiros

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
type|string|Obrigatório|Tipo de plataforma 1:facebook,2:google,3:wechat,4:qq,5:weibo,6:twitter
platform_id|string|Obrigatório|ID do usuário da plataforma de terceiros
platform_token|string|Obrigatório|Token da plataforma de terceiros

Conteúdo da resposta:
```json
{
    "code": "200",
    "result": {
        "s_token": "string, identificador de sessão do usuário",
        "s_token_expire": "string, tempo de expiração do identificador de sessão do usuário, 0 não expira",
        "username": "string, nome de usuário",
        "nickname": "string, apelido do usuário",
        "avatar": "string, avatar do usuário",
        "gender": "string, gênero do usuário, male:masculino, female:feminino, other:desconhecido",
    }
}
```

5. Interface de Modificação de Informações do Usuário

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
username|string|Não obrigatório|Conta de usuário
nickname|string|Não obrigatório|Apelido
avatar|string|Não obrigatório|URL do avatar
gender|string|Não obrigatório|Gênero do usuário, male:masculino, female:feminino, other:desconhecido

Conteúdo da resposta:
```json
{
    "code": "200",
    "result": {
        "username": "string, nome de usuário",
        "nickname": "string, apelido do usuário",
        "avatar": "string, avatar do usuário",
        "gender": "string, gênero do usuário, male:masculino, female:feminino, other:desconhecido",
    }
}
```

6. Verificação de Status de Login do Usuário

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
s_token|string|Obrigatório|Identificador de sessão do usuário

Conteúdo da resposta:
```json
{
    "code": "200",
    "result": {
        "s_token_expire": "string, tempo de expiração do identificador de sessão do usuário, 0 não expira, -1 login inválido",
    }
}
```

### Interfaces de Serviço (Serviço Básico, Interno)

**Serviço de Conta:**

1. Cadastro

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
username|string|Não obrigatório|Conta de usuário
email|string|Escolha um entre email/phone|E-mail do usuário
phone|string|Escolha um entre email/phone|Celular do usuário

Resposta do método de interação um (ir para a página de login):
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
username|string|Não obrigatório|Conta de usuário
email|string|Escolha um entre email/phone|E-mail do usuário
phone|string|Escolha um entre email/phone|Celular do usuário
password|string|Obrigatório|Senha

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
type|string|Obrigatório|Tipo de plataforma 1:facebook,2:google,3:wechat,4:qq,5:weibo,6:twitter
platform_id|string|Obrigatório|ID do usuário da plataforma de terceiros
platform_token|string|Obrigatório|Token da plataforma de terceiros

Conteúdo da resposta:
```json
{
    "code": "200",
    "result": {
        "uid": "string, ID da conta",
        "nickname": "string, Apelido do usuário",
        "avatar": "string, Avatar do usuário",
    }
}
```

**Serviço de Permissões**

1. Obter Menu do Sistema

Parâmetros de requisição:

Campo|Tipo|Obrigatório|Descrição
------------|------------|------------|------------
ms_id|string|Obrigatório|ID do sistema

Conteúdo da resposta:
```json
{
    "code": "200",
    "msg": "OK",
    "result": {
        "ms_name": "string, Nome do sistema",
        "ms_desc": "string, Descrição do sistema",
        "ms_domain": "string, Domínio do sistema",
        "list": [
            {
                "parent_id": "string, ID do menu pai",
                "menu_id": "string, ID do menu",
                "menu_name": "string, ID do menu",
                "menu_desc": "string, Descrição do menu",
                "menu_uri": "string, URI do menu",
                "child" : [
                    {
                        "parent_id": "string, ID do menu pai",
                        "menu_id": "string, ID do menu",
                        "menu_name": "string, ID do menu",
                        "menu_desc": "string, Descrição do menu",
                        "menu_uri": "string, URI do menu",
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
menu_id|string|Obrigatório|ID do menu

Conteúdo da resposta:
```json
{
    "code": "200",
    "msg": "OK",
    "result": []
}
```
