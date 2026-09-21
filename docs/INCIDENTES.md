# Incidentes

## 2026-09-21 — Queda e desconexões do realtime

### Sintoma

Login e chamadas da aplicação falharam durante uma queda do backend. O Nginx registrou upstream fechado e conexão recusada.

### Causa confirmada

O processo Node atingiu o limite de heap e encerrou. Durante o encerramento/reconexão, um `write EPIPE` em `server/presence.mjs` ficou sem tratamento e agravou a indisponibilidade.

### Evidências

- `FATAL ERROR: Reached heap limit Allocation failed - JavaScript heap out of memory`.
- `Error: write EPIPE` em `sendFrame`/`broadcastPresence`.
- Ausência de novos OOM após a recuperação observada.

### Correção aplicada

- Proteção de escrita e handlers de erro no WebSocket.
- Uma conexão de presença por aba.
- Reconexão com backoff para presença e SSE.
- Service Worker e bundle frontend versionados.

### Pendências conhecidas

- Mensagens de falha de Web Push e logout sem token devem ser tratadas em alteração separada, sem misturar com este incidente.

## 2026-09-21 — Sobrecarga por polling completo de conversas

### Sintoma

O frontend consultava `/api/internal/conversations` aproximadamente a cada 3 segundos, transferindo cerca de 944 KB com 193 conversas e 11.768 mensagens por ciclo.

### Causa confirmada

O endpoint usado para montar a lista lateral também devolvia o histórico completo de todas as conversas. Além disso, havia um segundo polling no Topbar para detectar mensagens novas. Isso pressionava rede, serialização do `database.json`, memória do Node e renderização React.

### Correção aplicada

- A lista passou a usar resumos de conversa, sem o array `messages`.
- O histórico passou a ser carregado sob demanda, em páginas de até 50 mensagens.
- O SSE `/api/internal/events` passou a transportar evento pequeno (`message.created`) com resumo, identificadores e prévia mínima.
- O polling rápido de mensagens foi removido; reconciliação da lista ficou espaçada e o SSE é o mecanismo principal.
- O histórico anterior pode ser carregado ao chegar ao topo da conversa.

### Validação

Após a publicação, uma resposta autenticada da lista observada no access log ficou na faixa de 54–68 KB, sem a carga de 944 KB. O serviço permaneceu ativo, o health local/público respondeu 200 e os assets novos retornaram 200.

## 2026-09-21 — Tela branca após otimização de conversas

### Causa real

O componente `App` avaliava uma dependência inexistente em um `useEffect`:

```jsx
}, [currentUser?.id, enabled]);
```

`enabled` não existia no escopo de `App`, provocando `ReferenceError: enabled is not defined` durante a montagem global do frontend. O backend e os assets continuavam respondendo normalmente, por isso health HTTP 200 não detectava o incidente.

### Correção

- Removida a dependência inválida, mantendo somente `currentUser?.id`.
- Adicionado Error Boundary global envolvendo o componente `App`.
- A falha agora apresenta mensagem amigável e botão `Recarregar Chat`, sem expor stack trace.
- Mantida a otimização de resumos, histórico sob demanda, paginação e SSE.

### Validação

Após a publicação, a interface renderizou visualmente no navegador e exibiu o formulário de login. O bundle novo retornou HTTP 200, o Service Worker foi versionado e o health local/público respondeu 200.
