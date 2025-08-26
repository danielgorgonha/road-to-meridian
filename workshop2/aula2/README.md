# Workshop 2: Smartcontracts Básico na Stellar com Soroban

## Aula 2: Smart Contract + Frontend Descentralizado (Tap Game)

### 📚 O que aprendemos nesta aula

Nesta aula, aprendemos como criar um **jogo descentralizado na blockchain** com frontend conectando diretamente ao smart contract. O resultado foi o projeto **Tap Game** que está disponível no repositório: [nearx-tap-game](https://github.com/danielgorgonha/nearx-tap-game)

### 🎯 Principais conceitos aprendidos

- ✅ **Storage na Stellar**: Como armazenar dados no smart contract
- ✅ **Smart Contract**: Estrutura do jogo Tap Game
- ✅ **Frontend Descentralizado**: Interface React conectando diretamente
- ✅ **Sistema Completo**: Jogo funcional sem backend centralizado

---

## 💾 Storage na Stellar

### Tipos de Storage

Na Stellar, existem 3 tipos principais de storage:

1. **Instance**: Dados do contrato (vida útil atrelada ao contrato)
2. **Persistent**: Dados independentes (podem expirar separadamente)
3. **Temporary**: Dados temporários (vida útil curta)

### Sistema de Aluguel

- Contratos e dados são **alugados**, não "pagos para sempre"
- Precisa renovar periodicamente

---

## 🎮 Tap Game - Smart Contract

### Estrutura do Jogo

```rust
#[contracttype]
pub struct Game {
    pub player: Address,
    pub score: u32,
    pub nickname: String,
}

#[contractimpl]
impl TapContract {
    // Salvar pontuação do jogador
    pub fn save_game(env: Env, player: Address, score: u32, nickname: String)
    
    // Obter ranking dos jogadores
    pub fn get_ranking(env: Env) -> Vec<Game>
    
    // Inicializar o contrato
    pub fn initialize(env: Env)
}
```

### Como funciona

1. **Jogador clica** no botão no frontend
2. **Frontend chama** a função `save_game` no contrato
3. **Contrato salva** a pontuação na blockchain
4. **Ranking é atualizado** automaticamente

---

## 🌐 Frontend React Descentralizado

### Abordagem Web2-like

- **Geração automática** de carteiras (sem wallet complexa)
- **Interface simples** para jogar
- **Ranking em tempo real**
- **Conexão direta** com smart contract

### Tecnologias usadas

- **React** para interface
- **Stellar SDK** para interação com blockchain
- **TypeScript** para tipagem

### Arquitetura Descentralizada

```
Frontend React
    ↓ (conexão direta)
Smart Contract (Rust)
    ↓ (dados na blockchain)
Stellar Network
```

**Sem backend centralizado!** O frontend conecta diretamente ao smart contract.

---

## 🔗 Integração Frontend + Smart Contract

### Fluxo do Jogo

```
1. Usuário acessa o site
2. Sistema gera carteira automaticamente
3. Usuário clica para jogar
4. Frontend chama smart contract DIRETAMENTE
5. Pontuação é salva na blockchain
6. Ranking é atualizado
```

### Comandos principais

```bash
# Deploy do contrato
stellar contract deploy --wasm tap_game.wasm

# Interagir com o contrato
stellar contract invoke --id <contract_id> -- save_game player "G..." score 100 nickname "Player1"
```

---

## 📁 Estrutura do Projeto

O projeto completo está em: [nearx-tap-game](https://github.com/danielgorgonha/nearx-tap-game)

```
nearx-tap-game/
├── web3/           # Smart Contract (Rust)
├── frontend/       # Interface React (DESCENTRALIZADO)
│                   # └── Conecta DIRETAMENTE ao smart contract
└── backend/        # Demonstração Python (próxima aula)
                    # └── Exemplo de outras linguagens
```

---

## 🎯 Próximos Passos

### Aula 3 (Próxima)
- **Stellar SDK Python** para demonstrar outras linguagens
- **Mesma funcionalidade** em Python
- **Multi-linguagem** - não é backend para o frontend

### Desafios Práticos
- ✅ Implementar o Tap Game descentralizado
- ✅ Adicionar mais funcionalidades
- ✅ Criar interface personalizada
- ✅ Explorar outras funcionalidades do smart contract

---

## 📚 Recursos

- **Repositório**: [nearx-tap-game](https://github.com/danielgorgonha/nearx-tap-game)
- **Documentação Stellar**: [soroban.stellar.org](https://soroban.stellar.org)
- **Stellar SDK**: [stellar-sdk](https://github.com/stellar/js-stellar-sdk)

---

*Desenvolvido para Meridian Hackathon 2025 - Rio de Janeiro*



