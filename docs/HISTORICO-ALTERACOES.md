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
