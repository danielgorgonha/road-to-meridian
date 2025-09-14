# Workshop 2: Smartcontracts Básico na Stellar com Soroban

## 📚 Visão Geral

O Workshop 2 é o segundo nível da série "Road to Meridian", focado no desenvolvimento de smart contracts na Stellar Network usando Rust e Soroban. Este workshop constrói sobre os fundamentos do Workshop 1 e prepara os participantes para o desenvolvimento de aplicações blockchain.

### 🎯 Objetivos do Workshop

- ✅ Compreender os fundamentos da blockchain e Stellar Network
- ✅ Dominar o desenvolvimento de smart contracts com Soroban
- ✅ Criar aplicações full-stack com smart contracts
- ✅ Implementar padrões de segurança básicos
- ✅ Preparar para o Workshop 3 avançado

---

## 📋 Estrutura das Aulas

### [📖 Aula 1: Básico de Blockchain e Hello World](./aula1/README.md)
**Conteúdo:** Fundamentos da blockchain, Stellar Network, Soroban e primeiro smart contract

**Principais tópicos:**
- 🏗️ Fundamentos da blockchain (4 pilares)
- ⭐ Stellar Network em detalhes
- 🔧 Smart contracts na Stellar (Soroban)
- 🚀 Demonstração prática: Hello World
- 🧪 Testes e segurança básica

**[📋 Ver conteúdo da Aula 1](./aula1/README.md)**

---

### [🚀 Aula 2: Smartcontracts e Integração com Backend](./aula2/README.md)
**Conteúdo:** Storage em smart contracts e integração com sistemas backend

**Principais tópicos:**
- 💾 Storage em smart contracts
- 🔗 Integração com backend
- 🔐 Segurança em integrações
- 🏗️ Aplicação full-stack
- 🧪 Testes de integração

**[📋 Ver conteúdo da Aula 2](./aula2/README.md)**

---

### [🌐 Aula 3: Smartcontracts e Integração com Frontend](./aula3/README.md)
**Conteúdo:** Interfaces de usuário e aplicações web completas com smart contracts

**Principais tópicos:**
- 🌐 Frontend para smart contracts
- 🔗 Integração frontend-backend
- 👛 Wallet integration
- 🎨 Interface de usuário
- 🎯 Aplicação completa: Sistema de votação

**[📋 Ver conteúdo da Aula 3](./aula3/README.md)**

---

## 🏆 Desafios do Workshop 2

### Desafio Aula 1 - 🔄 EM ANDAMENTO
- 🔄 Reproduzir o "Hello World" e criar variações
- 🔄 Implementar contador simples
- 🔄 Criar contrato de votação básico
- 🔄 Publicar sobre aprendizado no LinkedIn

### Desafio Aula 2 - 📅 FUTURO
- 📋 Sistema de votação com storage
- 📋 API REST completa
- 📋 Autenticação JWT
- 📋 Cache Redis

### Desafio Aula 3 - 📅 FUTURO
- 📋 Aplicação full-stack completa
- 📋 Sistema de NFTs
- 📋 DEX simples
- 📋 Gráficos de votação

---

## 🏗️ Fundamentos da Blockchain

### Os 4 Pilares Fundamentais

#### 1. **Carteiras (Wallets)**
- **Função**: Armazenam chaves criptográficas (pública e privada)
- **Chave Pública**: Como um e-mail (identificação)
- **Chave Privada**: Como uma senha (controle)
- **Ciclo de Vida**: Criar par de chaves → Assinar transações → Enviar para blockchain

#### 2. **Transações**
- **Definição**: Instruções para a blockchain realizar ações
- **Exemplos**: Transferir tokens, NFTs, criar contas, interagir com smart contracts
- **Característica**: Alteram o "estado" da blockchain (salvo nos blocos)
- **Imutabilidade**: Registros passados são imutáveis

#### 3. **Blocos (Ledgers)**
- **Na Stellar**: Chamados de "ledgers"
- **Conteúdo**: Conjuntos de transações agrupadas, ordenadas e executadas
- **Estrutura**: 
  - **Header**: Metadados (número, hash, tempo de criação)
  - **Body**: Lista de transações
- **Encadeamento**: Blocos conectados criptograficamente através de hashes

#### 4. **Consenso**
- **Objetivo**: Garantir que todos os nós concordem sobre o próximo estado
- **Complexidade**: Problema da ciência da computação distribuída
- **Tipos**: Proof of Work (Bitcoin), Proof of Stake (Ethereum)

---

## ⭐ Stellar Network

### Características Principais
- **Foco**: Pagamentos globais rápidos e baratos
- **História**: Fundada em 2014 por Jed McCaleb
- **Smart Contracts**: Lançados em 2023 (Soroban)
- **Token**: Stellar Lumens (XLM)
- **Consenso**: Stellar Consensus Protocol (SCP)

### Tipos de Rede
- **Mainnet**: Rede pública com tokens de valor real
- **Testnet**: Rede de testes com 10.000 XLM gratuitos
- **Futurenet**: Rede de desenvolvimento com novas funcionalidades

---

## 🔧 Soroban (Smart Contracts)

### Características
- **Runtime**: WebAssembly (WASM)
- **Linguagem Principal**: Rust
- **Lançamento**: Mainnet em 2024
- **Pilares**: Desempenho, sustentabilidade e segurança

### Vantagens
- **Performance**: Alta velocidade de execução
- **Segurança**: Segurança de memória do Rust
- **Portabilidade**: WASM funciona em qualquer ambiente
- **Ecossistema**: Crescimento rápido com $100 milhões em financiamento

---

## 🚀 Exemplo: Hello World

### Código Básico
```rust
#![no_std]
use soroban_sdk::{contractimpl, Env, String, Vec};

pub struct HelloContract;

#[contractimpl]
impl HelloContract {
    pub fn hello(env: Env, message: String) -> Vec<String> {
        let mut result = Vec::new(&env);
        result.push_back(String::from_slice(&env, "Hello"));
        result.push_back(message);
        result
    }
}
```

### Comandos Básicos
```bash
# Criar projeto
stellar contract init HelloWord

# Compilar
stellar contract build

# Otimizar
stellar contract optimize

# Deploy
stellar contract deploy --wasm target/wasm32-unknown-unknown/release/hello_word_optimized.wasm --source Bob --network testnet

# Invocar
stellar contract invoke --id <contract_id> --source Bob -- hello message "Lucas"
```

---

## 🔐 Segurança Básica

### Boas Práticas
- **Testes**: 100% de cobertura de código
- **Validação**: Sempre validar inputs
- **Controle de Acesso**: Verificar permissões
- **Auditoria**: Revisar código antes do deploy

### Padrões de Segurança
```rust
// Access Control
pub fn require_owner(env: &Env, caller: &Address, owner: &Address) {
    if caller != owner {
        panic!("Unauthorized access");
    }
}

// Input Validation
pub fn validate_input(env: &Env, input: &str) -> Result<(), Error> {
    if input.len() > 100 {
        return Err(Error::from_type_and_code(1, 1));
    }
    Ok(())
}
```

---

## 📚 Recursos Adicionais

**Veja recursos completos no [README principal](../README.md#-recursos-adicionais).**

---

*Desenvolvido para Meridian Hackathon 2025 - Rio de Janeiro*



