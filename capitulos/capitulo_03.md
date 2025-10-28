# Capítulo 3: Abrindo Pacotes (Interação e Lógica do Jogo)

## 📑 Sumário do Capítulo

- [3.1 Lógica por trás do "Booster Pack": Simulando a aleatoriedade](#31-lógica-por-trás-do-booster-pack-simulando-a-aleatoriedade)
- [3.2 Projeto (Parte 1): A Rota /abrir_pacote (API Interna)](#32-projeto-parte-1-a-rota-abrir_pacote-api-interna)
- [3.3 Lógica por trás do JavaScript (Client-Side): O que o Flask não pode fazer](#33-lógica-por-trás-do-javascript-client-side-o-que-o-flask-não-pode-fazer)
- [3.4 Introdução ao fetch(): Chamando nossa API Flask](#34-introdução-ao-fetch-chamando-nossa-api-flask)
- [3.5 Projeto (Parte 2): Exibindo as Cartas Reveladas](#35-projeto-parte-2-exibindo-as-cartas-reveladas)
- [3.6 Validação de Dados do Cliente](#36-validação-de-dados-do-cliente)
- [3.7 Verifique seu Conhecimento](#37-verifique-seu-conhecimento)
- [3.8 Exercícios Práticos](#38-exercícios-práticos)

---

## 🎯 Introdução

No Capítulo 2, demos vida à nossa aplicação. Criamos a interface visual com Bootstrap, a estrutura com `base.html` e carregamos dinamicamente os 151 Pokémon da PokeAPI usando Jinja2. Nossa página exibe uma "Pokédex" completa, mas ainda é estática — o usuário não pode fazer nada.

Neste capítulo, vamos implementar a funcionalidade principal do nosso projeto: a **interação**. Vamos adicionar o botão "Abrir Pacote", criar a lógica no servidor para sortear 4 cartas e, o mais importante, aprender como o navegador (Cliente) pode se comunicar com nosso servidor Flask (Servidor) **sem recarregar a página**.

---

## 3.1 Lógica por trás do "Booster Pack": Simulando a aleatoriedade

### 🎲 Como funciona no mundo real?

Você compra um pacote de TCG fechado e não sabe o que vem dentro. As cartas são **aleatórias**. Precisamos replicar essa lógica em nosso código.

### 🐍 A ferramenta: módulo `random`

Como vimos no Capítulo 6 de "Fundamentos de Python 1", o módulo embutido `random` do Python é a ferramenta perfeita para isso.

Temos uma lista com os 151 Pokémon carregados da API. Nosso objetivo é sortear **4 itens únicos** dessa lista.

### ⚠️ O problema com `random.choice()`

Se usássemos `random.choice()` quatro vezes, poderíamos sortear o mesmo Pokémon duas vezes:

```python
# ❌ PODE REPETIR
carta1 = random.choice(lista_pokemon)
carta2 = random.choice(lista_pokemon)  # Pode ser a mesma que carta1!
carta3 = random.choice(lista_pokemon)
carta4 = random.choice(lista_pokemon)
```

### ✅ A solução: `random.sample()`

```python
# ✅ GARANTE 4 CARTAS ÚNICAS
cartas_sorteadas = random.sample(lista_pokemon, 4)
```

**`random.sample(populacao, k)`**: Retorna uma lista de `k` elementos **únicos** escolhidos da `populacao` (no nosso caso, a lista de 151 Pokémon).

---

## 3.2 Projeto (Parte 1): A Rota /abrir_pacote (API Interna)

### 🎯 Objetivo

Nossa primeira tarefa é ensinar ao nosso servidor Flask como "sortear 4 cartas" e entregar o resultado para quem pedir.

### 🔄 Mudança de paradigma

Até agora, nossa única rota (`/`) retorna uma página HTML renderizada. Agora, vamos criar uma rota diferente. Esta rota não retornará HTML. Ela será uma **API interna**, e sua única responsabilidade é processar a lógica do sorteio e retornar os dados brutos das cartas sorteadas em formato **JSON**.

### 💻 Passo 1: Adicionar as importações

Abra o `app.py` e adicione as importações de `random` e `jsonify`:

```python
from flask import Flask, render_template, jsonify  # <-- Adicione jsonify
import requests
import random  # <-- Adicione random
```

> 💡 **`jsonify`**: Função do Flask que converte um dicionário Python em uma resposta JSON formatada corretamente para o navegador.

### 💻 Passo 2: Refatorar para variável global

No Capítulo 2, a função `carregar_dados_pokemon()` era chamada toda vez que o usuário visitava a index. Isso é **lento**. Como os 151 Pokémon não mudam, podemos carregar essa lista **uma única vez** quando o servidor inicia e armazená-la em uma variável global.

```python
# ... (importações) ...

app = Flask(__name__)
POKEAPI_URL = "https://pokeapi.co/api/v2/pokemon?limit=151"

def carregar_dados_pokemon():
    # ... (código da função exatamente como antes) ...
    # ... (retorna lista_pokemon) ...
    pass  # Mantenha o código existente

# --- NOVO: Carrega a lista UMA VEZ quando o app inicia ---
LISTA_GLOBAL_POKEMON = carregar_dados_pokemon()
# -------------------------------------------------------

@app.route('/')
def index():
    nome_treinador = "Ash Ketchum"
    # Agora usamos a lista global, muito mais rápido!
    return render_template('index.html', 
                           treinador=nome_treinador,
                           lista_pokemon=LISTA_GLOBAL_POKEMON)

# ... (app.run) ...
```

> 📝 **Obs**: Isso é uma refatoração de performance. O app agora inicia um pouco mais devagar, mas a página index carrega instantaneamente.

### 💻 Passo 3: Criar a nova rota de API

```python
# ... (código anterior) ...

# --- NOVA ROTA DE API ---
@app.route('/abrir_pacote', methods=['POST'])
def abrir_pacote():
    """Sorteia 4 cartas da lista global e as retorna como JSON."""
    try:
        if not LISTA_GLOBAL_POKEMON:
            # Caso a API tenha falhado na inicialização
            return jsonify({'erro': 'Falha ao carregar dados dos Pokémon'}), 500
        
        # Usamos random.sample para sortear 4 cartas ÚNICAS
        cartas_sorteadas = random.sample(LISTA_GLOBAL_POKEMON, 4)
        
        # Retornamos os dados das 4 cartas em formato JSON
        return jsonify({'cartas': cartas_sorteadas})
    
    except Exception as e:
        # Captura qualquer erro inesperado
        return jsonify({'erro': str(e)}), 500

# -------------------------

if __name__ == '__main__':
    app.run(debug=True)
```

### 📊 Análise da Rota

| Parte do código | Explicação |
|-----------------|------------|
| `@app.route('/abrir_pacote', methods=['POST'])` | Define a URL e especifica que ela SÓ aceita requisições do tipo POST |
| `random.sample(LISTA_GLOBAL_POKEMON, 4)` | O coração da nossa lógica - sorteia 4 cartas únicas |
| `jsonify({'cartas': cartas_sorteadas})` | Nossa resposta - não é HTML, é um objeto JSON |

### 🔍 GET vs POST

| Método | Uso | Exemplo |
|--------|-----|---------|
| **GET** | Obter dados (como carregar uma página) | `/`, `/pokemon/25` |
| **POST** | Enviar dados ou realizar uma ação | `/abrir_pacote`, `/login` |

> 💡 **Abrir um pacote é uma ação**, por isso usamos POST.

---

## 3.3 Lógica por trás do JavaScript (Client-Side): O que o Flask não pode fazer

### 🤔 O Problema Fundamental

Temos um problema de **arquitetura**. O Python/Flask vive no **Servidor**. O botão HTML vive no **Cliente** (navegador). Eles são computadores diferentes, possivelmente a milhares de quilômetros de distância.

**Quando o usuário clica no botão "Abrir Pacote" no navegador, como podemos executar a função `abrir_pacote()` que está no `app.py`?**

### ❌ A Forma Antiga (Recarregando a Página)

Poderíamos envolver o botão em um `<form action="/abrir_pacote" method="POST">`. Quando clicado:

1. O navegador descartaria a página atual
2. Pediria a URL `/abrir_pacote`
3. O Flask executaria e... retornaria um JSON
4. O usuário veria apenas um texto JSON feio na tela

Teríamos que fazer o Flask retornar uma nova página HTML inteira. Isso é **lento** e uma **péssima experiência**.

### ✅ A Forma Moderna (AJAX)

Queremos que a página **permaneça onde está**. Quando o usuário clica, queremos que o navegador, silenciosamente e em segundo plano, envie a requisição POST para o servidor.

Isso é chamado de **AJAX** (Asynchronous JavaScript and XML).

### 🎯 Nossa Arquitetura

```
1. Cliente (HTML): 
   └─> O usuário clica em <button id="btn-abrir">

2. Cliente (JavaScript): 
   └─> Nosso script JS "ouve" esse clique

3. Cliente (JS - AJAX): 
   └─> O JS envia a requisição POST /abrir_pacote para o servidor (Flask)

4. Servidor (Flask): 
   └─> A rota /abrir_pacote executa, sorteia as 4 cartas e retorna o JSON

5. Cliente (JS): 
   └─> O JS recebe o JSON (as 4 cartas) em segundo plano

6. Cliente (JS - DOM): 
   └─> O JS manipula o HTML da página (sem recarregar!) e exibe as cartas
```

> 💡 **A única linguagem que roda dentro do navegador do cliente é o JavaScript.**

---

## 3.4 Introdução ao fetch(): Chamando nossa API Flask

### 🌐 O que é fetch()?

`fetch()` é a função moderna do JavaScript para fazer requisições de rede (AJAX). É o equivalente JavaScript do `requests.get()` do Python.

### 📁 Estrutura do Projeto Atualizada

```
poke_tcg/
├── venv/
├── app.py
├── templates/
│   ├── base.html
│   └── index.html
└── static/          ← NOVA PASTA
    └── app.js       ← NOVO ARQUIVO
```

### 💻 Passo 1: Criar a pasta static

O Flask procura por arquivos estáticos (CSS, JS, Imagens) em uma pasta chamada `static`. Crie-a no mesmo nível de `templates`.

### 💻 Passo 2: Criar o arquivo JS

Dentro de `static`, crie `app.js`.

### 💻 Passo 3: Linkar o arquivo JS no base.html

Precisamos que todas as nossas páginas carreguem esse script. Abra `templates/base.html` e adicione o `<script>` no final do `<body>`, logo após o script do Bootstrap.

```html
    <!-- Script do Bootstrap -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
    
    <!-- Nosso script customizado -->
    <script src="{{ url_for('static', filename='app.js') }}"></script>
</body>
</html>
```

> 📝 **Nota**: `url_for('static', ...)` é a forma correta do Jinja2 de gerar o caminho para arquivos estáticos.

### 💻 Passo 4: Adicionar o botão e modal em index.html

Coloque um botão e um lugar para exibir as cartas (usaremos um Modal do Bootstrap).

```html
{% extends 'base.html' %}
{% block title %}Minha Coleção{% endblock %}

{% block content %}
    <div class="d-flex justify-content-between align-items-center mb-4">
        <h2>Minha Coleção ({{ treinador }})</h2>
        <button id="btn-abrir-pacote" class="btn btn-primary btn-lg">
            Abrir Pacote!
        </button>
    </div>
    
    <!-- Galeria de Pokémon (código existente) -->
    <div class="row">
        {% for poke in lista_pokemon %}
            <!-- ... código dos cards ... -->
        {% endfor %}
    </div>
    
    <!-- Modal para exibir as cartas reveladas -->
    <div class="modal fade" id="modal-cartas-reveladas" tabindex="-1" aria-hidden="true">
        <div class="modal-dialog modal-lg modal-dialog-centered">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title">Cartas Reveladas!</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body" id="corpo-modal-cartas">
                    <!-- As cartas serão inseridas aqui pelo JavaScript -->
                </div>
            </div>
        </div>
    </div>
{% endblock %}
```

### 💻 Passo 5: Escrever o JavaScript em static/app.js

```javascript
// Espera o HTML ser totalmente carregado antes de rodar o script
document.addEventListener('DOMContentLoaded', () => {
    
    // 1. Encontra o botão e o modal na página
    const btnAbrir = document.getElementById('btn-abrir-pacote');
    const modalElement = document.getElementById('modal-cartas-reveladas');
    
    // Se o botão não existir nesta página, não faz nada
    if (!btnAbrir || !modalElement) {
        return;
    }
    
    // Cria uma instância do Modal do Bootstrap
    const meuModal = new bootstrap.Modal(modalElement);
    
    // 2. Adiciona um "ouvinte" de clique no botão
    btnAbrir.addEventListener('click', () => {
        console.log("Botão clicado! Pedindo pacote ao servidor...");
        
        // Desabilita o botão para evitar cliques duplos
        btnAbrir.disabled = true;
        btnAbrir.textContent = "Sorteando...";
        
        // 3. Chama a função fetch()
        fetch('/abrir_pacote', {
            method: 'POST'  // Especifica o método POST
        })
        .then(response => {
            if (!response.ok) {
                throw new Error('Falha na rede ou erro do servidor.');
            }
            return response.json();  // Converte a resposta do Flask em JSON
        })
        .then(data => {
            // 4. 'data' é o nosso objeto: {'cartas': [...]}
            console.log("Cartas recebidas:", data.cartas);
            
            // 5. Chama a função para exibir as cartas (próxima seção)
            exibirCartasNoModal(data.cartas);
            
            // 6. Mostra o modal
            meuModal.show();
        })
        .catch(error => {
            console.error('Erro ao abrir pacote:', error);
            alert('Não foi possível abrir o pacote. Tente novamente.');
        })
        .finally(() => {
            // 7. Reabilita o botão, independentemente de sucesso ou falha
            btnAbrir.disabled = false;
            btnAbrir.textContent = "Abrir Pacote!";
        });
    });
});

// Esta função será implementada na próxima seção
function exibirCartasNoModal(cartas) {
    // ... (lógica para criar o HTML das cartas) ...
}
```

### 📊 Fluxo do fetch()

```javascript
fetch('/abrir_pacote', { method: 'POST' })
    ↓
.then(response => response.json())  // Converte resposta em JSON
    ↓
.then(data => {                     // Usa os dados
    console.log(data.cartas);
    exibirCartasNoModal(data.cartas);
})
    ↓
.catch(error => {                   // Captura erros
    console.error(error);
})
    ↓
.finally(() => {                    // Executa sempre no final
    btnAbrir.disabled = false;
})
```

---

## 3.5 Projeto (Parte 2): Exibindo as Cartas Reveladas

### 🎯 Objetivo

O último passo é implementar a função `exibirCartasNoModal()`. Ela receberá a lista de 4 cartas (JSON) e deverá construir o HTML dinamicamente.

### 💻 Adicione a função em static/app.js

```javascript
// ... (código do addEventListener) ...

function exibirCartasNoModal(cartas) {
    const corpoModal = document.getElementById('corpo-modal-cartas');
    
    // 1. Limpa o modal de cartas anteriores
    corpoModal.innerHTML = '';
    
    // 2. Cria a <div class="row"> do Bootstrap
    const row = document.createElement('div');
    row.className = 'row';
    
    // 3. Faz um loop nas 4 cartas recebidas
    cartas.forEach(carta => {
        // 4. Cria o HTML do Card para cada carta
        // (Usamos o mesmo padrão de classes do Bootstrap do index.html)
        
        const col = document.createElement('div');
        col.className = 'col-md-6 mb-3';  // 2 colunas por linha no modal
        
        const cardHTML = `
            <div class="card text-center shadow-sm">
                <img src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${carta.id}.png" 
                     class="card-img-top p-3" 
                     alt="${carta.nome}">
                <div class="card-body">
                    <h5 class="card-title">${carta.nome}</h5>
                    <span class="badge bg-primary">Nova!</span>
                </div>
            </div>
        `;
        
        // 5. Adiciona o HTML na coluna e a coluna na linha
        col.innerHTML = cardHTML;
        row.appendChild(col);
    });
    
    // 6. Adiciona a linha (com todas as 4 cartas) ao corpo do modal
    corpoModal.appendChild(row);
}
```

### 🎨 Manipulação do DOM

```javascript
// Criar elemento
const div = document.createElement('div');

// Adicionar classe
div.className = 'minha-classe';

// Adicionar HTML interno
div.innerHTML = '<p>Olá</p>';

// Adicionar como filho
parentElement.appendChild(div);
```

### 🚀 Testando

Rode seu `app.py`, recarregue a página e clique no botão "Abrir Pacote!". 

Você verá:
1. O botão ser desabilitado
2. A mensagem "Sorteando..." aparecer
3. O modal surgir com 4 cartas aleatórias!

🎉 **Parabéns! Você implementou AJAX com sucesso!**

---

## 3.6 Validação de Dados do Cliente

### ⚠️ Por que Validar Dados do Cliente?

Quando desenvolvemos aplicações web, é fundamental entender que **nunca devemos confiar cegamente nos dados enviados pelo cliente**. O navegador do usuário está completamente fora do nosso controle.

### 🔓 Vulnerabilidades

Qualquer pessoa com conhecimentos básicos de desenvolvimento web pode:

- ✏️ Modificar requisições HTTP usando ferramentas de desenvolvedor
- 🤖 Enviar dados maliciosos através de scripts automatizados
- 🔧 Manipular parâmetros de URL e formulários
- 💉 Injetar código malicioso em campos de entrada

### 🛡️ Validação no Servidor é Obrigatória

Embora possamos (e devamos) implementar validações no lado do cliente usando JavaScript para melhorar a experiência do usuário, essas validações são facilmente contornáveis. 

> 🚨 **A validação no servidor é a única linha de defesa confiável.**

### 💻 Exemplo Prático: Validando a Rota /abrir_pacote

Em nossa aplicação PokéBooster, a rota `/abrir_pacote` sorteia 4 cartas aleatórias. Mas e se um usuário malicioso tentar manipular essa requisição para obter vantagens? Vamos implementar validações adequadas:

```python
from flask import Flask, jsonify, request
import random

@app.route('/abrir_pacote', methods=['POST'])
def abrir_pacote():
    # 1. Verificar se o método HTTP é o esperado
    if request.method != 'POST':
        return jsonify({'erro': 'Método não permitido'}), 405
    
    # 2. Validar parâmetros (se houver)
    # Por exemplo, se implementarmos um sistema de autenticação:
    user_id = request.args.get('user_id')
    if user_id:
        # Validar que user_id é um número inteiro válido
        try:
            user_id = int(user_id)
            if user_id <= 0:
                return jsonify({'erro': 'ID de usuário inválido'}), 400
        except ValueError:
            return jsonify({'erro': 'ID de usuário deve ser numérico'}), 400
    
    # 3. Implementar rate limiting (limitar requisições)
    # Prevenir que um usuário abra milhares de pacotes por segundo
    # (Veremos isso em detalhes no Capítulo 6)
    
    # 4. Validar integridade dos dados
    try:
        if not LISTA_GLOBAL_POKEMON:
            return jsonify({'erro': 'Dados não disponíveis'}), 500
            
        cartas_sorteadas = random.sample(LISTA_GLOBAL_POKEMON, 4)
        return jsonify({'cartas': cartas_sorteadas})
    except Exception as e:
        # Nunca expor detalhes do erro para o cliente
        return jsonify({'erro': 'Erro ao processar requisição'}), 500
```

### 📋 Princípios de Validação Segura

| Princípio | Descrição | Exemplo |
|-----------|-----------|---------|
| **Validação de Tipo** | Verifique se os dados são do tipo esperado | `int()`, `isinstance()` |
| **Validação de Intervalo** | Para números, verifique limites aceitáveis | `if valor < 1 or valor > 151` |
| **Sanitização** | Remova caracteres especiais perigosos | Use bibliotecas especializadas |
| **Lista Branca** | Aceite apenas valores conhecidos | `if valor in ['opcao1', 'opcao2']` |
| **Mensagens Genéricas** | Não exponha detalhes internos | "Erro ao processar" vs "Tabela X não existe" |

### 💻 Validação de Dados JSON

Quando recebemos dados JSON do cliente (como em requisições POST), devemos validar a estrutura:

```python
from flask import request, jsonify

@app.route('/salvar_colecao', methods=['POST'])
def salvar_colecao():
    # 1. Verificar se o Content-Type é JSON
    if not request.is_json:
        return jsonify({'erro': 'Content-Type deve ser application/json'}), 400
    
    # 2. Tentar parsear o JSON
    try:
        dados = request.get_json()
    except Exception:
        return jsonify({'erro': 'JSON inválido'}), 400
    
    # 3. Validar estrutura esperada
    if not isinstance(dados, dict):
        return jsonify({'erro': 'Formato de dados inválido'}), 400
    
    if 'cartas' not in dados:
        return jsonify({'erro': 'Campo "cartas" é obrigatório'}), 400
    
    if not isinstance(dados['cartas'], list):
        return jsonify({'erro': 'Campo "cartas" deve ser uma lista'}), 400
    
    # 4. Validar cada item da lista
    for carta in dados['cartas']:
        if not isinstance(carta, dict):
            return jsonify({'erro': 'Formato de carta inválido'}), 400
        
        if 'id' not in carta or 'nome' not in carta:
            return jsonify({'erro': 'Carta deve conter id e nome'}), 400
        
        # Validar que o ID está no intervalo válido (1-151)
        try:
            carta_id = int(carta['id'])
            if carta_id < 1 or carta_id > 151:
                return jsonify({'erro': 'ID de carta fora do intervalo válido'}), 400
        except ValueError:
            return jsonify({'erro': 'ID de carta deve ser numérico'}), 400
    
    # Se passou por todas as validações, processar os dados
    # ...
    return jsonify({'sucesso': True})
```

### 🚫 Proteção Contra Ataques de Negação de Serviço (DoS)

Validar também significa proteger contra requisições que possam sobrecarregar o servidor:

```python
# Limitar tamanho de requisições
app.config['MAX_CONTENT_LENGTH'] = 16 * 1024 * 1024  # 16 MB máximo

@app.route('/upload', methods=['POST'])
def upload():
    dados = request.get_json()
    
    # Validar quantidade de itens em listas
    if len(dados.get('cartas', [])) > 1000:
        return jsonify({'erro': 'Quantidade de cartas excede o limite'}), 400
    
    # Validar comprimento de strings
    if len(dados.get('nome', '')) > 100:
        return jsonify({'erro': 'Nome muito longo'}), 400
    
    return jsonify({'sucesso': True})
```

### 📝 Códigos HTTP de Erro

| Código | Significado | Quando usar |
|--------|-------------|-------------|
| **400** | Bad Request | Erro na requisição do cliente |
| **401** | Unauthorized | Não autenticado |
| **403** | Forbidden | Não autorizado |
| **404** | Not Found | Recurso não encontrado |
| **405** | Method Not Allowed | Método HTTP incorreto |
| **500** | Internal Server Error | Erro no servidor |

### ✅ Checklist de Validação

- [ ] Validar método HTTP (GET, POST, etc.)
- [ ] Validar Content-Type quando esperado JSON
- [ ] Validar tipos de dados (int, string, list, dict)
- [ ] Validar intervalos numéricos
- [ ] Validar comprimento de strings
- [ ] Validar estrutura de objetos JSON
- [ ] Limitar tamanho de requisições
- [ ] Retornar mensagens de erro apropriadas
- [ ] Não expor detalhes internos do sistema
- [ ] Logar tentativas suspeitas

### 🎯 Pontos-Chave

> 🚨 **Nunca confie nos dados do cliente**: Sempre valide no servidor

> ✅ **Valide tipo, formato e intervalo**: Não assuma que os dados estão corretos

> 📊 **Use códigos HTTP apropriados**: 400 para erros do cliente, 500 para erros do servidor

> 🔒 **Mensagens de erro genéricas**: Não exponha detalhes internos do sistema

> 🛡️ **Proteja contra DoS**: Limite tamanho e frequência de requisições

A validação adequada de dados é a primeira linha de defesa contra muitos tipos de ataques web. No próximo capítulo, veremos como proteger dados armazenados no navegador contra outros tipos de vulnerabilidades.

---

## 3.7 Verifique seu Conhecimento

### ❓ Questão 1
Qual função do Flask é usada para converter um dicionário Python em uma resposta JSON?

**a)** `render_template()`  
**b)** `json.dump()`  
**c)** `jsonify()` ✅  
**d)** `return_json()`

---

### ❓ Questão 2
Qual módulo Python é usado para sortear itens únicos de uma lista?

**a)** `math.random`  
**b)** `random.sample()` ✅  
**c)** `random.choice()`  
**d)** `list.sort(random=True)`

---

### ❓ Questão 3
Por que usamos JavaScript para lidar com o clique do botão?

**a)** Porque Python não sabe lidar com cliques.  
**b)** Para permitir que o Cliente (navegador) se comunique com o Servidor (Flask) sem recarregar a página. ✅  
**c)** Porque JavaScript é mais rápido que Python.  
**d)** Para estilizar o botão com cores.

---

### ❓ Questão 4
O que é AJAX?

**a)** Um framework CSS concorrente do Bootstrap.  
**b)** Uma técnica que usa JavaScript para enviar e receber dados do servidor em segundo plano. ✅  
**c)** Um tipo de banco de dados do Flask.  
**d)** O nome da API do Pokémon.

---

### ❓ Questão 5
Qual método HTTP é semanticamente correto para uma rota que realiza uma ação (como /abrir_pacote)?

**a)** GET  
**b)** POST ✅  
**c)** UPDATE  
**d)** DELETE

---

### ❓ Questão 6
Qual é a função nativa do JavaScript usada para fazer requisições de rede (AJAX)?

**a)** `requests.get()`  
**b)** `ajax.call()`  
**c)** `fetch()` ✅  
**d)** `http.send()`

---

### ❓ Questão 7
No JavaScript, o que o `.then(response => response.json())` faz?

**a)** Converte um objeto JSON do JavaScript em uma string.  
**b)** Pede ao Flask para enviar uma resposta em JSON.  
**c)** Pega a resposta bruta do servidor e a converte em um objeto JSON utilizável pelo JavaScript. ✅  
**d)** Cria um arquivo chamado `response.json`.

---

### ❓ Questão 8
Em qual pasta o Flask espera encontrar arquivos estáticos como app.js?

**a)** templates  
**b)** static ✅  
**c)** js  
**d)** No mesmo local do app.py

---

### ❓ Questão 9
Qual método do JavaScript usamos para alterar o HTML dentro de um `<div>`?

**a)** `element.innerHTML = "..."` ✅  
**b)** `element.setHTML("...")`  
**c)** `element.text = "..."`  
**d)** `element.jinja = "..."`

---

### ❓ Questão 10
Qual a diferença entre `random.sample(lista, 4)` e `random.choice(lista)` chamado 4 vezes?

**a)** Nenhuma, ambos fazem a mesma coisa.  
**b)** `sample` garante 4 itens únicos; `choice` pode repetir o mesmo item. ✅  
**c)** `sample` é para números; `choice` é para strings.  
**d)** `sample` é mais lento que `choice`.

---

## 3.8 Exercícios Práticos

### 🎯 Exercício 1: Melhorar Feedback Visual

No `static/app.js`, dentro da função `addEventListener`, o texto do botão muda para "Sorteando...". Use o Bootstrap para adicionar um **spinner** (ícone de carregamento) ao botão.

**Dica:** O HTML para um spinner é:
```html
<span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
```

Você pode adicionar isso ao `innerHTML` do botão junto com o texto.

---

### 🎯 Exercício 2: Exibir ID da Carta

Modifique o `cardHTML` dentro do `app.js` para que, abaixo do nome do Pokémon, ele exiba o ID da carta (ex: "ID: #25"). 

A variável `carta.id` já está disponível no loop.

**Exemplo de saída esperada:**
```
Pikachu
ID: #25
```

---

### 🎯 Exercício 3: Refatorar o JS (Modularização)

O código que gera o HTML do card em `app.js` é um bloco de string grande. Crie uma nova função JavaScript separada, chamada `criarCardHTML(carta)`, que recebe um objeto `carta` e retorna a string HTML do card. 

Em seguida, chame essa função dentro do loop `forEach`. 

Isso torna o código mais limpo (princípio da modularização, que vimos no Capítulo 5 de "Fundamentos de Python 1").

**Estrutura esperada:**
```javascript
function criarCardHTML(carta) {
    return `
        <div class="card">
            <!-- ... -->
        </div>
    `;
}

function exibirCartasNoModal(cartas) {
    // ...
    cartas.forEach(carta => {
        col.innerHTML = criarCardHTML(carta);
        // ...
    });
}
```

---

## 📚 Resumo do Capítulo

Neste capítulo, você aprendeu:

✅ Como simular aleatoriedade com `random.sample()`  
✅ Como criar rotas de API que retornam JSON  
✅ A diferença entre GET e POST  
✅ O que é AJAX e por que é importante  
✅ Como usar `fetch()` para fazer requisições assíncronas  
✅ Como manipular o DOM com JavaScript  
✅ Como criar e exibir Modals do Bootstrap dinamicamente  
✅ Princípios essenciais de validação de dados do cliente  
✅ Como proteger rotas contra requisições maliciosas

---

## 🎯 Próximo Capítulo

No [Capítulo 4](./capitulo_04.md), vamos implementar a **persistência de dados**! Aprenderemos a usar o **Local Storage** do navegador para salvar a coleção do usuário e implementar o timer de 1 hora, garantindo que o progresso não seja perdido ao recarregar a página.

---

**[← Capítulo Anterior](./capitulo_02.md)** | **[Voltar para README](../README.md)** | **[Próximo Capítulo →](./capitulo_04.md)**