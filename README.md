🔐 SSH Hardening & Security – Part 1


📌 Objective

Implement security hardening measures on an SSH server to reduce unauthorized access risks and prepare the environment for future SOC (Security Operations Center) integrations.

⚙️ Environment

Operating System: Arch Linux (Virtual Machine on Oracle VirtualBox)

Tools used:

OpenSSH (sshd)

iptables (firewall)

systemd / systemctl (service management)


🛡️ Applied Configurations

1️⃣ Default SSH Port Change

The default SSH port was changed to reduce exposure to automated scans and brute-force attempts.

Configuration file:

/etc/ssh/sshd_config

Change applied:

Port 2222

Result:

Port 22 filtered/blocked
SSH service accessible only via port 2222
Note: This measure does not replace proper authentication controls but helps reduce noise from automated attacks.

2️⃣ Public Key Authentication

Password-based authentication was disabled in favor of SSH key-based authentication.
Key generation (client-side):

ssh-keygen -t ed25519

Key deployment to server:

ssh-copy-id -p 2222 m4@server

sshd configuration:

PasswordAuthentication no

Result:

Password login disabled
Access allowed only via cryptographic keys

3️⃣ Firewall Configuration (iptables)

Firewall rules were implemented to explicitly allow SSH access only on the hardened port.

Rules applied:

sudo iptables -A INPUT -p tcp --dport 2222 -j ACCEPT

sudo iptables -A INPUT -p tcp --dport 22 -j DROP

Persistence:

sudo iptables-save > /etc/iptables/iptables.rules

sudo systemctl enable iptables

Result:
Port 22 fully blocked

Only port 2222 allowed for SSH connections

4️⃣ SSH Service Management

Service control was tested to validate hardening and containment scenarios.

Disable SSH service:

sudo systemctl stop sshd

sudo systemctl disable sshd

sudo systemctl mask sshd

Re-enable SSH service:

sudo systemctl unmask sshd

sudo systemctl enable sshd

sudo systemctl start sshd

Purpose:

Validate service exposure control

Simulate service shutdown and recovery scenarios

📊 Tests Performed
🔍 Nmap Scan

Before: Port 22 open
After: Port 22 filtered, port 2222 open

🔐 SSH Connection Test

ssh -p 2222 m4@server

Result:

Successful login using SSH key authentication only
Password authentication rejected
✅ Outcome
This hardening process significantly reduces the SSH attack surface and establishes a secure baseline for future SOC-oriented monitoring and network defense projects.


-------------------------------------------------------------------------------------------------------------------------------------------------------------------------


🚀 Rodando o Elasticsearch

cd elasticsearch-8.15.0
./bin/elasticsearch

O servidor sobe na porta 9200.

Teste com:

curl http://localhost:9200

Deve retornar um JSON com informações da versão.



📊 Rodando o Kibana

cd kibana-8.15.0
./bin/kibana

Acesse via navegador:

http://localhost:5601

⚠️ Dicas importantes

Versão igual: mantenha Elasticsearch e Kibana na mesma versão (ex.: ambos 8.15).

Memória: Elasticsearch exige pelo menos 2 GB livres.

Primeira execução: serão geradas senhas e tokens no terminal. Guarde-os, pois o Kibana solicitará.



📦 Filebeat: Coletando Logs do Sistema

Instalação e verificação

./filebeat version

Deve retornar 9.2.3

Habilitar módulo system (captura de syslog e SSH)

./filebeat modules enable system

Configuração do filebeat.yml

output.elasticsearch:
  hosts: ["localhost:9200"]
  username: "elastic"
  password: "sua_senha"
  ssl:
    certificate_authorities: ["/caminho/para/certificado.crt"]
setup.kibana:
  host: "localhost:5601"

Inicializar dashboards e ingest pipelines

./filebeat setup

Rodar o Filebeat

./filebeat -e



📁 Logs Capturados (exemplo real)

Dashboard Kibana: [Filebeat System] Syslog dashboard ECS

Bar chart: eventos por hostname → archlinux

Donut chart: processos → opera (59.65%)


Tabela de logs:


Timestamp     Hostname     Processo     Mensagem
06:02:19     archlinux      opera     Uncaught (in promise) Error
06:01:42     archlinux   wireplumber  wp-event-dispatcher failed
05:58:56     archlinux      opera     ERROR: gpu command buffer
05:58:30     archlinux     systemd    Started Session 6 of User m4


🛠️ Configuração Avançada: journald como input

- type: journald
  seek: head
  include_lines: ['ERR', 'WARN']
  #fields:
  #  level: debug
  #  review: 1


🔍 Segurança e Monitoramento de Rede

Scan de portas com Nmap

nmap -sV -p 9200 0.0.0.0

Porta 2222/tcp: OpenSSH 10.2

Porta 9200/tcp: Elasticsearch REST API 7.0+ com SSL


✅ Boas Práticas com Elasticsearch

Segurança: habilite autenticação e criptografia (TLS/SSL).

Indexação: defina políticas de ciclo de vida (ILM) para evitar sobrecarga.

Templates: se modificar o nome do índice, configure setup.template.name e setup.template.pattern.

Monitoramento: use Kibana ou Grafana para acompanhar métricas.

Backups: configure snapshots regulares.





🔮 Futuras Atualizações:

🌍 Inclusão de GeoIP

Ative o processador geoip no pipeline de ingestão.

Permite visualizar localização geográfica de conexões SSH.

Exemplo de configuração:

processors:
  - decode_json_fields:
      fields: ["message"]
      target: "json"
  - geoip:
      field: "source.ip"
      target_field: "geo"


📈 Alertas e Análises

Configure alertas no Kibana para:

Múltiplas falhas de login

Conexões de IPs suspeitos

Atividades fora do horário padrão


🧠 Conclusão

Com Elasticsearch, Kibana e Filebeat integrados, você tem uma stack poderosa para observabilidade, segurança e análise em tempo real. A inclusão de GeoIP e boas práticas de configuração garantem escalabilidade e confiabilidade para ambientes Linux com foco em logs de sistema e conexões SSH.
