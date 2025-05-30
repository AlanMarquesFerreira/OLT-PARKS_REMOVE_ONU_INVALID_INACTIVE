ONU Cleaner - Script de Automação para OLT Parks
Python Version
License

📌 Visão Geral
Script Python para automatizar a identificação e remoção de ONUs com status INACTIVE ou INVALID em OLTs Parks, com geração de relatório detalhado.

✨ Funcionalidades
🔍 Verificação automática em portas GPON (1/1 a 2/4)

🚨 Detecção de ONUs problemáticas por status e serial

🗑️ Remoção segura com confirmação interativa

📄 Geração de relatório com timestamp

📊 Log detalhado de todas as operações

🔒 Autenticação via usuário/senha

🛠️ Pré-requisitos
Python 3.6 ou superior

Acesso Telnet à OLT

Permissões de administrador na OLT

bash
python --version  # Verifique sua versão do Python
⚙️ Instalação
Clone o repositório ou copie o script:

bash
git clone https://github.com/seu-usuario/onu-cleaner.git
cd onu-cleaner
Instale as dependências:

bash
pip install pexpect
🚀 Como Usar
Configuração
Edite as variáveis no script:

python
OLT_IP = "10.0.0.1"      # IP da sua OLT
USERNAME = "admin"       # Usuário de acesso
PASSWORD = "fisa"        # Senha de acesso
TIMEOUT = 10             # Tempo de espera (segundos)
Execução
bash
python onu_cleaner.py
Fluxo de Trabalho
O script conecta à OLT via Telnet

Verifica todas as portas GPON especificadas

Identifica ONUs com status INACTIVE ou INVALID

Solicita confirmação antes da remoção

Remove as ONUs problemáticas

Gera relatório com detalhes das operações

📝 Exemplo de Saída
[2023-11-15 14:30:45] Conectando à OLT 10.0.0.1...
[2023-11-15 14:30:47] Conexão estabelecida com sucesso
[2023-11-15 14:30:47] Verificando: show interface gpon1/1 onu status inactive
[2023-11-15 14:30:50] ONU problemática encontrada - Porta: gpon1/1 Serial: prks00b282b8
[2023-11-15 14:31:20] ONUs identificadas para remoção:
gpon1/1: prks00b282b8
gpon1/3: prks00b1c5f2, prks00b1c5a9, prks00b2465f

Confirmar remoção? (s/n): s
[2023-11-15 14:31:25] Iniciando processo de remoção...
[2023-11-15 14:31:45] Configuração salva com sucesso
[2023-11-15 14:31:45] Relatório gerado: 10.0.0.1_onu_removidas_20231115-143145.txt
[2023-11-15 14:31:45] Processo concluído com sucesso
📁 Estrutura de Arquivos
onu_cleaner.py - Script principal

onu_cleaner.log - Log detalhado de execução

10.0.0.1_onu_removidas_*.txt - Relatórios de remoção

📜 Código Fonte
python
import pexpect
import re
import time
from datetime import datetime

# Configurações
OLT_IP = "10.0.0.1"
USERNAME = "admin"
PASSWORD = "fisa"
TIMEOUT = 10
LOG_FILE = "onu_cleaner.log"

def log_action(message):
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    log_entry = f"[{timestamp}] {message}"
    print(log_entry)
    with open(LOG_FILE, "a") as f:
        f.write(log_entry + "\n")

def connect_olt():
    try:
        log_action(f"Conectando à OLT {OLT_IP}...")
        session = pexpect.spawn(f'telnet {OLT_IP}', timeout=TIMEOUT)
        
        session.expect('Username:')
        session.sendline(USERNAME)
        
        session.expect('Password:')
        session.sendline(PASSWORD)
        
        result = session.expect(['>', '#', 'Login incorrect'])
        if result == 2:
            raise Exception("Falha na autenticação")
        
        log_action("Conexão estabelecida com sucesso")
        return session
        
    except Exception as e:
        log_action(f"Erro na conexão: {str(e)}")
        raise

def check_onu_status(session):
    onu_list = {}
    gpon_ports = [f"gpon1/{x}" for x in range(1,5)] + [f"gpon2/{x}" for x in range(1,5)]
    
    for port in gpon_ports:
        for status in ['inactive', 'invalid']:
            cmd = f"show interface {port} onu status {status}"
            log_action(f"Verificando: {cmd}")
            
            session.sendline(cmd)
            session.expect(['>', '#'])
            output = session.before.decode()
            
            matches = re.finditer(
                r'(\d+-\d+-\d+-\d+)\s+\((prks\w+)\):\s*\n\s*Status\s*:\s*(INACTIVE|INVALID)',
                output
            )
            
            for match in matches:
                serial = match.group(2)
                if port not in onu_list:
                    onu_list[port] = []
                if serial not in onu_list[port]:
                    onu_list[port].append(serial)
                    log_action(f"ONU problemática encontrada - Porta: {port} Serial: {serial}")
    
    return onu_list

def remove_problematic_onus(session, onu_list):
    if not onu_list:
        log_action("Nenhuma ONU problemática encontrada")
        return
    
    log_action("Iniciando processo de remoção...")
    
    try:
        session.sendline("configure terminal")
        session.expect(["#"])
        
        for port, serials in onu_list.items():
            log_action(f"Acessando porta {port}")
            session.sendline(f"interface {port}")
            session.expect(["#"])
            
            for serial in serials:
                cmd = f"no onu {serial}"
                log_action(f"Executando: {cmd}")
                session.sendline(cmd)
                session.expect(["#"])
                time.sleep(1)
            
            session.sendline("exit")
            session.expect(["#"])
        
        session.sendline("end")
        session.expect([">", "#"])
        
        log_action("Salvando configuração...")
        session.sendline("copy running-config startup-config")
        session.expect([">", "#"], timeout=15)
        log_action("Configuração salva com sucesso")
        
    except Exception as e:
        log_action(f"Erro durante remoção: {str(e)}")
        raise

def generate_report(onu_list):
    if not onu_list:
        return
        
    timestamp = datetime.now().strftime("%Y%m%d-%H%M%S")
    filename = f"{OLT_IP}_onu_removidas_{timestamp}.txt"
    
    with open(filename, "w") as f:
        f.write(f"Relatório de ONUs removidas - {OLT_IP}\n")
        f.write(f"Data: {timestamp}\n")
        f.write("="*50 + "\n")
        
        total = 0
        for port, serials in onu_list.items():
            for serial in serials:
                f.write(f"Porta: {port.ljust(8)} | Serial: {serial}\n")
                total += 1
        
        f.write("="*50 + "\n")
        f.write(f"Total removidas: {total}\n")
    
    log_action(f"Relatório gerado: {filename}")

def main():
    try:
        log_action("==== INÍCIO DO PROCESSO ====")
        
        session = connect_olt()
        problematic_onus = check_onu_status(session)
        
        if problematic_onus:
            log_action("\nONUs identificadas para remoção:")
            for port, serials in problematic_onus.items():
                log_action(f"{port}: {', '.join(serials)}")
            
            confirm = input("\nConfirmar remoção? (s/n): ").lower()
            if confirm == 's':
                remove_problematic_onus(session, problematic_onus)
                generate_report(problematic_onus)
            else:
                log_action("Operação cancelada pelo usuário")
        else:
            log_action("Nenhuma ONU problemática encontrada")
        
        session.sendline("exit")
        log_action("==== PROCESSO CONCLUÍDO ====")
        
    except Exception as e:
        log_action(f"ERRO: {str(e)}")
        log_action("==== PROCESSO INTERROMPIDO ====")

if __name__ == "__main__":
    main()
🤝 Contribuições
Contribuições são bem-vindas! Siga estes passos:

Faça um fork do projeto

Crie uma branch para sua feature (git checkout -b feature/awesome-feature)

Commit suas mudanças (git commit -m 'Add some awesome feature')

Push para a branch (git push origin feature/awesome-feature)

Abra um Pull Request

📄 Licença
Este projeto está licenciado sob a licença MIT - veja o arquivo LICENSE para detalhes.

✉️ Contato
Desenvolvedor: [Seu Nome]
Email: seu-email@exemplo.com
LinkedIn: seu-perfil

⌨️ com ❤️ por Seu Nome

New chat
Message DeepSeek
