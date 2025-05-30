# 🔌 Script de Limpeza Automática em OLT's PARKS de remoção de ONUs parks Inactive ou Invalid via Telnet

Este script em Python se conecta via Telnet a uma OLT Parks, verifica as ONUs com status `INACTIVE` ou `INVALID`, e remove automaticamente aquelas com serial iniciado por `prks00`. Ao final, um relatório com as ONUs removidas é salvo.

---

## ⚙️ Como usar

1. 🔧 Edite as variáveis no início do código abaixo com os dados da sua OLT:

```python
OLT_IP = "10.0.0.1"        # IP da sua OLT
USERNAME = "root"          # Usuário de acesso
PASSWORD = "123456"        # Senha de acesso
```

2. 🏃‍♂️ Execute o script com Python:

```bash
python nome_do_arquivo.py
```

3. 🧾 Um relatório será salvo automaticamente com o nome:

```
<IP>_relatorio_onus_removidas_<DATA-HORA>.txt
```

Exemplo: `10.0.0.1_relatorio_onus_removidas_20250530-140640.txt`

---

## 📜 Código completo

```python
import telnetlib
import time
import re
from datetime import datetime

OLT_IP = "10.0.0.1"
USERNAME = "root"
PASSWORD = "123456789"
TIMEOUT = 5

COMANDOS = [
    "sh interface gpon1/1 onu status inactive",
    "sh interface gpon1/2 onu status inactive",
    "sh interface gpon1/3 onu status inactive",
    "sh interface gpon1/4 onu status inactive",
    "sh interface gpon1/1 onu status invalid",
    "sh interface gpon1/2 onu status invalid",
    "sh interface gpon1/3 onu status invalid",
    "sh interface gpon1/4 onu status invalid",
    "sh interface gpon2/1 onu status inactive",
    "sh interface gpon2/2 onu status inactive",
    "sh interface gpon2/3 onu status inactive",
    "sh interface gpon2/4 onu status inactive",
    "sh interface gpon2/1 onu status invalid",
    "sh interface gpon2/2 onu status invalid",
    "sh interface gpon2/3 onu status invalid",
    "sh interface gpon2/4 onu status invalid"
]

REGEX_SERIAL = re.compile(r'\((prks\w+)\):')

def send_command(tn, command, wait=1):
    tn.write(command.encode('ascii') + b"\n")
    time.sleep(wait)
    return tn.read_very_eager().decode('ascii', errors='ignore')

def main():
    try:
        print(f"[INFO] Conectando à OLT {OLT_IP} via Telnet...")
        tn = telnetlib.Telnet(OLT_IP, timeout=TIMEOUT)

        tn.write(b"\n")
        time.sleep(1)

        tn.read_until(b"Username:", timeout=TIMEOUT)
        tn.write(USERNAME.encode('ascii') + b"\n")

        tn.read_until(b"keyuser:", timeout=TIMEOUT)
        tn.write(PASSWORD.encode('ascii') + b"\n")

        time.sleep(2)
        login_output = tn.read_very_eager().decode('ascii', errors='ignore')
        print("[LOGIN OUTPUT]:")
        print(login_output)

        if "%AUTH" in login_output or "Login failed" in login_output:
            print("[ERRO] Falha no login.")
            tn.close()
            return

        print("[SUCESSO] Login bem-sucedido!")

        onus_para_remover = {}

        for comando in COMANDOS:
            print(f"[INFO] Enviando comando: {comando}")
            output = send_command(tn, comando, wait=2)
            print("[RESPOSTA DO COMANDO]:")
            print(output)

            interface = comando.split()[2]

            for match in REGEX_SERIAL.finditer(output):
                serial = match.group(1)
                if serial.startswith("prks00"):
                    print(f"[DETECÇÃO] {interface} - {serial}")
                    onus_para_remover.setdefault(interface, []).append(serial)

        removidas = []

        if onus_para_remover:
            print("[INFO] Iniciando remoção de ONUs problemáticas...")
            send_command(tn, "configure terminal", wait=1)
            for interface, seriais in onus_para_remover.items():
                send_command(tn, f"interface {interface}", wait=1)
                for serial in seriais:
                    print(f"[REMOVENDO] {serial} da interface {interface}")
                    send_command(tn, f"no onu {serial}", wait=1)
                    removidas.append((interface, serial))
                send_command(tn, "exit", wait=0.5)
            send_command(tn, "end", wait=1)
            send_command(tn, "copy r s", wait=2)
            print("[SUCESSO] ONUs removidas e configuração salva.")
        else:
            print("[INFO] Nenhuma ONU INACTIVE ou INVALID encontrada.")

        tn.close()

        if removidas:
            data_str = datetime.now().strftime("%Y%m%d-%H%M%S")
            nome_arquivo = f"{OLT_IP}_relatorio_onus_removidas_{data_str}.txt"
            with open(nome_arquivo, "w") as f:
                f.write(f"Relatório de ONUs removidas - {OLT_IP} - {data_str}\n")
                f.write("="*60 + "\n")
                for interface, serial in removidas:
                    f.write(f"Interface: {interface} | Serial: {serial}\n")
                f.write("="*60 + "\n")
                f.write(f"Total removidas: {len(removidas)}\n")
            print(f"[RELATÓRIO] Relatório salvo em '{nome_arquivo}'.")
        else:
            print("[RELATÓRIO] Nenhuma ONU removida. Nenhum arquivo criado.")

    except Exception as e:
        print(f"[ERRO] {e}")

if __name__ == "__main__":
    main()
```

---

## 🧠 Autor

Desenvolvido por [Alan Marques Ferreira] 🧑‍💻  
Especialista em automações de rede e scripts para OLTs Parks

---

## 🐍 Requisitos de Python

- ✅ Python 3.6 até 3.10: nada precisa ser instalado  
- ⚠️ Python 3.11 ou superior: instale a alternativa `telnetlib3`:

```bash
pip install telnetlib3
```

---

## 🛡️ Licença

Este projeto é de uso livre para fins administrativos.  
Dê os créditos ao autor original ao compartilhar ou modificar o script.
