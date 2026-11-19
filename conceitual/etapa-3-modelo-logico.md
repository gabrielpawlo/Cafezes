# Etapa 3 – Modelo lógico do CafezesBI

Mapeamento do diagrama ER (Etapa 2) para o modelo relacional.
Legenda: **PK** = chave primária, **FK** = chave estrangeira, `NN` = não nulo, `UQ` = único.

## Esquema relacional

```
Cliente        (cpf PK, pnome, unome, premium)

Funcionario    (id PK, pnome, unome, cpf UQ, funcao)

Fornecedor     (id PK, nome, prazo_entrega)

Produto        (codigo PK, nome, valor, qntd_estoque, estoque_min, validade)

Venda          (codigo_venda PK, data,
                cpf_cliente FK -> Cliente.cpf,
                id_funcionario FK -> Funcionario.id  NN)

Item_Venda     (codigo_venda PK/FK -> Venda.codigo_venda,
                codigo_produto PK/FK -> Produto.codigo,
                qntd, preco_praticado)

Pedido         (numero PK, data_pedido, status,
                id_fornecedor FK -> Fornecedor.id  NN)

Item_Pedido    (numero_pedido PK/FK -> Pedido.numero,
                codigo_produto PK/FK -> Produto.codigo,
                qntd_produto)
```

## Cardinalidades do ER e mapeamento

| Relacionamento | Entidade (mín,máx) | Entidade (mín,máx) | Mapeamento |
|---|---|---|---|
| Realiza | Cliente (0,n) | Venda (0,1) | `cpf_cliente` em Venda, aceita nulo |
| Registra | Funcionário (0,n) | Venda (1,1) | `id_funcionario` em Venda, NN |
| Contém | Venda (1,n) | Produto (0,n) | tabela `Item_Venda` (`qntd`, `preco_praticado`) |
| Inclui | Produto (0,n) | Pedido (1,n) | tabela `Item_Pedido` (`qntd_produto`) |
| Solicita | Fornecedor (0,n) | Pedido (1,1) | `id_fornecedor` em Pedido, NN |

Regra usada: relacionamento 1:N leva a chave estrangeira para o lado N; relacionamento N:N vira uma tabela própria, com chave primária composta pelas duas chaves e os atributos do relacionamento.

## Restrições de integridade

- `Funcionario.funcao` ∈ {`estoquista`, `vendedor`} (`CHECK`).
- `Funcionario.cpf` único e `Cliente.cpf` como chave primária (não repetir cadastros).
- `Produto.valor` > 0, `Produto.qntd_estoque` >= 0 e `Produto.estoque_min` >= 0 (`CHECK`).
- `Item_Venda.qntd` > 0, `Item_Venda.preco_praticado` > 0 e `Item_Pedido.qntd_produto` > 0 (`CHECK`).
- `Pedido.status` ∈ {`pendente`, `enviado`, `recebido`} (`CHECK`).

## Regras de negócio (a implementar com trigger ou transação)

- Não permitir venda de produto sem estoque suficiente (`qntd` > `qntd_estoque`).
- Ao registrar um item de venda, diminuir `qntd_estoque` do produto.
- Ao marcar um pedido como `recebido`, aumentar `qntd_estoque` com as quantidades de `Item_Pedido`.
- Alertar quando `qntd_estoque` < `estoque_min` e sugerir reposição com base na média de vendas.

## Observações

- `Produto.validade` guarda uma única data por produto. Se for necessário registrar uma validade para cada recebimento, a data passa a ser atributo de `Item_Pedido`.
- O ER atual não tem atributos de temporada. Para impor a regra "café de temporada só é vendido dentro do período", seria preciso acrescentar `inicio_temporada` e `fim_temporada` em `Produto`.
