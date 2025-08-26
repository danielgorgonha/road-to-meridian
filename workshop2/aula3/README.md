# Workshop 2: Smartcontracts Básico na Stellar com Soroban

## Aula 3: Smartcontracts e Integração com Frontend

### 📚 Resumo da Aula

Esta aula completa o Workshop 2 focando na integração de smart contracts com interfaces de usuário, criando aplicações web completas que interagem com a blockchain Stellar.

### 🎯 Objetivos da Aula

- ✅ Criar interfaces de usuário para smart contracts
- ✅ Integrar frontend com Stellar Network
- ✅ Desenvolver aplicações web completas
- ✅ Implementar wallets e autenticação
- ✅ Deploy de aplicações full-stack

---

## 🌐 Frontend para Smart Contracts

### Arquitetura Frontend

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   React     │───▶│   API       │───▶│   Stellar   │
│   App       │    │  Backend    │    │  Network    │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │
       ▼                   ▼
┌─────────────┐    ┌─────────────┐
│   Wallet    │    │   Cache     │
│  Connect    │    │  Layer      │
└─────────────┘    └─────────────┘
```

### Tecnologias Frontend

#### **React + TypeScript**
- **Vantagens**: Componentização, tipagem forte, ecossistema rico
- **Integração**: Hooks para interação com blockchain
- **Estado**: Context API para gerenciar wallet e contratos

#### **Stellar SDK**
- **Função**: Cliente JavaScript para Stellar
- **Recursos**: Criação de transações, assinatura, envio
- **Integração**: Hooks customizados

---

## 🔗 Integração Frontend-Backend

### Padrões de Comunicação

#### **1. REST API Pattern**
```typescript
// api/stellar.ts
export class StellarAPI {
  private baseURL: string;
  
  constructor(baseURL: string) {
    this.baseURL = baseURL;
  }
  
  async invokeContract(contractId: string, function: string, args: any[]) {
    const response = await fetch(`${this.baseURL}/contract/invoke`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        contractId,
        function,
        args,
      }),
    });
    
    return response.json();
  }
}
```

#### **2. WebSocket Pattern**
```typescript
// hooks/useStellarWebSocket.ts
import { useEffect, useState } from 'react';

export function useStellarWebSocket(contractId: string) {
  const [events, setEvents] = useState([]);
  
  useEffect(() => {
    const ws = new WebSocket(`ws://localhost:8080/contract/${contractId}`);
    
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      setEvents(prev => [...prev, data]);
    };
    
    return () => ws.close();
  }, [contractId]);
  
  return events;
}
```

---

## 👛 Wallet Integration

### Stellar Wallet Adapter

```typescript
// hooks/useWallet.ts
import { useState, useEffect } from 'react';
import { StellarSdk } from 'stellar-sdk';

export function useWallet() {
  const [wallet, setWallet] = useState(null);
  const [publicKey, setPublicKey] = useState(null);
  const [connected, setConnected] = useState(false);
  
  const connect = async () => {
    try {
      // Detectar wallet disponível
      if (window.freighter) {
        const freighter = window.freighter;
        await freighter.enable();
        const publicKey = await freighter.getPublicKey();
        
        setWallet(freighter);
        setPublicKey(publicKey);
        setConnected(true);
      }
    } catch (error) {
      console.error('Failed to connect wallet:', error);
    }
  };
  
  const disconnect = () => {
    setWallet(null);
    setPublicKey(null);
    setConnected(false);
  };
  
  const signTransaction = async (transaction: any) => {
    if (!wallet) throw new Error('Wallet not connected');
    return await wallet.signTransaction(transaction);
  };
  
  return {
    wallet,
    publicKey,
    connected,
    connect,
    disconnect,
    signTransaction,
  };
}
```

### Componente de Conexão

```typescript
// components/WalletConnect.tsx
import React from 'react';
import { useWallet } from '../hooks/useWallet';

export function WalletConnect() {
  const { connected, publicKey, connect, disconnect } = useWallet();
  
  return (
    <div className="wallet-connect">
      {connected ? (
        <div>
          <span>Connected: {publicKey?.slice(0, 8)}...</span>
          <button onClick={disconnect}>Disconnect</button>
        </div>
      ) : (
        <button onClick={connect}>Connect Wallet</button>
      )}
    </div>
  );
}
```

---

## 🎨 Interface de Usuário

### Componente de Smart Contract

```typescript
// components/SmartContractInterface.tsx
import React, { useState } from 'react';
import { useWallet } from '../hooks/useWallet';
import { StellarAPI } from '../api/stellar';

interface SmartContractInterfaceProps {
  contractId: string;
  functions: string[];
}

export function SmartContractInterface({ contractId, functions }: SmartContractInterfaceProps) {
  const { connected, publicKey, signTransaction } = useWallet();
  const [selectedFunction, setSelectedFunction] = useState('');
  const [args, setArgs] = useState('');
  const [result, setResult] = useState('');
  const [loading, setLoading] = useState(false);
  
  const api = new StellarAPI('http://localhost:8080');
  
  const invokeFunction = async () => {
    if (!connected) {
      alert('Please connect your wallet first');
      return;
    }
    
    setLoading(true);
    try {
      const parsedArgs = JSON.parse(args);
      const response = await api.invokeContract(contractId, selectedFunction, parsedArgs);
      setResult(JSON.stringify(response, null, 2));
    } catch (error) {
      setResult(`Error: ${error.message}`);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="smart-contract-interface">
      <h3>Smart Contract: {contractId}</h3>
      
      <div className="function-selector">
        <label>Function:</label>
        <select value={selectedFunction} onChange={(e) => setSelectedFunction(e.target.value)}>
          <option value="">Select a function</option>
          {functions.map(func => (
            <option key={func} value={func}>{func}</option>
          ))}
        </select>
      </div>
      
      <div className="args-input">
        <label>Arguments (JSON):</label>
        <textarea
          value={args}
          onChange={(e) => setArgs(e.target.value)}
          placeholder='["arg1", "arg2"]'
        />
      </div>
      
      <button 
        onClick={invokeFunction} 
        disabled={!connected || !selectedFunction || loading}
      >
        {loading ? 'Executing...' : 'Invoke Function'}
      </button>
      
      {result && (
        <div className="result">
          <h4>Result:</h4>
          <pre>{result}</pre>
        </div>
      )}
    </div>
  );
}
```

---

## 🎯 Aplicação Completa: Sistema de Votação

### Smart Contract de Votação

```rust
#![no_std]
use soroban_sdk::{contractimpl, contracttype, Env, Symbol, Address, Vec, String};

#[contracttype]
#[derive(Clone, Debug, Eq, PartialEq)]
pub struct Poll {
    pub id: String,
    pub question: String,
    pub options: Vec<String>,
    pub votes: Vec<u32>,
    pub voters: Vec<Address>,
    pub active: bool,
}

pub struct VotingContract;

#[contractimpl]
impl VotingContract {
    pub fn create_poll(env: Env, creator: Address, question: String, options: Vec<String>) -> String {
        let poll_id = format!("poll_{}", env.ledger().timestamp());
        
        let votes = Vec::new(&env);
        for _ in 0..options.len() {
            votes.push_back(0);
        }
        
        let voters = Vec::new(&env);
        
        let poll = Poll {
            id: poll_id.clone(),
            question,
            options,
            votes,
            voters,
            active: true,
        };
        
        env.storage().set(&poll_id, &poll);
        poll_id
    }
    
    pub fn vote(env: Env, voter: Address, poll_id: String, option_index: u32) -> bool {
        let mut poll: Poll = env.storage().get(&poll_id).unwrap();
        
        // Verificar se já votou
        for voter_addr in poll.voters.iter() {
            if voter_addr == voter {
                return false; // Já votou
            }
        }
        
        // Adicionar voto
        let mut new_votes = Vec::new(&env);
        for (i, vote_count) in poll.votes.iter().enumerate() {
            if i == option_index as usize {
                new_votes.push_back(vote_count + 1);
            } else {
                new_votes.push_back(vote_count);
            }
        }
        
        poll.votes = new_votes;
        poll.voters.push_back(voter);
        
        env.storage().set(&poll_id, &poll);
        true
    }
    
    pub fn get_poll(env: Env, poll_id: String) -> Poll {
        env.storage().get(&poll_id).unwrap()
    }
    
    pub fn get_polls(env: Env) -> Vec<String> {
        // Implementar listagem de polls
        Vec::new(&env)
    }
}
```

### Frontend para Sistema de Votação

```typescript
// components/VotingSystem.tsx
import React, { useState, useEffect } from 'react';
import { useWallet } from '../hooks/useWallet';
import { StellarAPI } from '../api/stellar';

export function VotingSystem() {
  const { connected, publicKey } = useWallet();
  const [polls, setPolls] = useState([]);
  const [newPoll, setNewPoll] = useState({ question: '', options: ['', ''] });
  const [loading, setLoading] = useState(false);
  
  const api = new StellarAPI('http://localhost:8080');
  
  const createPoll = async () => {
    if (!connected) return;
    
    setLoading(true);
    try {
      const options = newPoll.options.filter(opt => opt.trim() !== '');
      await api.invokeContract('VOTING_CONTRACT_ID', 'create_poll', [
        publicKey,
        newPoll.question,
        options
      ]);
      
      setNewPoll({ question: '', options: ['', ''] });
      loadPolls();
    } catch (error) {
      console.error('Failed to create poll:', error);
    } finally {
      setLoading(false);
    }
  };
  
  const vote = async (pollId: string, optionIndex: number) => {
    if (!connected) return;
    
    try {
      await api.invokeContract('VOTING_CONTRACT_ID', 'vote', [
        publicKey,
        pollId,
        optionIndex
      ]);
      
      loadPolls();
    } catch (error) {
      console.error('Failed to vote:', error);
    }
  };
  
  const loadPolls = async () => {
    try {
      const pollsData = await api.invokeContract('VOTING_CONTRACT_ID', 'get_polls', []);
      setPolls(pollsData);
    } catch (error) {
      console.error('Failed to load polls:', error);
    }
  };
  
  useEffect(() => {
    loadPolls();
  }, []);
  
  return (
    <div className="voting-system">
      <h2>Voting System</h2>
      
      {connected && (
        <div className="create-poll">
          <h3>Create New Poll</h3>
          <input
            type="text"
            placeholder="Question"
            value={newPoll.question}
            onChange={(e) => setNewPoll({ ...newPoll, question: e.target.value })}
          />
          
          {newPoll.options.map((option, index) => (
            <input
              key={index}
              type="text"
              placeholder={`Option ${index + 1}`}
              value={option}
              onChange={(e) => {
                const newOptions = [...newPoll.options];
                newOptions[index] = e.target.value;
                setNewPoll({ ...newPoll, options: newOptions });
              }}
            />
          ))}
          
          <button onClick={() => setNewPoll({ ...newPoll, options: [...newPoll.options, ''] })}>
            Add Option
          </button>
          
          <button onClick={createPoll} disabled={loading}>
            {loading ? 'Creating...' : 'Create Poll'}
          </button>
        </div>
      )}
      
      <div className="polls-list">
        <h3>Active Polls</h3>
        {polls.map((poll: any) => (
          <div key={poll.id} className="poll">
            <h4>{poll.question}</h4>
            {poll.options.map((option: string, index: number) => (
              <div key={index} className="poll-option">
                <span>{option}</span>
                <span>Votes: {poll.votes[index]}</span>
                {connected && (
                  <button onClick={() => vote(poll.id, index)}>
                    Vote
                  </button>
                )}
              </div>
            ))}
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## 🎨 Styling e UX

### CSS Modules

```css
/* VotingSystem.module.css */
.votingSystem {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.createPoll {
  background: #f5f5f5;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.createPoll input {
  width: 100%;
  padding: 10px;
  margin: 5px 0;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.poll {
  border: 1px solid #ddd;
  padding: 15px;
  margin: 10px 0;
  border-radius: 8px;
}

.pollOption {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  margin: 5px 0;
  background: #f9f9f9;
  border-radius: 4px;
}

.button {
  background: #007bff;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
}

.button:disabled {
  background: #ccc;
  cursor: not-allowed;
}
```

---

## 🚀 Deploy e Produção

### Build e Deploy Frontend

```bash
# Build da aplicação React
npm run build

# Deploy no Vercel
vercel --prod

# Ou deploy no Netlify
netlify deploy --prod --dir=build
```

### Configuração de Ambiente

```typescript
// config/environment.ts
export const config = {
  development: {
    apiUrl: 'http://localhost:8080',
    network: 'testnet',
    contractId: 'TEST_CONTRACT_ID',
  },
  production: {
    apiUrl: 'https://api.yourapp.com',
    network: 'public',
    contractId: 'PROD_CONTRACT_ID',
  },
};

export const currentConfig = config[process.env.NODE_ENV || 'development'];
```

### Docker Compose para Full-Stack

```yaml
# docker-compose.yml
version: '3.8'
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://backend:8080
    depends_on:
      - backend
  
  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      - STELLAR_NETWORK=testnet
      - CONTRACT_ID=${CONTRACT_ID}
    volumes:
      - ./backend:/app
```

---

## 🧪 Testes Frontend

### Testes de Componentes

```typescript
// __tests__/VotingSystem.test.tsx
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import { VotingSystem } from '../components/VotingSystem';

// Mock do hook useWallet
jest.mock('../hooks/useWallet', () => ({
  useWallet: () => ({
    connected: true,
    publicKey: 'G...',
    connect: jest.fn(),
    disconnect: jest.fn(),
  }),
}));

test('renders voting system', () => {
  render(<VotingSystem />);
  expect(screen.getByText('Voting System')).toBeInTheDocument();
});

test('creates new poll', async () => {
  render(<VotingSystem />);
  
  const questionInput = screen.getByPlaceholderText('Question');
  fireEvent.change(questionInput, { target: { value: 'Test question?' } });
  
  const createButton = screen.getByText('Create Poll');
  fireEvent.click(createButton);
  
  // Verificar se o poll foi criado
  expect(await screen.findByText('Test question?')).toBeInTheDocument();
});
```

---

## 🎯 Próximos Passos

### Workshop 3 (Futuro)
- Segurança avançada em smart contracts
- Composabilidade entre contratos
- Autenticação Passkey (FIDO)
- Otimizações de performance

### Desafios Práticos
- ✅ Implementar sistema de NFTs
- ✅ Criar DEX simples
- ✅ Adicionar gráficos de votação
- ✅ Implementar notificações em tempo real

---

## 📚 Recursos Adicionais

### Documentação
- [Stellar JavaScript SDK](https://stellar.github.io/js-stellar-sdk/)
- [React Documentation](https://reactjs.org/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

### Ferramentas
- [Stellar Laboratory](https://laboratory.stellar.org/)
- [Freighter Wallet](https://www.freighter.app/)
- [Vercel](https://vercel.com/) / [Netlify](https://netlify.com/)

---

*Desenvolvido para Meridian Hackathon 2025 - Rio de Janeiro*



