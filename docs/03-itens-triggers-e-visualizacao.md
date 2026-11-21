#  Módulo 3: Itens, Triggers, Visualização e Alertas

Este módulo documenta a camada de telemetria, inteligência analítica e visualização nativa do **Zabbix 7.0 LTS**. Aqui, os dados coletados pelos agentes são estruturados em itens ativos, passivos e dependentes, classificados por **Tags**, avaliados por expressões de alerta (**Triggers**), consolidados em **Templates** reutilizáveis e exibidos em **Dashboards** e **Mapas Topológicos**.

---

##  Passo 1: Arquitetura de Coleta (Itens Passivos, Ativos e Dependentes)

Para otimizar o consumo de recursos nos servidores e estações monitoradas, a coleta foi dividida entre itens mestres (passivos no Linux e ativos no Windows sob CGNAT) e **Itens Dependentes** (*Dependent Items*), que calculam métricas derivadas diretamente no Zabbix Server sem gerar requisições extras na rede.

### 1. Catálogo de Telemetria — Linux (`HostCloud-A` e `HostCloud-B` — 21 Itens)
Configurados com coleta passiva (`Zabbix agent`) e intervalos otimizados entre `30s` (disponibilidade) e `15m` (integridade de arquivos):

| Categoria / Tag | Nome do Item | Chave (*Key*) | Tipo | Intervalo |
| :--- | :--- | :--- | :--- | :--- |
| `component: health` | **Zabbix Agent Ping** | `agent.ping` | Zabbix agent | `1m` |
| `component: system` | **System uptime** | `system.uptime` | Zabbix agent | `30s` |
| `component: system` | **Number of processes** | `proc.num` | Zabbix agent | `1m` |
| `component: cpu` | **Number of CPUs** | `system.cpu.num` | Zabbix agent | `1m` |
| `component: cpu` | **CPU (User) / CPU idle time** | `system.cpu.util[,user]` / `[,idle]` | Zabbix agent | `1m` |
| `component: cpu` | **CPU utilization** *(Derivado)* | `system.cpu.util` | **Dependent item** | — |
| `component: cpu` | **Load average (1m / 5m / 15m)** | `system.cpu.load[all,avg1..15]` | Zabbix agent | `1m` |
| `component: memory` | **Total / Available memory** | `vm.memory.size[total]` / `[available]` | Zabbix agent | `1m` |
| `component: memory` | **Available memory in %** | `vm.memory.size[pavailable]` | Zabbix agent | `1m` |
| `component: memory` | **Memory utilization** *(Derivado)* | `vm.memory.util` | **Dependent item** | — |
| `component: storage` | **Free Space (`/`)** | `vfs.fs.size[/,free]` | Zabbix agent | `1m` |
| `component: memory` | **Swap (Total / Free / Free %)** | `system.swap.size[,total..pfree]` | Zabbix agent | `1m` |
| `component: security` | **Checksum `/etc/passwd` (FIM)** | `vfs.file.cksum[/etc/passwd,sha256]` | Zabbix agent | `15m` |
| `component: security` | **Checksum `zabbix_agentd/2.conf`** | `vfs.file.cksum[/etc/zabbix/...,sha256]` | Zabbix agent | `15m` |

### 2. Catálogo de Telemetria — Windows (`DESKTOP-SGH3G6V` — 18 Itens)
Como a estação Windows opera atrás de **CGNAT**, todos os itens mestres foram parametrizados como **`Zabbix agent (active)`**, enviando dados ativamente para a porta `10051` do Zabbix Server:
- **Disponibilidade e Sistema:** `agent.ping` (`1m`), `system.uptime` (`30s`), `proc.num` (`1m`) e `system.cpu.num` (`1m`).
- **Processamento:** `system.cpu.util`, `system.cpu.util[,,avg1]`, `system.cpu.util[,,avg5]` e `system.cpu.util[,,avg15]`.
- **Memória e Paginação:** `vm.memory.size[total]`, `vm.memory.size[free]`, `vm.memory.size[available]`, `vm.memory.size[pavailable]`, `vm.memory.util` (*Dependent item*) e métricas de Swap (`system.swap.size`).
- **Armazenamento:** `vfs.fs.size[C:,free]` (`1m`).
- **Segurança (Event Log):** `eventlog[Security,,,,4625,,skip]` para detecção em tempo real de falhas de logon (*detalhado no [Módulo 4](./04-logs-web-e-preprocessamento.md)*).

---

## 🚨 Passo 2: Modelagem de Triggers Analíticas e Controle de Flapping

Com os dados chegando em **Monitoring > Latest data**, foram configuradas triggers nas métricas críticas para detecção proativa de indisponibilidade, gargalos de recursos e alterações de segurança:

| Escopo / Métrica | Trigger Configurada | Severidade | Lógica Operacional |
| :--- | :--- | :--- | :--- |
| **Conectividade** | **NODATA 60 seconds** (`agent.ping`) | `Disaster` | Dispara alerta crítico se o agente ficar 60 segundos sem comunicação. |
| **Armazenamento** | **Less than 5GB free** (`vfs.fs.size`) | `High` | Alerta quando o espaço livre em disco cai abaixo de 5 GB (`5000000000` bytes). |
| **CPU (Média Móvel)** | **High CPU Usage 75% for 2m / 5m** | `Warning` | Avalia médias móveis (`avg5` / `2m`) para evitar falsos positivos em picos curtos. |
| **Carga Linux** | **Load Average (1m / 5m / 15m)** | `Warning` / `Average` | Monitora saturação da fila de processos em relação ao `Number of CPUs`. |
| **Segurança /FIM** | **Falha no Login (Event ID 4625)** | `Average` | Alerta imediato sobre tentativas inválidas de autenticação no Windows. |

### Boas Práticas Aplicadas nas Triggers
- **Macros Dinâmicas:** Uso de `{ITEM.VALUE}` nos títulos de alerta para exibir o valor exato da leitura no momento do incidente.
- **Mitigação de *Alert Flapping*:** Ajuste de histerese na expressão de recuperação (*OK event generation*) e permissão de fechamento manual (*Manual close*) em eventos de segurança, evitando abertura e fechamento repetitivo de alertas.

---
## 📦 Passo 3: Gráficos Personalizados e Padronização em Templates

1. **Gráficos Customizados (`Graphs`):** Criação de gráficos combinando `Total Memory` vs. `Free Memory` e curvas de `CPU Utilization` (`1m`, `5m` e `15m`) com visualização das linhas de trigger (`Show triggers`).
2. **Templates Unificados (`Data collection > Templates`):** Consolidação dos itens passivos/ativos, itens dependentes, tags e triggers em templates padronizados separados por sistema operacional.
3. **Exportação como Código (`YAML`):** Templates exportados e versionados na pasta [`templates/`](../templates/) deste repositório para replicação imediata.

---
