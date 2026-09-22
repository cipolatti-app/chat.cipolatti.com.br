# Histórico de alterações

## 2026-09-21 — Estabilidade de conexões realtime

- Investigado crash do Node causado por `JavaScript heap out of memory` e `EPIPE` não tratado durante broadcast de presença.
- Criado backup de produção antes da intervenção.
- Consolidada a presença para evitar duas conexões WebSocket na mesma aba.
- Adicionado backoff de reconexão para WebSocket e SSE.
- Service Worker versionado como `20260921-realtime-connection-lifecycle-v1`.
- Frontend versionado como `2026.09.21.1`.
- Build validado e assets publicados na produção.

Detalhes operacionais e evidências permanecem em `docs/INCIDENTES.md`.

## 2026-09-21 — Conversas resumidas e histórico sob demanda

- Backend: `/api/internal/conversations` agora retorna somente resumo da conversa e última mensagem.
- Backend: adicionada leitura paginada em `/api/internal/conversations/:id/messages?limit=50`, com cursor `before`.
- SSE: eventos realtime deixaram de incluir a conversa completa; carregam `conversationSummary`, identificadores e a mensagem afetada em formato mínimo.
- Frontend: a conversa ativa busca as últimas 50 mensagens e carrega páginas anteriores ao rolar para o topo.
- Frontend: remoção do polling de mensagens de 8 segundos; fallback da lista passou para 120 segundos e notificações/contadores para 30 segundos.
- Service Worker: `20260921-conversation-summaries-sse-v1`.
- Frontend: `2026.09.21.2`.
- Build validado e assets publicados na produção `.144`.

## 2026-09-21 — Lazy loading real de mensagens

- A conversa ativa carrega somente as últimas 50 mensagens por `GET /api/internal/conversations/:id/messages?limit=50`.
- A rolagem para o topo usa o cursor `before` retornado pela API para carregar páginas adicionais de 50 mensagens, preservando a posição visual.
- Eventos SSE de mensagens acrescentam apenas a mensagem afetada na conversa ativa; outras conversas recebem somente resumo, prévia, horário e contador.
- O endpoint de leitura passou a retornar resumo e contadores, sem transportar novamente o histórico completo.
- O cache de históricos no frontend foi limitado às conversas recentes e a 250 mensagens por conversa, preservando a conversa ativa.
- Build e publicação validados na `.144`; bundle publicado: `index-BG42-xaf.js`.
- Service Worker publicado: `20260921-lazy-history-sse-append-v1`.

## 2026-09-21 — Correção estrutural do scroll ao inserir histórico

- O prepend de páginas antigas passou a bloquear explicitamente qualquer caminho genérico de auto-scroll durante `PREPEND_HISTORY`.
- O `ResizeObserver` e o efeito de mudança de mensagens deixaram de inferir autorização para ir ao final somente porque o usuário estava perto do rodapé.
- O efeito de abertura da conversa não é mais rearmado a cada mudança de quantidade de mensagens.
- O `overflow-anchor` nativo do container é suspenso somente durante o prepend, evitando disputa com a restauração da âncora DOM.
- A paginação de 50 mensagens, o cursor `before`, SSE e o armazenamento do histórico não foram alterados.
- Build e publicação validados na `.144`; bundle final: `index-BxG9d3gd.js`.
## 2026-09-22 — Reconciliação do estado de mensagens

- Corrigida uma condição de corrida em que a reconciliação de resumos substituía a lista inteira de conversas e removia do estado React o histórico já carregado.
- O carregamento inicial das últimas 50 mensagens agora faz merge por `messageId`, preservando mensagens recebidas via SSE enquanto a requisição estava pendente.
- A reconciliação de conversas preserva os históricos carregados localmente e continua usando a política de cache existente.
- Não houve alteração no banco, na persistência, no SSE, no cursor de paginação ou na quantidade de mensagens por página.
- Frontend: `2026.09.22.1`; Service Worker: `20260922-message-state-reconciliation-v1`.
