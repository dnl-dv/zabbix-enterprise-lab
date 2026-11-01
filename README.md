# 🚀 Zabbix Enterprise Infrastructure & Monitoring Lab

> Documentação técnica de um laboratório *hands-on* de observabilidade corporativa implementado do zero. O projeto simula a estruturação de um ambiente de monitoramento resiliente e distribuído, cobrindo desde o provisionamento em nuvem até automações avançadas de segurança, descoberta de baixo nível (LLD), análise de logs de segurança e integrações com Grafana e API REST.

---

## 🛠️ Stack Tecnológica & Arquitetura
* **Sistema Operacional:** Ubuntu 26.04 Minimal (aarch64 / ARM64)
* **Nuvem / Infraestrutura:** Oracle Cloud Infrastructure (OCI)
* **Plataforma de Monitoramento:** Zabbix 7.0 LTS
* **Banco de Dados & Web:** MySQL & Apache
* **Segurança & Redes:** Certbot (SSL/Let's Encrypt), UFW / IPTables, Criptografia TLS/PSK
* **Visualização & Integrações:** Grafana OSS, Zabbix Global Dashboards, Network Maps e API REST

---

## 📂 Sumário e Módulos do Laboratório

### 🏁 Módulo 1: O Início — Fundação, Infraestrutura e Segurança de Perímetro
*Estabelecendo a base do servidor de monitoramento em nuvem com alta segurança.*
* **Provisionamento em Nuvem:** Migração de ambiente inicial para a **Oracle Cloud (OCI)** utilizando instâncias ARM64 com Ubuntu Minimal e acesso via SSH/PuTTY.
* **Deploy do Zabbix Core 7.0 LTS:** Instalação dos pacotes oficiais via APT, configuração do banco de dados MySQL, importação do esquema inicial e ajustes no arquivo `zabbix_server.conf`.
* **Frontend & Domínio:** Configuração inicial da interface web, aquisição de subdomínio corporativo (`zbx-lab.space` via Namecheap) e implementação de certificado **SSL/TLS com Certbot (Let's Encrypt)**.
* **Hardening de Perímetro:** Configuração de regras restritivas de firewall (UFW e IPTables) para controle rigoroso de portas e tráfego de rede.

### ⚙️ Módulo 2: O Meio (Parte 1) — Descoberta, Agentes Distribuídos e Automação
*Expandindo o monitoramento para múltiplos ambientes heterogêneos de forma automatizada e segura.*
* **Deploy de Agentes Distribuídos:** Instalação de agentes Zabbix em servidores Linux locais (`HostCloud-A`), nós externos em nuvem (`HostCloud-B` em CentOS) e ambientes Windows.
* **Auto Registration (Registro Automático):** Configuração de regras dinâmicas no servidor para descoberta e cadastro automático de novos hosts através de metadados.
* **Criptografia Ponta a Ponta (TLS/PSK):** Geração de chaves pré-compartilhadas de 256 bits via OpenSSL e blindagem total da comunicação entre servidor, proxies e agentes.
* **Ferramentas de Diagnóstico:** Utilização prática dos utilitários de linha de comando `zabbix_get` e `zabbix_sender` para testes e envio forçado de métricas.
* **Arquitetura Distribuída com Zabbix Proxy:** Instalação e configuração de um Zabbix Proxy (SQLite/Raspberry Pi) para gerenciamento de redes isoladas.

### 📈 Módulo 3: O Meio (Parte 2) — Observabilidade Ativa, LLD e Alertas
*Transformando métricas brutas em inteligência acionável e notificações eficientes.*
* **Métricas e Triggers Customizadas:** Criação de itens de host (Ping, espaço em disco, memória) e configuração de *Triggers* analíticas avançadas (`NODATA 60s`, limiares críticos de CPU e espaço).
* **Pré-processamento Avançado:** Manipulação de dados com Expressões Regulares (Regex), scripts em JavaScript e extração via JSONPath.
* **Monitoramento Especializado:** Leitura de logs de eventos do Windows (foco em auditoria de segurança — Event ID `4625` de falha de login), *Web Scenarios* e parâmetros de usuário customizados (`UserParameters`).
* **Baixo Nível de Descoberta (LLD):** Criação de regras automatizadas para descoberta de serviços do sistema operacional e métricas de infraestrutura.
* **Canais de Notificação:** Configuração de mídias de alerta via E-mail (Postfix), integração com mensageria via Webhook (Slack e Telegram) e estratégias para evitar *alert flapping*.

### 🎯 Módulo 4: O Fim / Nível Enterprise — Integrações Avançadas e Visão Executiva
*Consolidando o ambiente corporativo com topologias complexas, bancos de dados e painéis gerenciais.*
* **Monitoramento de Redes via SNMP:** Configuração de consultas SNMP (v2c e v3) com MIBs e OIDs, tradução via `snmpwalk2zabbix` e recepção de *SNMP Traps*.
* **Monitoramento de Bancos e Aplicações:** Configuração de templates nativos para instâncias MySQL, PostgreSQL e servidores de aplicação Java via JMX (Tomcat).
* **Dashboards Globais e Mapas:** Construção de painéis unificados de visualização (*Global Dashboards*) e criação de *Network Maps* topológicos interativos.
* **Grafana & API REST:** Integração do Zabbix ao Grafana OSS (via plugin oficial e conexão direta ao banco MySQL) e automação de tarefas utilizando a API REST do Zabbix com scripts Python.

---

## 💡 Como Navegar por este Repositório
Cada diretório e arquivo de documentação neste repositório contém o passo a passo detalhado, trechos de arquivos de configuração modificados (`.conf`), scripts de automação e evidências visuais para replicar este laboratório corporativo completo.
