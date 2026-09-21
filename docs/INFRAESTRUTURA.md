# Infraestrutura

## Produção

- VM: `192.168.0.144`
- Serviço: `kalion-connect`
- Porta interna: `127.0.0.1:3002`
- Domínio: `chat.cipolatti.com.br`
- Assets publicados em: `/opt/kalion-connect/external-dist`
- Código ativo em: `/opt/kalion-connect/current`
- Health local: `http://127.0.0.1:3002/api/health`
- Health público: `https://chat.cipolatti.com.br/api/health`

## Rollback

A VM `192.168.0.148` deve permanecer intacta e com o backend parado. Não fazer merge automático de bancos entre as VMs.

## Backups

Backups locais ficam sob `/var/backups/kalion-connect`. Cópias externas, quando configuradas, devem usar exclusivamente o destino autorizado do NAS e nunca incluir credenciais no script.

## Observabilidade

Consultar:

- `journalctl -u kalion-connect`
- `/var/log/kalion-connect/backend.log`
- `/var/log/kalion-connect/backend-error.log`
- `/var/log/nginx/error.log`

Não registrar senhas, cookies, tokens, endpoints completos de push ou conteúdo de mensagens.
