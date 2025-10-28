# Capítulo 4: Salvando o Progresso (Persistência no Navegador)


## Sumário do Capítulo

- [4.1 Lógica por trás da Persistência de Dados
](#lógica-por-trás-da-persistência-de-dados)
- [4.2 Opção 1 (Client-Side): O Local Storage do Navegador
](#opção-1-client-side-o-local-storage-do-navegador)
- [4.3 Projeto (Parte 1): Salvando a Coleção no Local Storage
](#projeto-parte-1-salvando-a-coleção-no-local-storage)
- [4.4 Projeto (Parte 2): O Timer de 1 Hora
](#projeto-parte-2-o-timer-de-1-hora)
- [4.5 XSS: O Perigo de Dados Não Sanitizados
](#xss-o-perigo-de-dados-não-sanitizados)
- [4.6 Verifique seu Conhecimento
](#verifique-seu-conhecimento)
- [4.7 Exercícios Práticos
](#exercícios-práticos)

---

No Capítulo 3, demos vida à nossa aplicação. Agora, o usuário pode clicar em um botão, chamar uma **API** no nosso servidor **Flask** e ver 4 cartas aleatórias aparecerem em um modal. No entanto, nosso projeto tem um problema fundamental: ele não tem memória.

Se você abrir um pacote, capturar um "Pikachu" e recarregar a página, seu progresso desaparece. A Pokédex volta a mostrar os 151 Pokémon como "Não Capturado". O botão "Abrir Pacote" está imediatamente disponível, mesmo que o usuário devesse esperar uma hora.

Neste capítulo, vamos resolver isso implementando a persistência de dados. Vamos aprender a salvar o estado do jogo (a coleção do usuário e o timer) diretamente no navegador.


## 4.1 Lógica por trás da Persistência de Dados


Lógica por trás da Persistência: A Web, por padrão, é "sem estado" (stateless). O servidor **Flask** não "se lembra" de quem você é entre uma requisição e outra. Cada visita à página / é tratada como se fosse a primeira vez.

Para "lembrar" do progresso de um usuário, precisamos armazenar seus dados em algum lugar. Existem duas localizações principais para isso:

Server-Side (Lado do **Servidor**): Os dados são armazenados em um banco de dados (como o **MySQL** que veremos no próximo capítulo) no nosso servidor.

Pró: Seguro, permanente, acessível de qualquer dispositivo (celular, outro PC).

Contra: Mais complexo, exige login, consome recursos do servidor.

Client-Side (Lado do **Cliente**): Os dados são armazenados diretamente no navegador do usuário.

Pró: Extremamente rápido, funciona offline, não exige um servidor complexo, perfeito para salvar preferências ou estado de um jogo simples.

Contra: Inseguro (o usuário pode ver e editar), preso àquele navegador e dispositivo, pode ser perdido se o usuário limpar o cache.

Para nosso projeto, começaremos com a abordagem Client-Side, que é mais simples e ideal para a nossa necessidade. A ferramenta que usaremos para isso é o Local Storage.


## 4.2 Opção 1 (Client-Side): O Local Storage do Navegador


O Local Storage (Armazenamento Local) é um pequeno "banco de dados" em formato chave-valor que todo navegador moderno oferece. Pense nele como um dicionário **Python** que sobrevive mesmo se você fechar o navegador.

O Local Storage só pode armazenar strings. Isso é uma limitação importante. Não podemos salvar uma lista ou um dicionário **JavaScript** diretamente.

A interface do **JavaScript** para usá-lo é simples:

localStorage.setItem('chave', 'valor_string'): Salva um valor.

localStorage.getItem('chave'): Recupera um valor.

localStorage.removeItem('chave'): Remove um valor.

O Problema: Como salvamos nossa coleção de cartas, que é um objeto (ex: {1: true, 4: true, 25: true}), se ele só armazena strings?

A Solução: JSON.stringify() e JSON.parse()

Usamos o formato JSON como uma "ponte":

Para Salvar: Convertemos nosso objeto **JavaScript** em uma string JSON.

```javascript
const stringJSON = JSON.stringify(minhaColecao);

localStorage.setItem('minhaColecao', stringJSON); 

// Salva: "{"1":true,"25":true}"

Para Ler: Lemos a string JSON do Local Storage e a convertemos de volta em um objeto **JavaScript**.

const stringSalva = localStorage.getItem('minhaColecao');

const minhaColecao = JSON.parse(stringSalva); 

// Converte: "{"1":true,"25":true}" -> { 1: true, 25: true }

```


## 4.3 Projeto (Parte 1): Salvando a Coleção no Local Storage


Nossa arquitetura mudará significativamente. Em vez do **Flask**/**Jinja2** construir a galeria de 151 Pokémon (que era estática), faremos o **JavaScript** construir essa galeria no carregamento da página. Isso nos permite verificar antes quais cartas o usuário já possui no Local Storage.

Passo 1: Refatorar app.py para ser uma **API** pura

O index.html não precisa mais receber a lista_pokemon. O **JavaScript** vai pedir essa lista.

Crie uma nova rota de **API** em app.py que apenas retorna a lista global.

Simplifique a rota index para apenas renderizar o "molde" da página.

```python
```

# ... (importações, app, POKEAPI_URL, carregar_dados_pokemon) ...



LISTA_GLOBAL_POKEMON = carregar_dados_pokemon()



# --- ROTA INDEX SIMPLIFICADA ---

@app.route('/')

def index():

    # Não precisamos mais passar a lista de pokémon por aqui

    return render_template('index.html', treinador="Ash Ketchum")



# --- NOVA ROTA DE **API** ---

@app.route('/api/pokemon')

def api_pokemon():

    """Retorna a lista completa de 151 Pokémon como JSON."""

    if not LISTA_GLOBAL_POKEMON:

        return jsonify({'erro': 'Falha ao carregar dados dos Pokémon'}), 500

    

    return jsonify(LISTA_GLOBAL_POKEMON)



# ... (rota /abrir_pacote e app.run) ...

Passo 2: Refatorar index.html para ter uma galeria vazia

Remova o loop {% for ... %}. O **JavaScript** irá preencher esta div agora.

```html
{% block title %}Minha Coleção{% endblock %}

```

{% block content %}

    <div class="d-flex justify-content-between align-items-center mb-4">

        <h2>Minha Coleção ({{ treinador }})</h2>

        <button id="btn-abrir-pacote" class="btn btn-primary btn-lg">Abrir Pacote!</button>

    </div>

    

    <div id="galeria-principal" class="row">

        <div class="text-center mt-5">

            <div class="spinner-border text-primary" role="status">

                <span class="visually-hidden">Carregando Pokédex...</span>

            </div>

            <p>Carregando Pokédex...</p>

        </div>

    </div>

    

    {% endblock %}

Passo 3: Atualizar static/app.js (A Mágica Acontece Aqui)

Este é o nosso maior passo. Vamos reestruturar o app.js para:

Ter uma variável global minhaColecao.

Funções para carregar e salvar do Local Storage.

Uma função para renderizar a galeria principal, verificando a coleção.

Atualizar a coleção ao abrir um pacote.

```javascript
```

// Nossas "constantes" de chaves do Local Storage

const CHAVE_COLECAO = 'poke_colecao';

const CHAVE_TIMER = 'poke_timer';



// Variável global para guardar o estado da coleção

let minhaColecao = {};



// --- NOVO: Funções de Lógica da Coleção ---



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

    // Classe **CSS** dinâmica para o card (cinza se não capturado)

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

    // Adicionamos um **CSS** customizado (nao-capturado)

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

        

        // 2. Constrói o **HTML**

        let htmlGaleria = '';

        for (const poke of listaMestre) {

            // 3. Verifica se o ID do poke está na 'minhaColecao'

            const capturado = minhaColecao.hasOwnProperty(poke.id);

            htmlGaleria += criarCardHTML(poke, capturado);

        }

        

        // 4. Insere todo o **HTML** de uma vez (mais rápido)

        galeria.innerHTML = htmlGaleria;



    } catch (error) {

        console.error("Erro ao renderizar galeria:", error);

        galeria.innerHTML = '<p class="text-danger">Erro ao carregar a Pokédex. Tente recarregar a página.</p>';

    }

}



// --- Modificação da Lógica do Modal ---



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

            minhaColecao[carta.id] = true; // Adiciona à coleção

            novasCartasCapturadas = true;

        }



        const cardHTML = `

            <div class="col-md-6 mb-3">

                <div class="card text-center shadow-sm">

                    <img src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${carta.id}.png" 

                         class="card-img-top p-3" alt="${carta.nome}">

                    <div class="card-body">

                        <h5 class="card-title">${carta.nome}</h5>

                        ${ehNova ? '<span class="badge bg-primary">Nova!</span>' : '<span class="badge bg-warning">Duplicata</span>'}

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

        renderizarGaleriaPrincipal(); // Atualiza a galeria principal em tempo real

    }

}



// --- NOVO: Lógica do Timer (Próxima Seção) ---

// ... (será adicionado na seção 4.4) ...





// --- Lógica Principal (DOMContentLoaded) ---



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

    iniciarGerenciadorTimer(btnAbrir); // (será adicionado na seção 4.4)



    btnAbrir.addEventListener('click', () => {

        // ... (código do fetch '/abrir_pacote') ...

        // ...

        .then(data => {

            console.log("Cartas recebidas:", data.cartas);

            exibirCartasNoModal(data.cartas);

            

            // --- NOVO: Salva o timer ---

            // Salva o timestamp de QUANDO o usuário pode abrir de novo (1h = 3600 * 1000 ms)

            const proximaAbertura = new Date().getTime() + (3600 * 1000); 

            localStorage.setItem(CHAVE_TIMER, proximaAbertura);

            

            // Atualiza o botão imediatamente

            atualizarBotaoTimer(btnAbrir, proximaAbertura);

            

            meuModal.show();

        })

        // ... (código do .catch e .finally) ...

    });

});

Passo 4: Adicionar o **CSS** para "Não Capturado"

Para o class="nao-capturado" funcionar, crie o arquivo static/style.css e adicione a regra.

```css
.nao-capturado img {

    opacity: 0.3;

    filter: grayscale(100%);

}

E não se esqueça de linkar este **CSS** em templates/base.html (dentro do <head>):

<head>

    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">

    

    <title>{% block title %}...{% endblock %}</title>

</head>

Recarregue a página. Agora, a galeria carrega e todos os Pokémon aparecem em cinza. Abra um pacote. Quando o modal fechar, você verá que as 4 cartas que você tirou agora estão coloridas na galeria principal! Se você recarregar a página... o progresso foi salvo!

```


## 4.4 Projeto (Parte 2): O Timer de 1 Hora


Agora vamos implementar o timer, também usando Local Storage. A lógica é:

Quando um pacote é aberto, salvamos o timestamp (o momento exato) em que o próximo pacote estará disponível (agora + 1 hora).

A cada segundo, verificamos a hora atual contra o timestamp salvo.

Se a hora atual for menor que o timestamp salvo, o botão fica desabilitado e mostramos o tempo restante.

Se for maior (ou se não houver timer salvo), o botão fica habilitado.

Adicione as funções de timer no static/app.js:

```javascript
```

// ... (código anterior) ...



// --- NOVO: Funções de Lógica do Timer ---



let idIntervaloTimer = null; // Guarda a referência do nosso 'setInterval'



function atualizarBotaoTimer(btn, proximaAbertura) {

    const agora = new Date().getTime();

    const tempoRestante = proximaAbertura - agora; // Em milissegundos



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





// --- ATUALIZAÇÃO NO 'DOMContentLoaded' ---



document.addEventListener('DOMContentLoaded', () => {

    carregarColecaoDoStorage();

    renderizarGaleriaPrincipal();



    const btnAbrir = document.getElementById('btn-abrir-pacote');

    const modalElement = document.getElementById('modal-cartas-reveladas');

    if (!btnAbrir || !modalElement) return;

    

    const meuModal = new bootstrap.Modal(modalElement);

    

    // Inicia o gerenciador do timer

    iniciarGerenciadorTimer(btnAbrir); // <--- JÁ ESTÁ AQUI



    btnAbrir.addEventListener('click', () => {

        btnAbrir.disabled = true;

        btnAbrir.textContent = "Sorteando...";



        fetch('/abrir_pacote', { method: 'POST' })

        .then(response => response.json())

        .then(data => {

            if (data.erro) throw new Error(data.erro);



            console.log("Cartas recebidas:", data.cartas);

            exibirCartasNoModal(data.cartas);

            

            // Salva o timestamp (1h = 3600 * 1000 ms)

            // (Para testar, mude para 10 segundos: 10 * 1000)

            const proximaAbertura = new Date().getTime() + (3600 * 1000); 

            localStorage.setItem(CHAVE_TIMER, proximaAbertura);

            

            // Inicia o timer novamente

            iniciarGerenciadorTimer(btnAbrir); // <--- ATUALIZA O TIMER

            

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

Abra seu pacote. O modal fechará e o botão iniciará uma contagem regressiva de 1 hora. Se você recarregar a página, a contagem continua!

Concluímos a Fase 1 do nosso projeto: uma aplicação web totalmente funcional, com lógica de jogo e persistência no lado do cliente.

Desafio do Capítulo 4: O que acontece se você abrir o jogo em outro navegador ou limpar o cache? Todo o seu progresso é perdido. No próximo capítulo, vamos resolver isso, dando ao usuário a opção de salvar seu progresso na "nuvem" (nosso banco de dados **MySQL**).



## 4.5 XSS: O Perigo de Dados Não Sanitizados

### O que é XSS (Cross-Site Scripting)?

**XSS (Cross-Site Scripting)** é uma das vulnerabilidades mais comuns e perigosas em aplicações web. Ela ocorre quando um atacante consegue injetar código malicioso (geralmente **JavaScript**) em páginas visualizadas por outros usuários.

O nome "Cross-Site" vem do fato de que o ataque permite que um site malicioso execute scripts no contexto de outro site confiável.

### Como XSS Funciona?

Imagine que sua aplicação permite que usuários salvem um "apelido" no **Local Storage** e depois exibe esse apelido na página:

```javascript
// Código VULNERÁVEL - NÃO USE!
let apelido = localStorage.getItem('apelido');
document.getElementById('saudacao').innerHTML = 'Olá, ' + apelido + '!';
```

Se um usuário malicioso salvar o seguinte "apelido":

```javascript
localStorage.setItem('apelido', '<img src=x onerror="alert(\'XSS Attack!\')">');
```

Quando a página carregar, o código malicioso será executado! O atacante poderia:

- Roubar cookies de sessão
- Redirecionar o usuário para sites maliciosos
- Modificar o conteúdo da página
- Capturar informações sensíveis (senhas, dados de cartão)

### Tipos de XSS

#### 1. XSS Armazenado (Stored XSS)

O código malicioso é armazenado no servidor (banco de dados) e executado toda vez que a página é carregada.

**Exemplo**: Um comentário em um blog contendo código malicioso.

#### 2. XSS Refletido (Reflected XSS)

O código malicioso vem de uma requisição (URL, formulário) e é imediatamente refletido na resposta.

**Exemplo**: Um parâmetro de busca que é exibido na página sem sanitização.

#### 3. XSS Baseado em DOM (DOM-based XSS)

O código malicioso manipula o DOM diretamente no navegador, sem passar pelo servidor. **Este é o tipo mais relevante para Local Storage!**

### XSS e Local Storage: Uma Combinação Perigosa

O **Local Storage** é especialmente vulnerável a XSS porque:

1. **Persistência**: Os dados permanecem mesmo após fechar o navegador
2. **Acesso Fácil**: Qualquer script na página pode ler/escrever no Local Storage
3. **Sem Proteção HTTP-Only**: Diferente de cookies, não há flag de proteção

### Como Proteger Contra XSS no Local Storage

#### 1. Nunca Use innerHTML com Dados Não Confiáveis

```javascript
// ❌ VULNERÁVEL
element.innerHTML = localStorage.getItem('dados');

// ✅ SEGURO
element.textContent = localStorage.getItem('dados');
```

A propriedade `textContent` trata tudo como texto puro, sem interpretar HTML/JavaScript.

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

### Implementando Proteção XSS no PokéBooster

Vamos proteger nossa aplicação contra XSS ao trabalhar com **Local Storage**:

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
        if (!Array.isArray(colecao)) {
            throw new Error('Coleção deve ser um array');
        }
        
        // Validar cada carta
        const colecaoValidada = colecao.map(carta => {
            if (!carta.id || !carta.nome) {
                throw new Error('Carta inválida');
            }
            
            return {
                id: parseInt(carta.id),
                nome: sanitizeHTML(carta.nome),
                imagem: sanitizeHTML(carta.imagem)
            };
        });
        
        localStorage.setItem('minhaColecao', JSON.stringify(colecaoValidada));
        return true;
    } catch (e) {
        console.error('Erro ao salvar coleção:', e);
        return false;
    }
}

// Função segura para carregar coleção
function carregarColecaoSegura() {
    try {
        const dados = localStorage.getItem('minhaColecao');
        
        if (!dados) {
            return [];
        }
        
        const colecao = JSON.parse(dados);
        
        // Validar estrutura
        if (!Array.isArray(colecao)) {
            throw new Error('Formato inválido');
        }
        
        // Validar cada item
        return colecao.filter(carta => {
            return carta.id && 
                   typeof carta.id === 'number' && 
                   carta.id >= 1 && 
                   carta.id <= 151 &&
                   carta.nome && 
                   typeof carta.nome === 'string';
        });
        
    } catch (e) {
        console.error('Erro ao carregar coleção:', e);
        // Limpar dados corrompidos
        localStorage.removeItem('minhaColecao');
        return [];
    }
}

// Exibir cartas de forma segura
function exibirCartas(cartas) {
    const container = document.getElementById('cartas-container');
    
    // Limpar container
    container.innerHTML = '';
    
    cartas.forEach(carta => {
        const div = document.createElement('div');
        div.className = 'col-md-3 mb-4';
        
        // Criar elementos de forma segura
        const card = document.createElement('div');
        card.className = 'card';
        
        const img = document.createElement('img');
        img.className = 'card-img-top';
        // Validar URL da imagem
        img.src = carta.imagem.startsWith('http') ? carta.imagem : '/static/placeholder.png';
        img.alt = sanitizeHTML(carta.nome);
        
        const cardBody = document.createElement('div');
        cardBody.className = 'card-body';
        
        const titulo = document.createElement('h5');
        titulo.className = 'card-title';
        // Usar textContent ao invés de innerHTML
        titulo.textContent = carta.nome;
        
        cardBody.appendChild(titulo);
        card.appendChild(img);
        card.appendChild(cardBody);
        div.appendChild(card);
        container.appendChild(div);
    });
}
```

### Content Security Policy (CSP)

Uma camada adicional de proteção é implementar **CSP** no servidor **Flask**:

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

### Checklist de Segurança para Local Storage

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

### Exercício Prático

Revise o código JavaScript da sua aplicação PokéBooster e:

1. Identifique todos os usos de `innerHTML`
2. Substitua por `textContent` ou criação de elementos DOM
3. Adicione validação ao carregar dados do Local Storage
4. Implemente a função `sanitizeHTML` e use-a em todos os inputs
5. Adicione CSP headers no Flask

### Pontos-Chave

- **XSS é uma ameaça real**: Pode comprometer completamente a segurança da aplicação
- **Local Storage é vulnerável**: Qualquer script pode acessá-lo
- **Nunca confie em dados armazenados**: Sempre valide e sanitize
- **Use textContent, não innerHTML**: Para exibir dados não confiáveis
- **Implemente múltiplas camadas de defesa**: Validação + Sanitização + CSP

A proteção contra XSS é fundamental para qualquer aplicação web moderna. No próximo capítulo, veremos como proteger dados no servidor contra outros tipos de ataques.




## 4.5 XSS: O Perigo de Dados Não Sanitizados

### O que é XSS (Cross-Site Scripting)?

**XSS (Cross-Site Scripting)** é uma das vulnerabilidades mais comuns e perigosas em aplicações web. Ela ocorre quando um atacante consegue injetar código malicioso (geralmente **JavaScript**) em páginas visualizadas por outros usuários.

O nome "Cross-Site" vem do fato de que o ataque permite que um site malicioso execute scripts no contexto de outro site confiável.

### Como XSS Funciona?

Imagine que sua aplicação permite que usuários salvem um "apelido" no **Local Storage** e depois exibe esse apelido na página:

```javascript
// Código VULNERÁVEL - NÃO USE!
let apelido = localStorage.getItem('apelido');
document.getElementById('saudacao').innerHTML = 'Olá, ' + apelido + '!';
```

Se um usuário malicioso salvar o seguinte "apelido":

```javascript
localStorage.setItem('apelido', '<img src=x onerror="alert(\'XSS Attack!\')">');
```

Quando a página carregar, o código malicioso será executado! O atacante poderia:

- Roubar cookies de sessão
- Redirecionar o usuário para sites maliciosos
- Modificar o conteúdo da página
- Capturar informações sensíveis (senhas, dados de cartão)

### Tipos de XSS

#### 1. XSS Armazenado (Stored XSS)

O código malicioso é armazenado no servidor (banco de dados) e executado toda vez que a página é carregada.

**Exemplo**: Um comentário em um blog contendo código malicioso.

#### 2. XSS Refletido (Reflected XSS)

O código malicioso vem de uma requisição (URL, formulário) e é imediatamente refletido na resposta.

**Exemplo**: Um parâmetro de busca que é exibido na página sem sanitização.

#### 3. XSS Baseado em DOM (DOM-based XSS)

O código malicioso manipula o DOM diretamente no navegador, sem passar pelo servidor. **Este é o tipo mais relevante para Local Storage!**

### XSS e Local Storage: Uma Combinação Perigosa

O **Local Storage** é especialmente vulnerável a XSS porque:

1. **Persistência**: Os dados permanecem mesmo após fechar o navegador
2. **Acesso Fácil**: Qualquer script na página pode ler/escrever no Local Storage
3. **Sem Proteção HTTP-Only**: Diferente de cookies, não há flag de proteção

### Como Proteger Contra XSS no Local Storage

#### 1. Nunca Use innerHTML com Dados Não Confiáveis

```javascript
// ❌ VULNERÁVEL
element.innerHTML = localStorage.getItem('dados');

// ✅ SEGURO
element.textContent = localStorage.getItem('dados');
```

A propriedade `textContent` trata tudo como texto puro, sem interpretar HTML/JavaScript.

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

### Implementando Proteção XSS no PokéBooster

Vamos proteger nossa aplicação contra XSS ao trabalhar com **Local Storage**:

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
        if (!Array.isArray(colecao)) {
            throw new Error('Coleção deve ser um array');
        }
        
        // Validar cada carta
        const colecaoValidada = colecao.map(carta => {
            if (!carta.id || !carta.nome) {
                throw new Error('Carta inválida');
            }
            
            return {
                id: parseInt(carta.id),
                nome: sanitizeHTML(carta.nome),
                imagem: sanitizeHTML(carta.imagem)
            };
        });
        
        localStorage.setItem('minhaColecao', JSON.stringify(colecaoValidada));
        return true;
    } catch (e) {
        console.error('Erro ao salvar coleção:', e);
        return false;
    }
}

// Função segura para carregar coleção
function carregarColecaoSegura() {
    try {
        const dados = localStorage.getItem('minhaColecao');
        
        if (!dados) {
            return [];
        }
        
        const colecao = JSON.parse(dados);
        
        // Validar estrutura
        if (!Array.isArray(colecao)) {
            throw new Error('Formato inválido');
        }
        
        // Validar cada item
        return colecao.filter(carta => {
            return carta.id && 
                   typeof carta.id === 'number' && 
                   carta.id >= 1 && 
                   carta.id <= 151 &&
                   carta.nome && 
                   typeof carta.nome === 'string';
        });
        
    } catch (e) {
        console.error('Erro ao carregar coleção:', e);
        // Limpar dados corrompidos
        localStorage.removeItem('minhaColecao');
        return [];
    }
}

// Exibir cartas de forma segura
function exibirCartas(cartas) {
    const container = document.getElementById('cartas-container');
    
    // Limpar container
    container.innerHTML = '';
    
    cartas.forEach(carta => {
        const div = document.createElement('div');
        div.className = 'col-md-3 mb-4';
        
        // Criar elementos de forma segura
        const card = document.createElement('div');
        card.className = 'card';
        
        const img = document.createElement('img');
        img.className = 'card-img-top';
        // Validar URL da imagem
        img.src = carta.imagem.startsWith('http') ? carta.imagem : '/static/placeholder.png';
        img.alt = sanitizeHTML(carta.nome);
        
        const cardBody = document.createElement('div');
        cardBody.className = 'card-body';
        
        const titulo = document.createElement('h5');
        titulo.className = 'card-title';
        // Usar textContent ao invés de innerHTML
        titulo.textContent = carta.nome;
        
        cardBody.appendChild(titulo);
        card.appendChild(img);
        card.appendChild(cardBody);
        div.appendChild(card);
        container.appendChild(div);
    });
}
```

### Content Security Policy (CSP)

Uma camada adicional de proteção é implementar **CSP** no servidor **Flask**:

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

### Checklist de Segurança para Local Storage

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

### Exercício Prático

Revise o código JavaScript da sua aplicação PokéBooster e:

1. Identifique todos os usos de `innerHTML`
2. Substitua por `textContent` ou criação de elementos DOM
3. Adicione validação ao carregar dados do Local Storage
4. Implemente a função `sanitizeHTML` e use-a em todos os inputs
5. Adicione CSP headers no Flask

### Pontos-Chave

- **XSS é uma ameaça real**: Pode comprometer completamente a segurança da aplicação
- **Local Storage é vulnerável**: Qualquer script pode acessá-lo
- **Nunca confie em dados armazenados**: Sempre valide e sanitize
- **Use textContent, não innerHTML**: Para exibir dados não confiáveis
- **Implemente múltiplas camadas de defesa**: Validação + Sanitização + CSP

A proteção contra XSS é fundamental para qualquer aplicação web moderna. No próximo capítulo, veremos como proteger dados no servidor contra outros tipos de ataques.



## 4.6 Verifique seu Conhecimento


Qual é a principal desvantagem da persistência de dados no Local Storage (Client-Side)? a) É muito lenta para ler e escrever. b) Os dados ficam presos àquele navegador/dispositivo e podem ser perdidos. c) Só funciona com o framework **Flask**. d) Ocupa muito espaço no servidor.

O Local Storage pode armazenar diretamente quais tipos de dados? a) Apenas Listas e Dicionários **JavaScript**. b) Apenas Números. c) Apenas Strings. d) Listas, Dicionários, Números e Strings.

Qual par de funções é usado para salvar um objeto **JavaScript** no Local Storage? a) JSON.parse() para salvar, JSON.stringify() para ler. b) localStorage.setObject() para salvar, localStorage.getObject() para ler. c) JSON.stringify() para salvar, JSON.parse() para ler. d) localStorage.save() para salvar, localStorage.load() para ler.

Por que refatoramos o app.py para carregar a galeria principal via **JavaScript** em vez de **Jinja2**? a) Porque **JavaScript** é mais rápido que **Jinja2**. b) Porque o **Jinja2** não sabe fazer loops {% for %}. c) Para permitir que o **JavaScript** verifique o Local Storage antes de exibir os cards, decidindo quais estão "Capturados". d) Para reduzir o uso de memória do servidor **Flask**.

Em static/app.js, qual é o propósito da função async/await em renderizarGaleriaPrincipal()? a) Ela permite que o fetch('/api/pokemon') aconteça em segundo plano, e o código "espera" (await) pela resposta antes de continuar. b) Ela criptografa os dados do Pokémon. c) Ela garante que o loop for rode mais rápido. d) async/await é uma sintaxe do **Python**, não do **JavaScript**.

O que a função minhaColecao.hasOwnProperty(poke.id) verifica? a) Se o Pokémon é uma propriedade do poke.id. b. Se a chave (ex: "25") existe dentro do nosso objeto minhaColecao. c) Se o Pokémon é raro. d) Se a PokeAPI possui aquele ID.

Qual é a principal falha de usar setTimeout(..., 3600000) para um timer de 1 hora? a) Ele não é preciso e pode errar por alguns minutos. b) Ele consome muita CPU do cliente. c) Ele é resetado e perdido se o usuário fechar a aba ou recarregar a página. d) setTimeout não existe em **JavaScript**.

Como nossa lógica de timer resolve o problema de recarregar a página? a) Ela salva o timestamp exato da "próxima abertura" no Local Storage e o verifica a cada segundo. b) Ela pede ao **Flask** para lembrar do timer. c) Ela impede o usuário de recarregar a página. d) Ela usa setTimeout e o impede de ser resetado.

Qual é o propósito do setInterval(funcao, 1000)? a) Executar a funcao uma única vez após 1000 segundos. b) Chamar o servidor 1000 vezes. c) Criar um timer de 1000 segundos. d) Executar a funcao repetidamente, uma vez a cada 1000 milissegundos (1 segundo).

No app.js, por que atualizamos a minhaColecao e chamamos salvarColecaoNoStorage()? a) minhaColecao é a variável em memória (rápida), salvarColecao... é a persistência no disco (lenta). b) É redundante, apenas uma é necessária. c. minhaColecao salva no servidor, salvarColecao... salva no cliente. d) minhaColecao é para o timer, salvarColecao... é para as cartas.


## 4.7 Exercícios Práticos


Botão de Reset: Adicione um botão "Resetar Jogo" em index.html. No app.js, adicione um "ouvinte" de clique para este novo botão. Ao ser clicado, ele deve: a) Perguntar ao usuário "Tem certeza?" (usando confirm('Tem certeza?')). b) Se o usuário confirmar, ele deve remover os itens do Local Storage (localStorage.removeItem(CHAVE_COLECAO) e CHAVE_TIMER). c) Recarregar a página (usando location.reload()) para que o jogo volte ao estado inicial.

Contador de Cartas: Em index.html, perto do título "Minha Coleção", mostre quantas cartas únicas o usuário já capturou.

Dica: Em app.js, dentro da renderizarGaleriaPrincipal(), você pode calcular o número de cartas em minhaColecao (usando Object.keys(minhaColecao).length) e inserir esse número em um <span> no **HTML**.

Ajuste Fino do Timer (Testes): No app.js, encontre a linha const proximaAbertura = ... + (3600 * 1000); e mude o tempo de 1 hora (3600 * 1000) para 10 segundos (10 * 1000). Teste o ciclo completo: abra um pacote, veja o timer de 10 segundos, espere ele acabar e veja o botão ser reabilitado. (Lembre-se de voltar para 1 hora depois).
