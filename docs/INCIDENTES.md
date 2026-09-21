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

## 2026-09-21 — Scroll retornando ao fim durante prepend de histórico

### Causa confirmada

O fluxo de histórico tinha a restauração de âncora, mas ainda havia dois caminhos concorrentes que podiam chamar `scrollMessagesToBottom`: o efeito de mensagens e o `ResizeObserver` usavam `isNearMessagesBottom()` como autorização implícita. Além disso, o efeito de abertura da conversa era rearmado quando `messages.length` mudava. Durante um prepend, esses caminhos podiam vencer a restauração da âncora e deslocar a viewport para baixo; o `overflow-anchor` nativo também podia competir com a compensação manual.

### Correção aplicada

- `PREPEND_HISTORY` agora desativa `followLatest` e bloqueia o auto-scroll centralizado.
- A restauração usa a mensagem DOM âncora em múltiplos frames e suspende `overflow-anchor` somente enquanto o prepend está pendente.
- `ResizeObserver` só acompanha o final quando existe intenção explícita de seguir mensagens novas; proximidade do rodapé não é mais suficiente.
- O efeito de abertura depende da conversa, não da quantidade de mensagens carregadas.
- O botão de novas mensagens continua sendo a ação explícita para voltar ao fim.

### Validação

Na produção `.144`, uma conversa longa foi aberta e navegada manualmente por páginas sucessivas, observando as transições de 50 para 100, 150 e 200 mensagens. O histórico continuou sendo inserido acima, sem retorno automático ao fim ou tela branca; o bundle final renderizou a lista, a conversa e o composer.
