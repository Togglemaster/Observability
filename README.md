# Observability 📡

Stack de observabilidade do **ToggleMaster**. Este diretório contém configurações de monitoramento, logging, tracing e alertas para toda a plataforma.

## 🎯 Descrição do Serviço

O stack de observabilidade fornece visibilidade completa sobre a saúde, performance e comportamento de todos os serviços do ToggleMaster. Ele inclui:

1. **Datadog Agent** - Coleta de métricas, logs e traces
2. **Prometheus** - Scraping de métricas (opcional)
3. **Grafana** - Dashboards de visualização
4. **Alertas** - Notificações automáticas de problemas
5. **Tracing Distribuído** - Rastreamento de requisições entre serviços

**Função crítica:** Sem observabilidade, é impossível detectar e diagnosticar problemas em produção.

## 📦 Stack Técnico

- **APM:** Datadog
- **Métricas:** Prometheus (opcional), Datadog metrics
- **Visualização:** Grafana
- **Logs:** Datadog, CloudWatch (AWS)
- **Alertas:** Datadog, PagerDuty (integração)

## 🚀 Como Usar

### Pré-requisitos

- Conta Datadog ativa
- Chave de API Datadog
- Kubernetes cluster (ou Docker Compose para desenvolvimento)
- Helm (para deployment em K8s)

### Setup Local via Docker Compose

#### 1. Configure as Variáveis de Ambiente

Crie um arquivo `.env` na raiz do diretório:

```env
# Datadog
DATADOG_API_KEY=seu_api_key_aqui
DATADOG_AGENT_HOST=datadog-agent
DATADOG_AGENT_PORT=8126
DATADOG_ENV=development
DATADOG_SERVICE=togglemaster

# Prometheus (opcional)
PROMETHEUS_RETENTION=15d

# Grafana
GRAFANA_ADMIN_PASSWORD=seu_password_seguro
GRAFANA_DOMAIN=localhost:3000

# Logs
LOG_LEVEL=INFO
```

#### 2. Inicie o Stack de Observabilidade

```bash
# Com Docker Compose
docker-compose -f docker-compose.observability.yaml up -d

# Ou com Helm (K8s)
helm install observability ./helm -f helm/values.yaml
```

#### 3. Acesse os Dashboards

- **Grafana:** http://localhost:3000
  - Usuário: admin
  - Senha: (a que você definiu em GRAFANA_ADMIN_PASSWORD)

- **Datadog:** https://app.datadoghq.com
  - Login com sua conta

- **Prometheus:** http://localhost:9090 (se habilitado)

### Configuração do Datadog Agent

O Datadog Agent coleta dados de todos os serviços. Exemplo de configuração:

```yaml
# datadog-agent.yaml
agent:
  image: datadog/agent:latest
  environment:
    - DD_API_KEY=${DATADOG_API_KEY}
    - DD_AGENT_HOST=datadog-agent
    - DD_TRACE_ENABLED=true
    - DD_PROFILING_ENABLED=true
    - DD_LOGS_ENABLED=true
  ports:
    - "8126:8126/tcp"
    - "8125:8125/udp"
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock:ro
```

## 🔧 Variáveis de Ambiente

### Obrigatórias
| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| `DATADOG_API_KEY` | Chave API Datadog | `xxxxxxxxxxxxxxx...` |
| `DATADOG_SERVICE` | Nome do serviço | `togglemaster` |
| `DATADOG_ENV` | Ambiente (dev/staging/prod) | `development` |

### Opcionais
| Variável | Descrição | Padrão |
|----------|-----------|--------|
| `DATADOG_AGENT_HOST` | Host do Datadog Agent | `datadog-agent` |
| `DATADOG_AGENT_PORT` | Porta do Datadog Agent | `8126` |
| `DD_TRACE_ENABLED` | Habilita tracing | `true` |
| `DD_PROFILING_ENABLED` | Habilita profiling | `true` |
| `DD_LOGS_ENABLED` | Habilita coleta de logs | `true` |
| `LOG_LEVEL` | Nível de log | `INFO` |

## 🔐 GitHub Secrets Necessários

Configure os seguintes secrets no GitHub para CI/CD:

```yaml
DATADOG_API_KEY
  Descrição: Chave API Datadog
  Valor: <sua-api-key>

DATADOG_APP_KEY
  Descrição: Chave de aplicação Datadog
  Valor: <sua-app-key>

GRAFANA_ADMIN_PASSWORD
  Descrição: Senha admin do Grafana
  Valor: <senha-forte>

GRAFANA_DOMAIN
  Descrição: Domínio do Grafana
  Valor: grafana.example.com

PAGERDUTY_INTEGRATION_KEY
  Descrição: Chave de integração PagerDuty
  Valor: <sua-integration-key>

SLACK_WEBHOOK_URL
  Descrição: Webhook URL do Slack para alertas
  Valor: https://hooks.slack.com/services/YOUR/WEBHOOK/URL

PROMETHEUS_RETENTION
  Descrição: Retenção de dados Prometheus
  Valor: 15d

ELASTICSEARCH_PASSWORD
  Descrição: Senha do Elasticsearch (se usado)
  Valor: <seu-password>
```

## 📊 Dashboards Principais

### 1. Overview do ToggleMaster
- Status de todos os serviços
- Requisições por segundo
- Latência P95, P99
- Taxa de erro

### 2. Performance dos Serviços
- CPU/Memória por serviço
- Taxa de requisições
- Latência por endpoint
- Erros e exceções

### 3. Banco de Dados
- Conexões ativas
- Query latency
- Replicação lag
- Espaço utilizado

### 4. Cache (Redis)
- Taxa de hit/miss
- Memória utilizada
- Keys por database
- Latência de operações

### 5. AWS SQS
- Mensagens enfileiradas
- Taxa de processamento
- Dead letter queue
- Latência de entrega

### 6. Analytics
- Eventos processados
- Distribuição por flag
- Taxa de erro

## 🚨 Alertas Configurados

### Alertas Críticos (P1)
```
- Qualquer serviço down
- Taxa de erro > 5%
- Latência P95 > 5s
- Database replication lag > 30s
```

### Alertas Altos (P2)
```
- CPU > 80%
- Memória > 85%
- Latência P95 > 1s
- SQS queue depth > 10000
```

### Alertas Médios (P3)
```
- Cache hit rate < 70%
- Disk space > 80%
- Network I/O > 1Gbps
```

## 📈 Métricas Importantes

### Por Serviço

**Auth Service:**
- `auth.requests.total` - Total de requisições
- `auth.validation.success` - Validações bem-sucedidas
- `auth.validation.failed` - Validações falhadas
- `auth.latency.ms` - Latência

**Flag Service:**
- `flags.requests.total` - Total de requisições
- `flags.created` - Flags criadas
- `flags.updated` - Flags atualizadas
- `flags.deleted` - Flags deletadas

**Evaluation Service:**
- `evaluation.requests.total` - Total de avaliações
- `evaluation.cache.hits` - Cache hits
- `evaluation.cache.misses` - Cache misses
- `evaluation.latency.ms` - Latência

**Targeting Service:**
- `targeting.rules.total` - Total de regras
- `targeting.rules.updated` - Regras atualizadas
- `targeting.latency.ms` - Latência

**Analytics Service:**
- `analytics.events.processed` - Eventos processados
- `analytics.events.failed` - Eventos falhados
- `analytics.processing.latency.ms` - Latência

## 🔍 Boas Práticas de Observabilidade

### Instrumentação
1. Sempre use distributed tracing (Datadog APM)
2. Instrumente operações de banco de dados
3. Instrumente chamadas HTTP externas
4. Adicione custom tags a spans

### Logs
1. Use níveis apropriados (INFO, ERROR, DEBUG)
2. Inclua contexto (user_id, request_id, etc)
3. Estruture logs em JSON
4. Evite logs sensíveis (senhas, tokens)

### Métricas
1. Use histogramas para latência
2. Use gauges para estado
3. Use counters para eventos
4. Agregue por dimensões importantes

### Alertas
1. Evite falsos positivos
2. Configure escalações apropriadas
3. Valide alertas regularmente
4. Documente runbooks para cada alerta

## 🐛 Troubleshooting

### Problema: "Datadog Agent not connecting"
**Solução:** Verifique a chave API e conectividade de rede
```bash
docker logs datadog-agent
```

### Problema: "Grafana dashboard vazio"
**Solução:** Verifique se dados estão chegando do Prometheus
```bash
curl http://localhost:9090/api/v1/targets
```

### Problema: "Alertas não disparando"
**Solução:** Valide a condição e integração com PagerDuty/Slack

## 📚 Recursos Adicionais

- [Datadog Documentation](https://docs.datadoghq.com/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [ToggleMaster Architecture](../README.md)

## 🔗 Links Importantes

- Datadog Dashboard: https://app.datadoghq.com
- Grafana Local: http://localhost:3000
- Prometheus Local: http://localhost:9090
- PagerDuty: https://www.pagerduty.com

## 👥 Suporte

Para dúvidas ou problemas com observabilidade, abra uma issue no repositório ou entre em contato com o time DevOps.
