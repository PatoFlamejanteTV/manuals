# Sistema de Compras

## Serviço de Carrinho de Compras

Para um e-commerce, o carrinho de compras é a etapa mais importante de todo o processo de compra. Isso porque, no desenvolvimento do e-commerce até hoje, o carrinho de compras não serve apenas para a função de empacotar e fazer o pedido; é também uma importante janela de exibição para favoritos, comparação, lembretes de promoção e recomendações relacionadas. Com tantas capacidades, como devemos projetar para garantir o alto desempenho do carrinho de compras e uma boa capacidade de expansão para atender ao desenvolvimento futuro?

A partir de hoje, usaremos um cenário hipotético para apresentar um design de carrinho de compras: uma determinada plataforma de e-commerce, que é um modelo multi-inquilino (nossos designs anteriores são todos modelos multi-inquilino), onde os usuários podem adicionar produtos ao carrinho de compras e exibi-los e ordená-los de acordo com a dimensão do comerciante. Claro, o carrinho de compras também suporta várias operações convencionais: selecionar, excluir, limpar, invalidar produtos, etc. E há promoções relacionadas que podem lembrar os usuários. Ao mesmo tempo, para monitoramento e operação, é necessário suportar a sincronização dos dados do carrinho de compras para monitoramento, data warehouse e outras capacidades.

Este artigo explicará as capacidades do sistema a partir da perspectiva do usuário e do servidor. O objetivo principal deste artigo é esclarecer as capacidades do carrinho de compras e algumas lógicas. O próximo artigo abordará o design do modelo do carrinho de compras e a definição da interface.

### Perspectiva do Usuário

Vamos primeiro definir quais são as funções de operação do carrinho de compras no lado do usuário?

![Requisitos do usuário](https://dayutalk.cn/img/user-cart-c.png)

As capacidades básicas de um carrinho de compras estão basicamente na figura acima, vamos decompor uma a uma abaixo.

#### Operações

Do ponto de vista do usuário, o carrinho de compras permite adicionar produtos (adicionar ao carrinho, comprar agora são formas de adição); após adicionar ao carrinho, se não quiser mais, pode excluir o produto (excluir um, excluir vários, limpar); se quiser comprar mais, pode modificar a quantidade de compra, se descobrir que não tem dinheiro suficiente, pode reduzir a quantidade; ou se descobrir que o vermelho é mais bonito que o branco, pode alterar a especificação convenientemente no carrinho; para alguns produtos muito caros, pode adicionar alguns serviços de garantia no carrinho (na verdade, são produtos virtuais vinculados); ao ir para o checkout, também fornecerá a capacidade de seleção para o usuário decidir quais produtos realmente comprar desta vez.

Através da descrição acima, podemos ver que este processo tem suas conexões internas. Aqui falamos sobre a função de seleção, existem duas práticas na indústria, cada uma com vantagens e desvantagens, vamos ver. O estado de seleção do produto do Taobao é salvo no cliente e, por padrão, não é selecionado, o estado desaparece ao atualizar ou reabrir o APP; JD, Suning e similares são salvos no servidor, registrando o estado de seleção do usuário. Existem prós e contras para essas duas situações.

**Cliente:**

1. Desempenho: a lógica de selecionado/não selecionado é feita diretamente no local, reduzindo as requisições de rede.
2. Experiência: não pode sincronizar em múltiplos terminais, mas o carrinho de compras é relativamente mais como uma lista de favoritos, não há problema se o usuário selecionar novamente a cada vez.
3. Cálculo: ao calcular o preço, é necessário enviar os produtos selecionados localmente (também pode ser calculado localmente).
4. Implementação: depende principalmente da implementação do cliente, irrelevante para o servidor, desacoplamento de desenvolvimento.

**Servidor:**

1. Desempenho: cada operação de seleção precisa chamar o servidor, e essa operação pode ser muito frequente, além da perda de rede, o servidor também precisa considerar como encontrar rapidamente o produto modificado.
2. Experiência: sincronização de estado em múltiplos terminais, registro de histórico de estado.
3. Cálculo: o servidor pode obter dados, sem necessidade de enviar dados extras na requisição.
4. Implementação: o servidor e o cliente precisam concordar sobre como interagir e retornar dados (cada seleção causará alteração de preço), acoplados.

Pessoalmente, acho que nenhum dos dois métodos tem uma vantagem óbvia, é completamente uma escolha baseada no modelo de negócios e na situação da equipe. Nosso design subsequente aqui será baseado em salvar o estado de seleção do produto no servidor.

Em toda a lógica de operação, há dois lugares importantes para explicar separadamente: método de compra e modificação de atributos de compra no carrinho.

##### Método de Compra

Os principais métodos de compra são: comprar agora, adicionar ao carrinho e compra em grupo.

Primeiro, a adição comum ao carrinho de compras não tem muito o que dizer. Vamos focar em comprar agora e compra em grupo.

Comprar agora, em termos de operação, é selecionar o produto e ir diretamente para a página de confirmação do pedido, sem a etapa de ir ao carrinho para finalizar a compra. Mas sua implementação pode depender da lógica do carrinho de compras. Vamos ver qual a diferença entre usar o carrinho de compras e não usar o carrinho de compras para implementar essa lógica?

Se usarmos o carrinho de compras para implementar, ou seja, quando o usuário clica em comprar agora, o produto é essencialmente adicionado ao carrinho, mas este carrinho é diferente do carrinho original, pois este carrinho só pode adicionar um produto e será sobrescrito a cada operação. No efeito visual, também salta diretamente da página de detalhes do produto para a página de confirmação do pedido. Vamos ver os benefícios desta abordagem:

1. Consistente com a lógica de confirmação de pedido e checkout do carrinho de compras, internamente pode obter dados diretamente através do carrinho.
2. Requer um carrinho de compras independente dedicado para compra com um clique, com consumo de memória.

Outra forma de implementação usa uma nova estrutura de dados, porque geralmente a compra com um clique é mais simples, precisando apenas de informações do produto e informações de preço. Cada interação pode ser obtida com base no sku_id.

1. A lógica de confirmação de pedido e checkout precisa ser reformulada, parâmetros acordados devem ser passados entre cada requisição.
2. Economia de memória, interação superior e inferior garantida pelo sku_id.

Adotaremos a implementação usando um carrinho de compras independente para compra com um clique no servidor. O modelo de dados do carrinho de compras é consistente, garantindo consistência no fluxo de processamento subsequente.

Para a compra em grupo, ela é dividida em duas partes: primeiro é a ação de abrir o grupo. Quando o grupo é formado, podemos optar por adicionar os produtos do grupo ao carrinho comum, e ao mesmo tempo adicionar outros produtos. Também podemos optar por adicionar os produtos do grupo ao carrinho de compra com um clique, garantindo que o produto do grupo só possa ser comprado uma vez. O modo de compra em grupo é mais como uma pré-condição para adicionar ao carrinho. Essencialmente, não afeta o design do carrinho de compras.

##### Modificação de Atributos de Compra no Carrinho

Aqui refere-se principalmente à conveniência de operar algumas coisas que precisam ser operadas na dimensão spu no carrinho de compras, por exemplo: alterar especificações (ou seja, trocar sku), e selecionar serviços vinculados à dimensão spu (seguro, garantia estendida, etc.).

Vamos focar na seleção de serviços vinculados. Por exemplo: compramos um celular, o fabricante oferece garantia estendida, vários outros serviços adicionais, geralmente esses serviços são produtos virtuais. Mas há uma situação especial. Esses serviços de garantia primeiro não podem ser comprados separadamente, segundo, estão intimamente ligados à quantidade do produto principal. Por exemplo, se comprar dois celulares, se escolher adicionar serviços, a quantidade desses serviços deve ser 2, isso será uma relação vinculada.

Esses serviços de garantia não podem ser comprados separadamente, devem ser vendidos vinculados a produtos específicos.

Ao armazenar essa parte dos dados no servidor, deve-se considerar como salvar essa relação hierárquica, essa parte veremos no design do modelo mais tarde.

![Relação de produtos vinculados](https://dayutalk.cn/img/product-relations.png)

#### Lembrete

O lembrete de promoção é muito simples, os dados do carrinho retornados, cada produto deve carregar as informações atuais da promoção. O ponto chave aqui é como obter as informações da promoção, o que veremos no servidor.

Então fale sobre o lembrete da quantidade do carrinho, ou seja, exibir a quantidade atual de produtos no carrinho. Geralmente, ao entrar no APP, uma interface será chamada para obter o número de mensagens não lidas do usuário, o número de produtos no carrinho, etc. Isso requer uma velocidade de leitura muito alta. Então, como essa demanda pode ser atendida?

**Plano 1:** Podemos projetar uma estrutura para salvar o número dessas informações de lembrete relacionadas ao usuário e ler esses dados diretamente a cada vez. Não há necessidade de lidar com o serviço de mensagens ou serviço de carrinho para obter esses dados.

**Plano 2:** Projetar um campo para salvar a quantidade total nos modelos de mensagem e carrinho. Na interface de leitura de dados, chamar esses serviços simultaneamente para obter os dados e agregá-los, de modo que a velocidade dependa apenas do serviço mais lento.

Aqui nosso design adotará o **Plano 2**, porque assim a eficiência pode ser garantida até certo ponto, e a consistência dos dados da estrutura de todo o sistema é mais fácil de garantir. Claro, há um detalhe aqui que deve ser observado: a leitura concorrente deve ter um timeout projetado, para não arrastar o desempenho de toda a interface devido a problemas de leitura de um determinado serviço.

Em seguida, vamos olhar para as promoções. Além dos lembretes, esta parte também precisa fornecer as entradas correspondentes para permitir que os usuários concluam as operações de promoção. Por exemplo, se um produto tiver um cupom, uma entrada pode ser fornecida diretamente para coletá-lo; se puder ser combinado para desconto, haverá uma entrada para a lista de combinação e seleção de produtos, etc. O problema que precisa ser resolvido nesta parte é como o servidor pode obter essas atividades promocionais da dimensão do produto em tempo hábil.

Visto da perspectiva do usuário, vamos olhar para o que o servidor precisa fazer da perspectiva de pesquisa e desenvolvimento.

### Perspectiva de P&D

Vamos olhar para o gráfico de resumo dos requisitos novamente:

![Requisitos do servidor](https://dayutalk.cn/img/user-cart-s.png)

#### Armazenamento

Para armazenamento, a primeira escolha é definitivamente armazenamento em memória. Quanto a salvar no banco de dados, acho que não é necessário. Aqui estão meus motivos:

1. Os dados do carrinho de compras mudam com muita frequência, o custo de salvar no banco de dados é relativamente alto. Se for salvo de forma assíncrona, é difícil garantir a consistência.
2. Em casos extremos, se o cache falhar, o usuário só precisa adicionar ao carrinho novamente, e podemos garantir a recuperação de dados através do mecanismo de persistência do cache.

Portanto, para o carrinho de compras, salvaremos os dados inteiramente na memória.

#### Mudança no Tipo de Venda do Produto

Agora vamos discutir o problema da **Mudança no Tipo de Venda do Produto**. O que isso significa? Pense nisso: por exemplo, adicionei o produto A ao carrinho, mas não finalizei a compra. Neste momento, a operação diz que há uma atividade para o produto A, pegue 10 estoques com 50% de desconto. Então vem o problema: como lidar com usuários que já tinham esse produto no carrinho antes? **O principal problema a ser resolvido aqui é: usuários que já têm o produto no carrinho não podem comprar diretamente com 50% de desconto**. Vários planos, vamos ver:

**Plano 1:** Após a configuração da promoção, todos os usuários que tiverem esse produto no carrinho terão o produto invalidado ou excluído. Este plano é descartado primeiro, o custo da operação é muito alto e a experiência do usuário é ruim.

**Plano 2:** O carrinho de compras deve distinguir o mesmo SKU com diferentes tipos de venda. Ou seja, em nosso carrinho de compras, não adicionamos produtos pela dimensão SKU, mas geramos um código de identificação único através de **SKU + Tipo de Venda**.

Pode-se ver que o **Plano 2** resolve o problema da coexistência do mesmo sku no carrinho e o estoque não afeta um ao outro. Mas há outro problema aqui? O tipo de venda do produto (ou esta marca), onde deve ser configurado? Parece que o sistema de produtos pode projetar, e o sistema de promoção também pode configurar. Em nossa lógica, configuraremos no sistema de promoção. Porque o produto pertence à lógica básica, se alterado, o estoque global será afetado. Após o término da atividade, é difícil conseguir a venda normal automática. Portanto, esta marca deve ser configurada na atividade (ao configurar a atividade, será verificado através do sistema de promoção se a atividade anterior do produto é mutuamente exclusiva, para garantir que as atividades configuradas não se contradigam).

#### Sistemas Dependentes

O sistema de carrinho de compras depende de muitos outros sistemas.

- Sistema de Produtos
- Sistema de Estoque
- Sistema de Promoção
- Sistema de Checkout

Esses sistemas dependentes, alguns são para transmissão de dados, outros para obtenção de dados. Vamos olhar para essas duas dimensões.

##### Lembrete de Promoção e Cálculo

O servidor precisa resolver os problemas de lembrete de promoção e cálculo de preço.

Falando em cálculo, a melhor maneira para essa parte é chamar o cálculo de preço do centro de checkout. Vamos ver a diferença entre o cálculo de preço no carrinho de compras e o cálculo de preço no checkout do pedido.

Primeiro, ao calcular o preço no carrinho de compras, o endereço do usuário é desconhecido, o que afetará o cálculo do frete; segundo, o uso de cupons é desconhecido. Então, se resolvermos esses dois problemas, podemos fazer o cálculo de preço vir da mesma lógica, apenas alguns parâmetros de entrada são diferentes. Portanto, podemos calcular com base no frete mais alto aqui e, ao mesmo tempo, assumir que nenhum cupom é usado no carrinho por padrão. Para problemas de promoção, aqui é possível confirmar através do sistema de promoção quais preços os produtos selecionados podem desfrutar. Portanto, o preço da promoção deve ser calculado.

Em seguida, vamos falar sobre como fornecer informações de promoção aos usuários de forma eficiente. Começando pela nossa visão de configuração.

Ao configurar uma atividade promocional ou emitir um cupom, agrupamos vários produtos sob uma atividade promocional ou cupom. Se obtivermos produtos com base na dimensão de atividade e cupom, a eficiência é relativamente alta.

![Atividade-Produto](https://dayutalk.cn/img/activity-product.png)

Mas houve uma mudança no cenário do carrinho de compras. Precisamos obter todas as informações de atividade desse produto (atividades de toda a plataforma, atividades da loja) a partir da dimensão do produto;
Então, como fazer para exibir essas informações no carrinho? Uma prática muito comum (e de fato muitas empresas fazem isso): retirar todas as informações de atividade e percorrer todas as informações relacionadas a esse produto. Essa abordagem é muito ineficiente e não pode atender a cenários de aplicação em grande escala, como durante o Double 11.

Portanto, para atender a essa demanda aqui, o sistema de promoção precisa fornecer uma capacidade de obter promoções correspondentes (atividades, cupons) de acordo com o produto. Portanto, geralmente, as atividades configuradas pelo sistema de promoção não podem ser armazenadas apenas na dimensão da atividade, mas também precisam gerar uma cópia das informações de promoção na dimensão do produto.

![Produto-Atividade](https://dayutalk.cn/img/product-activity.png)

#### Análise de Dados do Carrinho

Para dados do carrinho de compras, o frontend registrará a adição de dados ao carrinho através de rastreamento (tracking), mas o rastreamento frontend geralmente registra que uma operação frontend foi acionada, mas não sabe se a operação foi bem-sucedida ou não. E não é possível entender a situação geral dos dados do carrinho em tempo hábil.

Para permitir que a equipe de operação entenda a situação atual do carrinho de compras de forma mais completa, registramos logs locais através do backend e, em seguida, sincronizamos os logs com serviços de dados, monitoramento, etc., através da coleta de logs.

#### Invalidação e Ordenação

Há mais duas pequenas partes que não foram mencionadas: primeiro, como o produto deve ser invalidado, por exemplo: sem estoque, fora de linha; segundo, os produtos no carrinho são de várias lojas, qual é a estratégia de ordenação?

Como este artigo discute apenas requisitos e não envolve design de modelo específico, apresentamos apenas o plano. Primeiro é a invalidação do produto, que é muito parecida com uma operação de soft delete. Uma vez definida, o produto visto pelo lado do usuário não poderá ser finalizado, apenas a operação de exclusão poderá ser realizada.

Para ordenação, o design que adotaremos é: de acordo com o tempo da última operação de uma determinada loja no carrinho, a operação mais recente certamente estará no topo.

### Conclusão

Através do acima, basicamente esclarecemos o que devemos fazer no design do carrinho de compras e quais capacidades os sistemas dependentes devem fornecer. O próximo artigo começará a entrar no design do modelo de dados e no design da interface frontend e backend.

Se você tiver algum complemento aos requisitos do carrinho de compras acima, deixe uma mensagem. Vamos aperfeiçoar juntos.


## Arquitetura do Carrinho de Compras

No artigo anterior [Análise de Requisitos de Design de Carrinho de Compras](https://dayutalk.cn/2019/12/09/%E8%B4%AD%E7%89%A9%E8%BD%A6%E8%AE%BE%E8%AE%A1%E4%B9%8B%E9%9C%80%E6%B1%82%E5%88%86%E6%9E%90/) foram descritos os requisitos gerais do carrinho de compras. Este artigo foca no design da arquitetura (negócios + arquitetura do sistema) sobre como implementá-lo.

### Explicação

O design da arquitetura pode ser dividido em três níveis:
- Arquitetura de Negócios
- Arquitetura de Sistema
- Arquitetura Técnica

Explicação rápida e simples dos três significados de arquitetura; quando recebemos os requisitos do carrinho de compras, dizemos usar Golang para implementar e Redis para armazenamento; isso descreve a arquitetura técnica; realizamos a divisão em camadas do código do projeto do carrinho de compras, especificações de design e planejamento de sistemas dependentes, isso é chamado de arquitetura de sistema;

Então, o que é arquitetura de negócios? A arquitetura de negócios é essencialmente a descrição textual da arquitetura do sistema; o que isso significa? Quando recebemos um requisito, primeiro precisamos nos comunicar com o solicitante para estabelecer um entendimento unificado. Por exemplo: padronizar terminologia (o significado de produto no carrinho de compras e produto no sistema de produtos é diferente); estabelecer modelos que todos possam entender, a interação entre entidades como carrinho de compras, usuário, produto, pedido e quais funções cada um possui.

Existem muitas metodologias na análise de arquitetura de negócios, por exemplo: Domain Driven Design (DDD), mas não é o único método de análise de arquitetura de negócios, nem necessariamente o melhor. O que serve para você é o melhor. Nossos diagramas de relacionamento de entidades e diagramas UML comumente usados também pertencem ao campo da arquitetura de negócios;

O que precisa ser enfatizado aqui é que, não importa qual método você use para modelar e projetar, ter um design é melhor do que não ter design. Em segundo lugar, certifique-se de refletir o conteúdo da modelagem em seu código.

A análise da arquitetura de negócios neste artigo utiliza o pensamento `DDD` (Domain Driven Design); novamente, `o que serve é o melhor`.

### Arquitetura de Negócios

Através da análise de requisitos anterior, já esclarecemos o que nosso carrinho de compras deve fazer. Vamos primeiro ver um processo típico de operação do usuário no carrinho de compras.

![Jornada do Usuário](https://dayutalk.cn/img/cart-sys-00.png)

Neste processo, o usuário utiliza o veículo carrinho de compras para completar o processo de compra de produtos; o dado que flui constantemente é o produto, e o veículo carrinho de compras é estável. Estes são os pontos estáveis e pontos de mudança em nosso sistema.

A forma como os produtos fluem pode variar, por exemplo, adicionar ao carrinho de lugares diferentes, adicionar ao carrinho de maneiras diferentes, o ciclo de vida no carrinho também é diferente; mas esse fluxo é estável, deve-se primeiro deixar o produto existir no carrinho e depois ir para o checkout para gerar o pedido.

O ciclo de vida do produto no carrinho de compras é o seguinte:

![Processo](https://dayutalk.cn/img/cart-sys-01.jpg)

De acordo com este processo, vamos ver as operações correspondentes a cada etapa.

![Operações correspondentes ao processo](https://dayutalk.cn/img/cart-sys-02.jpg)

Note aqui que a operação antes de adicionar ao carrinho pode ser colocada na operação de adição do carrinho, mas como esta parte é muito instável e variável, nós a isolamos para facilitar a expansão subsequente sem afetar a etapa de carrinho relativamente estável.

> As três etapas acima, de acordo com o conceito em DDD, devem ser chamadas de Entidades, e juntas constituem o domínio do carrinho de compras; hoje não vamos falar sobre esses conceitos, vamos pular, e se houver oportunidade mais tarde, publicarei um artigo separado para explicar.

#### Antes de Adicionar ao Carrinho

Através da análise de fluxo, resumimos as interfaces de operação que o sistema precisa ter e as entidades correspondentes a essas interfaces. Agora vamos ver o que precisa ser feito antes de adicionar ao carrinho;

Antes de adicionar ao carrinho, é basicamente realizar verificações em várias dimensões para os produtos que estão prestes a ser adicionados, verificando se atendem aos requisitos.

Antes de permitir que o usuário adicione ao carrinho, a primeira coisa que resolvemos é de onde o usuário está comprando e depois verificamos? Porque o mesmo produto comprado de canais diferentes tem situações diferentes, por exemplo: celular Xiaomi, compramos via seckill (oferta relâmpago), ou via crowdfunding de amigos, ou compra direta no shopping, o preço é diferente, mas na verdade é o mesmo produto;

A segunda questão é se possui qualificação de compra. Como mencionado acima, a operação de adicionar ao carrinho de seckill ou crowdfunding não está disponível para qualquer um, deve-se ter qualificação. Então a verificação de qualificação também é colocada aqui;

A terceira questão é verificar os atributos do produto comprado, como se está disponível/fora de estoque, se tem estoque, limite de quantidade de compra, etc.

E todos descobrirão que as condições de verificação aqui podem ser muito variáveis. Como construir um código fácil de estender?

![Verificação de Adição ao Carrinho](https://dayutalk.cn/img/cart-sys-03.jpg)

Em todo o processo de adição ao carrinho, o importante é distinguir diferentes verificações com base na fonte. Temos duas opções de escolha.

Opção 1: Fazer através do modo Strategy + modo Facade. A estratégia é realizar diferentes verificações com base em diferentes fontes de adição ao carrinho, e a fachada é encapsular estratégias individuais com base em diferentes fontes;

Opção 2: Através do modo Chain of Responsibility (Cadeia de Responsabilidade), mas aqui precisa haver uma mudança, esta cadeia no processo de execução pode escolher pular certos nós, por exemplo: seckill não precisa de verificação de estoque, nem de crowdfunding;

Através de uma análise abrangente, escolhi o modo Chain of Responsibility. Postando o código principal:

```go
// Interface que cada lógica de verificação deve implementar
type Handler interface {
	Skipped(in interface{}) bool // Aqui decide se deve pular
	HandleRequest(in interface{}) error // Aqui realiza várias verificações
}

// Nó da Cadeia de Responsabilidade
type RequestChain struct {
	Handler
	Next *RequestChain
}

// Definir handler
func (h *RequestChain) SetNextHandler(in *RequestChain) *RequestChain {
	h.Next = in
	return in
}
```

Sobre padrões de design, vocês podem ver o github do meu colega: https://github.com/TIGERB/easy-tips/tree/master/go/src/patterns

#### Carrinho de Compras

Tendo falado sobre antes de adicionar ao carrinho, agora vamos ver a parte do carrinho de compras. Discutimos anteriormente que o carrinho de compras pode ter várias formas, por exemplo: armazenar vários produtos para checkout juntos, checkout imediato de um determinado produto, etc. Portanto, o carrinho de compras certamente selecionará o tipo de carrinho com base no canal.

As operações desta parte são relativamente estáveis. Escolhemos algumas operações importantes para falar sobre a ideia.

##### Adicionar ao Carrinho de Compras

Ao colocar a verificação de condições como pré-requisito, descobrirá que ao realizar a operação de adicionar ao carrinho, essa parte da lógica tornou-se muito leve. O que precisa ser feito é principalmente a lógica das seguintes partes.

![Adicionar ao Carrinho de Compras](https://dayutalk.cn/img/cart-sys-04.jpg)

Existem alguns truques aqui. Primeiro é a lógica de obter o produto. Como também será usada na verificação anterior, aqui após a obtenção anterior, será passada adiante via parâmetros, então não há necessidade de ler o banco de dados ou chamar o serviço para obter novamente;

Em segundo lugar, aqui é necessário obter os dados do carrinho existente do usuário atual e, em seguida, adicionar este produto. Esta é uma operação semelhante a uma fusão (merge). Se o produto já existir, equivale a adicionar um na quantidade; é necessário observar se este produto tem relação pai-filho com produtos existentes, se é possível que a adição altere alguma regra de atividade, por exemplo: originalmente comprava 2 e ganhava 1 brinde, agora adicionando mais um vira 3 e ganha 2 brindes;

> Nota: A adição aqui não é necessariamente alterar a quantidade diretamente no carrinho, pode ser adicionar diretamente na lista ou página de detalhes.

Ao fundir os dados do carrinho e confirmar que está ok através da verificação de atividade de marketing, grave diretamente no armazenamento.

##### Fundir Carrinho de Compras

Por que existe a operação de fundir carrinhos? Porque geralmente o e-commerce permite operações como visitante, então quando o usuário faz login, é necessário fundir os dois.

A lógica de muitas partes da fusão aqui pode ser reutilizada com a lógica de adicionar ao carrinho. Por exemplo: os dados após a fusão precisam ser verificados quanto à legalidade e, em seguida, sobrescritos no armazenamento. Portanto, todos podem ver a correlação aqui. O método de design deve ser genérico até certo ponto.

##### Lista do Carrinho de Compras

A lista do carrinho de compras é uma interface muito importante. Em princípio, a interface do carrinho fornecerá dois tipos, uma versão simplificada e uma versão completa;

A interface de lista simplificada é usada principalmente para obter informações simples, como no canto superior direito da página inicial do PC; a versão completa é usada na lista do carrinho de compras.

Na implementação real, o carrinho de compras definitivamente não é tão simples quanto uma interface de leitura. Porque todos sabemos que, seja informações do produto ou informações da atividade, estão mudando constantemente. Portanto, cada interface de leitura deve verificar a legalidade dos dados no carrinho atual e, se encontrar inconsistência, deve sobrescrever os dados originais armazenados.

![Lista do Carrinho de Compras](https://dayutalk.cn/img/cart-sys-05.jpg)

Também existem algumas práticas que verificam a legalidade dos dados em cada interface. Sugiro que, por questões de desempenho, algumas interfaces podem relaxar a verificação adequadamente e realizar uma verificação completa ao obter a lista. Por exemplo, na interface de adição, verificarei apenas a legalidade do produto que adicionei, e nunca verificarei todo o carrinho. Porque após essa operação geralmente a operação de lista será chamada, e então a verificação será feita. As duas operações se repetem, então apenas a última é tomada.

#### Checkout

O checkout inclui duas partes: informações detalhadas da página de checkout e envio do pedido. A página de checkout pode ser considerada um "wrapper" sobre a lista do carrinho, pois a maior diferença entre a página de checkout e a página de lista é que o usuário precisa selecionar o endereço de entrega (produtos virtuais à parte). Neste momento, informações de preço mais explícitas serão geradas, o resto é basicamente o mesmo. Portanto, ao projetar a interface da lista do carrinho, deve-se considerar a generalidade suficiente.

Outra coisa a notar aqui é: Compra Imediata. Também implementaremos através da interface da página de checkout, mas internamente ainda chamará a interface de adição para adicionar o produto ao carrinho; há três lugares para prestar atenção. Primeiro, esta operação de adição é concluída dentro do serviço, e a parte que chama o serviço não precisa perceber a existência dessa operação de adição; segundo, a Key deste carrinho no Redis é independente do carrinho comum, caso contrário, os produtos dos dois acoplados juntos são muito difíceis de operar e processar; finalmente, o carrinho de compra imediata deve considerar que, quando a conta está logada em vários terminais, os dados não podem afetar uns aos outros. Aqui, o uuid de cada terminal pode ser usado como a marca do carrinho para evitar essa situação.

O último passo do carrinho de compras é gerar o pedido. O mais importante nesta etapa é bloquear o carrinho para evitar que os dados sejam adulterados durante o processo de envio. Apenas para acrescentar, muitos códigos de bloqueio distribuído Redis escritos por muitas pessoas têm defeitos, todos devem prestar atenção à questão da atomicidade. Existem muitos artigos desse tipo na rede, não vou repetir.

Após o bloqueio bem-sucedido, temos várias práticas aqui. Uma é organizar os dados de acordo com o design do DB e começar a gravar na tabela. Isso é adequado para situações onde o volume de negócios não é grande, por exemplo, o volume de pedidos não excede 2000K por segundo; E se o requisito de concorrência do seu sistema for muito alto?

Na verdade, também é muito simples, uma das três armas mágicas de alto desempenho: assincronia; quando enviamos, gravamos diretamente o snapshot dos dados no MQ e, em seguida, processamos o consumo de forma assíncrona. A capacidade de processamento pode ser melhorada controlando o número de consumidores. Embora este método melhore o desempenho, a complexidade também aumentará. Todos precisam escolher de acordo com sua situação real.

Sobre o design da arquitetura de negócios, paramos por aqui. Em seguida, vamos ver a arquitetura do sistema.

### Arquitetura de Sistema

A estrutura do sistema inclui principalmente como mapear a arquitetura de negócios, bem como a explicação dos parâmetros de entrada e saída correspondentes. Como a entrada e a saída são determinadas de acordo com seus respectivos negócios e não há dificuldade, aqui falaremos apenas sobre como mapear a arquitetura de negócios para a arquitetura do sistema, bem como a escolha da estrutura de dados Redis mais central na arquitetura do sistema e o design da estrutura de dados armazenada.

#### Estrutura do Código

O diretório de código abaixo é projetado de acordo com `Golang`. Vamos ver como mapear a arquitetura de negócios acima para o nível do código.

```golang
├── addproducts.go
├── cartlist.go
├── mergecart.go
├── entity
│   ├── cart
│   │   ├── add.go
│   │   ├── cart.go
│   │   └── list.go
│   ├── order
│   │   ├── checkout.go
│   │   ├── order.go
│   │   └── submit.go
│   └── precart
├── event
│   └── sendorder.go
├── facade
│   ├── activity.go
│   └── product.go
└── repo
```

Existem quatro diretórios na camada externa: `entity`, `event`, `facade`, `repo`. As responsabilidades são as seguintes:

**entity**: Armazena as três entidades do domínio de compras que analisamos anteriormente; todas as principais operações estão nessas três entidades;

**event**: Isso é usado para processar eventos gerados. Por exemplo, como acabamos de dizer, se enviarmos pedidos de forma assíncrona, este diretório deve completar como enviar dados para o MQ;

**facade**: Para que serve este diretório? Isso ocorre principalmente porque nosso serviço também precisa depender de serviços como produtos e atividades de marketing. Não devemos chamá-los diretamente na entidade, porque terceiros podem mudar, ou ter adições ou reduções. Fazemos o seguinte encapsulamento simples aqui (padrão Facade nos padrões de design);

**repo**: Este diretório pode ser entendido até certo ponto como a camada `Model`. Em todo o serviço de domínio, se lidar com persistência, é feito através dele.

Finalmente, os vários arquivos na camada externa são os serviços de domínio que fornecemos para a camada de aplicação chamar.

> Para garantir a compacidade do conteúdo, abandonei a introdução do diretório de todo o microsserviço aqui e apresentei apenas o serviço de domínio separadamente. Posteriormente, escreverei um artigo separado para apresentar toda a arquitetura do sistema de microsserviços.

Através da divisão acima, completamos duas coisas:

1. A estrutura da análise da arquitetura de negócios tem um mapeamento no código do sistema, eles se refletem mutuamente. O maior benefício disso é garantir a consistência entre design e código. Ao ler a documentação, você sabe onde está o código correspondente;

2. O foco de cada diretório é separado, mais coeso e mais fácil de desenvolver e manter.

#### Armazenamento Redis

Agora, escolhemos o Redis como armazenamento de dados de produtos de compras. Temos que resolver dois problemas: primeiro, quais dados precisamos armazenar? Segundo, qual estrutura usamos para armazenar?

Muitos carrinhos de compras na internet salvam apenas um id de produto. O cenário real dificilmente atende à demanda. Pense nisso, como um id de produto pode lembrar o brinde escolhido pelo usuário? A atividade escolhida pelo usuário da última vez? E o canal do produto comprado?

Comparando cenários gerais de forma abrangente, dou uma estrutura de referência:

```golang
// Dados do carrinho de compras
type ShoppingData struct {
	Item       []*Item `json:"item"`
	UpdateTime int64   `json:"update_time"`
	Version    int32   `json:"version"`
}

// Elemento de item de produto único
type Item struct {
	ItemId       string          `json:"item_id"`
	ParentItemId string          `json:"parent_item_id,omitempty"` // id do item pai vinculado
	OrderId      string          `json:"order_id,omitempty"`       // número do pedido vinculado
	Sku          int64           `json:"sku"`
	Spu          int64           `json:"spu"`
	Channel      string          `json:"channel"`
	Num          int32           `json:"num"`
	Status       int32           `json:"status"`
	TTL          int32           `json:"ttl"`                     // Tempo de validade
	SalePrice    float64         `json:"sale_price"`              // Registrar preço de venda ao adicionar ao carrinho
	SpecialPrice float64         `json:"special_price,omitempty"` // Preço especificado ao adicionar ao carrinho
	PostFree     bool            `json:"post_free,omitempty"`     // Se é frete grátis
	Activities   []*ItemActivity `json:"activities,omitempty"`    // Registro de atividades participadas
	AddTime      int64           `json:"add_time"`
	UpdateTime   int64           `json:"update_time"`
}

// Atividade
type ItemActivity struct {
	ActID    string `json:"act_id"`
	ActType  string `json:"act_type"`
	ActTitle string `json:"act_title"`
}
```

Vamos focar na estrutura `Item`. O campo `item_id` é uma marca única para marcar um determinado produto no carrinho, porque dissemos antes que o mesmo sku, devido a canais diferentes, será dois itens diferentes no carrinho; o próximo campo `parent_item_id` é usado para marcar o relacionamento pai-filho. Aqui, a estrutura de árvore que pode existir é convertida em uma estrutura sequencial. Independentemente de ser um produto pai ou um produto filho, usamos armazenamento sequencial e depois associamos através deste campo; alguns alunos podem achar estranho, por que salvar o campo order id? Preste atenção ao seu negócio diário, por exemplo: mais um pedido, pré-venda com depósito, esse tipo deve estar associado a um pedido, seja para verificação de qualificação ou estatísticas de dados. Os campos restantes são campos muito convencionais, não vou apresentá-los um por um;

> Tipo de campo, todos modificam de acordo com suas próprias necessidades.

Em seguida, devemos falar sobre como escolher a estrutura de armazenamento do Redis. As cinco comumente usadas no Redis são `Hash Table, Set, Sorted Set, List, String`. Vamos analisar uma por uma.

Primeiro, a compra de carro deve ter uma key para marcar a qual usuário este carrinho pertence. Para simplificar, assumimos que nossa key é: `uid:cart_type`.

Vamos ver se usamos `Hash Table`; ao adicionar, precisamos usar o seguinte comando: `HSET uid:cart_type sku ShoppingData`; parece que não há problema, podemos localizar rapidamente um produto com base no sku e depois fazer modificações relacionadas, etc. Mas note que ShoppingData é uma string json. Se houver muitos produtos no carrinho do usuário, a complexidade de tempo que usamos `HGETALL uid:cart_type` para obter é O(n), e então o código precisa ser desserializado um a um, o que é outra complexidade O(n).

Se usarmos `Set`, encontraremos problemas semelhantes. Cada carrinho é visto como um conjunto, e cada elemento no conjunto é ShoppingData. Ao obter no código, ainda precisa ser desserializado um a um (desserialização é custo). Sobre conjuntos ordenados e listas, não analisarei, todos podem seguir a ideia acima para tentar encontrar o problema.

Parece que não temos escolha a não ser usar `String`. Vamos ver como é a adequação de `String`. Primeiro `SET uid:cart_type ShoppingDataArr`; serializamos todos os dados do carrinho em uma string para armazenamento. A complexidade de tempo para retirá-la a cada vez é O(1), e a serialização e desserialização requerem apenas uma vez. Parece ser uma escolha muito boa. Mas todos ainda precisam prestar atenção a alguns pontos no uso.

1. Um único Value não pode ser muito grande, caso contrário, haverá problemas de big key, então geralmente o carrinho tem um limite superior, por exemplo, o item não pode exceder quantos;
2. O desempenho da operação no redis melhorou, mas a inconveniência do código é ao modificar um único item, deve-se ler tudo a cada vez e depois encontrar o item correspondente para modificar; aqui podemos construir uma Hash Table na memória após ler os dados do redis para reduzir a complexidade de cada travessia;

Também vi muitas combinações de estruturas de dados Redis na Internet para salvar dados do carrinho de compras, mas isso sem dúvida aumenta a sobrecarga da rede. Em comparação, String ainda é a mais econômica.

### Resumo

Até aqui, o design de implementação do carrinho de compras está concluído. O design da tabela de pedidos será colocado separadamente no módulo de pedidos.

Para todo o serviço de carrinho de compras, embora não esteja escrito detalhadamente para uma interface específica, acredito que, ao analisar até este passo, todos tenham um plano em mente e possam combiná-lo com seu próprio negócio para implementá-lo.

Existem alguns lugares interessantes no texto, sugiro que todos tentem fazer. Se houver algum problema, podemos nos comunicar a qualquer momento.

- Versão adaptada do padrão Chain of Responsibility
- Implementação de bloqueio de transação distribuída Redis
