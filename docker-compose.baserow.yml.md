# Links

https://baserow.io/docs/installation/install-using-standalone-images

https://baserow.io/docs/installation/configuration

https://github.com/baserow/baserow/blob/develop/docker-compose.yml

# Baserow Celery Workers - Resumo

## baserow-celery (Worker Principal)
**Status: OBRIGATÓRIO**

- Processa tarefas assíncronas gerais do Baserow
- Exemplos: criação em massa, duplicação de tabelas, webhooks, notificações
- Executa também as tarefas agendadas pelo celery-beat
- Sem ele, operações básicas falham

---

## baserow-celery-export-worker
**Status: OPCIONAL (recomendado se usar exportações)**

- Processa **exclusivamente** exportações de tabelas (CSV, JSON, XML)
- Worker dedicado para não sobrecarregar o worker principal
- **Importante:** Tarefas de exportação vão para uma fila separada (`export`)
  - Se este worker não estiver ativo, exportações ficam **pendentes** (não são processadas pelo worker principal)
  - Não há fallback automático para o celery principal

**Quando usar:**
- ✅ Se você ou seus usuários fazem exportações de dados
- ❌ Pode remover se nunca exporta dados

---

## baserow-celery-beat-worker
**Status: OPCIONAL (apenas para tarefas agendadas)**

- **Função:** Agenda tarefas periódicas (funciona como um cron interno)
- **NÃO executa** as tarefas, apenas coloca na fila no horário correto
- Quem executa: `baserow-celery` (worker principal)

**Tarefas agendadas típicas:**
- Limpeza de arquivos temporários antigos
- Envio de notificações periódicas
- Sincronização de dados externos (se configurado)
- Manutenção automática do sistema

**Quando usar:**
- ✅ Se precisa de automações com agendamento
- ✅ Se quer limpeza automática de arquivos temporários
- ❌ Instalação básica sem necessidades de agendamento

---

## Resumo Visual

```
┌──────────────────────────┐
│ baserow-celery           │ → Executa tudo (obrigatório)
├──────────────────────────┤
│ celery-export-worker     │ → Processa exportações (opcional)
├──────────────────────────┤
│ celery-beat-worker       │ → Agenda tarefas (opcional)
└──────────────────────────┘
```

## Configurações Recomendadas

### Instalação Mínima
```yaml
✅ baserow-celery
❌ baserow-celery-export-worker
❌ baserow-celery-beat-worker
```

### Instalação Completa (Produção)
```yaml
✅ baserow-celery
✅ baserow-celery-export-worker
✅ baserow-celery-beat-worker
```