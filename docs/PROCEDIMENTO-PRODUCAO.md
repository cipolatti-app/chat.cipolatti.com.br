# Procedimento de produção

## Antes da alteração

1. Confirmar o host e o escopo da mudança.
2. Verificar `systemctl status kalion-connect` e os health checks.
3. Criar backup dos arquivos/configurações envolvidos.
4. Confirmar que nenhum segredo ou dado de produção será incluído no repositório.

## Alteração e validação

1. Executar `node --check` nos módulos Node alterados.
2. Executar `npm run build` para alterações frontend.
3. Publicar os assets no diretório servido pelo Nginx.
4. Reiniciar somente o serviço necessário.
5. Confirmar `systemctl is-active kalion-connect`.
6. Testar health local e público.
7. Verificar logs do backend e Nginx após a publicação.

## Realtime

Validar WebSocket de presença e SSE de eventos. Uma aba deve manter uma conexão de cada tipo. Reconexões devem usar backoff e limpar timers/conexões anteriores.

## Rollback

Em falha crítica, interromper novas alterações, preservar evidências e restaurar somente a versão respaldada. Não reativar a VM antiga automaticamente se a nova produção já tiver recebido gravações reais.

## Relatório

Registrar causa, arquivos alterados, backup, build, serviço, health, testes funcionais e limitações da validação.
