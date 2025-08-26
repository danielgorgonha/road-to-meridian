# 🗳️ Sistema de Votação - Desafio 3

## 📋 Visão Geral

Este é o **Desafio 3** do Workshop 2 - Aula 1, focado na criação de um sistema de votação completo usando smart contracts na Stellar Network com Soroban.

### 🎯 Objetivos do Desafio

- ✅ Implementar sistema de votação descentralizado
- ✅ Gerenciar múltiplas opções de voto
- ✅ Validar votantes únicos
- ✅ Contar e exibir resultados
- ✅ Integrar com frontend 

---

## 🏗️ Arquitetura do Sistema

### 📊 Estrutura de Dados

```rust
#[contracttype]
pub struct VoteOption {
    pub name: String,        // Nome da opção (ex: "Rust", "Python")
    pub vote_count: u32,     // Número de votos recebidos
}

#[contracttype]
pub struct Voter {
    pub address: Address,    // Endereço do votante
    pub has_voted: bool,     // Se já votou
    pub voted_option: String, // Opção escolhida
}

#[contracttype]
pub struct VotingSession {
    pub title: String,           // Título da votação
    pub options: Vec<String>,    // Lista de opções
    pub total_votes: u32,        // Total de votos
    pub is_active: bool,         // Se a votação está ativa
    pub created_at: u64,         // Timestamp de criação
}
```

### 🔧 Funções do Contrato

```rust
#[contractimpl]
impl VotingContract {
    // Criar nova votação
    pub fn create_vote(env: Env, title: String, options: Vec<String>) -> String
    
    // Registrar voto de um endereço
    pub fn vote(env: Env, voter: Address, option: String) -> bool
    
    // Obter resultados da votação
    pub fn get_results(env: Env) -> Vec<VoteOption>
    
    // Verificar se endereço já votou
    pub fn has_voted(env: Env, voter: Address) -> bool
    
    // Obter informações da sessão de votação
    pub fn get_session_info(env: Env) -> VotingSession
    
    // Finalizar votação (apenas criador)
    pub fn end_vote(env: Env, creator: Address) -> bool
}
```

---

## 🚀 Implementação

### 📦 Estrutura do Projeto

```bash
voting-system/
├── Cargo.toml
├── src/
│   └── lib.rs
├── test/
│   └── test.rs
└── README.md
```

### 💻 Código do Contrato

```rust
#![no_std]
use soroban_sdk::{
    contract, contractimpl, contracttype, symbol_short, Address, Env, String, Vec, Symbol,
};

#[contracttype]
pub struct VoteOption {
    pub name: String,
    pub vote_count: u32,
}

#[contracttype]
pub struct Voter {
    pub address: Address,
    pub has_voted: bool,
    pub voted_option: String,
}

#[contracttype]
pub struct VotingSession {
    pub title: String,
    pub options: Vec<String>,
    pub total_votes: u32,
    pub is_active: bool,
    pub created_at: u64,
    pub creator: Address,
}

#[contract]
pub struct VotingContract;

#[contractimpl]
impl VotingContract {
    pub fn create_vote(env: Env, title: String, options: Vec<String>, creator: Address) -> String {
        // Verificar se já existe uma votação ativa
        let session_key = symbol_short!("session");
        if env.storage().has(&session_key) {
            panic!("Já existe uma votação ativa");
        }

        // Criar nova sessão de votação
        let session = VotingSession {
            title,
            options,
            total_votes: 0,
            is_active: true,
            created_at: env.ledger().timestamp(),
            creator,
        };

        // Salvar sessão
        env.storage().set(&session_key, &session);
        
        "Votação criada com sucesso".into()
    }

    pub fn vote(env: Env, voter: Address, option: String) -> bool {
        // Verificar se já votou
        if Self::has_voted(&env, voter.clone()) {
            panic!("Endereço já votou");
        }

        // Obter sessão atual
        let session_key = symbol_short!("session");
        let mut session: VotingSession = env.storage().get(&session_key).unwrap();
        
        // Verificar se votação está ativa
        if !session.is_active {
            panic!("Votação não está ativa");
        }

        // Verificar se opção é válida
        let mut option_valid = false;
        for opt in &session.options {
            if *opt == option {
                option_valid = true;
                break;
            }
        }
        if !option_valid {
            panic!("Opção inválida");
        }

        // Registrar voto
        let voter_key = Symbol::new(&env, &format!("voter_{}", voter.to_string()));
        let voter_data = Voter {
            address: voter.clone(),
            has_voted: true,
            voted_option: option.clone(),
        };
        env.storage().set(&voter_key, &voter_data);

        // Atualizar contagem
        session.total_votes += 1;
        env.storage().set(&session_key, &session);

        true
    }

    pub fn get_results(env: Env) -> Vec<VoteOption> {
        let session_key = symbol_short!("session");
        let session: VotingSession = env.storage().get(&session_key).unwrap();
        
        let mut results = Vec::new(&env);
        
        for option_name in &session.options {
            let mut vote_count = 0;
            
            // Contar votos para esta opção
            // (Implementação simplificada - em produção seria mais eficiente)
            for i in 0..session.total_votes {
                let voter_key = Symbol::new(&env, &format!("voter_{}", i));
                if let Ok(voter) = env.storage().get::<Voter>(&voter_key) {
                    if voter.voted_option == *option_name {
                        vote_count += 1;
                    }
                }
            }
            
            results.push_back(VoteOption {
                name: option_name.clone(),
                vote_count,
            });
        }
        
        results
    }

    pub fn has_voted(env: Env, voter: Address) -> bool {
        let voter_key = Symbol::new(&env, &format!("voter_{}", voter.to_string()));
        env.storage().has(&voter_key)
    }

    pub fn get_session_info(env: Env) -> VotingSession {
        let session_key = symbol_short!("session");
        env.storage().get(&session_key).unwrap()
    }

    pub fn end_vote(env: Env, creator: Address) -> bool {
        let session_key = symbol_short!("session");
        let mut session: VotingSession = env.storage().get(&session_key).unwrap();
        
        // Verificar se é o criador
        if session.creator != creator {
            panic!("Apenas o criador pode finalizar a votação");
        }
        
        session.is_active = false;
        env.storage().set(&session_key, &session);
        
        true
    }
}

#[cfg(test)]
mod test;
```

---

## 🧪 Testes

### 📝 Testes Unitários

```rust
#[cfg(test)]
mod test {
    use super::*;
    use soroban_sdk::testutils::{Address as _,};

    #[test]
    fn test_create_vote() {
        let env = Env::default();
        let contract = VotingContract;
        
        let creator = Address::random(&env);
        let title = String::from_slice(&env, "Melhor Linguagem");
        let mut options = Vec::new(&env);
        options.push_back(String::from_slice(&env, "Rust"));
        options.push_back(String::from_slice(&env, "Python"));
        
        let result = contract.create_vote(&env, title, options, creator.clone());
        assert_eq!(result, String::from_slice(&env, "Votação criada com sucesso"));
        
        let session = contract.get_session_info(&env);
        assert_eq!(session.title, String::from_slice(&env, "Melhor Linguagem"));
        assert_eq!(session.is_active, true);
    }

    #[test]
    fn test_vote() {
        let env = Env::default();
        let contract = VotingContract;
        
        // Criar votação
        let creator = Address::random(&env);
        let title = String::from_slice(&env, "Test Vote");
        let mut options = Vec::new(&env);
        options.push_back(String::from_slice(&env, "Option A"));
        options.push_back(String::from_slice(&env, "Option B"));
        
        contract.create_vote(&env, title, options, creator);
        
        // Votar
        let voter = Address::random(&env);
        let result = contract.vote(&env, voter.clone(), String::from_slice(&env, "Option A"));
        assert_eq!(result, true);
        
        // Verificar se votou
        assert_eq!(contract.has_voted(&env, voter), true);
    }

    #[test]
    #[should_panic(expected = "Endereço já votou")]
    fn test_double_vote() {
        let env = Env::default();
        let contract = VotingContract;
        
        // Setup
        let creator = Address::random(&env);
        let title = String::from_slice(&env, "Test");
        let mut options = Vec::new(&env);
        options.push_back(String::from_slice(&env, "A"));
        
        contract.create_vote(&env, title, options, creator);
        
        // Votar duas vezes
        let voter = Address::random(&env);
        contract.vote(&env, voter.clone(), String::from_slice(&env, "A"));
        contract.vote(&env, voter, String::from_slice(&env, "A")); // Deve falhar
    }
}
```

---

## 🎯 Exemplos de Uso

### 📋 Comandos CLI

```bash
# 1. Criar votação
stellar contract invoke \
  --id <contract_id> \
  --source Bob \
  --network testnet \
  -- create_vote \
  title "Qual é a melhor linguagem para blockchain?" \
  options '["Rust", "Python", "JavaScript", "Solidity"]' \
  creator "GBOB..."

# 2. Votar
stellar contract invoke \
  --id <contract_id> \
  --source Alice \
  --network testnet \
  -- vote \
  voter "GALICE..." \
  option "Rust"

# 3. Verificar se votou
stellar contract invoke \
  --id <contract_id> \
  --source Bob \
  --network testnet \
  -- has_voted \
  voter "GALICE..."

# 4. Obter resultados
stellar contract invoke \
  --id <contract_id> \
  --source Bob \
  --network testnet \
  -- get_results

# 5. Finalizar votação
stellar contract invoke \
  --id <contract_id> \
  --source Bob \
  --network testnet \
  -- end_vote \
  creator "GBOB..."
```

### 📊 Exemplo de Resultados

```json
{
  "results": [
    {
      "name": "Rust",
      "vote_count": 15
    },
    {
      "name": "Python", 
      "vote_count": 8
    },
    {
      "name": "JavaScript",
      "vote_count": 12
    },
    {
      "name": "Solidity",
      "vote_count": 5
    }
  ],
  "total_votes": 40,
  "winner": "Rust"
}
```

---

## 🔒 Funcionalidades de Segurança

### ✅ Validações Implementadas

- **Voto Único**: Cada endereço pode votar apenas uma vez
- **Opção Válida**: Só aceita votos em opções existentes
- **Votação Ativa**: Só aceita votos em votações ativas
- **Criador Único**: Apenas o criador pode finalizar a votação

### 🚧 Melhorias Futuras

- **Período de Votação**: Adicionar início e fim da votação
- **Votação Secreta**: Não revelar quem votou em quê
- **Múltiplos Votos**: Permitir múltiplos votos por endereço
- **Pesos de Voto**: Sistema de pesos diferentes por votante
- **Delegação**: Sistema de proxy voting

---

## 🎨 Integração com Frontend

### 📱 Interface Web (Próxima Aula)

```javascript
// Exemplo de integração futura
const votingContract = {
  async createVote(title, options) {
    return await stellar.invoke(contractId, 'create_vote', {
      title,
      options,
      creator: wallet.address
    });
  },
  
  async vote(option) {
    return await stellar.invoke(contractId, 'vote', {
      voter: wallet.address,
      option
    });
  },
  
  async getResults() {
    return await stellar.invoke(contractId, 'get_results');
  }
};
```

### 🎯 Componentes da Interface

- **Criar Votação**: Formulário para criar nova votação
- **Lista de Votações**: Mostrar votações ativas
- **Interface de Voto**: Botões para cada opção
- **Resultados em Tempo Real**: Gráficos e estatísticas
- **Histórico**: Votações finalizadas

---

## 🚀 Deploy e Testes

### 📦 Compilação

```bash
# Compilar contrato
stellar contract build

# Otimizar
stellar contract optimize

# Verificar tamanho
ls -la target/wasm32-unknown-unknown/release/
```

### 🧪 Testes Automatizados

```bash
# Executar testes
cargo test --target wasm32-unknown-unknown

# Testes específicos
cargo test test_vote
cargo test test_double_vote
```

### 🌐 Deploy na Testnet

```bash
# Deploy
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/voting_system_optimized.wasm \
  --source Bob \
  --network testnet

# Verificar no Stellar Expert
# https://testnet.stellar.expert/
```

---

## 📚 Recursos Adicionais

### 🔗 Documentação
- [Soroban Storage](https://soroban.stellar.org/docs/fundamentals-and-concepts/storage)
- [Soroban SDK](https://soroban.stellar.org/docs/reference/sdk)
- [Stellar CLI](https://soroban.stellar.org/docs/getting-started/setup)

### 🛠️ Ferramentas
- [Stellar Expert](https://stellar.expert/) - Explorador de blockchain
- [Freighter](https://www.freighter.app/) - Carteira para desenvolvimento
- [Rust Playground](https://play.rust-lang.org/) - Testar código Rust

### 💡 Dicas
- Sempre teste localmente antes do deploy
- Use otimização para reduzir custos
- Implemente validações robustas
- Documente todas as funções

---

*Desenvolvido para Meridian Hackathon 2025 - Road to Meridian Workshop 2*
