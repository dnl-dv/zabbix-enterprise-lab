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

### 2. Catálogo de Telemetria — Windows (`WIN-WORKSTATION-01` — 18 Itens)
Como a estação Windows opera atrás de **CGNAT**, todos os itens mestres foram parametrizados como **`Zabbix agent (active)`**, enviando dados ativamente para a porta `10051` do Zabbix Server:
- **Disponibilidade e Sistema:** `agent.ping` (`1m`), `system.uptime` (`30s`), `proc.num` (`1m`) e `system.cpu.num` (`1m`).
- **Processamento:** `system.cpu.util`, `system.cpu.util[,,avg1]`, `system.cpu.util[,,avg5]` e `system.cpu.util[,,avg15]`.
- **Memória e Paginação:** `vm.memory.size[total]`, `vm.memory.size[free]`, `vm.memory.size[available]`, `vm.memory.size[pavailable]`, `vm.memory.util` (*Dependent item*) e métricas de Swap (`system.swap.size`).
- **Armazenamento:** `vfs.fs.size[C:,free]` (`1m`).
- **Segurança (Event Log):** `eventlog[Security,,,,4625,,skip]` para detecção em tempo real de falhas de logon (*detalhado no [Módulo 4](./04-logs-web-e-preprocessamento.md)*).

---

