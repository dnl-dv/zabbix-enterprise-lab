# 📋 Templates (Zabbix) e Dashboards (Grafana)

Este diretório centraliza a inteligência de coleta e visualização do laboratório, permitindo a restauração imediata do ambiente com arquivos estruturados.

## 📦 Arquivos Disponíveis

- **`zbx_export_templates.yaml`**: Template do Zabbix contendo os itens, triggers de alerta e parametrizações de coleta criadas para os hosts Linux e Windows.
- **`Zabbix-lab-GrafanaDashboards.json`**: Estrutura completa do painel executivo criado no Grafana, já mapeado para consumir as métricas geradas pelo Zabbix.

## 📥 Como Importar

### No Zabbix (YAML)
1. Acesse o menu **Data collection > Templates**.
2. Clique em **Import** (canto superior direito).
3. Selecione o arquivo `.yaml` e confirme.

### No Grafana (JSON)
1. Acesse o menu lateral **Dashboards**.
2. Clique em **New** e depois em **Import**.
3. Faça o upload do arquivo `.json` ou cole seu conteúdo.
4. Vincule o dashboard ao seu *Data Source* do Zabbix e salve.
