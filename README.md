🔐 SSH Hardening & Security – Parte 1
📌 Objetivo
Implementar medidas de segurança no servidor SSH para reduzir riscos de acesso não autorizado e preparar o ambiente para futuras integrações com SOC (Security Operations Center).

⚙️ Ambiente
- Sistema operacional: Arch Linux (VM no Oracle VirtualBox)
- Ferramentas utilizadas:
- sshd (OpenSSH Server)
- iptables (firewall)
- systemctl (gerenciamento de serviços)

🛡️ Configurações aplicadas
1. Alteração da porta padrão
- Editado /etc/ssh/sshd_config:
Port 2222
- Resultado: Porta 22 filtrada, acesso apenas pela porta 2222.

2. Autenticação por chave pública
- Gerada chave SSH no cliente:
ssh-keygen -t ed25519
- Copiada chave para o servidor:
ssh-copy-id -p 2222 m4@servidor
- Configurado PasswordAuthentication no no sshd_config.

3. Firewall (iptables)
- Permitido acesso apenas na porta 2222:
sudo iptables -A INPUT -p tcp --dport 2222 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j DROP
- Regras salvas:
sudo iptables-save > /etc/iptables/iptables.rules
sudo systemctl enable iptables

4. Gerenciamento do serviço SSH
- Parar serviço:
sudo systemctl stop sshd
- Desativar no boot:
sudo systemctl disable sshd
- Bloquear totalmente:
sudo systemctl mask sshd
- Reativar:
sudo systemctl unmask sshd
sudo systemctl enable sshd
sudo systemctl start sshd


📊 Testes realizados
- Nmap:
- Antes: Porta 22 aberta.
- Depois: Porta 22 filtrada, porta 2222 aberta.
- Conexão SSH:
ssh -p 2222 m4@servidor

- → Login apenas com chave pública.