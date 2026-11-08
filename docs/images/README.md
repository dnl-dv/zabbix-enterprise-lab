## 🖼️ Galeria de Evidências (NOC & Grafana)

### 1. Topologia de Rede e Conectividade (Zabbix Map)
Mapeamento ativo na Oracle Cloud (OCI) detalhando IPs públicos/privados, regras de Firewall/Portas e agentes operando em modo passivo e ativo (contornando CGNAT):
![Mapa de Topologia Zabbix](zabbix-network-map.png)

### 2. Disponibilidade (SLA) e Uptime
![Disponibilidade](grafana-system-uptime-(tempo-de-atividade).png)
![SLA](grafana-status-atual-(zabbix-agent-ping).png)

### 3. Desempenho de Processamento, Memória e Disco (I/O)
![CPU](grafana-utilização-geral-de-cpu(%25).png)
![Enfileiramento de tarefas no Processador](grafana-load-avarage-(1m%2C5m%2C15m).png)
![Memória e Swap](grafana-memória-e-swap-disponivel.png)
![Armazenamento](grafana-espaço-livre-nos-discos-(lld).png)
![Disco](grafana-desempenho-de-disco-(leitura-escrita)(lld).png)

### 4. Tráfego de Rede e Auditoria de Segurança (FIM & Logon)
Monitoramento de integridade de arquivos críticos (`/etc/passwd` e arquivos `.conf` via checksum) e auditoria de falhas de login:
![Tráfego de Rede](grafana-tráfego-de-rede-(in-out)-(lld).png)
![HostCloud-B](grafana-fim-integridade-de-arquivos-crfticos-hostcloud-b.png)
![HostCloud-A](grafana-fim-integridade-de-arquivos-cnticos-hostcloud-a.png)
![Zabbix server](grafana-fim-integridade-de-arquivos-cnticos-zabbix-server.png)
![Auditoria Falha no login](grafana-auditoria-falhas-de-login.png)
