# Orientações para agentes

## Escopo

- Produção oficial: `192.168.0.144`.
- Domínio oficial: `https://chat.cipolatti.com.br`.
- Backend local esperado: `127.0.0.1:3002`.
- A VM `192.168.0.148` é somente rollback e não deve ser reativada sem autorização explícita.

## Segurança operacional

### Regra máxima

A continuidade do Chat | Cipolatti em produção tem prioridade sobre GitHub,
sincronização, documentação, atualização ou melhoria. A produção funcional é
a referência de comportamento e não deve ser usada como ambiente de merge,
build ou teste.

- `VERIFIQUE` significa somente diagnóstico: não alterar produção, reiniciar
  serviço ou publicar.
- `PODE CORRIGIR` autoriza alteração controlada seguindo esta sequência:
  consultar GitHub, comparar estados, criar backup, corrigir, testar,
  publicar, validar, documentar, commit e push.
- GitHub não é mecanismo automático de deploy.
- Nunca executar `git pull` cegamente na `.144`.
- Antes de qualquer trabalho, consultar este arquivo, a documentação e os
  commits recentes para descobrir alterações feitas por Alexandre ou Wesley.

- Criar backup antes de alterações de produção.
- Não versionar banco, mensagens, uploads, backups, credenciais, tokens VAPID ou subscriptions.
- Não alterar NAS, `.148` ou dados persistentes fora do escopo solicitado.
- Diagnosticar antes de reiniciar ou publicar.
- Validar build, serviço e health após mudanças.

## Publicação

1. Auditar o estado atual.
2. Criar backup identificável por data/hora.
3. Alterar somente os arquivos necessários.
4. Executar validações estáticas e `npm run build` quando houver frontend.
5. Publicar assets em `/opt/kalion-connect/external-dist` quando aplicável.
6. Validar `kalion-connect`, health local e health público.
7. Registrar a alteração em `docs/HISTORICO-ALTERACOES.md` e incidentes em `docs/INCIDENTES.md`.
