# Capítulo 6: Segurança Web Essencial para Flask

## Sumário do Capítulo

- [6.1 Por que Segurança Importa? Os Riscos Reais](#por-que-segurança-importa-os-riscos-reais)
- [6.2 Autenticação e Autorização: Protegendo Rotas](#autenticação-e-autorização-protegendo-rotas)
- [6.3 Senhas Seguras: Hashing com bcrypt](#senhas-seguras-hashing-com-bcrypt)
- [6.4 XSS (Cross-Site Scripting): O que é e como prevenir](#xss-cross-site-scripting-o-que-é-e-como-prevenir)
- [6.5 CSRF (Cross-Site Request Forgery): Proteção com Flask-WTF](#csrf-cross-site-request-forgery-proteção-com-flask-wtf)
- [6.6 SQL Injection: Por que ORM nos protege](#sql-injection-por-que-orm-nos-protege)
- [6.7 HTTPS e Variáveis de Ambiente: Protegendo Dados Sensíveis](#https-e-variáveis-de-ambiente-protegendo-dados-sensíveis)
- [6.8 Projeto: Implementando Login Seguro no PokéBooster](#projeto-implementando-login-seguro-no-pokébooster)
- [6.9 Verifique seu Conhecimento](#verifique-seu-conhecimento)
- [6.10 Exercícios Práticos](#exercícios-práticos)

---

Nos capítulos anteriores, construímos uma aplicação web funcional e interativa. Agora chegou o momento crucial de protegê-la. **Segurança não é um recurso opcional** — é uma necessidade fundamental em qualquer aplicação web moderna.

Neste capítulo, abordaremos as principais vulnerabilidades de segurança web e como implementar proteções eficazes no **Flask**. Você aprenderá a proteger sua aplicação contra os ataques mais comuns e a implementar práticas de segurança que são padrão na indústria.

## 6.1 Por que Segurança Importa? Os Riscos Reais

### O Cenário Atual

Aplicações web são alvos constantes de ataques. Segundo o **OWASP (Open Web Application Security Project)**, milhões de aplicações são comprometidas anualmente devido a vulnerabilidades básicas que poderiam ter sido evitadas.

### Consequências de Falhas de Segurança

Quando uma aplicação é comprometida, as consequências podem ser devastadoras:

1. **Roubo de Dados**: Informações pessoais, senhas, dados financeiros podem ser expostos
2. **Perda de Confiança**: Usuários abandonam aplicações que não protegem seus dados
3. **Impacto Financeiro**: Multas por violação de privacidade (LGPD, GDPR) e custos de recuperação
4. **Danos à Reputação**: Manchetes negativas podem destruir a credibilidade de uma empresa
5. **Responsabilidade Legal**: Desenvolvedores e empresas podem ser processados

### O OWASP Top 10

O **OWASP** mantém uma lista das 10 vulnerabilidades mais críticas em aplicações web:

1. **Broken Access Control**: Falhas no controle de acesso
2. **Cryptographic Failures**: Falhas na proteção de dados sensíveis
3. **Injection**: SQL Injection, Command Injection, etc.
4. **Insecure Design**: Falhas de design de segurança
5. **Security Misconfiguration**: Configurações inseguras
6. **Vulnerable Components**: Uso de bibliotecas desatualizadas
7. **Authentication Failures**: Falhas na autenticação
8. **Software and Data Integrity Failures**: Falhas de integridade
9. **Security Logging Failures**: Falta de monitoramento
10. **Server-Side Request Forgery (SSRF)**: Requisições forjadas

Neste capítulo, focaremos nas vulnerabilidades mais relevantes para aplicações **Flask**.

## 6.2 Autenticação e Autorização: Protegendo Rotas

### Diferença entre Autenticação e Autorização

- **Autenticação**: Verificar **quem** é o usuário (login)
- **Autorização**: Verificar **o que** o usuário pode fazer (permissões)

### Implementando Autenticação Básica

```python
from flask import Flask, session, redirect, url_for, request
from functools import wraps

app = Flask(__name__)
app.secret_key = 'sua-chave-secreta-forte'

# Decorator para proteger rotas
def login_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if 'user_id' not in session:
            return redirect(url_for('login', next=request.url))
        return f(*args, **kwargs)
    return decorated_function

@app.route('/area_protegida')
@login_required
def area_protegida():
    return f"Bem-vindo, usuário {session['username']}!"
```

### Implementando Autorização por Roles

```python
def role_required(role):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            if 'user_role' not in session:
                return "Acesso negado", 403
            
            if session['user_role'] != role:
                return "Você não tem permissão para acessar esta página", 403
            
            return f(*args, **kwargs)
        return decorated_function
    return decorator

@app.route('/admin')
@login_required
@role_required('admin')
def admin_panel():
    return "Painel de Administração"
```

## 6.3 Senhas Seguras: Hashing com bcrypt

### Por que Nunca Armazenar Senhas em Texto Plano

Armazenar senhas em texto plano é uma das piores práticas de segurança. Se o banco de dados for comprometido, todas as senhas ficam expostas.

### Instalando bcrypt

```bash
pip install flask-bcrypt
```

### Implementação Completa

```python
from flask_bcrypt import Bcrypt

app = Flask(__name__)
bcrypt = Bcrypt(app)

# Ao criar usuário
@app.route('/registro', methods=['POST'])
def registro():
    senha = request.form['senha']
    
    # Validar força da senha
    if len(senha) < 8:
        return "Senha deve ter no mínimo 8 caracteres", 400
    
    # Gerar hash
    senha_hash = bcrypt.generate_password_hash(senha).decode('utf-8')
    
    # Salvar senha_hash no banco de dados
    # ...
    
    return "Usuário criado com sucesso!"

# Ao fazer login
@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    senha = request.form['senha']
    
    # Buscar usuário no banco
    usuario = Usuario.query.filter_by(username=username).first()
    
    if usuario and bcrypt.check_password_hash(usuario.senha_hash, senha):
        session['user_id'] = usuario.id
        return redirect(url_for('index'))
    
    return "Credenciais inválidas", 401
```

### Boas Práticas para Senhas

1. **Requisitos Mínimos**: Pelo menos 8 caracteres
2. **Complexidade**: Combinar letras, números e símbolos
3. **Validação no Servidor**: Nunca confiar apenas na validação do cliente
4. **Rate Limiting**: Limitar tentativas de login
5. **Recuperação Segura**: Usar tokens temporários para reset de senha

## 6.4 XSS (Cross-Site Scripting): O que é e como prevenir

### O que é XSS?

**XSS** ocorre quando um atacante injeta código malicioso (geralmente **JavaScript**) em páginas visualizadas por outros usuários.

### Tipos de XSS

1. **Stored XSS**: Código malicioso armazenado no servidor
2. **Reflected XSS**: Código malicioso refletido imediatamente
3. **DOM-based XSS**: Manipulação do DOM no cliente

### Como o Jinja2 Protege Contra XSS

O **Jinja2** automaticamente escapa variáveis por padrão:

```html
<!-- Seguro - Jinja2 escapa automaticamente -->
<p>{{ nome_usuario }}</p>

<!-- Se nome_usuario = "<script>alert('XSS')</script>" -->
<!-- Resultado: &lt;script&gt;alert('XSS')&lt;/script&gt; -->
```

### Quando NÃO Usar |safe

```html
<!-- ❌ PERIGOSO - Desabilita proteção XSS -->
<div>{{ conteudo_usuario|safe }}</div>

<!-- ✅ SEGURO - Mantém proteção -->
<div>{{ conteudo_usuario }}</div>
```

### Proteção no JavaScript

```javascript
// ❌ VULNERÁVEL
element.innerHTML = dados_usuario;

// ✅ SEGURO
element.textContent = dados_usuario;

// ✅ SEGURO - Criar elementos via DOM
const p = document.createElement('p');
p.textContent = dados_usuario;
element.appendChild(p);
```

### Content Security Policy (CSP)

```python
@app.after_request
def set_security_headers(response):
    response.headers['Content-Security-Policy'] = (
        "default-src 'self'; "
        "script-src 'self' https://cdn.jsdelivr.net; "
        "style-src 'self' https://cdn.jsdelivr.net;"
    )
    response.headers['X-Content-Type-Options'] = 'nosniff'
    response.headers['X-Frame-Options'] = 'DENY'
    response.headers['X-XSS-Protection'] = '1; mode=block'
    return response
```

## 6.5 CSRF (Cross-Site Request Forgery): Proteção com Flask-WTF

### O que é CSRF?

**CSRF** é um ataque onde um site malicioso força o navegador do usuário a fazer requisições não autorizadas para outro site onde o usuário está autenticado.

### Exemplo de Ataque CSRF

```html
<!-- Site malicioso -->
<img src="http://seusite.com/deletar_conta?confirmar=sim">
```

Se o usuário estiver logado no seu site, a conta seria deletada!

### Instalando Flask-WTF

```bash
pip install flask-wtf
```

### Implementação

```python
from flask_wtf.csrf import CSRFProtect

app = Flask(__name__)
app.config['SECRET_KEY'] = 'chave-secreta-forte'
csrf = CSRFProtect(app)
```

### Usando em Templates

```html
<form method="POST">
    {{ csrf_token() }}
    <input type="text" name="username">
    <button type="submit">Enviar</button>
</form>
```

### Proteção em AJAX

```javascript
// Obter token CSRF
const token = document.querySelector('meta[name="csrf-token"]').content;

fetch('/api/endpoint', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRFToken': token
    },
    body: JSON.stringify(dados)
});
```

```html
<!-- No base.html -->
<head>
    <meta name="csrf-token" content="{{ csrf_token() }}">
</head>
```

## 6.6 SQL Injection: Por que ORM nos protege

### O que é SQL Injection?

**SQL Injection** ocorre quando um atacante insere código SQL malicioso através de inputs do usuário.

### Exemplo de Código Vulnerável

```python
# ❌ EXTREMAMENTE PERIGOSO - NÃO FAÇA ISSO!
username = request.form['username']
query = f"SELECT * FROM usuarios WHERE username = '{username}'"
result = db.execute(query)
```

Se o atacante inserir: `' OR '1'='1`, a query se torna:
```sql
SELECT * FROM usuarios WHERE username = '' OR '1'='1'
```

Isso retorna **todos** os usuários!

### Como o SQLAlchemy Protege

```python
# ✅ SEGURO - SQLAlchemy usa prepared statements
username = request.form['username']
usuario = Usuario.query.filter_by(username=username).first()
```

O **SQLAlchemy** automaticamente escapa os valores, prevenindo SQL Injection.

### Quando Usar Raw SQL com Segurança

Se precisar usar SQL bruto, use parâmetros:

```python
# ✅ SEGURO - Usando parâmetros
from sqlalchemy import text

username = request.form['username']
query = text("SELECT * FROM usuarios WHERE username = :username")
result = db.session.execute(query, {"username": username})
```

## 6.7 HTTPS e Variáveis de Ambiente: Protegendo Dados Sensíveis

### Por que HTTPS é Essencial

**HTTPS** criptografa toda a comunicação entre cliente e servidor, protegendo contra:

- Interceptação de dados (man-in-the-middle)
- Roubo de credenciais
- Modificação de dados em trânsito

### Forçando HTTPS no Flask

```python
from flask_talisman import Talisman

app = Flask(__name__)
Talisman(app, force_https=True)
```

### Variáveis de Ambiente

**Nunca** coloque dados sensíveis diretamente no código:

```python
# ❌ PERIGOSO
app.config['SECRET_KEY'] = 'minha-chave-123'
app.config['DATABASE_URI'] = 'mysql://root:senha123@localhost/db'
```

Use variáveis de ambiente:

```python
# ✅ SEGURO
import os
from dotenv import load_dotenv

load_dotenv()

app.config['SECRET_KEY'] = os.getenv('SECRET_KEY')
app.config['DATABASE_URI'] = os.getenv('DATABASE_URI')
```

Arquivo `.env`:
```
SECRET_KEY=chave-super-secreta-gerada-aleatoriamente
DATABASE_URI=mysql://user:pass@localhost/pokebooster
```

**Importante**: Adicione `.env` ao `.gitignore`!

## 6.8 Projeto: Implementando Login Seguro no PokéBooster

Vamos implementar um sistema de login completo e seguro:

```python
# app.py
from flask import Flask, render_template, request, redirect, url_for, session
from flask_sqlalchemy import SQLAlchemy
from flask_bcrypt import Bcrypt
from flask_wtf.csrf import CSRFProtect
from functools import wraps
import os
from dotenv import load_dotenv

load_dotenv()

app = Flask(__name__)
app.config['SECRET_KEY'] = os.getenv('SECRET_KEY')
app.config['SQLALCHEMY_DATABASE_URI'] = os.getenv('DATABASE_URI')

db = SQLAlchemy(app)
bcrypt = Bcrypt(app)
csrf = CSRFProtect(app)

# Modelo de Usuário
class Usuario(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    senha_hash = db.Column(db.String(128), nullable=False)
    cartas = db.relationship('Carta', backref='dono', lazy=True)
    
    def set_senha(self, senha):
        self.senha_hash = bcrypt.generate_password_hash(senha).decode('utf-8')
    
    def check_senha(self, senha):
        return bcrypt.check_password_hash(self.senha_hash, senha)

# Decorator de proteção
def login_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if 'user_id' not in session:
            return redirect(url_for('login'))
        return f(*args, **kwargs)
    return decorated_function

# Rotas
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
        
        # Criar usuário
        usuario = Usuario(username=username, email=email)
        usuario.set_senha(senha)
        
        db.session.add(usuario)
        db.session.commit()
        
        return redirect(url_for('login'))
    
    return render_template('registro.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        senha = request.form['senha']
        
        usuario = Usuario.query.filter_by(username=username).first()
        
        if usuario and usuario.check_senha(senha):
            session['user_id'] = usuario.id
            session['username'] = usuario.username
            return redirect(url_for('index'))
        
        return "Credenciais inválidas", 401
    
    return render_template('login.html')

@app.route('/logout')
def logout():
    session.clear()
    return redirect(url_for('index'))

@app.route('/minha_colecao')
@login_required
def minha_colecao():
    user_id = session['user_id']
    cartas = Carta.query.filter_by(usuario_id=user_id).all()
    return render_template('colecao.html', cartas=cartas)

if __name__ == '__main__':
    app.run(debug=True)
```

## 6.9 Verifique seu Conhecimento

1. Qual é a diferença entre autenticação e autorização?

2. Por que nunca devemos armazenar senhas em texto plano no banco de dados?

3. O que é XSS e como o Jinja2 nos protege contra ele?

4. O que é CSRF e como o Flask-WTF nos protege?

5. Por que usar ORM (SQLAlchemy) ajuda a prevenir SQL Injection?

6. Qual a importância de usar HTTPS em produção?

7. Por que devemos usar variáveis de ambiente para dados sensíveis?

8. O que é o OWASP Top 10 e por que é importante conhecê-lo?

## 6.10 Exercícios Práticos

1. **Sistema de Login Completo**: Implemente registro, login e logout no PokéBooster com todas as proteções de segurança abordadas.

2. **Validação de Senha Forte**: Crie uma função que valide se uma senha tem pelo menos 8 caracteres, uma letra maiúscula, uma minúscula, um número e um símbolo.

3. **Rate Limiting**: Instale e configure o Flask-Limiter para limitar tentativas de login a 5 por minuto.

4. **Headers de Segurança**: Implemente todos os headers de segurança recomendados (CSP, X-Frame-Options, etc.).

5. **Recuperação de Senha**: Implemente um sistema seguro de recuperação de senha usando tokens temporários.

6. **Auditoria de Segurança**: Revise todo o código do PokéBooster e identifique possíveis vulnerabilidades.

---

**Parabéns!** Você agora conhece os fundamentos essenciais de segurança web. Lembre-se: segurança não é um recurso único, mas um processo contínuo de melhoria e vigilância.

No próximo capítulo, aprenderemos como fazer o deploy da nossa aplicação de forma segura e profissional.

