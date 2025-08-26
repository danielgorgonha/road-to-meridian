# Workshop 1: Introdução ao Rust

## 📚 Visão Geral

O Workshop 1 é o primeiro nível da série "Road to Meridian", focado nos fundamentos da linguagem Rust e desenvolvimento de aplicações básicas. Este workshop estabelece as bases necessárias para os workshops avançados de smart contracts.

### 🎯 Objetivos do Workshop

- ✅ Dominar os fundamentos da linguagem Rust
- ✅ Criar e publicar bibliotecas no Crates.io
- ✅ Desenvolver APIs REST completas
- ✅ Integrar WebAssembly com Rust
- ✅ Preparar para desenvolvimento de smart contracts

---

## 📋 Estrutura das Aulas

### [📖 Aula 1: Criar e Publicar Bibliotecas em Rust](./aula1/README.md)
**Conteúdo:** Introdução ao Rust, configuração do ambiente, criação de bibliotecas, testes e publicação

**Principais tópicos:**
- 🛠️ Instalação e configuração do ambiente Rust
- 🦀 Hello World e primeiros passos
- 📝 Types, functions, and modules
- 🧮 Criação de biblioteca calculadora
- 🧪 Testes automatizados
- 📦 Publicação no Crates.io
- 💻 Programa interativo com input do usuário

**[📋 Ver conteúdo da Aula 1](./aula1/README.md)** | **[🎥 Assistir Aula 1 no YouTube](https://www.youtube.com/watch?v=aF98JOeoKV8&ab_channel=NearX)**

---

### [🚀 Aula 2: Criar e Deployar Rest API CRUD em Rust](./aula2/README.md)
**Conteúdo:** Desenvolvimento completo de APIs REST com Rust

**Principais tópicos:**
- 🔄 Operações CRUD (Create, Read, Update, Delete)
- 🛠️ Framework web Tide e runtime async-std
- 🔒 Gerenciamento de estado concorrente com Arc<Mutex<HashMap>>
- 📡 Endpoints REST API e métodos HTTP
- 🧪 Testes manuais com cURL
- 🚀 Deploy em produção com Railway

**[📋 Ver conteúdo da Aula 2](./aula2/README.md)** | **[🎥 Assistir Aula 2 no YouTube](https://www.youtube.com/watch?v=XozUaYcCQoo&t=2s&ab_channel=NearX)** | **[🚀 Demo Live](https://learn-rust-crud-production.up.railway.app)**

---

### [🌐 Aula 3: Criar e Integrar WebAssembly em Rust](./aula3/README.md)
**Conteúdo:** Desenvolvimento de aplicações WebAssembly com Rust, integrando módulos WASM em APIs CRUD

**Principais tópicos:**
- ⚡ Introdução ao WebAssembly (WASM, WASI, WAT)
- 🔧 Compilação e otimização Rust para WASM
- 🌐 Runtimes WebAssembly (Wasmi, Wasmtime, Wasmer)
- 🚀 Integração com APIs CRUD (CRUDE - Create, Read, Update, Delete, Execute)
- 🎯 Aplicações práticas e casos de uso blockchain

**[📋 Ver conteúdo da Aula 3](./aula3/README.md)** | **[🎥 Assistir Aula 3 no YouTube](https://www.youtube.com/watch?v=vcP9JGYUZHo&ab_channel=NearX)**

---

## 🏆 Desafios do Workshop 1

### Desafio Aula 1 - ✅ COMPLETADO
- ✅ Adicionar função power e sua inversa, logarithm à biblioteca **calculator** em **calc3.rs**
- ✅ Escrever testes e publicar a nova versão (0.2.0) no **crates.io**

### Desafio Aula 2 - ✅ COMPLETADO
- ✅ **Autenticação JWT**: Sistema completo de autenticação baseado em tokens implementado
- ✅ **Refresh Tokens**: Mecanismo de renovação automática de tokens com lifecycle seguro
- ✅ **Variáveis de Ambiente**: Integração dotenv para gerenciamento seguro de configuração
- ✅ **Deploy em Produção**: API live deployada na plataforma Railway
- ✅ **Collection Postman**: Documentação completa de testes com arquivos de importação
- ✅ **Testes Automatizados**: Suite abrangente de testes com scripts shell

### Desafio Aula 3 - ✅ COMPLETADO
- ✅ **Criação de Módulo WASM**: Criadas 9 funções matemáticas e compiladas para WebAssembly
- ✅ **Integração API CRUDE**: API CRUD estendida com funcionalidade execute
- ✅ **Execução Dinâmica**: Implementado carregamento e execução de módulos WASM em runtime
- ✅ **Autenticação Avançada**: JWT + Refresh tokens + Acesso owner-only
- ✅ **Otimização de Performance**: Cache de módulos + Rate limiting + Métricas
- ✅ **Deploy em Produção**: API live no Railway com testes abrangentes
- ✅ **Recursos Enterprise**: Documentação completa + Collection Postman + Testes automatizados

---

## 🚀 **Expansões de Desenvolvedor Sênior**

### **🎯 Implementações Avançadas**

Baseado em experiência de desenvolvimento backend sênior, os conceitos básicos do workshop foram expandidos em **aplicações de nível enterprise prontas para produção**:

#### **📦 Repositórios dos Projetos:**
- **[calculator](https://github.com/danielgorgonha/calculator)** - Biblioteca matemática avançada com funções power/logarithm
- **[interactive-calculator](https://github.com/danielgorgonha/interactive-calculator)** - Aplicação interativa funcional
- **[learn-rust-crud](https://github.com/danielgorgonha/learn-rust-crud)** - API CRUD + WASM pronta para produção

#### **🏆 Principais Conquistas:**
- **Excelência em Segurança**: Autenticação JWT + Refresh tokens + Acesso owner-only
- **Otimização de Performance**: Cache WASM + Rate limiting + Métricas abrangentes
- **Deploy em Produção**: APIs live no Railway com monitoramento
- **Estratégia de Testes**: Suites de testes automatizados com scripts shell
- **Documentação**: Docs completas da API + Collections Postman
- **Arquitetura Enterprise**: Design modular + Tratamento de erros + Logging

#### **💡 Práticas de Desenvolvedor Sênior Aplicadas:**
- **Escalabilidade**: Dos conceitos básicos às aplicações de produção
- **Security First**: Sistemas de autenticação e autorização
- **Performance Focus**: Cache, monitoramento e otimização
- **Testing Strategy**: Testes automatizados abrangentes
- **Documentation**: Documentação profissional e completa
- **Production Ready**: Deployado, monitorado e mantido

---

## 📚 Recursos Adicionais

### **Documentação Oficial**
- [📖 The Rust Programming Language Book](https://doc.rust-lang.org/book/) - Livro oficial do Rust
- [🔧 Rust Reference](https://doc.rust-lang.org/reference/) - Referência técnica completa
- [📚 Rust by Example](https://doc.rust-lang.org/rust-by-example/) - Aprenda Rust com exemplos
- [🛠️ Cargo Book](https://doc.rust-lang.org/cargo/) - Documentação do Cargo

### **Ferramentas e IDEs**
- [🦀 rust-analyzer](https://rust-analyzer.github.io/) - Language server para IDEs
- [📝 VS Code Rust Extension](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer)
- [🐛 Rust Playground](https://play.rust-lang.org/) - Teste código online

### **Comunidade e Aprendizado**
- [💬 Rust Discord](https://discord.gg/rust-lang) - Comunidade oficial
- [📱 r/rust](https://reddit.com/r/rust) - Subreddit do Rust
- [🎯 Rustlings](https://github.com/rust-lang/rustlings) - Exercícios interativos
- [🏆 Rust Quiz](https://dtolnay.github.io/rust-quiz/) - Teste seu conhecimento

### **Desenvolvimento Web e APIs**
- [📖 Tide Documentation](https://docs.rs/tide) - Documentação oficial do framework Tide
- [🔧 Serde Documentation](https://serde.rs/) - Serialização/deserialização JSON
- [🌐 Railway Documentation](https://docs.railway.app/) - Plataforma de deploy na nuvem
- [📡 REST API Best Practices](https://restfulapi.net/) - Diretrizes de design de API

### **Testes e Ferramentas de Desenvolvimento**
- [🧪 Postman](https://www.postman.com/) - Testes e desenvolvimento de API
- [📱 Insomnia](https://insomnia.rest/) - Alternativa ao Postman
- [🔍 cURL Documentation](https://curl.se/docs/) - Cliente HTTP via linha de comando

### **Vídeos e Cursos**
- [🎥 Rust Tutorial for Beginners](https://www.youtube.com/watch?v=zF34dRivLOw)
- [📺 Let's Get Rusty](https://www.youtube.com/c/LetsGetRusty) - Canal especializado
- [🎓 Rust Crash Course](https://www.youtube.com/watch?v=zF34dRivLOw)

---

## 🎯 Próximos Passos

### Após o Workshop 1
- 🎯 Continuar para o Workshop 2: Smartcontracts Básico
- 🏆 Aplicar conhecimentos em projetos pessoais
- 🌟 Contribuir para a comunidade Rust
- 💼 Explorar oportunidades de carreira

---

*Desenvolvido para Meridian Hackathon 2025 - Rio de Janeiro*



