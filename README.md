# Live coding: Dev Backend Sênior

Obrigado por participar! Este exercício dura cerca de **60 minutos** e não é uma prova com resposta certa.
Queremos ver **como você pensa**: como entende o problema, que perguntas faz, como divide o trabalho e
como organiza o código.

## Como vai funcionar

| Etapa | Tempo | O que acontece |
|---|---|---|
| 1. Entender o problema | ~10 min | Você lê o cenário, faz perguntas e conta como pretende resolver |
| 2. Código | ~35 min | Você implementa a solução, pensando em voz alta |
| 3. Conversa | ~15 min | Discutimos o que você fez e como a solução se comportaria em produção |

**Regras do jogo**

- Use a linguagem em que você se sente mais confortável. PHP é bem-vindo, mas não é obrigatório.
- Não precisa de framework, banco de dados nem servidor. Código simples rodando no terminal ou em testes basta.
- Pode pesquisar e pode usar IA (Copilot, Claude, ChatGPT etc.). Só pedimos que você **diga em voz alta
  o que pediu e por que aceitou ou mudou o que ela sugeriu**.
- Não precisa terminar tudo. Preferimos uma parte bem feita e bem explicada a tudo pela metade.
- Faça perguntas sempre que quiser. Elas fazem parte da avaliação.

---

## O cenário

Somos uma plataforma de entregas no mesmo dia. Lojas vendem em marketplaces, e o marketplace nos avisa
de cada mudança no pedido por meio de **eventos** (webhooks).

Cada evento tem este formato:

```json
{
  "event_id": "evt-001",
  "order_id": "PED-100",
  "type": "created",
  "occurred_at": "2026-11-26T09:00:00Z",
  "payload": { "store": "Loja A", "value": 150.00 }
}
```

Os tipos possíveis, na ordem natural da vida de um pedido, são:

```
created → ready_to_ship → shipped → delivered
                                  ↘ cancelled (pode acontecer em qualquer ponto antes de delivered)
```

Quando um pedido chega em `delivered`, precisamos **registrar o pagamento do motorista** chamando um
serviço externo:

```
PaymentGateway::pay(string $orderId, float $amount): void
```

O valor pago ao motorista é **10% do valor do pedido**.

### O problema

O marketplace não é perfeito. Na prática, os eventos chegam:

1. **Repetidos**: o mesmo `event_id` pode chegar mais de uma vez.
2. **Fora de ordem**: um `shipped` pode chegar antes do `ready_to_ship`.
3. **Com falhas do outro lado**: o `PaymentGateway` às vezes demora ou responde com erro.

E já aconteceu de **um motorista receber o mesmo pagamento duas vezes**.

---

## O que queremos que você construa

Um processador que recebe uma lista de eventos e:

1. Mantém o **status atual correto** de cada pedido.
2. **Nunca paga o mesmo pedido duas vezes**, mesmo com eventos repetidos ou com o processador rodando de novo.
3. **Não perde pagamento** quando o `PaymentGateway` falha.
4. Deixa claro o que aconteceu com cada evento: aplicado, ignorado (e por quê) ou com erro.

### Dados de exemplo

```json
[
  { "event_id": "evt-001", "order_id": "PED-100", "type": "created",       "occurred_at": "2026-11-26T09:00:00Z", "payload": { "store": "Loja A", "value": 150.00 } },
  { "event_id": "evt-002", "order_id": "PED-100", "type": "ready_to_ship", "occurred_at": "2026-11-26T09:10:00Z", "payload": {} },
  { "event_id": "evt-004", "order_id": "PED-100", "type": "delivered",     "occurred_at": "2026-11-26T11:00:00Z", "payload": {} },
  { "event_id": "evt-003", "order_id": "PED-100", "type": "shipped",       "occurred_at": "2026-11-26T09:30:00Z", "payload": {} },
  { "event_id": "evt-004", "order_id": "PED-100", "type": "delivered",     "occurred_at": "2026-11-26T11:00:00Z", "payload": {} },

  { "event_id": "evt-010", "order_id": "PED-200", "type": "created",       "occurred_at": "2026-11-26T10:00:00Z", "payload": { "store": "Loja B", "value": 80.00 } },
  { "event_id": "evt-011", "order_id": "PED-200", "type": "cancelled",     "occurred_at": "2026-11-26T10:05:00Z", "payload": {} },
  { "event_id": "evt-012", "order_id": "PED-200", "type": "shipped",       "occurred_at": "2026-11-26T10:20:00Z", "payload": {} },

  { "event_id": "evt-020", "order_id": "PED-300", "type": "delivered",     "occurred_at": "2026-11-26T12:00:00Z", "payload": {} },
  { "event_id": "evt-021", "order_id": "PED-300", "type": "created",       "occurred_at": "2026-11-26T08:00:00Z", "payload": { "store": "Loja C", "value": 200.00 } }
]
```

Para simular o gateway, você pode usar uma versão falsa que **falha na primeira chamada** de cada pedido
e funciona da segunda em diante.

### Resultado esperado (para você conferir)

| Pedido | Status final | Pagamento ao motorista |
|---|---|---|
| PED-100 | `delivered` | R$ 15,00, **uma única vez** |
| PED-200 | `cancelled` | nenhum |
| PED-300 | `delivered` | R$ 20,00, **uma única vez** |

Repare que o PED-300 recebe o `delivered` **antes** do `created`. Decidir o que fazer nesse caso faz parte
do exercício. Conte para a gente qual caminho você escolheu e por quê.

---

## Na conversa final, vamos falar sobre

Não precisa preparar nada. São só os temas:

- Se em vez de 10 eventos fossem **2 milhões por dia**, o que mudaria na sua solução? Onde ela quebraria primeiro?
- Se isso rodasse com **vários processos ao mesmo tempo**, o que poderia dar errado? Como você evitaria?
- Como você guardaria esses dados num banco relacional? Que tabelas e índices criaria?
- Se um motorista reclamasse que **não recebeu**, como você investigaria?
- Como você testaria essa solução antes de ir para produção?

---

Boa sorte e, principalmente, fique à vontade!
