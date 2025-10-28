# Fundamentos de Python 2: Desenvolvimento Web com Flask

## Projeto Pokémon TCG - PokéBooster Simulator

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-2.0+-green.svg)](https://flask.palletsprojects.com/)
[![License](https://img.shields.io/badge/License-Educational-yellow.svg)]()

Este repositório contém o material didático completo do curso **Fundamentos de Python 2: Desenvolvimento Web com Flask**, elaborado para a **Fatec Jahu, 2025** por **Ronan Adriel Zenatti**.

## 📚 Sobre o Curso

Este curso é a continuação de "Fundamentos de Python 1" e tem como objetivo ensinar desenvolvimento web utilizando o framework **Flask** e **Bootstrap 5.3**. Através da construção de um projeto prático completo — o **PokéBooster TCG Simulator** — você aprenderá desde os conceitos básicos de aplicações web até tópicos avançados de segurança e persistência de dados.

## 🎯 Objetivo do Projeto

Construir um simulador web de abertura de pacotes de cartas do Trading Card Game (TCG) dos 151 Pokémon originais, implementando:

- Interface visual responsiva com Bootstrap
- Integração com a PokeAPI
- Sistema de coleção de cartas
- Persistência de dados (Local Storage e MySQL)
- Autenticação de usuários
- Práticas de segurança web

## 📖 Sumário dos Capítulos

### [Capítulo 1: Preparando o Terreno: O Projeto e o Flask](./capitulos/capitulo_01.md)

Introdução ao desenvolvimento web com Flask, configuração do ambiente e criação do primeiro servidor.

- 1.1 Do Console para a Web: O Próximo Passo
- 1.2 Lógica por trás do Flask: O que é um Micro-Framework?
- 1.3 Nosso Projeto: O "PokéBooster" TCG Simulator
- 1.4 Lógica por trás dos Ambientes Virtuais (venv)
- 1.5 Configurando o Ambiente do Mestre Pokémon
- 1.6 O "Olá, Mundo!" do Flask: Nosso primeiro App
- 1.7 Verifique seu Conhecimento
- 1.8 Exercícios Práticos

### [Capítulo 2: Renderizando o Visual (Templates com Jinja2 e Bootstrap)](./capitulos/capitulo_02.md)

Separação de lógica e apresentação usando templates Jinja2 e estilização com Bootstrap.

- 2.1 O que são Templates? A separação do HTML e do Python
- 2.2 Lógica por trás do Jinja2: Injetando dados do Python no HTML
- 2.3 Integrando o Bootstrap 5.3: Criando o base.html
- 2.4 Projeto (Parte 1): Exibindo a "Pokédex" (Galeria de Cartas)
- 2.5 Lógica por trás da API Pokémon (PokeAPI)
- 2.6 Projeto (Parte 2): Carregando os 151 Pokémon via API
- 2.7 Verifique seu Conhecimento
- 2.8 Exercícios Práticos

### [Capítulo 3: Abrindo Pacotes (Interação e Lógica do Jogo)](./capitulos/capitulo_03.md)

Implementação de interatividade com JavaScript, AJAX e comunicação cliente-servidor.

- 3.1 Lógica por trás do "Booster Pack": Simulando a aleatoriedade
- 3.2 Projeto (Parte 1): A Rota /abrir_pacote (API Interna)
- 3.3 Lógica por trás do JavaScript (Client-Side): O que o Flask não pode fazer
- 3.4 Introdução ao fetch(): Chamando nossa API Flask pelo Navegador
- 3.5 Projeto (Parte 2): Exibindo as Cartas Reveladas
- 3.6 **Validação de Dados do Cliente** ⚠️
- 3.7 Verifique seu Conhecimento
- 3.8 Exercícios Práticos

### [Capítulo 4: Salvando o Progresso (Persistência no Navegador)](./capitulos/capitulo_04.md)

Persistência de dados no lado do cliente usando Local Storage e implementação de timer.

- 4.1 Lógica por trás da Persistência de Dados
- 4.2 Opção 1 (Client-Side): O Local Storage do Navegador
- 4.3 Projeto (Parte 1): Salvando a Coleção no Local Storage
- 4.4 Projeto (Parte 2): O Timer de 1 Hora (Habilitando o Botão)
- 4.5 **XSS: O Perigo de Dados Não Sanitizados** ⚠️
- 4.6 Verifique seu Conhecimento
- 4.7 Exercícios Práticos

### [Capítulo 5: Evoluindo o Jogo (Banco de Dados com MySQL)](./capitulos/capitulo_05.md)

Persistência de dados no servidor usando MySQL e SQLAlchemy ORM.

- 5.1 Lógica por trás do Banco de Dados (Server-Side)
- 5.2 Configurando o MySQL e o Flask-SQLAlchemy
- 5.3 Lógica por trás do ORM (SQLAlchemy): Mapeando Classes para Tabelas
- 5.4 Projeto (Parte 1): Modelando o Banco de Dados (Usuario e Carta)
- 5.5 Projeto (Parte 2): Rotas de Autenticação e Salvamento
- 5.6 **Autenticação Segura e Proteção CSRF** ⚠️
- 5.7 Lógica por trás da Migração: Dando a opção ao usuário
- 5.8 Projeto (Parte 3): O Botão "Salvar na Nuvem"
- 5.9 Verifique seu Conhecimento
- 5.10 Exercícios Práticos

### [Capítulo 6: Segurança Web Essencial para Flask](./capitulos/capitulo_06.md) 🔒

**Novo capítulo dedicado à segurança web**, cobrindo as principais vulnerabilidades e proteções.

- 6.1 Por que Segurança Importa? Os Riscos Reais
- 6.2 Autenticação e Autorização: Protegendo Rotas
- 6.3 Senhas Seguras: Hashing com bcrypt
- 6.4 XSS (Cross-Site Scripting): O que é e como prevenir
- 6.5 CSRF (Cross-Site Request Forgery): Proteção com Flask-WTF
- 6.6 SQL Injection: Por que ORM nos protege
- 6.7 HTTPS e Variáveis de Ambiente: Protegendo Dados Sensíveis
- 6.8 Projeto: Implementando Login Seguro no PokéBooster
- 6.9 Verifique seu Conhecimento
- 6.10 Exercícios Práticos

### [Capítulo 7: Conclusão, Implantação e Próximos Passos](./capitulos/capitulo_07.md)

Revisão do projeto, deploy da aplicação e direções para continuar aprendendo.

- 7.1 Revisão do Projeto: O que construímos
- 7.2 Lógica por trás do "Deploy": Colocando seu app na web
- 7.3 Próximos Passos na sua Jornada Web

## 🔒 Ênfase em Segurança

Este curso inclui seções dedicadas de segurança integradas ao longo do conteúdo:

- **Capítulo 3.6**: Validação de Dados do Cliente
- **Capítulo 4.5**: XSS - O Perigo de Dados Não Sanitizados
- **Capítulo 5.6**: Autenticação Segura e Proteção CSRF
- **Capítulo 6**: Capítulo completo sobre Segurança Web Essencial

## 🛠️ Tecnologias Utilizadas

- **Python 3.8+**
- **Flask 2.0+** - Micro-framework web
- **Jinja2** - Motor de templates
- **Bootstrap 5.3** - Framework CSS
- **MySQL** - Banco de dados relacional
- **SQLAlchemy** - ORM (Object-Relational Mapping)
- **Flask-Bcrypt** - Hashing de senhas
- **Flask-WTF** - Proteção CSRF
- **PokeAPI** - API pública de dados Pokémon

## 📋 Pré-requisitos

- Conhecimento básico de Python (variáveis, loops, funções, classes)
- Noções de HTML e CSS
- Ter completado "Fundamentos de Python 1" ou conhecimento equivalente

## 🚀 Como Usar Este Material

1. **Clone o repositório**:
   ```bash
   git clone https://github.com/seu-usuario/fundamentos-python-web.git
   cd fundamentos-python-web
   ```

2. **Leia os capítulos em ordem**:
   - Cada capítulo está em um arquivo Markdown separado na pasta `capitulos/`
   - Siga a ordem numérica para melhor compreensão

3. **Pratique com os exercícios**:
   - Cada capítulo contém exercícios práticos
   - Tente resolver antes de consultar soluções

4. **Construa o projeto**:
   - Acompanhe a construção do PokéBooster passo a passo
   - Implemente as melhorias sugeridas

## 📝 Estrutura do Repositório

```
fundamentos-python-web/
├── README.md                 # Este arquivo
├── capitulos/
│   ├── capitulo_01.md       # Preparando o Terreno
│   ├── capitulo_02.md       # Renderizando o Visual
│   ├── capitulo_03.md       # Abrindo Pacotes
│   ├── capitulo_04.md       # Salvando o Progresso
│   ├── capitulo_05.md       # Evoluindo o Jogo
│   ├── capitulo_06.md       # Segurança Web Essencial
│   └── capitulo_07.md       # Conclusão e Deploy
└── LICENSE
```

## 🎓 Objetivos de Aprendizagem

Ao completar este curso, você será capaz de:

- ✅ Criar aplicações web completas com Flask
- ✅ Implementar interfaces responsivas com Bootstrap
- ✅ Integrar APIs externas em suas aplicações
- ✅ Trabalhar com persistência de dados (client-side e server-side)
- ✅ Implementar autenticação e autorização de usuários
- ✅ Proteger aplicações contra vulnerabilidades comuns (XSS, CSRF, SQL Injection)
- ✅ Fazer deploy de aplicações Flask em produção
- ✅ Seguir boas práticas de desenvolvimento web

## 🤝 Contribuindo

Este é um material educacional aberto. Contribuições são bem-vindas:

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/melhoria`)
3. Commit suas mudanças (`git commit -m 'Adiciona nova seção'`)
4. Push para a branch (`git push origin feature/melhoria`)
5. Abra um Pull Request

## 📄 Licença

Este material é disponibilizado para fins educacionais. Sinta-se livre para usar, modificar e compartilhar, mantendo os créditos originais.

## 👨‍🏫 Autor

**Ronan Adriel Zenatti**  
Fatec Jahu, 2025

## 📞 Contato e Suporte

Para dúvidas, sugestões ou reportar problemas:

- Abra uma [Issue](https://github.com/seu-usuario/fundamentos-python-web/issues)
- Entre em contato através da Fatec Jahu

## 🌟 Agradecimentos

- **Fatec Jahu** - Pela oportunidade de criar este material
- **PokeAPI** - Pela API pública e gratuita de dados Pokémon
- **Comunidade Flask** - Pela excelente documentação e suporte
- **Todos os alunos** - Cujo feedback ajuda a melhorar este curso

---

**Bons estudos e boa sorte na sua jornada de desenvolvimento web!** 🚀

*"O conhecimento é o único bem que aumenta quando é compartilhado."*

