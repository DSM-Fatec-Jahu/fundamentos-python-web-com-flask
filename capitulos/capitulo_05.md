# Capítulo 5: Evoluindo o Jogo (Banco de Dados com MySQL)


## Sumário do Capítulo

- [5.1 Lógica por trás do Banco de Dados (Server-Side)
](#lógica-por-trás-do-banco-de-dados-server-side)
- [5.2 Configurando o MySQL e o Flask-SQLAlchemy
](#configurando-o-mysql-e-o-flask-sqlalchemy)
- [5.3 Lógica por trás do ORM (SQLAlchemy)
](#lógica-por-trás-do-orm-sqlalchemy)
- [5.4 Projeto (Parte 1): Modelando o Banco de Dados
](#projeto-parte-1-modelando-o-banco-de-dados)
- [5.5 Projeto (Parte 2): Rotas de Autenticação
](#projeto-parte-2-rotas-de-autenticação)
- [5.6 Autenticação Segura e Proteção CSRF
](#autenticação-segura-e-proteção-csrf)
- [5.7 Lógica por trás da Migração: Dando a opção ao usuário
](#lógica-por-trás-da-migração-dando-a-opção-ao-usuário)
- [5.8 Projeto (Parte 3): O Botão "Salvar na Nuvem"
](#projeto-parte-3-o-botão-salvar-na-nuvem)
- [5.9 Verifique seu Conhecimento
](#verifique-seu-conhecimento)
- [5.10 Exercícios Práticos
](#exercícios-práticos)

---

Nosso simulador está 100% funcional e divertido, mas tem uma fraqueza crítica: todo o progresso do usuário (sua coleção e seu timer) está salvo no Local Storage do navegador.

Isso significa que:

Se o usuário limpar o cache do navegador, ele perde tudo.

Se ele jogar no celular, seu progresso não aparecerá no computador.

Os dados não são permanentes e estão "presos" a um único dispositivo.

Neste capítulo, vamos implementar a solução definitiva para a persistência: um banco de dados no lado do servidor (Server-Side). Daremos ao usuário a opção de criar uma conta e salvar seu progresso na "nuvem" (nosso servidor **MySQL**), permitindo que ele acesse sua coleção de qualquer lugar.


## 5.1 Lógica por trás do Banco de Dados (Server-Side)


Lógica por trás do Banco de Dados: A lógica aqui é a de persistência centralizada e autoritativa. Em vez de cada cliente (navegador) guardar sua própria versão dos dados, todos os dados são armazenados em um local central e seguro: o nosso servidor.

O navegador do usuário (**Cliente**) fará uma requisição ao nosso aplicativo (**Servidor** **Flask**). Se essa requisição precisar de dados (como a coleção salva), o **Flask** irá "perguntar" a um terceiro serviço, o **Servidor** de Banco de Dados (no nosso caso, o **MySQL**), que é especializado em armazenar, organizar e recuperar dados de forma eficiente.

**MySQL** é um RDBMS (Sistema Gerenciador de Banco de Dados Relacional). Pense nele como uma coleção de planilhas Excel (tabelas) superpotentes, onde as tabelas podem se relacionar (ex: a tabela "Usuários" se relaciona com a tabela "Cartas").


## 5.2 Configurando o MySQL e o Flask-SQLAlchemy


Poderíamos escrever comandos SQL puros em nosso **Python** (ex: INSERT INTO ...), mas isso é complexo e suscetível a erros. Em vez disso, usaremos uma biblioteca chamada **SQLAlchemy**, que é um **ORM**.

Para facilitar ainda mais, usaremos a extensão **Flask**-**SQLAlchemy**, que integra perfeitamente o **SQLAlchemy** ao **Flask**.

Instale as bibliotecas:

Certifique-se de que seu **venv** está ativo e instale os pacotes necessários. mysqlclient é o "conector" que permite ao **Python** conversar com o **MySQL**.

```bash
(Nota: Se você tiver problemas ao instalar mysqlclient no seu sistema, uma alternativa popular é pip install pymysql e ajustar a string de conexão).

Configure o app.py:

Precisamos informar ao **Flask**-**SQLAlchemy** onde nosso banco de dados está. Isso é feito através de uma "string de conexão".

(Pré-requisito: Este capítulo assume que você tem um servidor **MySQL** rodando. Crie um banco de dados vazio chamado, por exemplo, poke_tcg_db.)

# app.py

```

from flask import **Flask**, render_template, jsonify, request, session, redirect, url_for, flash

from flask_sqlalchemy import **SQLAlchemy**

import requests

import random



app = **Flask**(__name__)



# --- Configuração do Banco de Dados ---

# String de conexão: 'mysql+mysqlclient://USUARIO:SENHA@SERVIDOR/NOME_DO_DB'

# Troque 'root:senha' pelo seu usuário e senha do **MySQL**.

app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+mysqlclient://root:senha@localhost/poke_tcg_db'

app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False # Desativa avisos desnecessários



# --- Chave Secreta para Sessão ---

# Necessário para 'session' e 'flash'

app.config['SECRET_KEY'] = 'uma-chave-secreta-muito-dificil-de-adivinhar'



# --- Inicializa a extensão ---

db = **SQLAlchemy**(app)



# ... (POKEAPI_URL, carregar_dados_pokemon, LISTA_GLOBAL_POKEMON) ...

# ... (Rotas / e /api/pokemon) ...

# ... (Rota /abrir_pacote) ...

# ... (if __name__ == '__main__': ...)


## 5.3 Lógica por trás do ORM (SQLAlchemy)


Lógica por trás do **ORM** (Object-Relational Mapper): A lógica aqui é a de abstração. Um **ORM** atua como um tradutor. Ele nos permite pensar e escrever código **Python** "Orientado a Objetos" e traduz isso automaticamente para código SQL "Relacional".

Isso é poderoso porque nos permite focar na lógica da aplicação (Classes, Objetos) sem nos preocuparmos com a sintaxe exata do SQL.


## 5.4 Projeto (Parte 1): Modelando o Banco de Dados


Vamos definir nossas "tabelas" como classes **Python**. Precisamos de duas: Usuario e Carta.

Instale Werkzeug para senhas: Nunca salve senhas como texto puro! Usaremos o Werkzeug (instalado automaticamente com o **Flask**) para gerar "hashes" seguros.

```bash
(**venv**) pip install werkzeug

Defina os Modelos em app.py:

Adicione este código logo após a linha db = **SQLAlchemy**(app).

# app.py

```

from werkzeug.security import generate_password_hash, check_password_hash



# ... (db = **SQLAlchemy**(app)) ...



# --- MODELOS DO BANCO DE DADOS ---



class Usuario(db.Model):

    id = db.Column(db.Integer, primary_key=True)

    username = db.Column(db.String(80), unique=True, nullable=False)

    password_hash = db.Column(db.String(128))



    # Relação: Um Usuário pode ter muitas Cartas

    # 'backref' cria um atributo virtual 'usuario' na classe Carta

    cartas = db.relationship('Carta', backref='usuario', lazy=True)



    def set_password(self, password):

        self.password_hash = generate_password_hash(password)



    def check_password(self, password):

        return check_password_hash(self.password_hash, password)



class Carta(db.Model):

    id = db.Column(db.Integer, primary_key=True)

    pokemon_id = db.Column(db.Integer, nullable=False)

    nome = db.Column(db.String(100), nullable=False)



    # Chave Estrangeira: Liga esta carta a um usuário

    user_id = db.Column(db.Integer, db.ForeignKey('usuario.id'), nullable=False)



    def __repr__(self):

        return f'<Carta {self.nome} (User {self.user_id})>'

Análise dos Modelos:

db.Column: Define uma coluna na tabela.

db.Integer, db.String: Define o tipo de dado.

primary_key=True: Identificador único da linha.

unique=True: Garante que não haverá dois usuários com o mesmo username.

db.ForeignKey('usuario.id'): O "link". Diz que Carta.user_id deve ser um id válido da tabela usuario.

db.relationship(...): A "mágica" do **ORM**. Define a relação One-to-Many (Um-para-Muitos). Agora podemos fazer meu_usuario.cartas para obter uma lista de todas as cartas daquele usuário.

Crie as Tabelas no DB:

Nossos modelos estão definidos, mas as tabelas ainda não existem no **MySQL**.

Abra um terminal, ative seu **venv** e rode o shell do **Python**:

```bash
>>> from app import app, db

>>> with app.app_context():

...     db.create_all()

...

>>> exit()

Se você verificar seu banco poke_tcg_db agora, verá as tabelas usuario e carta criadas!

```


## 5.5 Projeto (Parte 2): Rotas de Autenticação


Precisamos de páginas para o usuário se registrar, logar e deslogar. O **Flask** usa um objeto session para "lembrar" quem está logado entre as requisições.

Crie os Templates **HTML** (register.html e login.html)

Crie estes dois arquivos dentro da pasta templates/.

```html
{% block title %}Login{% endblock %}

{% block content %}

    <h2>Login</h2>

    <form method="POST" action="{{ url_for('login') }}">

        <div class="mb-3">

            <label for="username" class="form-label">Usuário</label>

            <input type="text" class="form-control" id="username" name="username" required>

        </div>

        <div class="mb-3">

            <label for="password" class="form-label">Senha</label>

            <input type="password" class="form-control" id="password" name="password" required>

        </div>

        <button type="submit" class="btn btn-primary">Entrar</button>

    </form>

{% endblock %}

{% extends 'base.html' %}

{% block title %}Registrar{% endblock %}

{% block content %}

    <h2>Registrar Nova Conta</h2>

    <form method="POST" action="{{ url_for('register') }}">

        <div class="mb-3">

            <label for="username" class="form-label">Usuário</label>

            <input type="text" class="form-control" id="username" name="username" required>

        </div>

        <div class="mb-3">

            <label for="password" class="form-label">Senha</label>

            <input type="password" class="form-control" id="password" name="password" required>

        </div>

        <button type="submit" class="btn btn-success">Criar Conta</button>

    </form>

{% endblock %}

Modifique base.html para exibir botões e mensagens:

Precisamos mostrar "Login/Register" se o usuário estiver deslogado, ou "Olá, [Nome] / Logout" se estiver logado. Também precisamos de um local para exibir as mensagens flash.

<nav class="navbar navbar-expand-lg navbar-dark bg-dark">

    <div class="container">

        <a class="navbar-brand" href="/">PokéBooster TCG</a>

        <div class="d-flex">

            {% if session.user_id %}

                <span class="navbar-text me-3">

                    Olá, {{ session.username }}!

                </span>

                <a href="{{ url_for('logout') }}" class="btn btn-outline-light">Logout</a>

            {% else %}

                <a href="{{ url_for('login') }}" class="btn btn-outline-light me-2">Login</a>

                <a href="{{ url_for('register') }}" class="btn btn-primary">Registrar</a>

            {% endif %}

        </div>

    </div>

</nav>

```

<main class="container mt-4">

    {% with messages = get_flashed_messages(with_categories=true) %}

        {% if messages %}

            {% for category, message in messages %}

                <div class="alert alert-{{ category or 'info' }} alert-dismissible fade show" role="alert">

                    {{ message }}

                    <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>

                </div>

            {% endfor %}

        {% endif %}

    {% endwith %}



    {% block content %}{% endblock %}

</main>

Crie as Rotas de Autenticação em app.py:

```python
```

# ... (código anterior) ...



# --- ROTAS DE AUTENTICAÇÃO ---



@app.route('/register', methods=['GET', 'POST'])

def register():

    if request.method == 'POST':

        username = request.form['username']

        password = request.form['password']



        # Verifica se o usuário já existe

        usuario_existente = Usuario.query.filter_by(username=username).first()

        if usuario_existente:

            flash('Este nome de usuário já existe.', 'warning')

            return redirect(url_for('register'))



        # Cria novo usuário

        novo_usuario = Usuario(username=username)

        novo_usuario.set_password(password)

        db.session.add(novo_usuario)

        db.session.commit()



        flash('Conta criada com sucesso! Faça o login.', 'success')

        return redirect(url_for('login'))



    return render_template('register.html')



@app.route('/login', methods=['GET', 'POST'])

def login():

    if request.method == 'POST':

        username = request.form['username']

        password = request.form['password']



        usuario = Usuario.query.filter_by(username=username).first()



        # Verifica o usuário e a senha

        if usuario and usuario.check_password(password):

            # Armazena o ID do usuário na 'session'

            session['user_id'] = usuario.id

            session['username'] = usuario.username

            flash('Login bem-sucedido!', 'success')

            return redirect(url_for('index'))

        else:

            flash('Usuário ou senha inválidos.', 'danger')

            return redirect(url_for('login'))



    return render_template('login.html')



@app.route('/logout')

def logout():

    # Remove o usuário da 'session'

    session.pop('user_id', None)

    session.pop('username', None)

    flash('Você foi desconectado.', 'info')

    return redirect(url_for('index'))



# ... (rota /abrir_pacote e app.run) ...



## 5.6 Autenticação Segura e Proteção CSRF

### Por que Autenticação Segura é Crítica?

Quando migramos nossa aplicação para usar banco de dados, precisamos identificar usuários de forma única e segura. A autenticação é o processo de verificar a identidade de um usuário, e implementá-la incorretamente pode comprometer toda a aplicação.

### Problemas Comuns em Autenticação

#### 1. Senhas em Texto Plano

**Nunca, jamais, em hipótese alguma** armazene senhas em texto plano no banco de dados!

```python
# ❌ EXTREMAMENTE PERIGOSO - NÃO FAÇA ISSO!
senha = request.form['senha']
usuario = Usuario(nome=nome, senha=senha)  # Senha em texto plano
db.session.add(usuario)
```

**Por quê?**
- Se o banco de dados for comprometido, todas as senhas são expostas
- Administradores do sistema podem ver as senhas dos usuários
- Usuários frequentemente reutilizam senhas em múltiplos sites

#### 2. Hashing Inadequado

Usar algoritmos de hash simples como MD5 ou SHA1 não é suficiente:

```python
# ❌ INSEGURO
import hashlib
senha_hash = hashlib.md5(senha.encode()).hexdigest()
```

**Problemas:**
- MD5 e SHA1 são muito rápidos, permitindo ataques de força bruta
- Não usam "salt", permitindo ataques com rainbow tables
- São considerados criptograficamente quebrados

### Hashing Seguro com bcrypt

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
- Lento por design (dificulta ataques de força bruta)
- Gera salt automaticamente
- Ajustável (pode aumentar a dificuldade com o tempo)

### Implementando Autenticação Segura no PokéBooster

#### Instalação

```bash
pip install flask-bcrypt
```

#### Modelo de Usuário Seguro

```python
# app.py
from flask import Flask, render_template, request, redirect, url_for, session
from flask_sqlalchemy import SQLAlchemy
from flask_bcrypt import Bcrypt
import secrets

app = Flask(__name__)
app.config['SECRET_KEY'] = secrets.token_hex(16)  # Gerar chave secreta forte
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://user:pass@localhost/pokebooster'
db = SQLAlchemy(app)
bcrypt = Bcrypt(app)

class Usuario(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    senha_hash = db.Column(db.String(128), nullable=False)
    
    def set_senha(self, senha):
        """Cria hash da senha"""
        self.senha_hash = bcrypt.generate_password_hash(senha).decode('utf-8')
    
    def check_senha(self, senha):
        """Verifica se a senha está correta"""
        return bcrypt.check_password_hash(self.senha_hash, senha)
```

#### Rota de Registro

```python
@app.route('/registro', methods=['GET', 'POST'])
def registro():
    if request.method == 'POST':
        username = request.form['username']
        email = request.form['email']
        senha = request.form['senha']
        
        # Validações
        if len(senha) < 8:
            return "Senha deve ter no mínimo 8 caracteres", 400
        
        if Usuario.query.filter_by(username=username).first():
            return "Nome de usuário já existe", 400
        
        if Usuario.query.filter_by(email=email).first():
            return "Email já cadastrado", 400
        
        # Criar usuário
        usuario = Usuario(username=username, email=email)
        usuario.set_senha(senha)
        
        db.session.add(usuario)
        db.session.commit()
        
        return redirect(url_for('login'))
    
    return render_template('registro.html')
```

#### Rota de Login

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        senha = request.form['senha']
        
        usuario = Usuario.query.filter_by(username=username).first()
        
        if usuario and usuario.check_senha(senha):
            # Login bem-sucedido
            session['user_id'] = usuario.id
            session['username'] = usuario.username
            return redirect(url_for('index'))
        else:
            # Mensagem genérica para não revelar se o usuário existe
            return "Credenciais inválidas", 401
    
    return render_template('login.html')
```

#### Proteção de Rotas

```python
from functools import wraps

def login_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if 'user_id' not in session:
            return redirect(url_for('login'))
        return f(*args, **kwargs)
    return decorated_function

@app.route('/minha_colecao')
@login_required
def minha_colecao():
    user_id = session['user_id']
    cartas = Carta.query.filter_by(usuario_id=user_id).all()
    return render_template('colecao.html', cartas=cartas)
```

### CSRF: Cross-Site Request Forgery

**CSRF** é um ataque onde um site malicioso força o navegador do usuário a fazer requisições não autorizadas para outro site onde o usuário está autenticado.

#### Exemplo de Ataque CSRF

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

### Proteção CSRF com Flask-WTF

#### Instalação

```bash
pip install flask-wtf
```

#### Configuração

```python
from flask_wtf.csrf import CSRFProtect

app = Flask(__name__)
app.config['SECRET_KEY'] = secrets.token_hex(16)
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

O Flask-WTF automaticamente:
1. Gera um token único por sessão
2. Valida o token em todas as requisições POST
3. Rejeita requisições sem token válido

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

### Configuração de Sessões Seguras

```python
from datetime import timedelta

app.config['SECRET_KEY'] = secrets.token_hex(16)
app.config['SESSION_COOKIE_SECURE'] = True  # Apenas HTTPS
app.config['SESSION_COOKIE_HTTPONLY'] = True  # Não acessível via JavaScript
app.config['SESSION_COOKIE_SAMESITE'] = 'Lax'  # Proteção CSRF adicional
app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(hours=1)
```

### Prevenção de Enumeração de Usuários

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

### Rate Limiting

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

### Checklist de Segurança de Autenticação

- [ ] Usar bcrypt para hash de senhas
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

### Exercício Prático

Implemente um sistema de autenticação completo no PokéBooster:

1. Crie o modelo Usuario com senha_hash
2. Implemente rotas de registro e login
3. Adicione proteção CSRF
4. Proteja rotas que requerem autenticação
5. Implemente rate limiting
6. Configure sessões seguras

### Pontos-Chave

- **Bcrypt é essencial**: Use sempre para hash de senhas
- **CSRF é real**: Proteja todos os formulários
- **Sessões devem ser seguras**: HttpOnly, Secure, SameSite
- **Rate limiting previne força bruta**: Limite tentativas de login
- **Mensagens genéricas**: Não revele informações sobre usuários

A autenticação segura é a base de qualquer aplicação web que lida com dados de usuários. No próximo capítulo, veremos práticas avançadas de segurança e como fazer deploy seguro da aplicação.




## 5.6 Autenticação Segura e Proteção CSRF

### Por que Autenticação Segura é Crítica?

Quando migramos nossa aplicação para usar banco de dados, precisamos identificar usuários de forma única e segura. A autenticação é o processo de verificar a identidade de um usuário, e implementá-la incorretamente pode comprometer toda a aplicação.

### Problemas Comuns em Autenticação

#### 1. Senhas em Texto Plano

**Nunca, jamais, em hipótese alguma** armazene senhas em texto plano no banco de dados!

```python
# ❌ EXTREMAMENTE PERIGOSO - NÃO FAÇA ISSO!
senha = request.form['senha']
usuario = Usuario(nome=nome, senha=senha)  # Senha em texto plano
db.session.add(usuario)
```

**Por quê?**
- Se o banco de dados for comprometido, todas as senhas são expostas
- Administradores do sistema podem ver as senhas dos usuários
- Usuários frequentemente reutilizam senhas em múltiplos sites

#### 2. Hashing Inadequado

Usar algoritmos de hash simples como MD5 ou SHA1 não é suficiente:

```python
# ❌ INSEGURO
import hashlib
senha_hash = hashlib.md5(senha.encode()).hexdigest()
```

**Problemas:**
- MD5 e SHA1 são muito rápidos, permitindo ataques de força bruta
- Não usam "salt", permitindo ataques com rainbow tables
- São considerados criptograficamente quebrados

### Hashing Seguro com bcrypt

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
- Lento por design (dificulta ataques de força bruta)
- Gera salt automaticamente
- Ajustável (pode aumentar a dificuldade com o tempo)

### Implementando Autenticação Segura no PokéBooster

#### Instalação

```bash
pip install flask-bcrypt
```

#### Modelo de Usuário Seguro

```python
# app.py
from flask import Flask, render_template, request, redirect, url_for, session
from flask_sqlalchemy import SQLAlchemy
from flask_bcrypt import Bcrypt
import secrets

app = Flask(__name__)
app.config['SECRET_KEY'] = secrets.token_hex(16)  # Gerar chave secreta forte
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://user:pass@localhost/pokebooster'
db = SQLAlchemy(app)
bcrypt = Bcrypt(app)

class Usuario(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    senha_hash = db.Column(db.String(128), nullable=False)
    
    def set_senha(self, senha):
        """Cria hash da senha"""
        self.senha_hash = bcrypt.generate_password_hash(senha).decode('utf-8')
    
    def check_senha(self, senha):
        """Verifica se a senha está correta"""
        return bcrypt.check_password_hash(self.senha_hash, senha)
```

#### Rota de Registro

```python
@app.route('/registro', methods=['GET', 'POST'])
def registro():
    if request.method == 'POST':
        username = request.form['username']
        email = request.form['email']
        senha = request.form['senha']
        
        # Validações
        if len(senha) < 8:
            return "Senha deve ter no mínimo 8 caracteres", 400
        
        if Usuario.query.filter_by(username=username).first():
            return "Nome de usuário já existe", 400
        
        if Usuario.query.filter_by(email=email).first():
            return "Email já cadastrado", 400
        
        # Criar usuário
        usuario = Usuario(username=username, email=email)
        usuario.set_senha(senha)
        
        db.session.add(usuario)
        db.session.commit()
        
        return redirect(url_for('login'))
    
    return render_template('registro.html')
```

#### Rota de Login

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        senha = request.form['senha']
        
        usuario = Usuario.query.filter_by(username=username).first()
        
        if usuario and usuario.check_senha(senha):
            # Login bem-sucedido
            session['user_id'] = usuario.id
            session['username'] = usuario.username
            return redirect(url_for('index'))
        else:
            # Mensagem genérica para não revelar se o usuário existe
            return "Credenciais inválidas", 401
    
    return render_template('login.html')
```

#### Proteção de Rotas

```python
from functools import wraps

def login_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if 'user_id' not in session:
            return redirect(url_for('login'))
        return f(*args, **kwargs)
    return decorated_function

@app.route('/minha_colecao')
@login_required
def minha_colecao():
    user_id = session['user_id']
    cartas = Carta.query.filter_by(usuario_id=user_id).all()
    return render_template('colecao.html', cartas=cartas)
```

### CSRF: Cross-Site Request Forgery

**CSRF** é um ataque onde um site malicioso força o navegador do usuário a fazer requisições não autorizadas para outro site onde o usuário está autenticado.

#### Exemplo de Ataque CSRF

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

### Proteção CSRF com Flask-WTF

#### Instalação

```bash
pip install flask-wtf
```

#### Configuração

```python
from flask_wtf.csrf import CSRFProtect

app = Flask(__name__)
app.config['SECRET_KEY'] = secrets.token_hex(16)
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

O Flask-WTF automaticamente:
1. Gera um token único por sessão
2. Valida o token em todas as requisições POST
3. Rejeita requisições sem token válido

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

### Configuração de Sessões Seguras

```python
from datetime import timedelta

app.config['SECRET_KEY'] = secrets.token_hex(16)
app.config['SESSION_COOKIE_SECURE'] = True  # Apenas HTTPS
app.config['SESSION_COOKIE_HTTPONLY'] = True  # Não acessível via JavaScript
app.config['SESSION_COOKIE_SAMESITE'] = 'Lax'  # Proteção CSRF adicional
app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(hours=1)
```

### Prevenção de Enumeração de Usuários

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

### Rate Limiting

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

### Checklist de Segurança de Autenticação

- [ ] Usar bcrypt para hash de senhas
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

### Exercício Prático

Implemente um sistema de autenticação completo no PokéBooster:

1. Crie o modelo Usuario com senha_hash
2. Implemente rotas de registro e login
3. Adicione proteção CSRF
4. Proteja rotas que requerem autenticação
5. Implemente rate limiting
6. Configure sessões seguras

### Pontos-Chave

- **Bcrypt é essencial**: Use sempre para hash de senhas
- **CSRF é real**: Proteja todos os formulários
- **Sessões devem ser seguras**: HttpOnly, Secure, SameSite
- **Rate limiting previne força bruta**: Limite tentativas de login
- **Mensagens genéricas**: Não revele informações sobre usuários

A autenticação segura é a base de qualquer aplicação web que lida com dados de usuários. No próximo capítulo, veremos práticas avançadas de segurança e como fazer deploy seguro da aplicação.



## 5.7 Lógica por trás da Migração (A Ponte)


Lógica por trás da Migração: Temos dois "mundos" agora: o mundo anônimo (Local Storage) e o mundo logado (Banco de Dados). Precisamos construir uma "ponte" para que o usuário possa levar seus dados do mundo anônimo para o mundo logado.

Criaremos uma nova rota de **API**, /api/migrar_para_nuvem. Essa rota fará o seguinte:

Exigir Login: Ela só funcionará se session['user_id'] existir.

Receber Dados: Ela receberá o minhaColecao (o JSON do Local Storage) enviado pelo **JavaScript**.

Processar: Ela fará um loop por essa coleção e criará as entradas Carta no banco de dados, associadas ao user_id da sessão.


## 5.8 Projeto (Parte 3): O Botão "Salvar na Nuvem"


Adicione a Rota **API** de Migração em app.py:

```python
```

# ... (código anterior) ...



@app.route('/api/migrar_para_nuvem', methods=['POST'])

def migrar_para_nuvem():

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

            carta_existente = Carta.query.filter_by(user_id=user_id, pokemon_id=poke_id).first()



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

        return jsonify({'erro': str(e)}), 500



# ... (app.run) ...

Adicione o botão em index.html:

Vamos exibir o botão "Salvar na Nuvem" apenas se o usuário estiver logado.

```html
{% block content %}

    <div class="d-flex justify-content-between align-items-center mb-4">

        <h2>Minha Coleção ({{ treinador }})</h2>

        <div class="d-flex">

            {% if session.user_id %}

                <button id="btn-salvar-nuvem" class="btn btn-success me-2">

                    <svg ...> </svg>

                    Salvar na Nuvem

                </button>

            {% endif %}

            <button id="btn-abrir-pacote" class="btn btn-primary btn-lg">Abrir Pacote!</button>

        </div>

    </div>

    {% endblock %}

Adicione o **JavaScript** em static/app.js:

Finalmente, precisamos "ligar" o novo botão.

// static/app.js

```

// ... (código anterior) ...



// --- NOVO: Lógica de Migração para Nuvem ---



function salvarNaVem() {

    console.log("Iniciando salvamento na nuvem...");

    const btnSalvar = document.getElementById('btn-salvar-nuvem');



    // Pega a coleção atual do Local Storage

    const colecaoSalva = localStorage.getItem(CHAVE_COLECAO);

    if (!colecaoSalva || colecaoSalva === '{}') {

        alert('Sua coleção local está vazia. Abra alguns pacotes primeiro!');

        return;

    }



    btnSalvar.disabled = true;

    btnSalvar.textContent = 'Salvando...';



    fetch('/api/migrar_para_nuvem', {

        method: 'POST',

        headers: {

            'Content-Type': 'application/json'

        },

        body: colecaoSalva // Envia a string JSON do Local Storage

    })

    .then(response => response.json())

    .then(data => {

        if (data.erro) {

            throw new Error(data.erro);

        }

        alert(data.mensagem); // Ex: "5 novas cartas salvas na sua conta."



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

        btnSalvar.textContent = 'Salvar na Nuvem';

    });

}





// --- ATUALIZAÇÃO NO 'DOMContentLoaded' ---



document.addEventListener('DOMContentLoaded', () => {

    carregarColecaoDoStorage();

    renderizarGaleriaPrincipal();



    // ... (código do btnAbrir e modal) ...



    // --- LIGA O BOTÃO DE SALVAR NA NUVEM ---

    const btnSalvar = document.getElementById('btn-salvar-nuvem');

    if (btnSalvar) {

        btnSalvar.addEventListener('click', salvarNaVem);

    }



    // ... (código do addEventListener de 'btnAbrir') ...

});

Agora, o ciclo está completo. O usuário pode jogar anonimamente. Se ele gostar, pode criar uma conta, logar, e com um clique, "subir" seu progresso do Local Storage para o banco de dados permanente no servidor!


## 5.9 Verifique seu Conhecimento


Qual é a principal vantagem de um banco de dados no servidor (**MySQL**) sobre o Local Storage?

a) É mais rápido para o **JavaScript** ler.

b) É centralizado, seguro e acessível de múltiplos dispositivos.

c) Não requer instalação, já vem no **Flask**.

d) Só pode armazenar strings.

O que significa **ORM** (Object-Relational Mapper)?

a) Um comando SQL para otimizar relações.

b) Uma técnica de design de **HTML**.

c) Um "tradutor" que converte Classes **Python** em Tabelas SQL.

d) Um tipo de servidor de banco de dados.

Qual biblioteca é o "conector" que permite ao **SQLAlchemy** falar com o **MySQL**?

a) requests

b) mysqlclient (ou pymysql)

c) **Jinja2**

d) Werkzeug

Em class Carta(db.Model), o que a linha user_id = db.Column(db.Integer, db.ForeignKey('usuario.id')) faz?

a) Cria uma cópia da tabela usuario.

b) Define a chave primária da tabela Carta.

c) Cria um "link" para um registro na tabela usuario, estabelecendo uma relação.

d) Define o ID do Pokémon.

Qual comando usamos no terminal para criar as tabelas no banco de dados a partir dos nossos modelos?

a) db.run()

b) db.create_all() (dentro de um contexto de aplicação)

c) flask run --create-db

d) pip install db-tables

Para que serve o app.config['SECRET_KEY'] no **Flask**?

a) Para criptografar o banco de dados inteiro.

b) É a senha de admin do **Flask**.

c) Para assinar digitalmente o cookie de session e protegê-lo contra falsificação.

d) Para se conectar à PokeAPI.

Qual função do werkzeug.security usamos para verificar se uma senha fornecida está correta?

a) generate_password_hash(senha_fornecida)

b) check_password(senha_fornecida, hash_salvo)

c) check_password_hash(hash_salvo, senha_fornecida)

d) compare_passwords(senha_fornecida, hash_salvo)

Como o **Flask** "lembra" que um usuário está logado entre diferentes requisições?

a) Armazenando o user_id no objeto session.

b) O **Flask** não lembra, o **JavaScript** envia o usuário e senha a cada clique.

c) Armazenando o user_id em uma variável global do **Python**.

d) Verificando o Local Storage do navegador.

Na rota /api/migrar_para_nuvem, como impedimos que usuários deslogados salvem dados?

a) Verificando se if 'user_id' not in session:.

b) O **JavaScript** esconde o botão, o que é segurança suficiente.

c) Pedindo a senha do usuário novamente na rota.

d) Verificando o request.method == 'POST'.

O que db.session.rollback() faz?

a) Salva permanentemente as mudanças.

b) Desfaz todas as alterações feitas na sessão do banco de dados desde o último commit (útil em caso de erro).

c) Apaga o banco de dados inteiro.

d) Deleta o usuário da sessão do **Flask**.


## 5.10 Exercícios Práticos


Carregar da Nuvem: A nossa migração é unidirecional (Local -> Nuvem). Crie a lógica inversa:

a) Adicione um botão "Carregar da Nuvem" em index.html (visível apenas se logado).

b) Crie uma nova rota **API** (GET) /api/carregar_da_nuvem.

c) Esta rota (protegida) deve buscar todas as cartas do usuário logado no DB (ex: user.cartas) e retorná-las como JSON.

d) No app.js, o fetch para esta rota deve, ao receber os dados, sobrescrever o minhaColecao e o localStorage com os dados do servidor, e então location.reload().

Carregamento Híbrido: (Desafio) Melhore a lógica de carregamento. Altere o app.py para que a rota index (/) faça o seguinte:

Se o usuário estiver logado (session.user_id existe):

Carregue a coleção do banco de dados.

Crie o formato {1: true, 4: true, ...} no **Python**.

Passe essa coleção para o index.html (ex: render_template(..., colecao_servidor=minha_colecao_db)).

Em index.html, passe essa coleção para o **JavaScript** (ex: <script>const COLECAO_SERVIDOR = {{ colecao_servidor|tojson }};</script>).

Em app.js, carregarColecaoDoStorage() deve agora verificar se COLECAO_SERVIDOR existe. Se sim, usa ela. Se não (usuário deslogado), usa o Local Storage.

Refatoração de Nomes: Na rota /api/migrar_para_nuvem, usamos um mapa_nomes para salvar o nome do Pokémon. Isso é ineficiente. Modifique o modelo Carta para ter apenas pokemon_id e user_id. O nome não precisa ser salvo no DB, pois já podemos obtê-lo da LISTA_GLOBAL_POKEMON usando o id ao renderizar. Refatore a rota migrar_para_nuvem para salvar apenas o pokemon_id.
