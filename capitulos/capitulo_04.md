# Capítulo 4: Salvando o Progresso (Persistência no Navegador)

## 📑 Sumário do Capítulo

- [4.1 Lógica por trás da Persistência de Dados](#41-lógica-por-trás-da-persistência-de-dados)
- [4.2 Opção 1 (Client-Side): O Local Storage do Navegador](#42-opção-1-client-side-o-local-storage-do-navegador)
- [4.3 Projeto (Parte 1): Salvando a Coleção no Local Storage](#43-projeto-parte-1-salvando-a-coleção-no-local-storage)
- [4.4 Projeto (Parte 2): O Timer de 1 Hora](#44-projeto-parte-2-o-timer-de-1-hora)
- [4.5 XSS: O Perigo de Dados Não Sanitizados](#45-xss-o-perigo-de-dados-não-sanitizados)
- [4.6 Verifique seu Conhecimento](#46-verifique-seu-conhecimento)
- [4.7 Exercícios Práticos](#47-exercícios-práticos)

---

## 🎯 Introdução

No Capítulo 3, demos vida à nossa aplicação. Agora, o usuário pode clicar em um botão, chamar uma API no nosso servidor Flask e ver 4 cartas aleatórias aparecerem em um modal. No entanto, nosso projeto tem um problema fundamental: **ele não tem memória**.

Se você abrir um pacote, capturar um "Pikachu" e recarregar a página, seu progresso desaparece. A Pokédex volta a mostrar os 151 Pokémon como "Não Capturado". O botão "Abrir Pacote" está imediatamente disponível, mesmo que o usuário devesse esperar uma hora.

Neste capítulo, vamos resolver isso implementando a **persistência de dados**. Vamos aprender a salvar o estado do jogo (a coleção do usuário e o timer) diretamente no navegador.

---

## 4.1 Lógica por trás da Persistência de Dados

### 🤔 O Problema: A Web é "Sem Estado"

A Web, por padrão, é **stateless** (sem estado). O servidor Flask não "se lembra" de quem você é entre uma requisição e outra. Cada visita à página `/` é tratada como se fosse a primeira vez.

Para "lembrar" do progresso de um usuário, precisamos armazenar seus dados em algum lugar.

### 🗂️ Duas Opções de Persistência

Existem duas localizações principais para armazenar dados:

#### 1. Server-Side (Lado do Servidor)

Os dados são armazenados em um banco de dados (como o MySQL que veremos no próximo capítulo) no nosso servidor.

| Prós | Contras |
|------|---------|
| ✅ Seguro e permanente | ❌ Mais complexo |
| ✅ Acessível de qualquer dispositivo | ❌ Exige login |
| ✅ Backup e controle centralizado | ❌ Consome recursos do servidor |

#### 2. Client-Side (Lado do Cliente)

Os dados são armazenados diretamente no navegador do usuário.

| Prós | Contras |
|------|---------|
| ✅ Extremamente rápido | ❌ Inseguro (usuário pode ver/editar) |
| ✅ Funciona offline | ❌ Preso àquele navegador/dispositivo |
| ✅ Não exige servidor complexo | ❌ Pode ser perdido se limpar cache |
| ✅ Perfeito para estado temporário | ❌ Não é adequado para dados sensíveis |

### 🎯 Nossa Estratégia

Para nosso projeto, começaremos com a abordagem **Client-Side**, que é mais simples e ideal para a nossa necessidade. A ferramenta que usaremos é o **Local Storage**.

---

## 4.2 Opção 1 (Client-Side): O Local Storage do Navegador

### 📦 O que é Local Storage?

O **Local Storage** (Armazenamento Local) é um pequeno "banco de dados" em formato **chave-valor** que todo navegador moderno oferece. Pense nele como um dicionário Python que sobrevive mesmo se você fechar o navegador.

```javascript
// Conceito similar a um dicionário Python
// Python: meu_dict = {'nome': 'Ash', 'idade': 10}
// Local Storage: localStorage.setItem('nome', 'Ash')
```

### ⚠️ Limitação Importante

**O Local Storage só pode armazenar strings.** Isso é uma limitação importante. Não podemos salvar uma lista ou um dicionário JavaScript diretamente.

### 🔧 Interface do JavaScript

| Método | Descrição | Exemplo |
|--------|-----------|---------|
| `localStorage.setItem('chave', 'valor')` | Salva um valor | `localStorage.setItem('nome', 'Ash')` |
| `localStorage.getItem('chave')` | Recupera um valor | `const nome = localStorage.getItem('nome')` |
| `localStorage.removeItem('chave')` | Remove um valor | `localStorage.removeItem('nome')` |
| `localStorage.clear()` | Limpa tudo | `localStorage.clear()` |

### 🤔 O Problema: Como salvar objetos?

Como salvamos nossa coleção de cartas, que é um objeto (ex: `{1: true, 4: true, 25: true}`), se o Local Storage só armazena strings?

### ✅ A Solução: JSON.stringify() e JSON.parse()

Usamos o formato **JSON** como uma "ponte":

#### Para Salvar (Objeto → String)

```javascript
const minhaColecao = {1: true, 25: true, 150: true};

// Converte objeto em string JSON
const stringJSON = JSON.stringify(minhaColecao);
// Resultado: '{"1":true,"25":true,"150":true}'

// Salva no Local Storage
localStorage.setItem('minhaColecao', stringJSON);
```

#### Para Ler (String → Objeto)

```javascript
// Lê a string do Local Storage
const stringSalva = localStorage.getItem('minhaColecao');
// Resultado: '{"1":true,"25":true,"150":true}'

// Converte de volta em objeto
const minhaColecao = JSON.parse(stringSalva);
// Resultado: {1: true, 25: true, 150: true}
```

### 🔄 Fluxo Completo

```
Objeto JS → JSON.stringify() → String → Local Storage
                                            ↓
Objeto JS ← JSON.parse() ← String ← Local Storage
```

---

## 4.3 Projeto (Parte 1): Salvando a Coleção no Local Storage

### 🎯 Mudança de Arquitetura

Nossa arquitetura mudará significativamente. Em vez do Flask/Jinja2 construir a galeria de 151 Pokémon (que era estática), faremos o **JavaScript** construir essa galeria no carregamento da página. Isso nos permite verificar antes quais cartas o usuário já possui no Local Storage.

### 📁 Nova Estrutura

```
Antes (Capítulo 2):
Flask/Jinja2 → Gera HTML completo → Envia para navegador

Agora (Capítulo 4):
Flask → Envia HTML "vazio" → JavaScript busca dados da API
                          → JavaScript verifica Local Storage
                          → JavaScript gera HTML dinamicamente
```

### 💻 Passo 1: Refatorar app.py para ser uma API pura

O `index.html` não precisa mais receber a `lista_pokemon`. O JavaScript vai pedir essa lista.

```python
from flask import Flask, render_template, jsonify
import requests
import random

app = Flask(__name__)

POKEAPI_URL = "https://pokeapi.co/api/v2/pokemon?limit=151"

def carregar_dados_pokemon():
    """Busca os dados dos 151 Pokémon na PokeAPI."""
    try:
        response = requests.get(POKEAPI_URL)
        response.raise_for_status()
        
        data = response.json()
        resultados = data['results']
        
        lista_pokemon = []
        for i, poke in enumerate(resultados):
            pokemon_id = i + 1
            lista_pokemon.append({
                'id': pokemon_id,
                'nome': poke['name'].title(),
                'capturado': False
            })
        return lista_pokemon
    
    except requests.RequestException as e:
        print(f"Erro ao buscar dados da PokeAPI: {e}")
        return []

# Carrega a lista UMA VEZ quando o app inicia
LISTA_GLOBAL_POKEMON = carregar_dados_pokemon()

# --- ROTA INDEX SIMPLIFICADA ---
@app.route('/')
def index():
    # Não precisamos mais passar a lista de pokémon por aqui
    return render_template('index.html', treinador="Ash Ketchum")

# --- NOVA ROTA DE API ---
@app.route('/api/pokemon')
def api_pokemon():
    """Retorna a lista completa de 151 Pokémon como JSON."""
    if not LISTA_GLOBAL_POKEMON:
        return jsonify({'erro': 'Falha ao carregar dados dos Pokémon'}), 500
    
    return jsonify(LISTA_GLOBAL_POKEMON)

@app.route('/abrir_pacote', methods=['POST'])
def abrir_pacote():
    """Sorteia 4 cartas da lista global e as retorna como JSON."""
    try:
        if not LISTA_GLOBAL_POKEMON:
            return jsonify({'erro': 'Falha ao carregar dados dos Pokémon'}), 500
        
        cartas_sorteadas = random.sample(LISTA_GLOBAL_POKEMON, 4)
        return jsonify({'cartas': cartas_sorteadas})
    
    except Exception as e:
        return jsonify({'erro': str(e)}), 500

if __name__ == '__main__':
    app.run(debug=True)
```

### 💻 Passo 2: Refatorar index.html para ter uma galeria vazia

Remova o loop `{% for ... %}`. O JavaScript irá preencher esta div agora.

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
    
    <!-- Galeria será preenchida pelo JavaScript -->
    <div id="galeria-principal" class="row">
        <div class="text-center mt-5">
            <div class="spinner-border text-primary" role="status">
                <span class="visually-hidden">Carregando Pokédex...</span>
            </div>
            <p>Carregando Pokédex...</p>
        </div>
    </div>
    
    <!-- Modal para cartas reveladas -->
    <div class="modal fade" id="modal-cartas-reveladas" tabindex="-1" aria-hidden="true">
        <div class="modal-dialog modal-lg modal-dialog-centered">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title">Cartas Reveladas!</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body" id="corpo-modal-cartas">
                    <!-- As cartas serão inseridas aqui -->
                </div>
            </div>
        </div>
    </div>
{% endblock %}
```

### 💻 Passo 3: Atualizar static/app.js (A Mágica Acontece Aqui)

Este é o nosso maior passo. Vamos reestruturar o `app.js` para:

1. Ter uma variável global `minhaColecao`
2. Funções para carregar e salvar do Local Storage
3. Uma função para renderizar a galeria principal, verificando a coleção
4. Atualizar a coleção ao abrir um pacote

```javascript
// ==========================================
// CONSTANTES E VARIÁVEIS GLOBAIS
// ==========================================

// Chaves do Local Storage
const CHAVE_COLECAO = 'poke_colecao';
const CHAVE_TIMER = 'poke_timer';

// Variável global para guardar o estado da coleção
let minhaColecao = {};

// ==========================================
// FUNÇÕES DE LÓGICA DA COLEÇÃO
// ==========================================

function carregarColecaoDoStorage() {
    console.log("Carregando coleção do Local Storage...");
    const colecaoSalva = localStorage.getItem(CHAVE_COLECAO);
    
    if (colecaoSalva) {
        minhaColecao = JSON.parse(colecaoSalva);
    } else {
        minhaColecao = {};
    }
}

function salvarColecaoNoStorage() {
    console.log("Salvando coleção...");
    localStorage.setItem(CHAVE_COLECAO, JSON.stringify(minhaColecao));
}

function criarCardHTML(carta, capturado = false) {
    // Classe CSS dinâmica para o card (cinza se não capturado)
    const classeCapturado = capturado ? "shadow-sm" : "nao-capturado";
    
    // Badge dinâmico (selo)
    const badge = capturado 
        ? '<span class="badge bg-success">Capturado</span>' 
        : '<span class="badge bg-secondary">Não Capturado</span>';
    
    return `
        <div class="col-md-3 mb-4">
            <div class="card text-center ${classeCapturado}">
                <img src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${carta.id}.png" 
                     class="card-img-top p-3" 
                     alt="${carta.nome}">
                <div class="card-body">
                    <h5 class="card-title">${carta.nome}</h5>
                    <p class="card-text small text-muted">ID: #${carta.id}</p>
                    ${badge}
                </div>
            </div>
        </div>
    `;
}

async function renderizarGaleriaPrincipal() {
    console.log("Renderizando galeria principal...");
    const galeria = document.getElementById('galeria-principal');
    if (!galeria) return;
    
    // Limpa a galeria (remove o "Carregando...")
    galeria.innerHTML = '';
    
    try {
        // 1. Busca a lista-mestre de 151 Pokémon
        const response = await fetch('/api/pokemon');
        if (!response.ok) throw new Error('Falha ao buscar /api/pokemon');
        const listaMestre = await response.json();
        
        // 2. Constrói o HTML
        let htmlGaleria = '';
        for (const poke of listaMestre) {
            // 3. Verifica se o ID do poke está na 'minhaColecao'
            const capturado = minhaColecao.hasOwnProperty(poke.id);
            htmlGaleria += criarCardHTML(poke, capturado);
        }
        
        // 4. Insere todo o HTML de uma vez (mais rápido)
        galeria.innerHTML = htmlGaleria;
        
    } catch (error) {
        console.error("Erro ao renderizar galeria:", error);
        galeria.innerHTML = '<p class="text-danger">Erro ao carregar a Pokédex. Tente recarregar a página.</p>';
    }
}

// ==========================================
// LÓGICA DO MODAL DE CARTAS
// ==========================================

function exibirCartasNoModal(cartas) {
    const corpoModal = document.getElementById('corpo-modal-cartas');
    corpoModal.innerHTML = '';
    const row = document.createElement('div');
    row.className = 'row';
    
    let novasCartasCapturadas = false;
    
    cartas.forEach(carta => {
        // Verifica se a carta JÁ estava na coleção
        const ehNova = !minhaColecao.hasOwnProperty(carta.id);
        
        if (ehNova) {
            console.log(`Nova captura: ${carta.nome}`);
            minhaColecao[carta.id] = true;  // Adiciona à coleção
            novasCartasCapturadas = true;
        }
        
        const cardHTML = `
            <div class="col-md-6 mb-3">
                <div class="card text-center shadow-sm">
                    <img src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${carta.id}.png" 
                         class="card-img-top p-3" alt="${carta.nome}">
                    <div class="card-body">
                        <h5 class="card-title">${carta.nome}</h5>
                        ${ehNova 
                            ? '<span class="badge bg-primary">Nova!</span>' 
                            : '<span class="badge bg-warning">Duplicata</span>'}
                    </div>
                </div>
            </div>
        `;
        row.innerHTML += cardHTML;
    });
    
    corpoModal.appendChild(row);
    
    // Se houveram cartas novas, salva e re-renderiza a galeria
    if (novasCartasCapturadas) {
        salvarColecaoNoStorage();
        renderizarGaleriaPrincipal();  // Atualiza a galeria principal em tempo real
    }
}

// ==========================================
// LÓGICA PRINCIPAL (DOMContentLoaded)
// ==========================================

document.addEventListener('DOMContentLoaded', () => {
    // 1. Carrega a coleção salva
    carregarColecaoDoStorage();
    
    // 2. Renderiza a galeria principal
    renderizarGaleriaPrincipal();
    
    const btnAbrir = document.getElementById('btn-abrir-pacote');
    const modalElement = document.getElementById('modal-cartas-reveladas');
    if (!btnAbrir || !modalElement) return;
    
    const meuModal = new bootstrap.Modal(modalElement);
    
    // 3. Evento de clique no botão
    btnAbrir.addEventListener('click', () => {
        btnAbrir.disabled = true;
        btnAbrir.textContent = "Sorteando...";
        
        fetch('/abrir_pacote', { method: 'POST' })
        .then(response => {
            if (!response.ok) throw new Error('Falha na requisição');
            return response.json();
        })
        .then(data => {
            if (data.erro) throw new Error(data.erro);
            
            console.log("Cartas recebidas:", data.cartas);
            exibirCartasNoModal(data.cartas);
            meuModal.show();
        })
        .catch(error => {
            console.error('Erro ao abrir pacote:', error);
            alert('Não foi possível abrir o pacote. Tente novamente.');
        })
        .finally(() => {
            btnAbrir.disabled = false;
            btnAbrir.textContent = "Abrir Pacote!";
        });
    });
});
```

### 💻 Passo 4: Adicionar o CSS para "Não Capturado"

Para a classe `nao-capturado` funcionar, crie o arquivo `static/style.css`:

```css
/* Estilo para Pokémon não capturados */
.nao-capturado img {
    opacity: 0.3;
    filter: grayscale(100%);
}

.nao-capturado {
    background-color: #f8f9fa;
}
```

E não se esqueça de linkar este CSS em `templates/base.html` (dentro do `<head>`):

```html
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    
    <!-- Nosso CSS customizado -->
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
    
    <title>{% block title %}PokéBooster{% endblock %}</title>
</head>
```

### 🎉 Testando

Recarregue a página. Agora:

1. A galeria carrega e todos os Pokémon aparecem em cinza
2. Abra um pacote
3. Quando o modal fechar, você verá que as 4 cartas que você tirou agora estão coloridas na galeria principal!
4. **Recarregue a página... o progresso foi salvo!** 🎊

---

## 4.4 Projeto (Parte 2): O Timer de 1 Hora

### 🎯 Objetivo

Agora vamos implementar o timer, também usando Local Storage. A lógica é:

1. Quando um pacote é aberto, salvamos o **timestamp** (o momento exato) em que o próximo pacote estará disponível (agora + 1 hora)
2. A cada segundo, verificamos a hora atual contra o timestamp salvo
3. Se a hora atual for menor que o timestamp salvo, o botão fica desabilitado e mostramos o tempo restante
4. Se for maior (ou se não houver timer salvo), o botão fica habilitado

### 💻 Adicione as funções de timer no static/app.js

```javascript
// ==========================================
// FUNÇÕES DE LÓGICA DO TIMER
// ==========================================

let idIntervaloTimer = null;  // Guarda a referência do nosso 'setInterval'

function atualizarBotaoTimer(btn, proximaAbertura) {
    const agora = new Date().getTime();
    const tempoRestante = proximaAbertura - agora;  // Em milissegundos
    
    if (tempoRestante > 0) {
        // Ainda em cooldown
        btn.disabled = true;
        
        // Converte ms para minutos e segundos
        const minutos = Math.floor((tempoRestante % (1000 * 60 * 60)) / (1000 * 60));
        const segundos = Math.floor((tempoRestante % (1000 * 60)) / 1000);
        
        // Formata para 00:00
        const minFormatado = String(minutos).padStart(2, '0');
        const segFormatado = String(segundos).padStart(2, '0');
        
        btn.textContent = `Próximo Pacote em ${minFormatado}:${segFormatado}`;
    } else {
        // Cooldown acabou
        btn.disabled = false;
        btn.textContent = 'Abrir Pacote!';
        
        // Para o 'setInterval' pois não é mais necessário
        if (idIntervaloTimer) {
            clearInterval(idIntervaloTimer);
            idIntervaloTimer = null;
        }
    }
}

function iniciarGerenciadorTimer(btn) {
    const proximaAbertura = localStorage.getItem(CHAVE_TIMER);
    
    if (proximaAbertura) {
        // Se temos um timer salvo, começa a verificar a cada segundo
        // Limpa qualquer timer antigo para evitar duplicatas
        if (idIntervaloTimer) clearInterval(idIntervaloTimer);
        
        idIntervaloTimer = setInterval(() => {
            // A função 'atualizarBotaoTimer' é chamada a cada 1000ms
            atualizarBotaoTimer(btn, proximaAbertura);
        }, 1000);
        
        // Chama uma vez imediatamente para definir o estado inicial
        atualizarBotaoTimer(btn, proximaAbertura);
    } else {
        // Sem timer, botão está pronto
        btn.disabled = false;
        btn.textContent = 'Abrir Pacote!';
    }
}
```

### 💻 Atualização no DOMContentLoaded

```javascript
document.addEventListener('DOMContentLoaded', () => {
    // 1. Carrega a coleção salva
    carregarColecaoDoStorage();
    
    // 2. Renderiza a galeria principal
    renderizarGaleriaPrincipal();
    
    const btnAbrir = document.getElementById('btn-abrir-pacote');
    const modalElement = document.getElementById('modal-cartas-reveladas');
    if (!btnAbrir || !modalElement) return;
    
    const meuModal = new bootstrap.Modal(modalElement);
    
    // 3. Inicia o gerenciador do timer
    iniciarGerenciadorTimer(btnAbrir);
    
    // 4. Evento de clique no botão
    btnAbrir.addEventListener('click', () => {
        btnAbrir.disabled = true;
        btnAbrir.textContent = "Sorteando...";
        
        fetch('/abrir_pacote', { method: 'POST' })
        .then(response => {
            if (!response.ok) throw new Error('Falha na requisição');
            return response.json();
        })
        .then(data => {
            if (data.erro) throw new Error(data.erro);
            
            console.log("Cartas recebidas:", data.cartas);
            exibirCartasNoModal(data.cartas);
            
            // Salva o timestamp (1h = 3600 * 1000 ms)
            // Para testar, mude para 10 segundos: 10 * 1000
            const proximaAbertura = new Date().getTime() + (3600 * 1000);
            localStorage.setItem(CHAVE_TIMER, proximaAbertura);
            
            // Inicia o timer novamente
            iniciarGerenciadorTimer(btnAbrir);
            
            meuModal.show();
        })
        .catch(error => {
            console.error('Erro ao abrir pacote:', error);
            alert('Não foi possível abrir o pacote. Tente novamente.');
            btnAbrir.disabled = false;
            btnAbrir.textContent = "Abrir Pacote!";
        });
        // Removido o .finally para que o timer controle o botão
    });
});
```

### 🎉 Testando

Abra seu pacote. O modal fechará e o botão iniciará uma contagem regressiva de 1 hora (ou 10 segundos se você mudou para testar). 

**Se você recarregar a página, a contagem continua!** 🎊

### 🏆 Conquista Desbloqueada

Concluímos a **Fase 1** do nosso projeto: uma aplicação web totalmente funcional, com lógica de jogo e persistência no lado do cliente.

> 💡 **Desafio do Capítulo 4:** O que acontece se você abrir o jogo em outro navegador ou limpar o cache? Todo o seu progresso é perdido. No próximo capítulo, vamos resolver isso, dando ao usuário a opção de salvar seu progresso na "nuvem" (nosso banco de dados MySQL).

---

## 4.5 XSS: O Perigo de Dados Não Sanitizados

### ⚠️ O que é XSS (Cross-Site Scripting)?

**XSS (Cross-Site Scripting)** é uma das vulnerabilidades mais comuns e perigosas em aplicações web. Ela ocorre quando um atacante consegue injetar código malicioso (geralmente JavaScript) em páginas visualizadas por outros usuários.

O nome "Cross-Site" vem do fato de que o ataque permite que um site malicioso execute scripts no contexto de outro site confiável.

### 🎭 Como XSS Funciona?

Imagine que sua aplicação permite que usuários salvem um "apelido" no Local Storage e depois exibe esse apelido na página:

```javascript
// ❌ Código VULNERÁVEL - NÃO USE!
let apelido = localStorage.getItem('apelido');
document.getElementById('saudacao').innerHTML = 'Olá, ' + apelido + '!';
```

Se um usuário malicioso salvar o seguinte "apelido":

```javascript
localStorage.setItem('apelido', '<img src=x onerror="alert(\'XSS Attack!\')">');
```

Quando a página carregar, o código malicioso será executado! O atacante poderia:

- 🔓 Roubar cookies de sessão
- 🔀 Redirecionar o usuário para sites maliciosos
- ✏️ Modificar o conteúdo da página
- 🎣 Capturar informações sensíveis (senhas, dados de cartão)

### 📊 Tipos de XSS

#### 1. XSS Armazenado (Stored XSS)

O código malicioso é armazenado no servidor (banco de dados) e executado toda vez que a página é carregada.

**Exemplo:** Um comentário em um blog contendo código malicioso.

#### 2. XSS Refletido (Reflected XSS)

O código malicioso vem de uma requisição (URL, formulário) e é imediatamente refletido na resposta.

**Exemplo:** Um parâmetro de busca que é exibido na página sem sanitização.

#### 3. XSS Baseado em DOM (DOM-based XSS)

O código malicioso manipula o DOM diretamente no navegador, sem passar pelo servidor. **Este é o tipo mais relevante para Local Storage!**

### 🚨 XSS e Local Storage: Uma Combinação Perigosa

O Local Storage é especialmente vulnerável a XSS porque:

1. **Persistência**: Os dados permanecem mesmo após fechar o navegador
2. **Acesso Fácil**: Qualquer script na página pode ler/escrever no Local Storage
3. **Sem Proteção HTTP-Only**: Diferente de cookies, não há flag de proteção

### 🛡️ Como Proteger Contra XSS no Local Storage

#### 1. Nunca Use innerHTML com Dados Não Confiáveis

```javascript
// ❌ VULNERÁVEL
element.innerHTML = localStorage.getItem('dados');

// ✅ SEGURO
element.textContent = localStorage.getItem('dados');
```

A propriedade `textContent` trata tudo como **texto puro**, sem interpretar HTML/JavaScript.

#### 2. Sanitize Dados Antes de Armazenar

```javascript
// Função para sanitizar strings
function sanitizeInput(input) {
    const div = document.createElement('div');
    div.textContent = input;
    return div.innerHTML;
}

// Uso
let nomeUsuario = sanitizeInput(document.getElementById('nome').value);
localStorage.setItem('nome', nomeUsuario);
```

#### 3. Valide Dados ao Recuperar do Local Storage

```javascript
function getSecureData(key) {
    let data = localStorage.getItem(key);
    
    // Validar que os dados são do tipo esperado
    if (typeof data !== 'string') {
        return null;
    }
    
    // Remover caracteres perigosos
    data = data.replace(/<script[^>]*>.*?<\/script>/gi, '');
    data = data.replace(/<iframe[^>]*>.*?<\/iframe>/gi, '');
    data = data.replace(/on\w+\s*=\s*["'][^"']*["']/gi, '');
    
    return data;
}
```

#### 4. Use JSON.parse com Cuidado

```javascript
// ❌ VULNERÁVEL - pode executar código malicioso
let dados = eval('(' + localStorage.getItem('dados') + ')');

// ✅ SEGURO - JSON.parse não executa código
try {
    let dados = JSON.parse(localStorage.getItem('dados'));
    
    // Validar estrutura esperada
    if (!dados || typeof dados !== 'object') {
        throw new Error('Dados inválidos');
    }
} catch (e) {
    console.error('Erro ao carregar dados:', e);
    // Usar valores padrão
}
```

### 💻 Implementando Proteção XSS no PokéBooster

Vamos proteger nossa aplicação contra XSS ao trabalhar com Local Storage:

```javascript
// static/app.js - Versão Segura

// Função para sanitizar dados
function sanitizeHTML(str) {
    const temp = document.createElement('div');
    temp.textContent = str;
    return temp.innerHTML;
}

// Função segura para salvar coleção
function salvarColecaoSegura(colecao) {
    try {
        // Validar estrutura antes de salvar
        if (typeof colecao !== 'object') {
            throw new Error('Coleção deve ser um objeto');
        }
        
        // Validar cada ID
        const colecaoValidada = {};
        for (const id in colecao) {
            const pokeId = parseInt(id);
            if (pokeId >= 1 && pokeId <= 151) {
                colecaoValidada[pokeId] = true;
            }
        }
        
        localStorage.setItem(CHAVE_COLECAO, JSON.stringify(colecaoValidada));
        return true;
    } catch (e) {
        console.error('Erro ao salvar coleção:', e);
        return false;
    }
}

// Função segura para carregar coleção
function carregarColecaoSegura() {
    try {
        const dados = localStorage.getItem(CHAVE_COLECAO);
        
        if (!dados) {
            return {};
        }
        
        const colecao = JSON.parse(dados);
        
        // Validar estrutura
        if (typeof colecao !== 'object') {
            throw new Error('Formato inválido');
        }
        
        // Validar cada item
        const colecaoValidada = {};
        for (const id in colecao) {
            const pokeId = parseInt(id);
            if (pokeId >= 1 && pokeId <= 151) {
                colecaoValidada[pokeId] = true;
            }
        }
        
        return colecaoValidada;
        
    } catch (e) {
        console.error('Erro ao carregar coleção:', e);
        // Limpar dados corrompidos
        localStorage.removeItem(CHAVE_COLECAO);
        return {};
    }
}

// Exibir cartas de forma segura
function criarCardHTMLSeguro(carta, capturado) {
    const div = document.createElement('div');
    div.className = 'col-md-3 mb-4';
    
    const card = document.createElement('div');
    card.className = capturado ? 'card text-center shadow-sm' : 'card text-center nao-capturado';
    
    const img = document.createElement('img');
    img.className = 'card-img-top p-3';
    // Validar ID antes de usar na URL
    const pokeId = parseInt(carta.id);
    if (pokeId >= 1 && pokeId <= 151) {
        img.src = `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${pokeId}.png`;
    } else {
        img.src = '/static/placeholder.png';
    }
    img.alt = sanitizeHTML(carta.nome);
    
    const cardBody = document.createElement('div');
    cardBody.className = 'card-body';
    
    const titulo = document.createElement('h5');
    titulo.className = 'card-title';
    // Usar textContent ao invés de innerHTML
    titulo.textContent = carta.nome;
    
    const id = document.createElement('p');
    id.className = 'card-text small text-muted';
    id.textContent = `ID: #${carta.id}`;
    
    const badge = document.createElement('span');
    badge.className = capturado ? 'badge bg-success' : 'badge bg-secondary';
    badge.textContent = capturado ? 'Capturado' : 'Não Capturado';
    
    cardBody.appendChild(titulo);
    cardBody.appendChild(id);
    cardBody.appendChild(badge);
    card.appendChild(img);
    card.appendChild(cardBody);
    div.appendChild(card);
    
    return div;
}
```

### 🔒 Content Security Policy (CSP)

Uma camada adicional de proteção é implementar **CSP** no servidor Flask:

```python
# app.py
from flask import Flask, make_response

@app.after_request
def set_csp(response):
    response.headers['Content-Security-Policy'] = (
        "default-src 'self'; "
        "script-src 'self' https://cdn.jsdelivr.net; "
        "style-src 'self' https://cdn.jsdelivr.net; "
        "img-src 'self' https://raw.githubusercontent.com data:;"
    )
    return response
```

### ✅ Checklist de Segurança para Local Storage

- [ ] Nunca usar `innerHTML` com dados do Local Storage
- [ ] Sempre usar `textContent` ou criar elementos via DOM
- [ ] Validar tipo e estrutura dos dados ao recuperar
- [ ] Sanitizar dados antes de armazenar
- [ ] Usar `JSON.parse` ao invés de `eval`
- [ ] Implementar tratamento de erros robusto
- [ ] Limpar dados corrompidos automaticamente
- [ ] Implementar CSP no servidor
- [ ] Validar URLs antes de usar em atributos `src`
- [ ] Nunca armazenar dados sensíveis (senhas, tokens) no Local Storage

### 🎯 Pontos-Chave

> 🚨 **XSS é uma ameaça real**: Pode comprometer completamente a segurança da aplicação

> 💾 **Local Storage é vulnerável**: Qualquer script pode acessá-lo

> ✅ **Nunca confie em dados armazenados**: Sempre valide e sanitize

> 📝 **Use textContent, não innerHTML**: Para exibir dados não confiáveis

> 🛡️ **Implemente múltiplas camadas de defesa**: Validação + Sanitização + CSP

A proteção contra XSS é fundamental para qualquer aplicação web moderna. No próximo capítulo, veremos como proteger dados no servidor contra outros tipos de ataques.

---

## 4.6 Verifique seu Conhecimento

### ❓ Questão 1
Qual é a principal desvantagem da persistência de dados no Local Storage (Client-Side)?

**a)** É muito lenta para ler e escrever.  
**b)** Os dados ficam presos àquele navegador/dispositivo e podem ser perdidos. ✅  
**c)** Só funciona com o framework Flask.  
**d)** Ocupa muito espaço no servidor.

---

### ❓ Questão 2
O Local Storage pode armazenar diretamente quais tipos de dados?

**a)** Apenas Listas e Dicionários JavaScript.  
**b)** Apenas Números.  
**c)** Apenas Strings. ✅  
**d)** Listas, Dicionários, Números e Strings.

---

### ❓ Questão 3
Qual par de funções é usado para salvar um objeto JavaScript no Local Storage?

**a)** `JSON.parse()` para salvar, `JSON.stringify()` para ler.  
**b)** `localStorage.setObject()` para salvar, `localStorage.getObject()` para ler.  
**c)** `JSON.stringify()` para salvar, `JSON.parse()` para ler. ✅  
**d)** `localStorage.save()` para salvar, `localStorage.load()` para ler.

---

### ❓ Questão 4
Por que refatoramos o app.py para carregar a galeria principal via JavaScript em vez de Jinja2?

**a)** Porque JavaScript é mais rápido que Jinja2.  
**b)** Porque o Jinja2 não sabe fazer loops `{% for %}`.  
**c)** Para permitir que o JavaScript verifique o Local Storage antes de exibir os cards, decidindo quais estão "Capturados". ✅  
**d)** Para reduzir o uso de memória do servidor Flask.

---

### ❓ Questão 5
Em `static/app.js`, qual é o propósito da função `async/await` em `renderizarGaleriaPrincipal()`?

**a)** Ela permite que o `fetch('/api/pokemon')` aconteça em segundo plano, e o código "espera" (await) pela resposta antes de continuar. ✅  
**b)** Ela criptografa os dados do Pokémon.  
**c)** Ela garante que o loop for rode mais rápido.  
**d)** `async/await` é uma sintaxe do Python, não do JavaScript.

---

### ❓ Questão 6
O que a função `minhaColecao.hasOwnProperty(poke.id)` verifica?

**a)** Se o Pokémon é uma propriedade do `poke.id`.  
**b)** Se a chave (ex: "25") existe dentro do nosso objeto `minhaColecao`. ✅  
**c)** Se o Pokémon é raro.  
**d)** Se a PokeAPI possui aquele ID.

---

### ❓ Questão 7
Qual é a principal falha de usar `setTimeout(..., 3600000)` para um timer de 1 hora?

**a)** Ele não é preciso e pode errar por alguns minutos.  
**b)** Ele consome muita CPU do cliente.  
**c)** Ele é resetado e perdido se o usuário fechar a aba ou recarregar a página. ✅  
**d)** `setTimeout` não existe em JavaScript.

---

### ❓ Questão 8
Como nossa lógica de timer resolve o problema de recarregar a página?

**a)** Ela salva o timestamp exato da "próxima abertura" no Local Storage e o verifica a cada segundo. ✅  
**b)** Ela pede ao Flask para lembrar do timer.  
**c)** Ela impede o usuário de recarregar a página.  
**d)** Ela usa `setTimeout` e o impede de ser resetado.

---

### ❓ Questão 9
Qual é o propósito do `setInterval(funcao, 1000)`?

**a)** Executar a `funcao` uma única vez após 1000 segundos.  
**b)** Chamar o servidor 1000 vezes.  
**c)** Criar um timer de 1000 segundos.  
**d)** Executar a `funcao` repetidamente, uma vez a cada 1000 milissegundos (1 segundo). ✅

---

### ❓ Questão 10
No `app.js`, por que atualizamos a `minhaColecao` e chamamos `salvarColecaoNoStorage()`?

**a)** `minhaColecao` é a variável em memória (rápida), `salvarColecao...` é a persistência no disco (lenta). ✅  
**b)** É redundante, apenas uma é necessária.  
**c)** `minhaColecao` salva no servidor, `salvarColecao...` salva no cliente.  
**d)** `minhaColecao` é para o timer, `salvarColecao...` é para as cartas.

---

## 4.7 Exercícios Práticos

### 🎯 Exercício 1: Botão de Reset

Adicione um botão "Resetar Jogo" em `index.html`. No `app.js`, adicione um "ouvinte" de clique para este novo botão. Ao ser clicado, ele deve:

**a)** Perguntar ao usuário "Tem certeza?" (usando `confirm('Tem certeza?')`)  
**b)** Se o usuário confirmar, remover os itens do Local Storage:
```javascript
localStorage.removeItem(CHAVE_COLECAO);
localStorage.removeItem(CHAVE_TIMER);
```
**c)** Recarregar a página (usando `location.reload()`) para que o jogo volte ao estado inicial

---

### 🎯 Exercício 2: Contador de Cartas

Em `index.html`, perto do título "Minha Coleção", mostre quantas cartas únicas o usuário já capturou.

**Dica:** Em `app.js`, dentro da `renderizarGaleriaPrincipal()`, você pode calcular o número de cartas em `minhaColecao` usando:

```javascript
const totalCapturadas = Object.keys(minhaColecao).length;
```

E inserir esse número em um `<span>` no HTML.

**Exemplo de resultado:**
```
Minha Coleção (Ash Ketchum) - 15/151 cartas
```

---

### 🎯 Exercício 3: Ajuste Fino do Timer (Testes)

No `app.js`, encontre a linha:

```javascript
const proximaAbertura = new Date().getTime() + (3600 * 1000);
```

Mude o tempo de 1 hora (`3600 * 1000`) para **10 segundos** (`10 * 1000`). 

Teste o ciclo completo:
1. Abra um pacote
2. Veja o timer de 10 segundos
3. Espere ele acabar
4. Veja o botão ser reabilitado

**Lembre-se de voltar para 1 hora depois dos testes!**

---

## 📚 Resumo do Capítulo

Neste capítulo, você aprendeu:

✅ A diferença entre persistência Client-Side e Server-Side  
✅ Como usar o Local Storage do navegador  
✅ Como converter objetos para JSON e vice-versa  
✅ Como refatorar a arquitetura para carregamento dinâmico com JavaScript  
✅ Como implementar um sistema de coleção persistente  
✅ Como criar um timer que sobrevive ao recarregamento da página  
✅ Os perigos do XSS e como proteger sua aplicação  
✅ Técnicas de sanitização e validação de dados

---

## 🎯 Próximo Capítulo

No [Capítulo 5](./capitulo_05.md), vamos **evoluir nosso jogo** implementando persistência no servidor! Aprenderemos a usar **MySQL** e **SQLAlchemy** para criar um banco de dados, implementar autenticação de usuários e dar aos jogadores a opção de "Salvar na Nuvem" sua coleção, tornando-a acessível de qualquer dispositivo.

---

**[← Capítulo Anterior](./capitulo_03.md)** | **[Voltar para README](../README.md)** | **[Próximo Capítulo →](./capitulo_05.md)**