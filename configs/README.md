# ⚙️ Arquivos de Configuração (Higienizados)

Esta pasta reúne os arquivos de configuração do **Zabbix Server** e dos **Agentes Zabbix** (Linux e Windows) utilizados no laboratório.

> **Nota de Segurança:** Senhas de banco de dados e chaves criptográficas PSK foram substituídas por marcadores genéricos como **<SUA_SENHA_AQUI>** e **<SUA_CHAVE_PSK_AQUI>**.

## 📄 Arquivos Disponíveis

- **zabbix_server.conf**: Parâmetros centrais do servidor Zabbix na Oracle Cloud conectados ao banco MySQL.
- **zabbix_agentd_ubuntu.conf**: Configuração do agente clássico no host **HostCloud-A** com Auto Registration e PSK.
- **zabbix_agent2_centos.conf**: Configuração do Zabbix Agent 2 no host **HostCloud-B** (CentOS Stream) via domínio externo e PSK.
- **zabbix_agent2_windows.conf**: Configuração do Zabbix Agent 2 na estação Windows (**DESKTOP-SGH3G6V**) com coleta ativa e leitura de Event Logs.

## 🔐 Como Gerar sua Chave PSK nos Hosts Linux

1. Gere uma chave hexadecimal de 256 bits e ajuste as permissões:
```bash
openssl rand -hex 32 > /home/zabbix/secret.psk
chown zabbix:zabbix /home/zabbix/secret.psk
chmod 640 /home/zabbix/secret.psk
```
2. Reinicie o agente após apontar o caminho da chave no arquivo de configuração:
```bash
sudo systemctl restart zabbix-agent
```
