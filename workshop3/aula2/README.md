# Aula 2: Composabilidade entre Contratos

## 🎯 **O que Aprendemos**

### **Conceitos Fundamentais de Composabilidade**
- **Cross Contract Calls**: Interação entre contratos inteligentes
- **Autenticação em Profundidade**: Passagem de contexto de autenticação entre contratos
- **Deploy Pattern**: Criação dinâmica de contratos
- **Upgrade Pattern**: Atualização de contratos mantendo o estado

---

## 🔗 **Cross Contract Calls - O Coração da Composabilidade**

### **Conceito Fundamental**
- **Composabilidade** é usar seu contrato para interagir com outro contrato na rede
- **Pensar como Lego**: Contratos se encaixam como blocos de construção
- **Separação de Responsabilidades**: Dividir lógica em múltiplos contratos especializados

### **Por que Usar Cross Contract Calls?**
- ✅ **Modularidade** - Cada contrato tem uma responsabilidade específica
- ✅ **Reutilização** - Usar protocolos existentes (Soroban, Reflector, Blend)
- ✅ **Eficiência** - Não reinventar a roda, usar o que já existe
- ✅ **Manutenibilidade** - Código mais organizado e fácil de manter

### **Como Funciona?**
```rust
// Interface para outro contrato
pub trait FlipperInterface {
    fn get_flipper_value(&self, contract_address: &Address) -> Result<u32, Error>;
    fn flip(&self, contract_address: &Address) -> Result<(), Error>;
}

// Implementação da interface
impl FlipperInterface for Client {
    fn get_flipper_value(&self, contract_address: &Address) -> Result<u32, Error> {
        let client = Client::new(&self.env, contract_address);
        client.get_flipper_value()
    }
}
```

### **Vantagens sobre Solidity**
- **Tratamento de Erros**: Usando `Result<Ok, Error>` do Rust
- **Type Safety**: Compilador Rust previne muitos erros
- **Abstração de Custo Zero**: Performance sem perder legibilidade

---

## 🔐 **Autenticação em Profundidade**

### **O Problema**
Quando o **User A** chama o **Contrato A** que chama o **Contrato B**:
- Contrato B recebe o endereço do Contrato A (não do User A)
- Precisa passar o contexto de autenticação para frente

### **A Solução**
```rust
// Passar autenticação para o próximo contrato
pub fn call_other_contract(&self, env: &Env, user: &Address) -> Result<(), Error> {
    // Autenticar o usuário original
    user.require_auth();
    
    // Chamar outro contrato passando o contexto
    let other_contract = Client::new(env, &other_contract_address);
    other_contract.some_function(user); // Passa o user original
}
```

### **Padrões de Segurança**
- **Princípio do Menor Privilégio**: Usuários têm apenas permissões necessárias
- **RBAC (Role-Based Access Control)**: Controle de acesso baseado em funções
- **Validação de Parâmetros**: Sempre validar inputs antes de passar adiante

### **Caso de Uso Real**
- **Soroban Protocol**: Múltiplos contratos (Router, Factory, Pool)
- **Fluxo**: User → Router → Factory → Pool
- **Autenticação**: Passa contexto do usuário original através de toda a cadeia

---

## 🚀 **Deploy Pattern - Criação Dinâmica de Contratos**

### **Conceito**
- **Deploy dinâmico**: Contrato que cria outros contratos na blockchain
- **On-chain deployment**: Tudo acontece na blockchain, sem backend
- **Eficiência**: Criação de contratos sob demanda

### **Exemplo Prático: Loja de Aplicativos**
```rust
pub fn buy_notepad(&self, env: &Env, buyer: &Address, name: String) -> Result<Address, Error> {
    // 1. Validar pagamento
    self.validate_payment(env, buyer)?;
    
    // 2. Coletar pagamento
    self.collect_payment(env, buyer)?;
    
    // 3. Fazer deploy do novo contrato
    let notepad_address = self.deploy_notepad(env, buyer, name)?;
    
    Ok(notepad_address)
}
```

### **Processo de Deploy**
1. **Upload do Código**: `stellar contract upload` - sobe o WASM
2. **Obter Hash**: Recebe hash que identifica o contrato
3. **Instanciar**: `stellar contract deploy` - cria instância com parâmetros
4. **Configurar**: Define admin e configurações iniciais

### **Vantagens**
- ✅ **Automação**: Deploy automático sem intervenção manual
- ✅ **Escalabilidade**: Cria contratos conforme demanda
- ✅ **Customização**: Cada contrato pode ter configurações únicas

---

## 🔄 **Upgrade Pattern - Atualização de Contratos**

### **O Problema**
- **Contratos são imutáveis** uma vez deployados
- **Como atualizar** sem perder dados?
- **Como adicionar funcionalidades** sem quebrar o sistema?

### **A Solução: Proxy Pattern**
```rust
pub fn upgrade(&self, env: &Env, new_wasm_hash: &BytesN<32>) -> Result<(), Error> {
    // Validar que apenas admin pode fazer upgrade
    self.require_admin(env)?;
    
    // Atualizar ponteiro para nova versão
    env.current_contract().set_wasm(new_wasm_hash);
    
    Ok(())
}
```

### **Como Funciona**
1. **Contrato Original**: Mantém o storage (dados)
2. **Novo Contrato**: Contém nova lógica
3. **Proxy**: Aponta para nova versão mantendo dados antigos
4. **Transparente**: Usuários não percebem a mudança

### **Vantagens sobre Solidity**
- **WebAssembly**: Facilita upgrades (binário executável)
- **Simplicidade**: Muito mais simples que Diamond Pattern
- **Segurança**: Menos complexidade = menos vulnerabilidades

### **Considerações de Governança**
- **Multi-signature**: Múltiplas pessoas devem aprovar upgrades
- **Timelock**: Período de espera antes do upgrade
- **Auditoria**: Código deve ser auditado antes do upgrade

---

## 🏗️ **Projeto Prático: Loja de Aplicativos**

### **Arquitetura do Sistema**
```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Store         │    │   Meridian Token │    │   Notepad V1    │
│   (Loja)        │◄──►│   (Token)        │    │   (App)         │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                                               │
         │ Deploy                                        │
         ▼                                               │
┌─────────────────┐                                     │
│   Notepad V2    │◄────────────────────────────────────┘
│   (App Atualizado)│
└─────────────────┘
```

### **Funcionalidades Implementadas**

#### **1. Store Contract (Loja)**
- ✅ **Validação de Pagamento**: Verifica se usuário aprovou tokens
- ✅ **Coleta de Pagamento**: Transfere tokens para a loja
- ✅ **Deploy Dinâmico**: Cria novos contratos Notepad
- ✅ **Gestão de Admin**: Apenas admin pode sacar fundos

#### **2. Meridian Token**
- ✅ **Token Fungível**: Implementação completa de token
- ✅ **Mint Inicial**: Cria tokens para o criador
- ✅ **Transferências**: Sistema completo de transferências

#### **3. Notepad V1 (Aplicativo Básico)**
- ✅ **CRUD de Notas**: Adicionar, buscar, listar notas
- ✅ **Controle de Acesso**: Apenas dono pode modificar
- ✅ **Contador**: Acompanha número de notas

#### **4. Notepad V2 (Aplicativo Avançado)**
- ✅ **Funcionalidades V1**: Mantém todas as funcionalidades
- ✅ **Cross Contract Calls**: Pode adicionar notas em outros contratos
- ✅ **Autenticação em Profundidade**: Passa contexto de autenticação
- ✅ **Upgrade Pattern**: Pode ser atualizado mantendo dados

### **Fluxo de Uso**
1. **Usuário compra app**: Chama `buy_notepad` na loja
2. **Validação**: Loja verifica pagamento
3. **Deploy**: Loja cria novo contrato Notepad
4. **Configuração**: Usuário vira admin do seu app
5. **Uso**: Usuário adiciona notas no seu app
6. **Upgrade**: Admin pode atualizar para V2
7. **Cross Contract**: V2 pode interagir com outros contratos

---

## 🛠️ **Implementação Técnica**

### **Storage Management**
```rust
// Usando update para operações atômicas
env.storage().persistent().update(
    &DataKey::Balance,
    |balance: Option<u32>| -> Result<u32, Error> {
        Ok(balance.unwrap_or(0) + amount)
    }
)?;
```

### **Error Handling**
```rust
// Tratamento robusto de erros
let wasm_hash = env.storage().persistent()
    .get(&DataKey::NotepadWasm)
    .ok_or(Error::from_type_and_code(1, 1))?;
```

### **Random Number Generation**
```rust
// Geração de números aleatórios para endereços únicos
let salt = env.prng().gen::<u32>();
```

---

## 🎯 **Aplicações no Meridian Hackathon**

### **Trilha de Composabilidade**
- **Obrigatório**: Usar protocolos Stellar (Soroban, Reflector, Blend)
- **Diferencial**: Implementar cross contract calls
- **Inovação**: Criar novos padrões de composição

### **Ideias de Projetos**
- **DeFi Composer**: Plataforma que combina múltiplos protocolos
- **NFT Marketplace**: Integração com tokens e liquidez
- **DAO Governance**: Sistema de votação com múltiplos contratos
- **Insurance Protocol**: Integração com oráculos e pools

### **Dicas para o Hackathon**
- ✅ **Comece cedo**: Faça deploy dos contratos antes do final
- ✅ **Teste tudo**: Valide todas as interações
- ✅ **Documente**: Salve hashes e endereços dos contratos
- ✅ **Backup**: Tenha planos B para problemas de rede

---

## 📚 **Recursos e Ferramentas**

### **OpenZeppelin Contracts**
- **Contratos Auditados**: Seguros para produção
- **Padrões Comprovados**: RBAC, Upgradeable, etc.
- **Múltiplas Linguagens**: Rust, Solidity, Cairo

### **Stellar CLI Commands**
```bash
# Upload do contrato
stellar contract upload --wasm target/wasm32-unknown-unknown/release/contract.wasm

# Deploy do contrato
stellar contract deploy --wasm target/wasm32-unknown-unknown/release/contract.wasm

# Invoke de função
stellar contract invoke --id <contract_id> --source <account> -- <function_name> <args>
```

### **Ferramentas de Desenvolvimento**
- **Stellar Expert**: Explorador de contratos
- **Stellar Laboratory**: Interface web para testes
- **Soroban CLI**: Ferramentas de desenvolvimento

---

## 🎓 **Conceitos Avançados**

### **Gas Optimization**
- **Estruturas Eficientes**: Use tipos otimizados
- **Caching**: Evite recálculos desnecessários
- **Batch Operations**: Agrupe operações quando possível

### **Security Patterns**
- **Reentrancy Protection**: Prevenção de ataques de reentrância
- **Access Control**: Controle rigoroso de permissões
- **Input Validation**: Validação de todos os inputs

### **Testing Strategy**
- **Unit Tests**: Teste cada função individualmente
- **Integration Tests**: Teste interações entre contratos
- **Fuzz Testing**: Teste com inputs aleatórios

---

## 🚀 **Próximos Passos**

### **Para o Meridian Hackathon**
- 🎯 **Implementar Cross Contract Calls** em seu projeto
- 🔐 **Aplicar Padrões de Segurança** aprendidos
- 🏗️ **Usar Deploy Pattern** para funcionalidades dinâmicas
- 🔄 **Considerar Upgrade Pattern** para evolução do projeto

### **Desafios Práticos**
1. **Criar um DeFi Composer** que integre múltiplos protocolos
2. **Implementar um Marketplace** com cross contract calls
3. **Desenvolver um DAO** com sistema de votação distribuído
4. **Construir um Insurance Protocol** com oráculos

---

## 💡 **Dicas do Professor Lucas**

### **Para Aprendizado**
- ❌ **NÃO use IA** para aprender do zero
- ❌ **NÃO use frameworks** na primeira vez
- ✅ **FAÇA na mão** para entender os fundamentos
- ✅ **ESCREVA código** com caneta e papel (melhor fixação)

### **Para Produção**
- ✅ **USE IA** para produtividade
- ✅ **USE frameworks** para eficiência
- ✅ **USE ferramentas** para otimização

### **Mentalidade de Desenvolvedor**
- **"Programe, não apenas estude"** - A prática é essencial
- **"Menos reuniões, mais código"** - Foco na execução
- **"Se está complexo, está errado"** - Simplicidade é chave

---

## 🎯 **Resumo da Aula**

### **O que Aprendemos:**
✅ **Cross Contract Calls** - Interação entre contratos inteligentes  
✅ **Autenticação em Profundidade** - Passagem de contexto de autenticação  
✅ **Deploy Pattern** - Criação dinâmica de contratos  
✅ **Upgrade Pattern** - Atualização mantendo estado  
✅ **Projeto Prático** - Loja de aplicativos completa  

### **Conceitos Técnicos:**
- **Composabilidade** como pilar fundamental da Web3
- **Separação de responsabilidades** em contratos
- **Gestão de estado** em upgrades
- **Padrões de segurança** para produção

### **Aplicação Prática:**
- **Sistema completo** de loja de aplicativos
- **Múltiplos contratos** interagindo
- **Upgrade pattern** implementado
- **Cross contract calls** funcionais

---

*Workshop 3, Aula 2 - Composabilidade entre Contratos*  
*Resumo dos entendimentos da aula*  
*Professor: Lucas Oliveira (Nerex Education)*

