Analisando o README e os capítulos 1-7, identifiquei que o README está **bastante completo e bem estruturado**, mas há **algumas atualizações necessárias** para refletir com precisão o conteúdo dos capítulos. Aqui está o README atualizado:

---

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
- Deploy em produção

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
- 3.4 Introdução ao fetch(): Chamando nossa API Flask
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

**Capítulo dedicado à segurança web**, cobrindo as principais vulnerabilidades e proteções.

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
- 7.3 Preparando para Produção: Checklist
- 7.4 Deploy no PythonAnywhere (Passo a Passo)
- 7.5 Próximos Passos na sua Jornada Web
- 7.6 Recursos e Referências
- 7.7 Verifique seu Conhecimento
- 7.8 Exercícios Práticos (Desafios Finais)

## 🔒 Ênfase em Segurança

Este curso integra **segurança desde o início**, com seções dedicadas ao longo do conteúdo:

- **Capítulo 3.6**: Validação de Dados do Cliente
- **Capítulo 4.5**: XSS - O Perigo de Dados Não Sanitizados
- **Capítulo 5.6**: Autenticação Segura e Proteção CSRF
- **Capítulo 6**: Capítulo completo sobre Segurança Web Essencial

### 🛡️ Tópicos de Segurança Cobertos

| Vulnerabilidade | Capítulo | Proteção Implementada |
|-----------------|----------|----------------------|
| **XSS** | 4.5, 6.4 | Escape automático Jinja2, sanitização, CSP |
| **CSRF** | 5.6, 6.5 | Flask-WTF, tokens CSRF |
| **SQL Injection** | 6.6 | SQLAlchemy ORM, prepared statements |
| **Senhas** | 5.5, 6.3 | bcrypt hashing, validação de força |
| **Validação** | 3.6, 6.2 | Server-side validation, rate limiting |
| **HTTPS** | 6.7 | SSL/TLS, headers de segurança |

## 🛠️ Tecnologias Utilizadas

### Backend
- **Python 3.8+** - Linguagem de programação
- **Flask 2.0+** - Micro-framework web
- **Jinja2** - Motor de templates
- **SQLAlchemy** - ORM (Object-Relational Mapping)
- **Flask-Bcrypt** - Hashing seguro de senhas
- **Flask-WTF** - Proteção CSRF
- **Requests** - Cliente HTTP

### Frontend
- **HTML5** - Estrutura
- **CSS3** - Estilização
- **JavaScript ES6+** - Interatividade
- **Bootstrap 5.3** - Framework CSS responsivo
- **Fetch API** - Requisições AJAX

### Banco de Dados
- **MySQL** - Banco de dados relacional
- **mysqlclient** - Driver Python para MySQL

### Integrações
- **PokeAPI** - API pública de dados Pokémon
- **GitHub** - CDN para sprites

### Deploy
- **Gunicorn** - Servidor WSGI
- **PythonAnywhere** - Plataforma de hosting
- **python-dotenv** - Gerenciamento de variáveis de ambiente

## 📋 Pré-requisitos

- Conhecimento básico de Python (variáveis, loops, funções, classes)
- Noções de HTML e CSS
- Ter completado "Fundamentos de Python 1" ou conhecimento equivalente
- Python 3.8+ instalado
- Git (opcional, mas recomendado)

## 🚀 Como Usar Este Material

### 1. Clone o repositório

```bash
git clone https://github.com/seu-usuario/fundamentos-python-web.git
cd fundamentos-python-web
```

### 2. Leia os capítulos em ordem

- Cada capítulo está em um arquivo Markdown separado na pasta `capitulos/`
- Siga a ordem numérica (1 → 7) para melhor compreensão
- Pratique os exercícios ao final de cada capítulo

### 3. Configure o ambiente de desenvolvimento

```bash
# Crie e ative um ambiente virtual
python -m venv venv

# Windows
venv\Scripts\activate

# Linux/Mac
source venv/bin/activate

# Instale as dependências
pip install -r requirements.txt
```

### 4. Construa o projeto

- Acompanhe a construção do PokéBooster passo a passo
- Implemente as melhorias sugeridas nos exercícios
- Teste cada funcionalidade antes de avançar

### 5. Faça o deploy

- Siga o tutorial completo do Capítulo 7
- Publique sua aplicação no PythonAnywhere
- Compartilhe com amigos e família!

## 📝 Estrutura do Repositório

```
fundamentos-python-web/
├── README.md                 # Este arquivo
├── LICENSE                   # Licença educacional
├── requirements.txt          # Dependências Python
├── .gitignore               # Arquivos ignorados pelo Git
├── capitulos/
│   ├── capitulo_01.md       # Preparando o Terreno
│   ├── capitulo_02.md       # Renderizando o Visual
│   ├── capitulo_03.md       # Abrindo Pacotes
│   ├── capitulo_04.md       # Salvando o Progresso
│   ├── capitulo_05.md       # Evoluindo o Jogo
│   ├── capitulo_06.md       # Segurança Web Essencial
│   └── capitulo_07.md       # Conclusão e Deploy
└── projeto/                  # (Opcional) Código do projeto
    ├── app.py
    ├── config.py
    ├── requirements.txt
    ├── .env.example
    ├── templates/
    ├── static/
    └── tests/
```

## 🎓 Objetivos de Aprendizagem

Ao completar este curso, você será capaz de:

### Backend
- ✅ Criar aplicações web completas com Flask
- ✅ Implementar autenticação e autorização de usuários
- ✅ Trabalhar com banco de dados MySQL usando ORM
- ✅ Criar APIs REST funcionais
- ✅ Gerenciar sessões e cookies
- ✅ Implementar validação de dados no servidor

### Frontend
- ✅ Criar interfaces responsivas com Bootstrap
- ✅ Implementar interatividade com JavaScript
- ✅ Fazer requisições AJAX com Fetch API
- ✅ Manipular o DOM dinamicamente
- ✅ Trabalhar com Local Storage

### Segurança
- ✅ Proteger aplicações contra XSS
- ✅ Implementar proteção CSRF
- ✅ Prevenir SQL Injection
- ✅ Fazer hashing seguro de senhas
- ✅ Configurar HTTPS e headers de segurança
- ✅ Validar e sanitizar inputs do usuário

### DevOps
- ✅ Fazer deploy de aplicações Flask em produção
- ✅ Configurar variáveis de ambiente
- ✅ Usar Git para controle de versão
- ✅ Gerenciar dependências com pip
- ✅ Configurar servidores WSGI (Gunicorn)

### Soft Skills
- ✅ Seguir boas práticas de desenvolvimento
- ✅ Escrever código limpo e documentado
- ✅ Resolver problemas sistematicamente
- ✅ Trabalhar com documentação oficial

## 🤝 Contribuindo

Este é um material educacional aberto. Contribuições são bem-vindas:

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/melhoria`)
3. Commit suas mudanças (`git commit -m 'Adiciona nova seção sobre X'`)
4. Push para a branch (`git push origin feature/melhoria`)
5. Abra um Pull Request

### Tipos de Contribuições Bem-Vindas

- 📝 Correções de erros gramaticais ou de código
- 💡 Sugestões de exercícios adicionais
- 🐛 Reportar bugs ou inconsistências
- 🌍 Traduções para outros idiomas
- 📚 Exemplos adicionais
- 🎨 Melhorias visuais nos exemplos

## 📄 Licença

Este material é disponibilizado para fins educacionais sob os seguintes termos:

- ✅ Livre para usar em contextos educacionais
- ✅ Livre para modificar e adaptar
- ✅ Livre para compartilhar
- ⚠️ Mantenha os créditos originais
- ⚠️ Não venda este material

## 👨‍🏫 Autor

**Ronan Adriel Zenatti**  
Fatec Jahu, 2025

## 📞 Contato e Suporte

Para dúvidas, sugestões ou reportar problemas:

- 🐛 Abra uma [Issue](https://github.com/seu-usuario/fundamentos-python-web/issues)
- 💬 Entre em contato através da Fatec Jahu
- 📧 Email: [seu-email@fatec.sp.gov.br]

## 🌟 Agradecimentos

- **Fatec Jahu** - Pela oportunidade de criar este material
- **PokeAPI** - Pela API pública e gratuita de dados Pokémon
- **Comunidade Flask** - Pela excelente documentação e suporte
- **Bootstrap Team** - Pelo framework CSS incrível
- **Comunidade Python Brasil** - Pelo apoio e inspiração
- **Todos os alunos** - Cujo feedback ajuda a melhorar este curso

## 📊 Estatísticas do Curso

```
📚 Capítulos: 7
📝 Páginas: ~200
💻 Linhas de Código: ~1.500+
🎯 Exercícios: 50+
⏱️ Tempo Estimado: 40-60 horas
🔧 Tecnologias: 10+
🏆 Projeto Final: 1 aplicação completa
```

## 🗺️ Roadmap do Curso

```
Semana 1-2: Capítulos 1-2 (Fundamentos Flask e Templates)
   └─> Projeto: Interface visual da Pokédex

Semana 3-4: Capítulos 3-4 (JavaScript e Persistência)
   └─> Projeto: Sistema de abertura de pacotes + Timer

Semana 5-6: Capítulo 5 (Banco de Dados)
   └─> Projeto: Autenticação + Migração para nuvem

Semana 7: Capítulo 6 (Segurança)
   └─> Projeto: Implementação completa de segurança

Semana 8: Capítulo 7 (Deploy e Conclusão)
   └─> Projeto: Deploy em produção
```

## 💡 Dicas para Aproveitar Melhor o Curso

1. **Pratique muito**: Não apenas leia, digite o código
2. **Erre e aprenda**: Bugs são oportunidades de aprendizado
3. **Personalize**: Adicione suas próprias funcionalidades
4. **Documente**: Escreva comentários explicando seu código
5. **Compartilhe**: Mostre seu projeto para amigos
6. **Ajude outros**: Ensinar solidifica o aprendizado
7. **Seja paciente**: Programação é uma skill que requer prática

## 🎯 Próximos Passos Após o Curso

Após completar este curso, considere:

- 🚀 **Django** - Framework web full-stack mais robusto
- ⚡ **FastAPI** - APIs modernas e rápidas
- ⚛️ **React/Vue** - Frameworks JavaScript frontend
- 🐳 **Docker** - Containerização de aplicações
- ☁️ **AWS/GCP/Azure** - Cloud computing
- 📊 **Data Science** - Análise de dados com Python
- 🤖 **Machine Learning** - IA e aprendizado de máquina

## 📚 Recursos Complementares

### Documentação Oficial
- [Python](https://docs.python.org/pt-br/3/)
- [Flask](https://flask.palletsprojects.com/)
- [Bootstrap](https://getbootstrap.com/)
- [MDN Web Docs](https://developer.mozilla.org/pt-BR/)

### Tutoriais Recomendados
- [Real Python](https://realpython.com/)
- [FreeCodeCamp](https://www.freecodecamp.org/)
- [Miguel Grinberg - Flask Mega Tutorial](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)

### Comunidades
- [Python Brasil](https://python.org.br/)
- [Stack Overflow em Português](https://pt.stackoverflow.com/)
- [Reddit - r/learnpython](https://www.reddit.com/r/learnpython/)

---

**Bons estudos e boa sorte na sua jornada de desenvolvimento web!** 🚀

*"O conhecimento é o único bem que aumenta quando é compartilhado."*

---

**Versão:** 1.0.0  
**Última Atualização:** Janeiro 2025  
**Status:** ✅ Completo