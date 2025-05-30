# OLT-PARKS_REMOVE_ONU_INVALID_INACTIVE
OLT-PARK Script automático para verificar e remover onu's invalidas ou inativas de olt parks.
# 🔌 Script de Limpeza Automática de ONUs Inativas ou Inválidas via Telnet

Este script em Python se conecta via Telnet a uma OLT Parks, verifica as ONUs com status `INACTIVE` ou `INVALID`, e remove automaticamente aquelas que estiverem com status problemático. Ao final, um relatório detalhado é salvo com os dados das ONUs removidas.

---

## 🚀 Funcionalidades

- ✅ Conexão automática com a OLT via Telnet  
- 🔍 Execução de comandos para listar ONUs por porta GPON  
- 🧠 Detecção de seriais iniciados por `prks00` com status `INACTIVE` ou `INVALID`  
- 🧹 Remoção automática das ONUs identificadas  
- 🧾 Geração de relatório `.txt` com IP da OLT, data e horário da operação  

---

## 📦 Instalação

### ✅ Requisitos

- Python 3.6 ou superior instalado  
- Acesso à rede com permissões Telnet até a OLT  
- Permissão para executar comandos de administração na OLT  

### 📥 Instalação das bibliotecas

O script utiliza apenas bibliotecas da **biblioteca padrão do Python**, então **não é necessário instalar nenhuma dependência externa**.

🔎 Mas se estiver usando uma versão desatualizada do Python que não possui `telnetlib`, você pode instalar via:

```bash
pip install telnetlib3
