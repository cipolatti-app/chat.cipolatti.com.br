# Contexto do projeto

O Chat | Cipolatti é uma aplicação web/PWA com frontend Vite/React, backend Node.js e persistência local controlada pelo serviço `kalion-connect`.

## Ambientes

| Ambiente | Função | Estado esperado |
|---|---|---|
| `192.168.0.144` | Produção oficial | Ativa |
| `192.168.0.148` | Rollback preservado | `kalion-connect` parado |

O endereço oficial é `https://chat.cipolatti.com.br`.

## Integrações relevantes

- Active Directory/LDAP para autenticação.
- WebSocket para presença.
- SSE em `/api/internal/events` para eventos de conversas.
- Web Push/FCM para notificações em background.
- Nginx/OpenResty como proxy e servidor de assets.

Segredos e dados de usuários devem permanecer fora do repositório.
