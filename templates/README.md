# 📋 Templates, Mapas e Dashboards

Este diretório centraliza a inteligência de coleta, topologia de rede e visualização do laboratório, permitindo a restauração imediata do ambiente.

## 📦 Arquivos Disponíveis

- **`Meus-Templates-Linux-by-Zabbix-agent.yaml`** e **`Meus-Templates-Windows-by-Zabbix-agent-Active.yaml`**: Templates do Zabbix contendo os itens, triggers de alerta e parametrizações de coleta ativas para os hosts Linux e Windows, que já estão enviando dados ao Grafana. *(Nota: As regras de LLD - Low-Level Discovery - estão estruturadas, mas aguardam implementação e geração de métricas nas próximas etapas do laboratório).*
- **`Zabbix-Topology-Map2.yaml`**: Mapa nativo do Zabbix detalhando a topologia da infraestrutura. Contém informações visuais e lógicas sobre IPs, tipos de agentes, regras de firewall e conectividade geral entre os nós.
- **`Zabbix-lab-GrafanaDashboards`**: Estrutura completa do painel executivo criado no Grafana, já mapeado para consumir as métricas geradas pelo Zabbix.

## 📥 Como Importar

### No Zabbix
1. **Para Templates:** Acesse o menu **Data collection > Templates**. Clique em **Import** no canto superior direito, selecione o arquivo `.yaml` e confirme.
2. **Para o Mapa de Rede:** Acesse o menu **Monitoring > Maps**. Clique em **Import**, selecione o arquivo do mapa (`.yaml`) e confirme.

### No Grafana
1. Acesse o menu lateral **Dashboards**.
2. Clique em **New** e depois em **Import**.
3. Faça o upload do arquivo `grafana_dashboard.json`.
4. Vincule o dashboard ao seu *Data Source* do Zabbix e salve.
