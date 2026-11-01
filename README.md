# 🛡️ Enterprise NOC & Observability Lab

![Zabbix](https://img.shields.io/badge/Zabbix-7.0_LTS-red?logo=zabbix&logoColor=white)
![Oracle Cloud](https://img.shields.io/badge/OCI-ARM64_Flex-F80000?logo=oracle&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-26.04_Minimal-E95420?logo=ubuntu&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?logo=mysql&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-OSS-orange?logo=grafana&logoColor=white)
![Security](https://img.shields.io/badge/TLS%2FPSK-256--bit-green)

Ambiente completo de **Network Operations Center (NOC)** e observabilidade corporativa, projetado e implantado do zero utilizando **Zabbix 7.0 LTS** na nuvem da Oracle Cloud Infrastructure (OCI). O projeto simula a gestão de infraestruturas heterogêneas (Linux e Windows), implementação de segurança de transporte com chaves pré-compartilhadas (TLS/PSK), automação de descoberta por metadados (*Auto Registration*), auditoria de segurança em logs de eventos e visualização avançada com Grafana.

---

## 📌 Sumário
- [Visão Geral da Arquitetura](#-visão-geral-da-arquitetura)
- [Tecnologias Utilizadas](#-tecnologias-utilizadas)
- [Fases de Implementação](#-fases-de-implementação)
- [Simulação de Incidentes (Engenharia de Resiliência)](#-simulação-de-incidentes-engenharia-de-resiliência)
- [Estrutura do Repositório](#-estrutura-do-repositório)
- [Como Reproduzir o Lab](#-como-reproduzir-o-lab)

---

## 🏗 Visão Geral da Arquitetura

O ambiente de monitoramento foi desenhado com topologia centralizada resiliente em arquitetura ARM64 na Oracle Cloud Infrastructure (OCI). A coleta de telemetria entre os ativos distribuídos e o servidor central opera sob dois modelos:

1. **Checagens Passivas e Híbridas Criptografadas:** O Zabbix Server consulta métricas nos nós de rede privada por meio de sessões TCP seguras na porta `10050`.
2. **Checagens Ativas com Auto Registration:** Agentes instalados em redes externas ou atrás de CGNAT conectam-se proativamente à porta `10051` do servidor central, autenticando-se via chaves pré-compartilhadas (PSK de 256 bits) e provisionando-se automaticamente com base na diretiva `HostMetadata`.

A interface web administrativa é protegida por certificados SSL/TLS (Let's Encrypt / Certbot) sob o domínio corporativo `zbx-lab.space`, com regras restritivas de firewall (UFW/IPTables e Cloud Security Lists) limitando a exposição pública exclusivamente a origens autorizadas.

| Host de Telemetria | Sistema Operacional | Modo de Coleta | Conectividade | Segurança em Trânsito |
| :--- | :--- | :--- | :--- | :--- |
| **Zabbix Server Core** | Ubuntu 26.04 Minimal (ARM64) | Servidor Central | Portas TCP 10051 (Ingestão) e 443 (HTTPS) | Apache TLS/SSL (Certbot) + Firewall Restrito |
| **HostCloud-A** | Ubuntu Server | Híbrido (Passivo/Ativo) | Rede Privada via IP Interno (Porta 10050) | TLS/PSK (`HostCloud-A` - 256-bit) |
| **HostCloud-B** | CentOS 9 Stream | Ativo (Zabbix Agent 2) | Rede Externa via FQDN `zbx-lab.space` | TLS/PSK (`HostCloud-B` - 256-bit) |
| **DESKTOP-SGH3G6V** | Windows 10 PRO | Ativo Exclusivo (Agent 2) | Saída TCP 10051 sob CGNAT Residencial | TLS/PSK (`DESKTOP-SGH3G6V` - 256-bit) |

---

## 💻 Tecnologias Utilizadas

| Tecnologia | Função na Arquitetura |
| :--- | :--- |
| **Oracle Cloud (OCI)** | Provedor de nuvem para hospedagem do servidor central (`VM.Standard.A1.Flex` ARM64) |
| **Ubuntu & CentOS Stream** | Sistemas operacionais Linux utilizados no servidor central e nos nós monitorados |
| **Zabbix 7.0 LTS** | Plataforma central de coleta de métricas, processamento de triggers e envio de alertas |
| **MySQL 8.0** | Banco de dados relacional configurado em `utf8mb4_bin` para retenção de histórico e tendências |
| **Apache HTTP Server** | Servidor web responsável pela disponibilização do frontend do Zabbix |
| **Certbot (Let's Encrypt)** | Automação de certificados digitais SSL/TLS e redirecionamento seguro HTTP para HTTPS |
| **OpenSSL** | Geração de chaves criptográficas pré-compartilhadas (PSK de 256 bits) |
| **UFW / IPTables** | Hardening de perímetro e filtragem de pacotes por porta e endereço IP |
| **Grafana OSS** | Plataforma analítica integrada via API JSON-RPC para dashboards executivos de NOC |

---

## 🚀 Fases de Implementação

### Fase 1: Fundação de Infraestrutura e Hardening de Perímetro
- Provisionamento da instância ARM64 na Oracle Cloud e acesso remoto via SSH/PuTTY.
- Instalação dos pacotes oficiais do Zabbix 7.0 LTS, MySQL Server e Apache.
- Criação do banco de dados `zabbix` (`utf8mb4`), ajuste temporário de `log_bin_trust_function_creators=1` e importação do esquema inicial (`server.sql.gz`).
- Configuração de DNS para o domínio `zbx-lab.space`, emissão de certificado SSL via Certbot e ajuste de loopback em `/etc/hosts` para prevenção de bloqueios de *hairpinning*.

### Fase 2: Conectividade Heterogênea, Auto Registration e TLS/PSK
- Deploy distribuído de agentes em servidores Linux (`HostCloud-A` e `HostCloud-B`) e estação Windows (`DESKTOP-SGH3G6V`).
- Automação de inventário via **Auto Registration**: leitura de `HostMetadata=linux` e `HostMetadata=WindowsDesktop` para alocação automática em grupos e vinculação de templates ativos.
- Blindagem ponta a ponta com geração de chaves de 256 bits (`openssl rand -hex 32 > secret.psk`), restrição de permissões (`chmod 640`) e ativação de `TLSConnect=psk` e `TLSAccept=psk`.

### Fase 3: Telemetria Avançada, Gatilhos Analíticos e Auditoria de Segurança
- Criação de itens customizados de verificação de disponibilidade (`agent.ping`), armazenamento (`vfs.fs.size`) e carga de processamento (`system.cpu.util`).
- Modelagem de triggers contra indisponibilidade (`nodata 60s` - Desastre), espaço em disco crítico (`< 5GB` - Alta) e exaustão de CPU (`avg 2m > 75%` - Aviso).
- **Auditoria de Segurança no Windows:** Monitoramento em tempo real do Windows Event Viewer através da chave `eventlog[Security,,,,4625,,skip]` para detecção de falhas de logon, utilizando pré-processamento com Regex e JavaScript para isolar o nome da conta atacada.
- Configuração de notificações de incidentes via SMTP (E-mail).

### Fase 4: Padronização, Topologia NOC e Integração com Grafana
- Conversão de itens, triggers e gráficos avulsos em **Templates reutilizáveis**.
- Construção de **Global Dashboards** e desenho de um **Network Map** interativo com mudança dinâmica de status.
- Integração com **Grafana OSS** utilizando conta de serviço com privilégio mínimo (*Read-only* e sem acesso ao frontend) conectada ao endpoint `api_jsonrpc.php`.

---

## 🚨 Simulação de Incidentes (Engenharia de Resiliência)

Para validar a eficácia dos alertas e a resposta operacional do NOC, foram executados três testes controlados de injeção de falha:

| Cenário Simulado | Ação Executada (Vetor de Falha) | Detecção e Comportamento do Sistema | Recuperação |
| :--- | :--- | :--- | :--- |
| **1. Queda de Agente** | Parada manual do serviço no Linux (`sudo systemctl stop zabbix-agent`). | A função `nodata(60s)` disparou alerta de **Desastre**, alterando o ícone do host para vermelho no Mapa de Rede. | Reinício do daemon e resolução automática do incidente no painel. |
| **2. Sobrecarga de CPU** | Estresse sintético de processador (`stress --cpu 4 --timeout 180`). | A média móvel superou 75% por 2 minutos, acionando alerta de **Aviso** no Zabbix e pico visual no Grafana. | Término do processo de estresse e normalização automática da métrica. |
| **3. Força Bruta no Windows** | Tentativas seguidas de login com senha incorreta na estação Windows. | O Agent 2 capturou o **Event ID 4625**, extraiu o usuário alvo via Regex/JS e gerou alerta de segurança no NOC. | Verificação pelo operador e encerramento manual (*Manual Close*) da trigger. |

---

## 📁 Estrutura do Repositório

```text
zabbix-enterprise-lab/
├── configs/           # Arquivos .conf higienizados do Zabbix Server e Agentes (Linux/Windows)
├── templates/         # Templates customizados (.yaml) e Mapa de Topologia NOC (.json)
├── scripts/           # Scripts de automação em Python (LLD de serviços) e Bash (Check SSL)
└── docs/evidencias/   # Capturas de tela dos dashboards, mapas, logs e testes de incidente
