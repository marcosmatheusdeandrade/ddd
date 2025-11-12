## 🎯 Introdução ao DDD

**Domain-Driven Design (DDD)** é uma abordagem de desenvolvimento de software que coloca o **domínio de negócio** no centro da aplicação. Não é sobre tecnologia, frameworks ou padrões arquiteturais - é sobre **modelar o conhecimento do negócio em código**.

---

### Por que vale a pena DDD?

- **CURTO PRAZO:**
  - ❌ Mais código inicial
  - ❌ Curva de aprendizado
  - ❌ Mais abstrações


- **LONGO PRAZO:**
  - ✅ Código auto-documentado
  - ✅ Fácil de manter
  - ✅ Fácil de testar
  - ✅ Fácil de evoluir
  - ✅ Menos bugs
  - ✅ Time alinhado com negócio

---

❌ Abordagem Tradicional (Database-First):
```
┌─────────────┐
│  Database   │  ← Começa aqui
└──────┬──────┘
       │
┌──────▼──────┐
│   Models    │  ← Anêmicos (getters/setters)
└──────┬──────┘
       │
┌──────▼──────┐
│  Services   │  ← Toda a lógica aqui
└──────┬──────┘
       │
┌──────▼──────┐
│ Controllers │
└─────────────┘
```
✅ Abordagem DDD (Domain-First):
```
┌─────────────┐
│   Domínio   │  ← Começa aqui (regras de negócio)
└──────┬──────┘
       │
┌──────▼──────┐
│  Aplicação  │  ← Orquestração (use cases)
└──────┬──────┘
       │
┌──────▼──────┐
│Infraestrutura│ ← Detalhes técnicos
└─────────────┘

```


## 🧩 Conceitos Fundamentais

-----

## 🌪️ Eventos de Domínio

Os **eventos de domínio** são o ponto de partida natural em uma modelagem orientada a domínio.  
Eles representam **fatos significativos que aconteceram no negócio** e que o sistema precisa registrar ou reagir.

Exemplo:
> `PedidoCriado`, `PagamentoConfirmado`, `ProdutoEstoqueEsgotado`.

Esses eventos ajudam a conectar o time técnico e o time de negócio em torno do **comportamento real da empresa** — e não apenas de tabelas ou APIs.


### Event Storming

O **Event Storming** é uma **técnica colaborativa de descoberta de domínio**, não exclusiva do DDD, mas **altamente recomendada dentro dele**.  
Durante uma sessão de Event Storming, o time mapeia os **eventos de domínio** em um quadro visual, revelando:
- A sequência natural das operações do negócio;
- Os **comandos** que causam os eventos;
- Os **agregados** e **contextos delimitados** que emergem do fluxo.

> Em resumo: o Event Storming ajuda a **descobrir o modelo de domínio e os limites contextuais** antes mesmo de escrever código.

---

## 🗣️ Linguagem Ubíqua

A **linguagem ubíqua** é um dos pilares do DDD.  
É a **linguagem compartilhada entre o time de desenvolvimento e o time de negócio**, usada em **reuniões, código, documentação e testes**.

Características:
- Baseada nos **termos reais do negócio** (ex: “pedido”, “cliente”, “nota fiscal”, “cancelamento”);
- Deve ser **consistente dentro de cada contexto delimitado**;
- Reduz ambiguidades e melhora a comunicação entre todos os envolvidos.

Exemplo prático:
```java
pedido.confirmar();
pedido.cancelar();
pedido.aplicarDesconto();
```



---
### 🏢 Bounded Contexts (Contextos Delimitados)
Bounded Context é o limite onde um modelo de domínio é válido. Cada contexto tem seu próprio modelo, linguagem e regras.
Nossa estrutura:

```
/main/java/com/marcosmatheus/
│
├── pedidos/          ← Bounded Context: PEDIDOS
│   ├── dominio/
│   ├── aplicacao/
│   └── infraestrutura/
│
├── clientes/         ← Bounded Context: CLIENTES
│   ├── dominio/
│   └── infraestrutura/
│
├── produtos/         ← Bounded Context: PRODUTOS (Catálogo)
│   └── dominio/
│
└── compartilhado/    ← Shared Kernel
    ├── dominio/
    └── infraestrutura/


```

Por que separar?

❌ Sem Bounded Contexts (Modelo Único), classe "Cliente" tentando servir TODO o sistema
```java
public class Cliente {
    // Dados de cadastro
    private String nome;
    private Email email;
    private Cpf cpf;

    // Dados de pedidos (poluindo o modelo)
    private List<Pedido> pedidos;
    private BigDecimal totalGasto;

    // Dados de marketing (poluindo o modelo)
    private List<Campanha> campanhas;
    private boolean recebeNewsletter;

    // Dados financeiros (poluindo o modelo)
    private BigDecimal limiteCredito;
    private List<Fatura> faturas;

    // MUITA RESPONSABILIDADE! ❌
    // DIFÍCIL DE MANTER! ❌
}
```
✅ Com Bounded Contexts (Modelos Separados)


```java
public class Cliente {
    private ClienteId id;
    private String nome;
    private Email email;
    private Cpf cpf;
    private TipoCliente tipo;     // VIP, Regular, etc
    private StatusCliente status;  // Ativo, Inativo

    // Foco: Gerenciar dados cadastrais ✅
}

```
 - CONTEXTO: PEDIDOS (Vendas):
```java
package com.marcosmatheus.pedidos.dominio.pedido;


public class Pedido {
    private PedidoId id;
    private ClienteId clienteId;  // ✅ Apenas referência (não é o objeto Cliente)
    private List<ItemPedido> itens;
    private StatusPedido status;

    // Foco: Gerenciar processo de vendas ✅
    // NÃO precisa saber tudo sobre cliente!
}

```
Comunicação entre Contextos

```java
// pedidos/infraestrutura/integracao/ClienteGateway.
public interface ClienteGateway {
    ClienteDTO buscarCliente(String clienteId);
    boolean clienteEstaAtivo(String clienteId);
    boolean clienteEhVip(String clienteId);
}
```
✅ Anti-Corruption Layer
- Protege o contexto de Pedidos das mudanças no contexto de Clientes

🎯
Agregados e Raiz de Agregado
Agregado: Pedido + ItemPedido


```
┌─────────────────────────────────────────────┐
│           AGREGADO: PEDIDO                  │
│                                             │
│  ┌──────────────────────────────────┐       │
│  │   Pedido (Raiz do Agregado)      │       │
│  │                                  │       │
│  │  - PedidoId (identidade)         │       │
│  │  - clienteId                     │       │
│  │  - status                        │       │
│  │  - total                         │       │
│  │  - desconto                      │       │
│  │  - frete                         │       │
│  │                                  │       │
│  │  + adicionarItem() ← Único ponto │       │
│  │  + removerItem()   ← de entrada  │       │
│  │  + confirmar()                   │       │
│  │  + cancelar()                    │       │
│  └──────────────────────────────────┘       │
│           │                                 │
│           │ governa                         │
│           ▼                                 │
│  ┌──────────────────────────────────┐       │
│  │    ItemPedido (Entidade)         │       │
│  │                                  │       │
│  │  - ItemPedidoId (identidade)     │       │
│  │  - produtoId                     │       │
│  │  - nomeProduto                   │       │
│  │  - quantidade                    │       │
│  │  - precoUnitario                 │       │
│  │  - subtotal                      │       │
│  └──────────────────────────────────┘       │
│                                             │
└─────────────────────────────────────────────┘
```
✅ ItemPedido NÃO pode existir sem Pedido    
✅ ItemPedido só é acessado através do Pedido     
✅ Pedido garante consistência dos itens

Implementação:

```java
// pedidos/dominio/pedido/Pedido.
public class Pedido extends RaizAgregado<PedidoId> {

    private PedidoId id;
    private ClienteId clienteId;
    private List<ItemPedido> itens = new ArrayList<>();
    private StatusPedido status;
    private EnderecoEntrega enderecoEntrega;
    private Dinheiro subtotal;
    private Dinheiro desconto;
    private Dinheiro frete;
    private Dinheiro total;

    // ✅ Factory Method (padrão DDD)
    public static Pedido criar(String clienteId, EnderecoEntrega endereco) {
        PedidoId id = PedidoId.gerar();
        Pedido pedido = new Pedido(id, clienteId, endereco);

        // Gera evento
        pedido.adicionarEvento(new PedidoCriadoEvento(
            id.obterValor(),
            clienteId,
            LocalDateTime.now()
        ));

        return pedido;
    }

    // ✅ Método de negócio encapsula regras
    public void adicionarItem(ProdutoId produtoId,
                             String nomeProduto,
                             Quantidade quantidade,
                             Dinheiro precoUnitario) {
        
        if (this.status != StatusPedido.PENDENTE) {
            throw new IllegalStateException("Não é possível adicionar itens a pedidos " + status);
        }

        ItemPedido item = ItemPedido.criar(
            ItemPedidoId.gerar(),
            produtoId,
            nomeProduto,
            quantidade,
            precoUnitario
        );

        this.itens.add(item);

        recalcularSubtotal();

        adicionarEvento(new ItemAdicionadoAoPedidoEvento(
            this.id.obterValor(),
            item.obterIdentificador().obterValor(),
            produtoId.obterValor(),
            quantidade.getValor()
        ));
    }

    public List<ItemPedido> getItens() {
        return Collections.unmodifiableList(itens);
    }

    // ✅ Invariante: total sempre correto
    private void recalcularSubtotal() {
        this.subtotal = itens.stream()
            .map(ItemPedido::getSubtotal)
            .reduce(Dinheiro.zero(), Dinheiro::somar);
        recalcularTotal();
    }

    private void recalcularTotal() {
        this.total = this.subtotal
            .subtrair(this.desconto)
            .somar(this.frete);
    }
}

// pedidos/dominio/pedido/ItemPedido.
public class ItemPedido extends Entidade<ItemPedidoId> {

    private ItemPedidoId id;
    private ProdutoId produtoId;
    private String nomeProduto; 
    private Quantidade quantidade;
    private Dinheiro precoUnitario;
    private Dinheiro subtotal;

    // ✅ Factory Method
    public static ItemPedido criar(ItemPedidoId id,
                                   ProdutoId produtoId,
                                   String nomeProduto,
                                   Quantidade quantidade,
                                   Dinheiro precoUnitario) {

        ItemPedido item = new ItemPedido();
        item.id = id;
        item.produtoId = produtoId;
        item.nomeProduto = nomeProduto;
        item.quantidade = quantidade;
        item.precoUnitario = precoUnitario;
        item.subtotal = precoUnitario.multiplicarBigDecimal.valueOf(quantidade.getValor());

        return item;
    }

    @Override
    public ItemPedidoId obterIdentificador() {
        return id;
    }

    // ✅ Sem setters - imutável após criação
    public Dinheiro getSubtotal() {
        return subtotal;
    }
}
```
Por que ItemPedido não é Raiz?

❌ Acesso direto ao item:
```java
 - ItemPedido item = itemRepository.findById(itemId);
 - item.setQuantidade(10);
```
❌ Total do pedido NÃO foi atualizado!


✅ CORRETO - Através da raiz

- pedido.atualizarQuantidadeItem(itemId, Quantidade.de(10));

-  Total recalculado automaticamente

---

### **Entidade (Entity)**

```java
// compartilhado/dominio/Entidade.```java
public interface Entidade<ID extends Identificador<?>> {
    ID obterIdentificador();
}
```

O que é?

Objeto com identidade única que persiste ao longo do tempo.   

Exemplo no nosso sistema:

```java
public class ItemPedido extends Entidade<ItemPedidoId> {
    private ItemPedidoId id;          
    private ProdutoId produtoId;
    private String nomeProduto;
    private Quantidade quantidade;
    private Dinheiro precoUnitario;
}
```
---

### **Objeto de Valor (Value Object)**

```java
// compartilhado/dominio/ObjetoValor.
public interface ObjetoValor {
}
```

**O que é?**

✅ Objeto imutável definido apenas por seus atributos.      
✅ Não tem identidade própria.     
✅ Dois objetos são iguais se todos os atributos são iguais.

Exemplos no nosso sistema:
```java
@Getter
@EqualsAndHashCode
public class Dinheiro implements ObjetoValor {
    private final BigDecimal valor;  // ✅ Imutável (final)

    private Dinheiro(BigDecimal valor) {
        this.valor = valor.setScale(2, RoundingMode.HALF_UP);
    }

    // ✅ Retorna NOVO objeto (não modifica o atual)
    public Dinheiro somar(Dinheiro outro) {
        return new Dinheiro(this.valor.add(outro.valor));
    }
}


@Getter
@EqualsAndHashCode
public class Quantidade implements ObjetoValor {
    private final int valor;

    // Valida na construção
    private Quantidade(int valor) {
        if (valor <= 0) {
            throw new IllegalArgumentException("Quantidade deve ser > 0");
        }
        this.valor = valor;
    }
}
```

Por que usar Value Objects?

❌ Sem Value objects, Código Primitivo (Primitive Obsession)

```java
public class Pedido {
    private BigDecimal total;  // Pode ser negativo? Qual moeda?
    private int quantidade;    // Pode ser zero? Negativo?

    public void calcular() {
        // Validações espalhadas por todo código
        if (quantidade <= 0) throw new Exception();
        if (total.compareTo(BigDecimal.ZERO) < 0) throw new Exception();
    }
}
```

 ✅ Com Value Objects (Validação Centralizada)
```java

public class Pedido {
    private Dinheiro total;       // ✅ Sempre válido
    private Quantidade quantidade; // ✅ Sempre válido

    // Não precisa validar, já vem validado!
}
```

---
### Raiz de Agregado (Aggregate Root)

```java
public interface RaizAgregado<ID extends Identificador<?>> extends Entidade<ID> {

    List<EventoDominio> obterEventosNaoPublicados();
    
    void limparEventos();
}
```

O que é?

✅ Ponto de entrada único para um grupo de objetos relacionados.   
✅ Garante consistência transacional dentro do agregado.    
✅ Controla todas as modificações nas entidades filhas.

**Exemplo: Pedido (Raiz de Agregado)**

```java
public class Pedido extends RaizAgregado<PedidoId> {

    private PedidoId id;                      // Identidade
    private List<ItemPedido> itens;          // Entidades filhas
    private StatusPedido status;
    private Dinheiro total;

    // ÚNICO ponto de entrada para adicionar itens
    public void adicionarItem(ProdutoId produtoId, 
                             String nomeProduto,
                             Quantidade quantidade, 
                             Dinheiro precoUnitario) {

        // Regras de negócio CENTRALIZADAS
        validarPodeAdicionarItem();

        ItemPedido item = ItemPedido.criar(
            ItemPedidoId.gerar(),
            produtoId,
            nomeProduto,
            quantidade,
            precoUnitario
        );

        this.itens.add(item);
        recalcularTotal();  // Mantém consistência

        adicionarEvento(new ItemAdicionadoAoPedidoEvento(...));
    }

    public List<ItemPedido> getItens() {
        return Collections.unmodifiableList(itens);
    }
}
```

Por que utilizar Raiz de Agregado?

❌ SEM Raiz de Agregado 
 - ❌(Inconsistência) Ex: pedido.getItens().add(item);  // Adiciona item
 - ❌ Total NÃO foi recalculado!
 - ❌ Status pode estar errado! 
 - ❌ Evento não foi gerado!

✅ COM Raiz de Agregado 
 - ✅ (Consistência Garantida) Ex: pedido.adicionarItem(...);
 - ✅ Total recalculado automaticamente!
 - ✅ Status atualizado se necessário!
 - ✅ Evento gerado!


---

###  Domain Services
 - Lógica que envolve múltiplos agregados ou conceitos

```java
public interface ServicoPrecificacao {
    Dinheiro calcularDesconto(Pedido pedido);
    Dinheiro calcularFrete(Pedido pedido);
}

```

## 🏗️ Estrutura de Camadas
```
┌─────────────────────────────────────────────────────────┐
│                     APRESENTAÇÃO                        │
│                   (Controllers/API)                     │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                      APLICAÇÃO                          │
│              (Use Cases / Orquestração)                 │
│                                                         │
│  pedidos/aplicacao/pedido/                              │
│    └── CriarPedido.java                                 │
│    └── comando/CriarPedidoComando.java                  │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                      DOMÍNIO                            │
│              (Regras de Negócio PURAS)                  │
│                                                         │
│  pedidos/dominio/pedido/                                │
│    ├── Pedido.java (Raiz de Agregado)                   │
│    ├── ItemPedido.java (Entidade)                       │
│    ├── StatusPedido.java (Enum)                         │
│    ├── repository/PedidoRepository.java (Interface)     │
│    ├── service/ServicoPrecificacao.java (Interface)     │
│    └── evento/PedidoCriadoEvento.java                   │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                  INFRAESTRUTURA                         │
│            (Detalhes Técnicos / Frameworks)             │
│                                                         │
│  pedidos/infraestrutura/                                │
│    ├── persistencia/                                    │
│    │   ├── PedidoEntity.java (JPA)                      │
│    │   ├── PedidoRepositoryImpl.java                    │
│    │   └── PedidoJpaRepository.java                     │
│    └── service/                                         │
│        ├── ServicoPrecificacaoImpl.java                 │
│        └── ServicoValidacaoPedidoImpl.java              │
└─────────────────────────────────────────────────────────┘
```
### 1. Camada de Domínio
/dominio/pedido/
```
├── Pedido.java                    ← Raiz de Agregado
├── ItemPedido.java                ← Entidade
├── ItemPedidoId.java              ← Identificador
├── PedidoId.java                  ← Identificador
├── StatusPedido.java              ← Enum
│
├── repository/
│   ├── PedidoRepository.java      ← Interface (PORT)
│   └── ProdutoRepository.java     ← Interface (PORT)
│
├── service/
│   ├── ServiceDomainPedido.java   ← Interface
│   ├── ServicoPrecificacao.java   ← Interface
│   └── ServicoEstoque.java        ← Interface
│
└── evento/
├── PedidoCriadoEvento.
    ├── ItemAdicionadoAoPedidoEvento.
└── PedidoConfirmadoEvento.`
```
Regras:

✅ SEM dependências de frameworks (Spring, Hibernate, etc)   
✅ SEM anotações de infraestrutura (@Entity, @Autowired, etc)   
✅ APENAS lógica de negócio pura   
✅ Interfaces (Ports) que a infraestrutura implementa


## Exemplo:

✅ CORRETO - Domínio puro
```java
public class Pedido extends RaizAgregado<PedidoId> {
    private PedidoId id;
    private List<ItemPedido> itens;

    public void confirmar() {
        if (this.itens.isEmpty()) {
            throw new IllegalStateException("Pedido sem itens");
        }
        this.status = StatusPedido.CONFIRMADO;
    }
}
```

❌ ERRADO - Domínio acoplado à infraestrutura

```java
@Entity  // ❌ Anotação JPA no domínio!

public class Pedido {
    @Id
    @GeneratedValue
    private Long id;  // ❌ Tipo primitivo, não Value Object

    @Autowired  // ❌ Dependência do Spring!
    private EmailService emailService;
}
```

### 2. Camada de Aplicação
/aplicacao/pedido/
```
├── CriarPedido.java               ← Use Case
├── ConfirmarPedido.java           ← Use Case
├── CancelarPedido.java            ← Use Case
│
└── comando/
├── CriarPedidoComando.java    ← DTO de entrada
└── ConfirmarPedidoComando.```java
```
Responsabilidades:

✅ Orquestrar o fluxo de negócio  
✅ Coordenar chamadas entre agregados  
✅ Gerenciar transações   
✅ Publicar eventos de domínio

Exemplo:

```java
// pedidos/aplicacao/pedido/CriarPedido.```java
@Service
@RequiredArgsConstructor
public class CriarPedido {

    private final PedidoRepository pedidoRepository;
    private final ServiceDomainPedido servicoDominio;
    private final ServicoPrecificacao servicoPrecificacao;
    private final ServicoEstoque servicoEstoque;
    private final PublicadorEventos publicadorEventos;

    @Transactional
    public Pedido executar(CriarPedidoComando comando) {
        // Validações (serviços de domínio)
        servicoDominio.validarNovoPedido(comando.getClienteId());

        for (var item : comando.getItens()) {
            servicoDominio.validarDisponibilidadeProduto(
                item.getProdutoId(),
                item.getQuantidade()
            );
        }

        Pedido pedido = Pedido.criar(omando.getClienteId(), comando.getEnderecoEntrega());

        for (var itemDTO : comando.getItens()) {
            pedido.adicionarItem(
                ProdutoId.de(itemDTO.getProdutoId()),
                itemDTO.getNomeProduto(),
                Quantidade.de(itemDTO.getQuantidade()),
                Dinheiro.de(itemDTO.getPrecoUnitario())
            );
        }

        Dinheiro desconto = servicoPrecificacao.calcularDesconto(pedido);
        Dinheiro frete = servicoPrecificacao.calcularFrete(pedido);
        pedido.aplicarDesconto(desconto);
        pedido.aplicarFrete(frete);

        servicoEstoque.reservarEstoque(pedido);

        Pedido pedidoSalvo = pedidoRepository.salvar(pedido);

        pedido.obterEventosNaoPublicados().forEach(publicadorEventos::publicar);
        pedido.limparEventos();

        return pedidoSalvo;
    }
}
```

**Por que um Use Case por operação?**

✅ 1. Single Responsibility Principle   
✅ 2. Fácil de testar individualmente    
✅ 3. Fácil de entender o fluxo     
✅ 4. Permite controle fino de transações  
✅ 5. Facilita auditoria e logs


**❌ Se fosse tudo em um "PedidoService":**
```java
@Service

public class PedidoService {
    public void criar(...) { }
    public void confirmar(...) { }
    public void cancelar(...) { }
    public void atualizar(...) { }
    public void enviar(...) { }
    // ... 20 métodos
    // DIFÍCIL de manter! ❌
}
```

### 3. Camada de Infraestrutura
/infraestrutura/
```
├── persistencia/
│   ├── PedidoEntity.java              ← JPA Entity
│   ├── ItemPedidoEntity.java          ← JPA Entity
│   ├── PedidoJpaRepository.java       ← Spring Data
│   └── PedidoRepositoryImpl.java      ← Implementação 
│
└── service/
├── ServicoValidacaoPedidoImpl.java ← Implementação
├── ServicoPrecificacaoImpl.java    ← Implementação
└── ServicoEstoqueImpl.java         ← Implementação
```


🔄 Shared Kernel (Núcleo Compartilhado)
```
/
├── dominio/
│   ├── Entidade.java              ← Interface base
│   ├── RaizAgregado.java          ← Interface base
│   ├── ObjetoValor.java           ← Interface base
│   ├── Identificador.java         ← Interface base
│   │
│   └── eventos/
│       ├── EventoDominio.java     ← Interface
│       └── PublicadorEventos.java ← Interface
│
├── ClienteId.java                 ← Value Object compartilhado
├── ProdutoId.java                 ← Value Object compartilhado
├── Dinheiro.java                  ← Value Object compartilhado
├── Quantidade.java                ← Value Object compartilhado
├── Email.java                     ← Value Object compartilhado
├── Cpf.java                       ← Value Object compartilhado
├── Telefone.java                  ← Value Object compartilhado
└── EnderecoEntrega.java           ← Value Object compartilhado

```

Por que compartilhar?   
✅ Todos os contextos entendem "Dinheiro" da mesma forma


```java
package com.marcosmatheus.compartilhado;

public class Dinheiro implements ObjetoValor {
    private final BigDecimal valor;
    // Lógica de validação e operações
}
```


📡 Domain Events
Estrutura de Eventos:
/dominio/pedido/evento/
```
├── PedidoCriadoEvento.```java
├── PedidoConfirmadoEvento.```java
├── PedidoCanceladoEvento.```java
├── PedidoPagoEvento.```java
├── PedidoEnviadoEvento.```java
├── PedidoEntregueEvento.```java
├── ItemAdicionadoAoPedidoEvento.```java
└── ItemRemovidoDoPedidoEvento.```java
```
Exemplo de Evento:
```java
// pedidos/dominio/pedido/evento/PedidoCriadoEvento.
public record PedidoCriadoEvento(
    String pedidoId,
    String clienteId,
    BigDecimal valorTotal,
    LocalDateTime ocorridoEm
) implements EventoDominio {

    @Override
    public String tipoEvento() {
        return "PedidoCriado";
    }
}
```
Geração no Agregado:
```java
public class Pedido extends RaizAgregado<PedidoId> {

    private List<EventoDominio> eventosNaoPublicados = new ArrayList<>();

    public static Pedido criar(String clienteId, EnderecoEntrega endereco) {
        Pedido pedido = new Pedido(...);

        pedido.adicionarEvento(new PedidoCriadoEvento(
            pedido.id.obterValor(),
            clienteId,
            pedido.total.getValor(),
            LocalDateTime.now()
        ));

        return pedido;
    }

    private void adicionarEvento(EventoDominio evento) {
        this.eventosNaoPublicados.add(evento);
    }
}
```

Publicação no Use Case:

```java
@Service
public class CriarPedido {

    private final PublicadorEventos publicadorEventos;

    @Transactional
    public Pedido executar(CriarPedidoComando comando) {
        Pedido pedido = Pedido.criar(...);
        Pedido pedidoSalvo = pedidoRepository.salvar(pedido);
        
        pedido.obterEventosNaoPublicados()
              .forEach(publicadorEventos::publicar);

        pedido.limparEventos();

        return pedidoSalvo;
    }
}
```




