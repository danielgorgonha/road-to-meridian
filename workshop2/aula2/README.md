# Workshop 2: Smartcontracts Básico na Stellar com Soroban

## Aula 2: Smartcontracts e Integração com Backend

### 📚 Resumo da Aula

Esta aula foca na integração de smart contracts com sistemas backend, explorando como armazenar dados no contrato e criar aplicações que interagem com a blockchain Stellar.

### 🎯 Objetivos da Aula

- ✅ Entender storage em smart contracts
- ✅ Criar contratos com estado persistente
- ✅ Integrar smart contracts com APIs backend
- ✅ Desenvolver aplicações full-stack com Stellar
- ✅ Implementar padrões de segurança

---

## 💾 Storage em Smart Contracts

### Conceitos Fundamentais

#### **Estado Persistente**
- **Definição**: Dados que persistem entre chamadas do contrato
- **Localização**: Armazenado na blockchain
- **Custo**: Cada operação de storage tem custo em XLM

#### **Tipos de Storage**
- **Temporary**: Dados que existem apenas durante a execução
- **Persistent**: Dados que persistem entre transações
- **Instance**: Dados específicos da instância do contrato

### Exemplo Prático: Contador

```rust
#![no_std]
use soroban_sdk::{contractimpl, contracttype, Env, Symbol, Address};

#[contracttype]
#[derive(Clone, Debug, Eq, PartialEq)]
pub struct DataKey {
    pub owner: Address,
    pub counter: Symbol,
}

pub struct CounterContract;

#[contractimpl]
impl CounterContract {
    pub fn increment(env: Env, owner: Address) -> u32 {
        let key = DataKey {
            owner: owner.clone(),
            counter: Symbol::new(&env, "counter"),
        };
        
        let current_count: u32 = env.storage().get(&key).unwrap_or(0);
        let new_count = current_count + 1;
        
        env.storage().set(&key, &new_count);
        new_count
    }
    
    pub fn get_count(env: Env, owner: Address) -> u32 {
        let key = DataKey {
            owner: owner.clone(),
            counter: Symbol::new(&env, "counter"),
        };
        
        env.storage().get(&key).unwrap_or(0)
    }
}
```

---

## 🔗 Integração com Backend

### Arquitetura de Integração

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Frontend  │───▶│    API      │───▶│   Stellar   │
│             │    │  Backend    │    │  Network    │
└─────────────┘    └─────────────┘    └─────────────┘
```

### Padrões de Integração

#### **1. API Gateway Pattern**
- **Função**: Centralizar chamadas para smart contracts
- **Vantagens**: Cache, rate limiting, autenticação
- **Implementação**: REST API que chama Stellar CLI

#### **2. Event-Driven Pattern**
- **Função**: Reagir a eventos da blockchain
- **Vantagens**: Desacoplamento, escalabilidade
- **Implementação**: Webhooks + message queues

### Exemplo: API Backend em Rust

```rust
use actix_web::{web, App, HttpServer, Result};
use serde::{Deserialize, Serialize};
use std::process::Command;

#[derive(Deserialize)]
struct IncrementRequest {
    owner_address: String,
}

#[derive(Serialize)]
struct IncrementResponse {
    new_count: u32,
    transaction_hash: String,
}

async fn increment_counter(req: web::Json<IncrementRequest>) -> Result<web::Json<IncrementResponse>> {
    // Chamar smart contract via Stellar CLI
    let output = Command::new("stellar")
        .args(&[
            "contract", "invoke",
            "--id", "CONTRACT_ID",
            "--source", "WALLET_KEY",
            "--", "increment",
            "owner", &req.owner_address
        ])
        .output()
        .expect("Failed to execute command");
    
    // Parse resultado
    let result = String::from_utf8_lossy(&output.stdout);
    
    Ok(web::Json(IncrementResponse {
        new_count: 0, // Parse from result
        transaction_hash: "hash".to_string(), // Parse from result
    }))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/increment", web::post().to(increment_counter))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

---

## 🔐 Segurança em Integrações

### Boas Práticas

#### **1. Autenticação e Autorização**
```rust
// Verificar assinatura da transação
pub fn verify_signature(env: &Env, signature: &[u8], message: &[u8]) -> bool {
    // Implementar verificação de assinatura
    true
}
```

#### **2. Rate Limiting**
- **Objetivo**: Prevenir spam e ataques
- **Implementação**: Contadores por endereço
- **Custo**: Considerar custos de storage

#### **3. Input Validation**
```rust
pub fn validate_input(env: &Env, input: &str) -> Result<(), Error> {
    if input.len() > 100 {
        return Err(Error::from_type_and_code(1, 1));
    }
    Ok(())
}
```

### Padrões de Segurança

#### **Access Control**
```rust
pub fn require_owner(env: &Env, caller: &Address, owner: &Address) {
    if caller != owner {
        panic!("Unauthorized access");
    }
}
```

#### **Reentrancy Protection**
```rust
use soroban_sdk::symbol_short;

pub fn protected_function(env: &Env) {
    let key = symbol_short!("locked");
    if env.storage().has(&key) {
        panic!("Function is locked");
    }
    
    env.storage().set(&key, &true);
    // Execute function logic
    env.storage().set(&key, &false);
}
```

---

## 🏗️ Aplicação Full-Stack

### Estrutura do Projeto

```
smart-contract-app/
├── contracts/
│   ├── counter/
│   └── voting/
├── backend/
│   ├── src/
│   ├── Cargo.toml
│   └── .env
├── frontend/
│   ├── src/
│   └── package.json
└── README.md
```

### Backend API (Rust + Actix)

```rust
// main.rs
use actix_web::{web, App, HttpServer};
use serde::{Deserialize, Serialize};

mod stellar_client;
mod auth;
mod routes;

#[derive(Serialize)]
struct ApiResponse<T> {
    success: bool,
    data: Option<T>,
    error: Option<String>,
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(routes::counter::scope())
            .service(routes::voting::scope())
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

### Cliente Stellar

```rust
// stellar_client.rs
use std::process::Command;
use serde_json::Value;

pub struct StellarClient {
    network: String,
    wallet_key: String,
}

impl StellarClient {
    pub fn new(network: String, wallet_key: String) -> Self {
        Self { network, wallet_key }
    }
    
    pub fn invoke_contract(&self, contract_id: &str, function: &str, args: &[&str]) -> Result<Value, Box<dyn std::error::Error>> {
        let mut command = Command::new("stellar");
        command.args(&["contract", "invoke", "--id", contract_id, "--source", &self.wallet_key, "--network", &self.network, "--", function]);
        command.args(args);
        
        let output = command.output()?;
        let result = String::from_utf8_lossy(&output.stdout);
        
        Ok(serde_json::from_str(&result)?)
    }
}
```

---

## 🧪 Testes de Integração

### Testes End-to-End

```rust
#[cfg(test)]
mod integration_tests {
    use super::*;
    use actix_web::test;

    #[actix_web::test]
    async fn test_increment_counter() {
        let app = test::init_service(App::new().service(increment_counter)).await;
        
        let req = test::TestRequest::post()
            .uri("/increment")
            .set_json(IncrementRequest {
                owner_address: "G...".to_string(),
            })
            .to_request();
        
        let resp = test::call_service(&app, req).await;
        assert!(resp.status().is_success());
    }
}
```

### Testes de Smart Contract

```rust
#[cfg(test)]
mod test {
    use super::*;
    use soroban_sdk::Address;

    #[test]
    fn test_increment() {
        let env = Env::default();
        let contract = CounterContract;
        let owner = Address::random(&env);
        
        let count1 = contract.increment(&env, &owner);
        assert_eq!(count1, 1);
        
        let count2 = contract.increment(&env, &owner);
        assert_eq!(count2, 2);
        
        let stored_count = contract.get_count(&env, &owner);
        assert_eq!(stored_count, 2);
    }
}
```

---

## 🚀 Deploy e Produção

### Configuração de Ambiente

```bash
# Variáveis de ambiente
export STELLAR_NETWORK=testnet
export CONTRACT_ID=your_contract_id
export WALLET_SECRET=your_wallet_secret
export API_PORT=8080
```

### Docker Deployment

```dockerfile
FROM rust:1.70 as builder
WORKDIR /app
COPY . .
RUN cargo build --release

FROM debian:bullseye-slim
RUN apt-get update && apt-get install -y ca-certificates
COPY --from=builder /app/target/release/backend /usr/local/bin/
EXPOSE 8080
CMD ["backend"]
```

### Monitoramento

```rust
use tracing::{info, error};

pub async fn increment_with_logging(req: web::Json<IncrementRequest>) -> Result<web::Json<IncrementResponse>> {
    info!("Incrementing counter for owner: {}", req.owner_address);
    
    match increment_counter(req).await {
        Ok(response) => {
            info!("Counter incremented successfully");
            Ok(response)
        }
        Err(e) => {
            error!("Failed to increment counter: {}", e);
            Err(e)
        }
    }
}
```

---

## 🎯 Próximos Passos

### Aula 3 (Próxima)
- Integração com frontend
- Interface de usuário para smart contracts
- Aplicações web completas

### Desafios Práticos
- ✅ Implementar sistema de votação
- ✅ Criar API REST completa
- ✅ Adicionar autenticação JWT
- ✅ Implementar cache Redis

---

## 📚 Recursos Adicionais

### Documentação
- [Soroban Storage](https://soroban.stellar.org/docs/fundamentals-and-concepts/storage)
- [Actix Web](https://actix.rs/)
- [Stellar CLI](https://soroban.stellar.org/docs/getting-started/setup)

### Exemplos
- [Soroban Examples](https://github.com/stellar/soroban-examples)
- [Stellar Quest](https://quest.stellar.org/)

---

*Desenvolvido para Meridian Hackathon 2025 - Rio de Janeiro*



