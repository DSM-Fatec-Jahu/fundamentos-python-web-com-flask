# Capítulo 5: Evoluindo o Jogo (Banco de Dados com MySQL)

## 📑 Sumário do Capítulo

- [5.1 Lógica por trás do Banco de Dados (Server-Side)](#51-lógica-por-trás-do-banco-de-dados-server-side)
- [5.2 Configurando o MySQL e o Flask-SQLAlchemy](#52-configurando-o-mysql-e-o-flask-sqlalchemy)
- [5.3 Lógica por trás do ORM (SQLAlchemy)](#53-lógica-por-trás-do-orm-sqlalchemy)
- [5.4 Projeto (Parte 1): Modelando o Banco de Dados](#54-projeto-parte-1-modelando-o-banco-de-dados)
- [5.5 Projeto (Parte 2): Rotas de Autenticação](#55-projeto-parte-2-rotas-de-autenticação)
- [5.6 Autenticação Segura e Proteção CSRF](#56-autenticação-segura-e-proteção-csrf)
- [5.7 Lógica por trás da Migração: Dando a opção ao usuário](#57-lógica-por-trás-da-migração-dando-a-opção-ao-usuário)
- [5.8 Projeto (Parte 3): O Botão "Salvar na Nuvem"](#58-projeto-parte-3-o-botão-salvar-na-nuvem)
- [5.9 Verifique seu Conhecimento](#59-verifique-seu-conhecimento)
- [5.10 Exercícios Práticos](#510-exercícios-práticos)

---

## 🎯 Introdução

Nosso simulador está 100% funcional e divertido, mas tem uma fraqueza crítica: todo o progresso do usuário (sua coleção e seu timer) está salvo no **Local Storage** do navegador.

Isso significa que:

- ❌ Se o usuário limpar o cache do navegador, ele perde tudo
- ❌ Se ele jogar no celular, seu progresso não aparecerá no computador
- ❌ Os dados não são permanentes e estão "presos" a um único dispositivo

Neste capítulo, vamos implementar a solução definitiva para a persistência: um **banco de dados no lado do servidor (Server-Side)**. Daremos ao usuário a opção de criar uma conta e salvar seu progresso na "nuvem" (nosso servidor MySQL), permitindo que ele acesse sua coleção de qualquer lugar.

---

## 5.1 Lógica por trás do Banco de Dados (Server-Side)

### 🎯 Persistência Centralizada e Autoritativa

A lógica aqui é a de **persistência centralizada e autoritativa**. Em vez de cada cliente (navegador) guardar sua própria versão dos dados, todos os dados são armazenados em um **local central e seguro**: o nosso servidor.

### 🔄 Arquitetura com Banco de Dados

```
┌─────────────┐           REQUEST            ┌─────────────┐
│   CLIENTE   │  ──────────────────────────> │   SERVIDOR  │
│ (Navegador) │                               │   (Flask)   │
│             │  <──────────────────────────  │      ↕      │
└─────────────┘          RESPONSE             │   QUERY     │
                                               │      ↕      │
                                               ┌─────────────┐
                                               │   BANCO DE  │
                                               │    DADOS    │
                                               │   (MySQL)   │
                                               └─────────────┘
```

O navegador do usuário (**Cliente**) fará uma requisição ao nosso aplicativo (**Servidor Flask**). Se essa requisição precisar de dados (como a coleção salva), o Flask irá "perguntar" a um terceiro serviço, o **Servidor de Banco de Dados** (no nosso caso, o MySQL), que é especializado em armazenar, organizar e recuperar dados de forma eficiente.

### 📊 O que é MySQL?

**MySQL** é um **RDBMS** (Sistema Gerenciador de Banco de Dados Relacional). Pense nele como uma coleção de planilhas Excel (tabelas) superpotentes, onde as tabelas podem se relacionar (ex: a tabela "Usuários" se relaciona com a tabela "Cartas").

| Conceito | Excel | MySQL |
|----------|-------|-------|
| Arquivo | Planilha Excel | Banco de Dados |
| Aba | Planilha | Tabela |
| Linha | Linha | Registro (Row) |
| Coluna | Coluna | Campo (Column) |
| Célula | Célula | Valor |

---

## 5.2 Configurando o MySQL e o Flask-SQLAlchemy

### 🤔 Por que não SQL puro?

Poderíamos escrever comandos SQL puros em nosso Python (ex: `INSERT INTO ...`), mas isso é complexo e suscetível a erros. Em vez disso, usaremos uma biblioteca chamada **SQLAlchemy**, que é um **ORM** (Object-Relational Mapping).

Para facilitar ainda mais, usaremos a extensão **Flask-SQLAlchemy**, que integra perfeitamente o SQLAlchemy ao Flask.

### 📦 Passo 1: Instale as bibliotecas

Certifique-se de que seu `venv` está ativo e instale os pacotes necessários:

```bash
pip install flask-sqlalchemy mysqlclient
```

> 📝 **Nota**: Se você tiver problemas ao instalar `mysqlclient` no seu sistema, uma alternativa popular é:
> ```bash
> pip install pymysql
> ```
> E ajustar a string de conexão conforme necessário.

### 🔧 Passo 2: Configure o app.py

Precisamos informar ao Flask-SQLAlchemy onde nosso banco de dados está. Isso é feito através de uma "string de conexão".

> ⚠️ **Pré-requisito**: Este capítulo assume que você tem um servidor MySQL rodando. Crie um banco de dados vazio chamado, por exemplo, `poke_tcg_db`.

**Como criar o banco de dados:**

```sql
-- No MySQL Workbench ou terminal MySQL
CREATE DATABASE poke_tcg_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

**Atualize seu app.py:**

```python
# app.py

from flask import Flask, render_template, jsonify, request, session, redirect, url_for, flash
from flask_sqlalchemy import SQLAlchemy
import requests
import random

app = Flask(__name__)

# --- Configuração do Banco de Dados ---
# String de conexão: 'mysql+mysqlclient://USUARIO:SENHA@SERVIDOR/NOME_DO_DB'
# Troque 'root:senha' pelo seu usuário e senha do MySQL
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+mysqlclient://root:senha@localhost/poke_tcg_db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False  # Desativa avisos desnecessários

# --- Chave Secreta para Sessão ---
# Necessário para 'session' e 'flash'
app.config['SECRET_KEY'] = 'uma-chave-secreta-muito-dificil-de-adivinhar'

# --- Inicializa a extensão ---
db = SQLAlchemy(app)

# ... (POKEAPI_URL, carregar_dados_pokemon, LISTA_GLOBAL_POKEMON) ...
# ... (Rotas / e /api/pokemon) ...
# ... (Rota /abrir_pacote) ...
# ... (if __name__ == '__main__': ...)
```

### 🔒 Segurança: Variáveis de Ambiente

> ⚠️ **Importante**: Nunca coloque senhas diretamente no código! Em produção, use variáveis de ambiente.

```python
import os

app.config['SQLALCHEMY_DATABASE_URI'] = os.getenv(
    'DATABASE_URL', 
    'mysql+mysqlclient://root:senha@localhost/poke_tcg_db'
)
app.config['SECRET_KEY'] = os.getenv('SECRET_KEY', 'chave-de-desenvolvimento')
```

---

## 5.3 Lógica por trás do ORM (SQLAlchemy)

### 🎯 O que é ORM?

**ORM** (Object-Relational Mapper) significa "Mapeador Objeto-Relacional". A lógica aqui é a de **abstração**.

Um ORM atua como um **tradutor**. Ele nos permite pensar e escrever código Python "Orientado a Objetos" e traduz isso automaticamente para código SQL "Relacional".

### 🔄 Comparação: Python vs SQL

**Python (Orientado a Objetos):**

```python
# Criar um usuário
usuario = Usuario(username='ash', email='ash@pokemon.com')
db.session.add(usuario)
db.session.commit()

# Buscar todos os usuários
usuarios = Usuario.query.all()

# Buscar um usuário específico
usuario = Usuario.query.filter_by(username='ash').first()
```

**SQL (Relacional):**

```sql
-- Criar um usuário
INSERT INTO usuario (username, email) VALUES ('ash', 'ash@pokemon.com');

-- Buscar todos os usuários
SELECT * FROM usuario;

-- Buscar um usuário específico
SELECT * FROM usuario WHERE username = 'ash' LIMIT 1;
```

### ✅ Vantagens do ORM

| Vantagem | Descrição |
|----------|-----------|
| 🐍 **Pythônico** | Escreva Python, não SQL |
| 🔒 **Segurança** | Proteção automática contra SQL Injection |
| 🔄 **Portabilidade** | Fácil trocar de MySQL para PostgreSQL |
| 📝 **Manutenibilidade** | Código mais legível e organizado |
| 🎯 **Produtividade** | Menos código, menos erros |

---

## 5.4 Projeto (Parte 1): Modelando o Banco de Dados

### 🗂️ Estrutura do Banco de Dados

Precisamos de duas tabelas:

1. **Usuario**: Armazena informações de login
2. **Carta**: Armazena as cartas capturadas por cada usuário

### 📊 Diagrama de Relacionamento

```
┌─────────────────┐         ┌─────────────────┐
│     USUARIO     │         │      CARTA      │
├─────────────────┤         ├─────────────────┤
│ id (PK)         │────┐    │ id (PK)         │
│ username        │    │    │ pokemon_id      │
│ password_hash   │    └────│ user_id (FK)    │
└─────────────────┘         │ nome            │
                            └─────────────────┘

        1                           N
  (Um usuário)              (Muitas cartas)
```

### 🔐 Passo 1: Instale Werkzeug para senhas

Nunca salve senhas como texto puro! Usaremos o **Werkzeug** (instalado automaticamente com o Flask) para gerar "hashes" seguros.

```bash
pip install werkzeug
```

### 💻 Passo 2: Defina os Modelos em app.py

Adicione este código logo após a linha `db = SQLAlchemy(app)`:

```python
# app.py

from werkzeug.security import generate_password_hash, check_password_hash

# ... (db = SQLAlchemy(app)) ...

# --- MODELOS DO BANCO DE DADOS ---

class Usuario(db.Model):
    """Modelo para armazenar informações dos usuários."""
    
    # Colunas da tabela
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    password_hash = db.Column(db.String(128))
    
    # Relação: Um Usuário pode ter muitas Cartas
    # 'backref' cria um atributo virtual 'usuario' na classe Carta
    cartas = db.relationship('Carta', backref='usuario', lazy=True)
    
    def set_password(self, password):
        """Cria o hash da senha."""
        self.password_hash = generate_password_hash(password)
    
    def check_password(self, password):
        """Verifica se a senha está correta."""
        return check_password_hash(self.password_hash, password)
    
    def __repr__(self):
        return f'<Usuario {self.username}>'


class Carta(db.Model):
    """Modelo para armazenar as cartas capturadas pelos usuários."""
    
    # Colunas da tabela
    id = db.Column(db.Integer, primary_key=True)
    pokemon_id = db.Column(db.Integer, nullable=False)
    nome = db.Column(db.String(100), nullable=False)
    
    # Chave Estrangeira: Liga esta carta a um usuário
    user_id = db.Column(db.Integer, db.ForeignKey('usuario.id'), nullable=False)
    
    def __repr__(self):
        return f'<Carta {self.nome} (User {self.user_id})>'
```

### 📊 Análise dos Modelos

| Elemento | Descrição | Exemplo |
|----------|-----------|---------|
| `db.Column` | Define uma coluna na tabela | `id = db.Column(db.Integer)` |
| `db.Integer` | Tipo de dado inteiro | ID, pokemon_id |
| `db.String(80)` | Tipo de dado texto (máx 80 caracteres) | username, nome |
| `primary_key=True` | Identificador único da linha | `id` |
| `unique=True` | Garante valores únicos | username (sem duplicatas) |
| `nullable=False` | Não pode ser nulo (obrigatório) | username, pokemon_id |
| `db.ForeignKey('usuario.id')` | Link para outra tabela | user_id → usuario.id |
| `db.relationship(...)` | Define relação entre tabelas | Usuario.cartas |

### 🔗 Entendendo o Relacionamento

```python
# db.relationship define a relação One-to-Many (Um-para-Muitos)
cartas = db.relationship('Carta', backref='usuario', lazy=True)
```

Isso permite fazer:

```python
# Buscar um usuário
usuario = Usuario.query.filter_by(username='ash').first()

# Acessar todas as cartas do usuário
for carta in usuario.cartas:
    print(carta.nome)

# Ou o inverso (graças ao backref)
carta = Carta.query.first()
print(carta.usuario.username)  # Nome do dono da carta
```

### 🗄️ Passo 3: Crie as Tabelas no DB

Nossos modelos estão definidos, mas as tabelas ainda não existem no MySQL.

**Abra um terminal Python:**

```bash
# Certifique-se de que o venv está ativo
python
```

**Execute no shell Python:**

```python
>>> from app import app, db
>>> with app.app_context():
...     db.create_all()
...
>>> exit()
```

> 📝 **Saída esperada**: Nenhum erro. Se você verificar seu banco `poke_tcg_db` agora, verá as tabelas `usuario` e `carta` criadas!

### 🔍 Verificando no MySQL

```sql
USE poke_tcg_db;
SHOW TABLES;

-- Saída:
-- +------------------------+
-- | Tables_in_poke_tcg_db  |
-- +------------------------+
-- | usuario                |
-- | carta                  |
-- +------------------------+

DESCRIBE usuario;
DESCRIBE carta;
```

---

## 5.5 Projeto (Parte 2): Rotas de Autenticação

### 🎯 Objetivo

Precisamos de páginas para o usuário:
1. 📝 **Registrar** uma nova conta
2. 🔑 **Logar** com suas credenciais
3. 🚪 **Deslogar** da aplicação

O Flask usa um objeto `session` para "lembrar" quem está logado entre as requisições.

### 💻 Passo 1: Criar os Templates HTML

#### 📄 templates/register.html

```html
{% extends 'base.html' %}
{% block title %}Registrar - PokéBooster{% endblock %}

{% block content %}
<div class="row justify-content-center mt-5">
    <div class="col-md-6">
        <div class="card shadow">
            <div class="card-body">
                <h2 class="card-title text-center mb-4">Criar Nova Conta</h2>
                
                <form method="POST" action="{{ url_for('register') }}">
                    <div class="mb-3">
                        <label for="username" class="form-label">Nome de Usuário</label>
                        <input type="text" 
                               class="form-control" 
                               id="username" 
                               name="username" 
                               required
                               minlength="3"
                               maxlength="80">
                        <div class="form-text">Mínimo 3 caracteres</div>
                    </div>
                    
                    <div class="mb-3">
                        <label for="password" class="form-label">Senha</label>
                        <input type="password" 
                               class="form-control" 
                               id="password" 
                               name="password" 
                               required
                               minlength="6">
                        <div class="form-text">Mínimo 6 caracteres</div>
                    </div>
                    
                    <div class="d-grid">
                        <button type="submit" class="btn btn-success btn-lg">
                            Criar Conta
                        </button>
                    </div>
                </form>
                
                <div class="text-center mt-3">
                    <p>Já tem uma conta? 
                        <a href="{{ url_for('login') }}">Fazer Login</a>
                    </p>
                </div>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

#### 📄 templates/login.html

```html
{% extends 'base.html' %}
{% block title %}Login - PokéBooster{% endblock %}

{% block content %}
<div class="row justify-content-center mt-5">
    <div class="col-md-6">
        <div class="card shadow">
            <div class="card-body">
                <h2 class="card-title text-center mb-4">Entrar</h2>
                
                <form method="POST" action="{{ url_for('login') }}">
                    <div class="mb-3">
                        <label for="username" class="form-label">Nome de Usuário</label>
                        <input type="text" 
                               class="form-control" 
                               id="username" 
                               name="username" 
                               required>
                    </div>
                    
                    <div class="mb-3">
                        <label for="password" class="form-label">Senha</label>
                        <input type="password" 
                               class="form-control" 
                               id="password" 
                               name="password" 
                               required>
                    </div>
                    
                    <div class="d-grid">
                        <button type="submit" class="btn btn-primary btn-lg">
                            Entrar
                        </button>
                    </div>
                </form>
                
                <div class="text-center mt-3">
                    <p>Não tem uma conta? 
                        <a href="{{ url_for('register') }}">Criar Conta</a>
                    </p>
                </div>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### 💻 Passo 2: Modificar base.html

Precisamos mostrar "Login/Register" se o usuário estiver deslogado, ou "Olá, [Nome] / Logout" se estiver logado. Também precisamos de um local para exibir as mensagens flash.

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    
    <!-- Nosso CSS -->
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
    
    <title>{% block title %}PokéBooster{% endblock %}</title>
</head>
<body>
    <!-- Menu de Navegação -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="/">
                <strong>PokéBooster TCG</strong>
            </a>
            
            <!-- Botões de Login/Logout -->
            <div class="d-flex">
                {% if session.user_id %}
                    <span class="navbar-text me-3 text-light">
                        Olá, <strong>{{ session.username }}</strong>!
                    </span>
                    <a href="{{ url_for('logout') }}" class="btn btn-outline-light btn-sm">
                        Sair
                    </a>
                {% else %}
                    <a href="{{ url_for('login') }}" class="btn btn-outline-light btn-sm me-2">
                        Entrar
                    </a>
                    <a href="{{ url_for('register') }}" class="btn btn-primary btn-sm">
                        Criar Conta
                    </a>
                {% endif %}
            </div>
        </div>
    </nav>
    
    <!-- Conteúdo Principal -->
    <main class="container mt-4">
        <!-- Mensagens Flash -->
        {% with messages = get_flashed_messages(with_categories=true) %}
            {% if messages %}
                {% for category, message in messages %}
                    <div class="alert alert-{{ category }} alert-dismissible fade show" role="alert">
                        {{ message }}
                        <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                    </div>
                {% endfor %}
            {% endif %}
        {% endwith %}
        
        {% block content %}{% endblock %}
    </main>
    
    <!-- Scripts -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="{{ url_for('static', filename='app.js') }}"></script>
</body>
</html>
```

### 💻 Passo 3: Criar as Rotas de Autenticação em app.py

```python
# app.py
# ... (imports e configurações anteriores) ...

# --- ROTAS DE AUTENTICAÇÃO ---

@app.route('/register', methods=['GET', 'POST'])
def register():
    """Rota para registro de novos usuários."""
    if request.method == 'POST':
        username = request.form['username'].strip()
        password = request.form['password']
        
        # Validações básicas
        if len(username) < 3:
            flash('Nome de usuário deve ter no mínimo 3 caracteres.', 'warning')
            return redirect(url_for('register'))
        
        if len(password) < 6:
            flash('Senha deve ter no mínimo 6 caracteres.', 'warning')
            return redirect(url_for('register'))
        
        # Verifica se o usuário já existe
        usuario_existente = Usuario.query.filter_by(username=username).first()
        if usuario_existente:
            flash('Este nome de usuário já existe. Escolha outro.', 'warning')
            return redirect(url_for('register'))
        
        # Cria novo usuário
        novo_usuario = Usuario(username=username)
        novo_usuario.set_password(password)
        
        try:
            db.session.add(novo_usuario)
            db.session.commit()
            flash('Conta criada com sucesso! Faça o login.', 'success')
            return redirect(url_for('login'))
        except Exception as e:
            db.session.rollback()
            flash('Erro ao criar conta. Tente novamente.', 'danger')
            print(f"Erro ao criar usuário: {e}")
            return redirect(url_for('register'))
    
    return render_template('register.html')


@app.route('/login', methods=['GET', 'POST'])
def login():
    """Rota para login de usuários."""
    if request.method == 'POST':
        username = request.form['username'].strip()
        password = request.form['password']
        
        # Busca o usuário no banco
        usuario = Usuario.query.filter_by(username=username).first()
        
        # Verifica o usuário e a senha
        if usuario and usuario.check_password(password):
            # Armazena o ID do usuário na 'session'
            session['user_id'] = usuario.id
            session['username'] = usuario.username
            flash('Login bem-sucedido! Bem-vindo de volta!', 'success')
            return redirect(url_for('index'))
        else:
            flash('Usuário ou senha inválidos.', 'danger')
            return redirect(url_for('login'))
    
    return render_template('login.html')


@app.route('/logout')
def logout():
    """Rota para logout de usuários."""
    # Remove o usuário da 'session'
    session.pop('user_id', None)
    session.pop('username', None)
    flash('Você foi desconectado. Até logo!', 'info')
    return redirect(url_for('index'))


# ... (rota /abrir_pacote e app.run) ...
```

### 🎨 Categorias de Flash Messages

| Categoria | Cor no Bootstrap | Uso |
|-----------|------------------|-----|
| `success` | Verde | Operações bem-sucedidas |
| `info` | Azul | Informações gerais |
| `warning` | Amarelo | Avisos |
| `danger` | Vermelho | Erros |

### 🧪 Testando

1. Acesse `http://127.0.0.1:5000/register`
2. Crie uma conta
3. Faça login
4. Observe que o menu agora mostra seu nome
5. Faça logout

---

## 5.6 Autenticação Segura e Proteção CSRF

### ⚠️ Por que Autenticação Segura é Crítica?

Quando migramos nossa aplicação para usar banco de dados, precisamos identificar usuários de forma única e segura. A autenticação é o processo de verificar a identidade de um usuário, e implementá-la incorretamente pode comprometer toda a aplicação.

### 🔒 Problemas Comuns em Autenticação

#### 1. Senhas em Texto Plano

**Nunca, jamais, em hipótese alguma** armazene senhas em texto plano no banco de dados!

```python
# ❌ EXTREMAMENTE PERIGOSO - NÃO FAÇA ISSO!
senha = request.form['senha']
usuario = Usuario(nome=nome, senha=senha)  # Senha em texto plano
db.session.add(usuario)
```

**Por quê?**
- 💣 Se o banco de dados for comprometido, todas as senhas são expostas
- 👀 Administradores do sistema podem ver as senhas dos usuários
- 🔑 Usuários frequentemente reutilizam senhas em múltiplos sites

#### 2. Hashing Inadequado

Usar algoritmos de hash simples como MD5 ou SHA1 não é suficiente:

```python
# ❌ INSEGURO
import hashlib
senha_hash = hashlib.md5(senha.encode()).hexdigest()
```

**Problemas:**
- ⚡ MD5 e SHA1 são muito rápidos, permitindo ataques de força bruta
- 🌈 Não usam "salt", permitindo ataques com rainbow tables
- 💔 São considerados criptograficamente quebrados

### ✅ Hashing Seguro com bcrypt

O **bcrypt** é um algoritmo de hashing projetado especificamente para senhas:

```python
from flask_bcrypt import Bcrypt

app = Flask(__name__)
bcrypt = Bcrypt(app)

# Criar hash de senha
senha_hash = bcrypt.generate_password_hash('senha123').decode('utf-8')

# Verificar senha
if bcrypt.check_password_hash(senha_hash, 'senha_digitada'):
    print("Senha correta!")
```

**Vantagens do bcrypt:**
- 🐌 Lento por design (dificulta ataques de força bruta)
- 🧂 Gera salt automaticamente
- 📈 Ajustável (pode aumentar a dificuldade com o tempo)

### 💻 Implementando bcrypt no PokéBooster

#### Instalação

```bash
pip install flask-bcrypt
```

#### Atualização do Modelo

```python
# app.py
from flask_bcrypt import Bcrypt

app = Flask(__name__)
# ... configurações ...
bcrypt = Bcrypt(app)
db = SQLAlchemy(app)

class Usuario(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    senha_hash = db.Column(db.String(128), nullable=False)
    
    def set_senha(self, senha):
        """Cria hash da senha usando bcrypt."""
        self.senha_hash = bcrypt.generate_password_hash(senha).decode('utf-8')
    
    def check_senha(self, senha):
        """Verifica se a senha está correta."""
        return bcrypt.check_password_hash(self.senha_hash, senha)
```

### 🛡️ CSRF: Cross-Site Request Forgery

**CSRF** é um ataque onde um site malicioso força o navegador do usuário a fazer requisições não autorizadas para outro site onde o usuário está autenticado.

#### 💣 Exemplo de Ataque CSRF

Imagine que você está logado no PokéBooster. Um atacante cria um site malicioso com:

```html
<!-- Site malicioso -->
<form action="http://pokebooster.com/deletar_conta" method="POST">
    <input type="hidden" name="confirmar" value="sim">
</form>
<script>
    document.forms[0].submit();
</script>
```

Se sua aplicação não tiver proteção CSRF, a conta do usuário seria deletada!

### 🔐 Proteção CSRF com Flask-WTF

#### Instalação

```bash
pip install flask-wtf
```

#### Configuração

```python
from flask_wtf.csrf import CSRFProtect

app = Flask(__name__)
app.config['SECRET_KEY'] = 'sua-chave-secreta-forte'
csrf = CSRFProtect(app)
```

#### Usando em Formulários HTML

```html
<!-- templates/login.html -->
<form method="POST">
    {{ csrf_token() }}
    <input type="text" name="username" required>
    <input type="password" name="senha" required>
    <button type="submit">Login</button>
</form>
```

#### Proteção em Requisições AJAX

```javascript
// static/app.js
// Obter token CSRF do meta tag
const csrfToken = document.querySelector('meta[name="csrf-token"]').content;

fetch('/abrir_pacote', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRFToken': csrfToken
    },
    body: JSON.stringify({})
})
.then(response => response.json())
.then(data => console.log(data));
```

```html
<!-- templates/base.html -->
<head>
    <meta name="csrf-token" content="{{ csrf_token() }}">
</head>
```

### 🔒 Configuração de Sessões Seguras

```python
from datetime import timedelta

app.config['SECRET_KEY'] = 'chave-super-secreta-aleatoria'
app.config['SESSION_COOKIE_SECURE'] = True  # Apenas HTTPS
app.config['SESSION_COOKIE_HTTPONLY'] = True  # Não acessível via JavaScript
app.config['SESSION_COOKIE_SAMESITE'] = 'Lax'  # Proteção CSRF adicional
app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(hours=1)
```

### 🚫 Prevenção de Enumeração de Usuários

Não revele se um usuário existe ou não:

```python
# ❌ INSEGURO - Revela se o usuário existe
if not usuario:
    return "Usuário não encontrado", 404
if not usuario.check_senha(senha):
    return "Senha incorreta", 401

# ✅ SEGURO - Mensagem genérica
if not usuario or not usuario.check_senha(senha):
    return "Credenciais inválidas", 401
```

### ⏱️ Rate Limiting

Prevenir ataques de força bruta limitando tentativas de login:

```python
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

limiter = Limiter(
    app=app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

@app.route('/login', methods=['POST'])
@limiter.limit("5 per minute")
def login():
    # ... código de login
    pass
```

### ✅ Checklist de Segurança de Autenticação

- [ ] Usar bcrypt (ou argon2) para hash de senhas
- [ ] Nunca armazenar senhas em texto plano
- [ ] Validar força da senha (mínimo 8 caracteres)
- [ ] Implementar proteção CSRF em todos os formulários
- [ ] Usar SECRET_KEY forte e aleatória
- [ ] Configurar cookies de sessão como HttpOnly e Secure
- [ ] Implementar rate limiting em rotas de autenticação
- [ ] Usar mensagens de erro genéricas
- [ ] Implementar logout seguro
- [ ] Validar e sanitizar todos os inputs
- [ ] Usar HTTPS em produção

---

## 5.7 Lógica por trás da Migração: Dando a opção ao usuário

### 🌉 A Ponte entre Dois Mundos

Temos dois "mundos" agora:

1. **Mundo Anônimo**: Local Storage (rápido, temporário)
2. **Mundo Logado**: Banco de Dados (permanente, sincronizado)

Precisamos construir uma "ponte" para que o usuário possa levar seus dados do mundo anônimo para o mundo logado.

### 🎯 Estratégia de Migração

Criaremos uma nova rota de API, `/api/migrar_para_nuvem`. Essa rota fará o seguinte:

1. **Exigir Login**: Ela só funcionará se `session['user_id']` existir
2. **Receber Dados**: Ela receberá o `minhaColecao` (o JSON do Local Storage) enviado pelo JavaScript
3. **Processar**: Ela fará um loop por essa coleção e criará as entradas `Carta` no banco de dados, associadas ao `user_id` da sessão

### 🔄 Fluxo de Migração

```
Local Storage (Cliente)
    ↓
Envia JSON via AJAX
    ↓
Flask recebe e valida
    ↓
Para cada Pokémon capturado:
  - Verifica se já existe no BD
  - Se não, cria nova Carta
    ↓
Salva no MySQL
    ↓
Retorna confirmação
```

---

## 5.8 Projeto (Parte 3): O Botão "Salvar na Nuvem"

### 💻 Passo 1: Adicionar a Rota API de Migração em app.py

```python
# app.py
# ... (código anterior) ...

@app.route('/api/migrar_para_nuvem', methods=['POST'])
def migrar_para_nuvem():
    """Migra a coleção do Local Storage para o banco de dados."""
    
    # 1. Verifica se o usuário está logado
    if 'user_id' not in session:
        return jsonify({'erro': 'Usuário não autenticado.'}), 401
    
    try:
        user_id = session['user_id']
        
        # 2. Recebe a coleção do Local Storage (enviada pelo JS)
        colecao_local = request.json
        if not colecao_local:
            return jsonify({'erro': 'Nenhum dado recebido.'}), 400
        
        # Carrega a lista-mestre de nomes (para referência)
        mapa_nomes = {poke['id']: poke['nome'] for poke in LISTA_GLOBAL_POKEMON}
        
        novas_cartas_adicionadas = 0
        
        # 3. Processa a migração
        for poke_id_str in colecao_local.keys():
            poke_id = int(poke_id_str)
            
            # Verifica se o usuário JÁ possui esta carta no DB (Evita duplicatas)
            carta_existente = Carta.query.filter_by(
                user_id=user_id, 
                pokemon_id=poke_id
            ).first()
            
            if not carta_existente:
                nome_pokemon = mapa_nomes.get(poke_id, 'Desconhecido')
                nova_carta = Carta(
                    pokemon_id=poke_id,
                    nome=nome_pokemon,
                    user_id=user_id
                )
                db.session.add(nova_carta)
                novas_cartas_adicionadas += 1
        
        db.session.commit()
        
        return jsonify({
            'sucesso': True, 
            'mensagem': f'{novas_cartas_adicionadas} novas cartas salvas na sua conta.'
        })
    
    except Exception as e:
        db.session.rollback()
        print(f"Erro na migração: {e}")
        return jsonify({'erro': 'Erro ao salvar dados.'}), 500


# ... (app.run) ...
```

### 💻 Passo 2: Adicionar o botão em index.html

Vamos exibir o botão "Salvar na Nuvem" apenas se o usuário estiver logado.

```html
{% extends 'base.html' %}
{% block title %}Minha Coleção{% endblock %}

{% block content %}
    <div class="d-flex justify-content-between align-items-center mb-4">
        <h2>Minha Coleção ({{ treinador }})</h2>
        
        <div class="d-flex gap-2">
            {% if session.user_id %}
                <button id="btn-salvar-nuvem" class="btn btn-success">
                    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-cloud-upload" viewBox="0 0 16 16">
                        <path fill-rule="evenodd" d="M4.406 1.342A5.53 5.53 0 0 1 8 0c2.69 0 4.923 2 5.166 4.579C14.758 4.804 16 6.137 16 7.773 16 9.569 14.502 11 12.687 11H10a.5.5 0 0 1 0-1h2.688C13.979 10 15 8.988 15 7.773c0-1.216-1.02-2.228-2.313-2.228h-.5v-.5C12.188 2.825 10.328 1 8 1a4.53 4.53 0 0 0-2.941 1.1c-.757.652-1.153 1.438-1.153 2.055v.448l-.445.049C2.064 4.805 1 5.952 1 7.318 1 8.785 2.23 10 3.781 10H6a.5.5 0 0 1 0 1H3.781C1.708 11 0 9.366 0 7.318c0-1.763 1.266-3.223 2.942-3.593.143-.863.698-1.723 1.464-2.383z"/>
                        <path fill-rule="evenodd" d="M7.646 4.146a.5.5 0 0 1 .708 0l3 3a.5.5 0 0 1-.708.708L8.5 5.707V14.5a.5.5 0 0 1-1 0V5.707L5.354 7.854a.5.5 0 1 1-.708-.708l3-3z"/>
                    </svg>
                    Salvar na Nuvem
                </button>
            {% endif %}
            
            <button id="btn-abrir-pacote" class="btn btn-primary btn-lg">
                Abrir Pacote!
            </button>
        </div>
    </div>
    
    <!-- Galeria de Pokémon -->
    <div id="galeria-principal" class="row">
        <div class="text-center mt-5">
            <div class="spinner-border text-primary" role="status">
                <span class="visually-hidden">Carregando Pokédex...</span>
            </div>
            <p>Carregando Pokédex...</p>
        </div>
    </div>
    
    <!-- Modal (código existente) -->
    <!-- ... -->
{% endblock %}
```

### 💻 Passo 3: Adicionar o JavaScript em static/app.js

```javascript
// static/app.js
// ... (código anterior) ...

// --- LÓGICA DE MIGRAÇÃO PARA NUVEM ---

function salvarNaNuvem() {
    console.log("Iniciando salvamento na nuvem...");
    const btnSalvar = document.getElementById('btn-salvar-nuvem');
    
    // Pega a coleção atual do Local Storage
    const colecaoSalva = localStorage.getItem(CHAVE_COLECAO);
    if (!colecaoSalva || colecaoSalva === '{}') {
        alert('Sua coleção local está vazia. Abra alguns pacotes primeiro!');
        return;
    }
    
    btnSalvar.disabled = true;
    const textoOriginal = btnSalvar.innerHTML;
    btnSalvar.innerHTML = '<span class="spinner-border spinner-border-sm"></span> Salvando...';
    
    fetch('/api/migrar_para_nuvem', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: colecaoSalva  // Envia a string JSON do Local Storage
    })
    .then(response => response.json())
    .then(data => {
        if (data.erro) {
            throw new Error(data.erro);
        }
        
        alert(data.mensagem);  // Ex: "5 novas cartas salvas na sua conta."
        
        // Opcional: Limpar o Local Storage após salvar na nuvem?
        // localStorage.removeItem(CHAVE_COLECAO);
        // location.reload();
    })
    .catch(error => {
        console.error('Erro ao salvar na nuvem:', error);
        alert(`Erro ao salvar: ${error.message}`);
    })
    .finally(() => {
        btnSalvar.disabled = false;
        btnSalvar.innerHTML = textoOriginal;
    });
}

// --- ATUALIZAÇÃO NO 'DOMContentLoaded' ---

document.addEventListener('DOMContentLoaded', () => {
    carregarColecaoDoStorage();
    renderizarGaleriaPrincipal();
    
    const btnAbrir = document.getElementById('btn-abrir-pacote');
    const modalElement = document.getElementById('modal-cartas-reveladas');
    
    if (btnAbrir && modalElement) {
        const meuModal = new bootstrap.Modal(modalElement);
        iniciarGerenciadorTimer(btnAbrir);
        
        // ... (código do addEventListener de 'btnAbrir') ...
    }
    
    // --- LIGA O BOTÃO DE SALVAR NA NUVEM ---
    const btnSalvar = document.getElementById('btn-salvar-nuvem');
    if (btnSalvar) {
        btnSalvar.addEventListener('click', salvarNaNuvem);
    }
});
```

### 🎉 Testando o Ciclo Completo

1. **Faça logout** (se estiver logado)
2. **Abra alguns pacotes** (captura cartas no Local Storage)
3. **Crie uma conta** e faça login
4. **Clique em "Salvar na Nuvem"**
5. **Verifique no banco de dados**:
   ```sql
   SELECT * FROM carta WHERE user_id = 1;
   ```

Agora, o ciclo está completo! O usuário pode jogar anonimamente. Se ele gostar, pode criar uma conta, logar, e com um clique, "subir" seu progresso do Local Storage para o banco de dados permanente no servidor! 🎊

---

## 5.9 Verifique seu Conhecimento

### ❓ Questão 1
Qual é a principal vantagem de um banco de dados no servidor (MySQL) sobre o Local Storage?

**a)** É mais rápido para o JavaScript ler.  
**b)** É centralizado, seguro e acessível de múltiplos dispositivos. ✅  
**c)** Não requer instalação, já vem no Flask.  
**d)** Só pode armazenar strings.

---

### ❓ Questão 2
O que significa ORM (Object-Relational Mapper)?

**a)** Um comando SQL para otimizar relações.  
**b)** Uma técnica de design de HTML.  
**c)** Um "tradutor" que converte Classes Python em Tabelas SQL. ✅  
**d)** Um tipo de servidor de banco de dados.

---

### ❓ Questão 3
Qual biblioteca é o "conector" que permite ao SQLAlchemy falar com o MySQL?

**a)** requests  
**b)** mysqlclient (ou pymysql) ✅  
**c)** Jinja2  
**d)** Werkzeug

---

### ❓ Questão 4
Em `class Carta(db.Model)`, o que a linha `user_id = db.Column(db.Integer, db.ForeignKey('usuario.id'))` faz?

**a)** Cria uma cópia da tabela usuario.  
**b)** Define a chave primária da tabela Carta.  
**c)** Cria um "link" para um registro na tabela usuario, estabelecendo uma relação. ✅  
**d)** Define o ID do Pokémon.

---

### ❓ Questão 5
Qual comando usamos no terminal para criar as tabelas no banco de dados a partir dos nossos modelos?

**a)** `db.run()`  
**b)** `db.create_all()` (dentro de um contexto de aplicação) ✅  
**c)** `flask run --create-db`  
**d)** `pip install db-tables`

---

### ❓ Questão 6
Para que serve o `app.config['SECRET_KEY']` no Flask?

**a)** Para criptografar o banco de dados inteiro.  
**b)** É a senha de admin do Flask.  
**c)** Para assinar digitalmente o cookie de session e protegê-lo contra falsificação. ✅  
**d)** Para se conectar à PokeAPI.

---

### ❓ Questão 7
Qual função do werkzeug.security usamos para verificar se uma senha fornecida está correta?

**a)** `generate_password_hash(senha_fornecida)`  
**b)** `check_password(senha_fornecida, hash_salvo)`  
**c)** `check_password_hash(hash_salvo, senha_fornecida)` ✅  
**d)** `compare_passwords(senha_fornecida, hash_salvo)`

---

### ❓ Questão 8
Como o Flask "lembra" que um usuário está logado entre diferentes requisições?

**a)** Armazenando o user_id no objeto session. ✅  
**b)** O Flask não lembra, o JavaScript envia o usuário e senha a cada clique.  
**c)** Armazenando o user_id em uma variável global do Python.  
**d)** Verificando o Local Storage do navegador.

---

### ❓ Questão 9
Na rota `/api/migrar_para_nuvem`, como impedimos que usuários deslogados salvem dados?

**a)** Verificando se `if 'user_id' not in session:`. ✅  
**b)** O JavaScript esconde o botão, o que é segurança suficiente.  
**c)** Pedindo a senha do usuário novamente na rota.  
**d)** Verificando o `request.method == 'POST'`.

---

### ❓ Questão 10
O que `db.session.rollback()` faz?

**a)** Salva permanentemente as mudanças.  
**b)** Desfaz todas as alterações feitas na sessão do banco de dados desde o último commit (útil em caso de erro). ✅  
**c)** Apaga o banco de dados inteiro.  
**d)** Deleta o usuário da sessão do Flask.

---

## 5.10 Exercícios Práticos

### 🎯 Exercício 1: Carregar da Nuvem

A nossa migração é unidirecional (Local → Nuvem). Crie a lógica inversa:

**a)** Adicione um botão "Carregar da Nuvem" em `index.html` (visível apenas se logado)

**b)** Crie uma nova rota API (GET) `/api/carregar_da_nuvem`:

```python
@app.route('/api/carregar_da_nuvem')
def carregar_da_nuvem():
    if 'user_id' not in session:
        return jsonify({'erro': 'Não autenticado'}), 401
    
    user_id = session['user_id']
    cartas = Carta.query.filter_by(user_id=user_id).all()
    
    # Converter para formato do Local Storage
    colecao = {str(carta.pokemon_id): True for carta in cartas}
    
    return jsonify(colecao)
```

**c)** No `app.js`, o fetch para esta rota deve, ao receber os dados, sobrescrever o `minhaColecao` e o `localStorage` com os dados do servidor, e então `location.reload()`.

---

### 🎯 Exercício 2: Carregamento Híbrido (Desafio)

Melhore a lógica de carregamento. Altere o `app.py` para que a rota `index` (`/`) faça o seguinte:

**Se o usuário estiver logado** (`session.user_id` existe):
- Carregue a coleção do banco de dados
- Crie o formato `{1: true, 4: true, ...}` no Python
- Passe essa coleção para o `index.html`:
  ```python
  return render_template('index.html', colecao_servidor=minha_colecao_db)
  ```

**Em `index.html`**, passe essa coleção para o JavaScript:
```html
<script>
    const COLECAO_SERVIDOR = {{ colecao_servidor|tojson }};
</script>
```

**Em `app.js`**, `carregarColecaoDoStorage()` deve agora verificar se `COLECAO_SERVIDOR` existe. Se sim, usa ela. Se não (usuário deslogado), usa o Local Storage.

---

### 🎯 Exercício 3: Refatoração de Nomes

Na rota `/api/migrar_para_nuvem`, usamos um `mapa_nomes` para salvar o nome do Pokémon. Isso é ineficiente.

**Refatore:**
- Modifique o modelo `Carta` para ter apenas `pokemon_id` e `user_id`
- Remova a coluna `nome` (o nome pode ser obtido da `LISTA_GLOBAL_POKEMON` usando o `id` ao renderizar)
- Refatore a rota `migrar_para_nuvem` para salvar apenas o `pokemon_id`

**Dica:** Você precisará recriar as tabelas:
```python
db.drop_all()
db.create_all()
```

---

## 📚 Resumo do Capítulo

Neste capítulo, você aprendeu:

✅ A diferença entre persistência Client-Side e Server-Side  
✅ Como configurar MySQL e Flask-SQLAlchemy  
✅ O que é ORM e suas vantagens  
✅ Como modelar banco de dados com relacionamentos  
✅ Como criar sistema de autenticação completo  
✅ Técnicas de segurança (bcrypt, CSRF, rate limiting)  
✅ Como migrar dados do Local Storage para o banco de dados  
✅ Como implementar a opção "Salvar na Nuvem"

---

## 🎯 Próximo Capítulo

No [Capítulo 6](./capitulo_06.md), vamos nos aprofundar em **Segurança Web Essencial**! Aprenderemos sobre XSS, CSRF, SQL Injection, HTTPS, variáveis de ambiente e todas as melhores práticas para manter sua aplicação segura em produção.

---

**[← Capítulo Anterior](./capitulo_04.md)** | **[Voltar para README](../README.md)** | **[Próximo Capítulo →](./capitulo_06.md)**