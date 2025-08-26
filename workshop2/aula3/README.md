# Workshop 2: Smartcontracts Básico na Stellar com Soroban

## Aula 3: Stellar SDK Python - Demonstração Prática

### 📚 O que aprendemos nesta aula

Nesta aula, aprendemos como usar o **Stellar SDK em Python** para interagir com smart contracts. Foi uma **demonstração prática** para mostrar como a mesma funcionalidade pode ser implementada em diferentes linguagens. O projeto **Tap Game** continua sendo descentralizado, com frontend conectando diretamente ao smart contract.

### 🎯 Principais conceitos aprendidos

- ✅ **Stellar SDK Python**: Como usar a biblioteca oficial
- ✅ **Criação de Chaves**: Gerar e gerenciar carteiras
- ✅ **Transações**: Criar contas, fazer pagamentos
- ✅ **Smart Contracts**: Interagir com contratos via Python
- ✅ **Multi-linguagem**: Como implementar em diferentes tecnologias

---

## 🏗️ Arquitetura do Projeto

### Estrutura Real (Descentralizada)

```
Tap Game (Projeto Final)
├── web3/           # Smart Contract (Rust)
├── frontend/       # Interface React (Aula 2)
│                   # └── Conecta DIRETAMENTE ao smart contract
└── backend/        # Demonstração Python (Aula 3)
                    # └── Exemplo de como usar Stellar SDK
```

### Fluxo Real

```
Frontend React (Aula 2)
    ↓ (conexão direta)
Smart Contract (Rust)
    ↓ (demonstração)
Backend Python (Aula 3)
```

**Importante**: O frontend da Aula 2 é **descentralizado** e não depende do backend Python.

---

## 🐍 Stellar SDK Python

### Instalação

```bash
# Criar ambiente virtual
python -m venv env
source env/bin/activate  # Linux/Mac
# ou
env\Scripts\activate     # Windows

# Instalar biblioteca
pip install stellar-sdk
```

### Importação básica

```python
from stellar_sdk import Keypair, Server, TransactionBuilder, Network
from stellar_sdk.operation.payment import Payment
from stellar_sdk.operation.create_account import CreateAccount
```

---

## 🔑 Criação de Chaves

### Gerar chave aleatória

```python
# Gerar chave aleatória
keypair = Keypair.random()
print(f"Public Key: {keypair.public_key}")
print(f"Secret Key: {keypair.secret}")

# Chaves começam com:
# - Secret: S (32 bytes)
# - Public: G
# - Contrato: C
```

### Gerar chave a partir de seed

```python
# Mais seguro - usar seed específica
import os
seed = os.urandom(32)  # 32 bytes aleatórios
keypair = Keypair.from_raw_ed25519_seed(seed)
```

---

## 🏦 Criação de Contas

### Método 1: Friendbot (Testnet)

```python
import requests

def create_account_friendbot(public_key):
    """Criar conta usando Friendbot da Stellar"""
    url = f"https://friendbot.stellar.org/?addr={public_key}"
    response = requests.get(url)
    
    if response.status_code == 200:
        print(f"Conta criada! Recebeu 10.000 XLM")
        return True
    else:
        print("Erro ao criar conta")
        return False

# Uso
keypair = Keypair.random()
create_account_friendbot(keypair.public_key)
```

### Método 2: Criação manual

```python
def create_account_manual(source_keypair, destination_public_key, amount=100):
    """Criar conta manualmente via transação"""
    server = Server("https://horizon-testnet.stellar.org")
    
    # Carregar conta fonte
    source_account = server.load_account(source_keypair.public_key)
    
    # Criar transação
    transaction = TransactionBuilder(
        source_account=source_account,
        network_passphrase=Network.TESTNET_NETWORK_PASSPHRASE,
        base_fee=100
    ).append_create_account_op(
        destination=destination_public_key,
        starting_balance=str(amount)
    ).set_timeout(30).build()
    
    # Assinar e enviar
    transaction.sign(source_keypair)
    response = server.submit_transaction(transaction)
    
    print(f"Conta criada! Hash: {response['hash']}")
    return response
```

---

## 💰 Pagamentos

### Pagamento simples

```python
def send_payment(source_keypair, destination_public_key, amount):
    """Enviar pagamento XLM"""
    server = Server("https://horizon-testnet.stellar.org")
    
    # Carregar conta fonte
    source_account = server.load_account(source_keypair.public_key)
    
    # Criar transação de pagamento
    transaction = TransactionBuilder(
        source_account=source_account,
        network_passphrase=Network.TESTNET_NETWORK_PASSPHRASE,
        base_fee=100
    ).append_payment_op(
        destination=destination_public_key,
        asset=Asset.native(),
        amount=str(amount)
    ).set_timeout(30).build()
    
    # Assinar e enviar
    transaction.sign(source_keypair)
    response = server.submit_transaction(transaction)
    
    print(f"Pagamento enviado! Hash: {response['hash']}")
    return response
```

### Múltiplos pagamentos

```python
def send_multiple_payments(source_keypair, payments):
    """Enviar múltiplos pagamentos em uma transação"""
    server = Server("https://horizon-testnet.stellar.org")
    source_account = server.load_account(source_keypair.public_key)
    
    transaction = TransactionBuilder(
        source_account=source_account,
        network_passphrase=Network.TESTNET_NETWORK_PASSPHRASE,
        base_fee=100
    )
    
    # Adicionar múltiplas operações
    for payment in payments:
        transaction.append_payment_op(
            destination=payment['destination'],
            asset=Asset.native(),
            amount=str(payment['amount'])
        )
    
    transaction.set_timeout(30).build()
    transaction.sign(source_keypair)
    
    response = server.submit_transaction(transaction)
    print(f"Múltiplos pagamentos enviados! Hash: {response['hash']}")
    return response
```

---

## 🤖 Smart Contract Integration

### Ler dados do contrato

```python
def read_contract(contract_id, function_name):
    """Ler dados de um smart contract"""
    server = Server("https://horizon-testnet.stellar.org")
    
    # Criar transação de leitura
    transaction = TransactionBuilder(
        source_account=source_account,
        network_passphrase=Network.TESTNET_NETWORK_PASSPHRASE,
        base_fee=100
    ).append_invoke_host_function_op(
        contract_id=contract_id,
        function_name=function_name,
        function_args=[]
    ).set_timeout(30).build()
    
    # Para leitura, não precisa assinar
    response = server.submit_transaction(transaction)
    return response
```

### Escrever no contrato

```python
def write_contract(contract_id, function_name, args):
    """Escrever dados em um smart contract"""
    server = Server("https://horizon-testnet.stellar.org")
    
    # Codificar argumentos em XDR
    from stellar_sdk import xdr
    
    encoded_args = []
    for arg in args:
        if isinstance(arg, int):
            encoded_args.append(xdr.SCVal.from_i32(arg))
        elif isinstance(arg, str):
            encoded_args.append(xdr.SCVal.from_string(arg))
        elif isinstance(arg, bytes):
            encoded_args.append(xdr.SCVal.from_address(arg))
    
    # Criar transação
    transaction = TransactionBuilder(
        source_account=source_account,
        network_passphrase=Network.TESTNET_NETWORK_PASSPHRASE,
        base_fee=100
    ).append_invoke_host_function_op(
        contract_id=contract_id,
        function_name=function_name,
        function_args=encoded_args
    ).set_timeout(30).build()
    
    # Assinar e enviar
    transaction.sign(source_keypair)
    response = server.submit_transaction(transaction)
    return response
```

---

## 🎮 Exemplo: Tap Game Integration

### Salvar pontuação no jogo

```python
def save_game_score(contract_id, player_address, score, nickname, game_time):
    """Salvar pontuação no Tap Game (demonstração)"""
    args = [
        player_address,  # Address
        score,           # u32
        nickname,        # String
        game_time        # u32
    ]
    
    return write_contract(contract_id, "new_game", args)
```

### Obter ranking

```python
def get_game_ranking(contract_id):
    """Obter ranking do Tap Game (demonstração)"""
    return read_contract(contract_id, "get_ranking")
```

---

## 🐛 Debugging e Dicas

### Dicas para Hackathon

```python
# 1. Sempre simular antes de enviar
def simulate_transaction(transaction):
    server = Server("https://horizon-testnet.stellar.org")
    try:
        simulation = server.simulate_transaction(transaction)
        print("Simulação bem-sucedida!")
        return True
    except Exception as e:
        print(f"Erro na simulação: {e}")
        return False

# 2. Aumentar taxa em momentos de congestionamento
def high_fee_transaction():
    # Usar taxa mais alta durante hackathon
    base_fee = 1000  # Em vez de 100
```

### Resolver erros

```python
# Usar Discord da Stellar para decodificar erros XDR
def decode_error(error_xdr):
    """
    Copiar erro XDR e colar no Discord da Stellar:
    https://discord.gg/stellar
    """
    print(f"Erro XDR: {error_xdr}")
    print("Cole no Discord da Stellar para decodificar")
```

---

## 📁 Estrutura do Projeto

O projeto completo está em: [nearx-tap-game](https://github.com/danielgorgonha/nearx-tap-game)

```
nearx-tap-game/
├── web3/           # Smart Contract (Rust)
├── frontend/       # Interface React (Aula 2)
│                   # └── Descentralizado - conecta direto ao contrato
└── backend/        # Demonstração Python (Aula 3)
    ├── main.py     # Exemplos de uso do Stellar SDK
    └── requirements.txt
```

---

## 🎯 Resumo do Workshop 2

### Aula 1: Fundamentos
- ✅ Blockchain e Stellar Network
- ✅ Smart Contracts básicos
- ✅ Hello World na prática

### Aula 2: Frontend Descentralizado
- ✅ Tap Game - Smart Contract
- ✅ Frontend React conectando diretamente
- ✅ Sistema descentralizado completo

### Aula 3: Multi-linguagem (Python)
- ✅ Stellar SDK em Python
- ✅ Demonstração de outras linguagens
- ✅ Mesma funcionalidade, tecnologia diferente

---

## 🎯 Próximos Passos

### Workshop 3 (Próximo)
- **Segurança avançada** em smart contracts
- **Composabilidade** entre contratos
- **Autenticação Passkey** (FIDO)

### Desafios Práticos
- ✅ Implementar Tap Game completo
- ✅ Testar com diferentes linguagens
- ✅ Criar variações do jogo
- ✅ Explorar outras funcionalidades

---

## 📚 Recursos

- **Repositório**: [nearx-tap-game](https://github.com/danielgorgonha/nearx-tap-game)
- **Stellar SDK Python**: [stellar-sdk](https://github.com/StellarCN/py-stellar-sdk)
- **Documentação**: [stellar.org/developers](https://stellar.org/developers)
- **Discord Stellar**: [discord.gg/stellar](https://discord.gg/stellar)

---

*Desenvolvido para Meridian Hackathon 2025 - Rio de Janeiro*



