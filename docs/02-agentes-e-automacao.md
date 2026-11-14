#  Módulo 2: Agentes Distribuídos, Auto Registration e Criptografia TLS/PSK

Documentação técnica passo a passo da expansão do laboratório para o monitoramento distribuído multi-plataforma (**Ubuntu**, **CentOS** e **Windows**), detalhando a arquitetura de coletas **Passivas e Ativas** (contornando redes restritas com NAT/CGNAT), o provisionamento zero-touch via **Auto Registration** e a criptografia ponta a ponta com chaves pré-compartilhadas (**TLS/PSK de 256 bits**).

---

##  Passo 1: Arquitetura de Conectividade dos Agentes (Passivo vs. Ativo)

O laboratório foi projetado para validar três cenários distintos de conectividade corporativa que se comunicam com o **Zabbix Server (`zbx-lab.space` / `150.136.108.22` / `10.0.0.161`)**:

| Hostname | Sistema Operacional | Tipo de Agente | Topologia de Rede | Modo de Operação | Metadado (`HostMetadata`) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`HostCloud-A`** | Ubuntu 26.04 LTS | Zabbix Agent (Clássico) | Rede Privada OCI (`10.0.0.176`) | Híbrido (Passivo + Ativo via IP Privado) | `linux` |
| **`HostCloud-B`** | CentOS 8 / Stream | Zabbix Agent 2 | Nuvem Externa / IP Público (`129.80.172.178`) | Híbrido (Passivo + Ativo via FQDN) | `linux` |
| **`DESKTOP-SGH3G6V`** | Windows 10 | Zabbix Agent 2 | Rede Doméstica / Corporativa sob **CGNAT** (`45.163.168.157`) | **100% Ativo** (Inicia conexão na porta `10051`) | `WindowsDesktop` |

> **Por que usar o Modo Ativo no Windows sob CGNAT?** Em provedores de internet que utilizam *Carrier-Grade NAT (CGNAT)*, não é possível abrir portas de entrada (`10050/TCP`) para que o Zabbix Server consulte o host passivamente. No modo **Ativo (`ServerActive`)**, o próprio agente Windows inicia a conexão de saída até a porta `10051/TCP` do servidor na nuvem (`zbx-lab.space`), baixando a lista de métricas solicitadas e enviando os resultados periodicamente sem necessidade de redirecionamento de portas local.

---

##  Passo 2: Implantação do Agente Linux em Rede Local (`HostCloud-A`)

O nó **`HostCloud-A`** reside na mesma Virtual Cloud Network (VCN) do Zabbix Server na Oracle Cloud, permitindo comunicação direta de baixa latência pelo endereço privado `10.0.0.161`.

### 1. Instalar o pacote do Zabbix Agent no Ubuntu
```bash
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu24.04_all.deb
sudo dpkg -i zabbix-release_7.0-1+ubuntu24.04_all.deb
sudo apt update
sudo apt install -y zabbix-agent
```

### 2. Parametrizar o arquivo `/etc/zabbix/zabbix_agentd.conf`
Edite o arquivo de configuração do agente clássico:
```bash
sudo nano /etc/zabbix/zabbix_agentd.conf
```
Diretivas ativas configuradas (arquivo higienizado disponível em [`configs/zabbix_agentd_ubuntu.conf`](../configs/zabbix_agentd_ubuntu.conf)):
```ini
PidFile=/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=10.0.0.161
ServerActive=10.0.0.161
Hostname=HostCloud-A
HostMetadata=linux
Include=/etc/zabbix/zabbix_agentd.d/*.conf
```

### 3. Liberar o Firewall e habilitar o serviço
No firewall do **`HostCloud-A`**, as portas `ICMP` e `10050/TCP` foram liberadas exclusivamente para o IP privado do Zabbix Server (`10.0.0.161/32`):
```bash
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
sudo systemctl status zabbix-agent
```

---

