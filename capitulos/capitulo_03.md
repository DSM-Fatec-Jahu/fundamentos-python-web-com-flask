# Capítulo 3: Abrindo Pacotes (Interação e Lógica do Jogo)


## Sumário do Capítulo

- [3.1 Lógica por trás do "Booster Pack": Simulando a aleatoriedade
](#lógica-por-trás-do-booster-pack-simulando-a-aleatoriedade)
- [3.2 Projeto (Parte 1): A Rota /abrir_pacote (API Interna)
](#projeto-parte-1-a-rota-abrir_pacote-api-interna)
- [3.3 Lógica por trás do JavaScript (Client-Side): O que o Flask não pode fazer
](#lógica-por-trás-do-javascript-client-side-o-que-o-flask-não-pode-fazer)
- [3.4 Introdução ao fetch(): Chamando nossa API Flask
](#introdução-ao-fetch-chamando-nossa-api-flask)
- [3.5 Projeto (Parte 2): Exibindo as Cartas Reveladas
](#projeto-parte-2-exibindo-as-cartas-reveladas)
- [3.6 Validação de Dados do Cliente
](#validação-de-dados-do-cliente)
- [3.7 Verifique seu Conhecimento
](#verifique-seu-conhecimento)
- [3.8 Exercícios Práticos
](#exercícios-práticos)

---

No Capítulo 2, demos vida à nossa aplicação. Criamos a interface visual com **Bootstrap**, a estrutura com base.html e carregamos dinamicamente os 151 Pokémon da PokeAPI usando **Jinja2**. Nossa página exibe uma "Pokédex" completa, mas ainda é estática — o usuário não pode fazer nada.

Neste capítulo, vamos implementar a funcionalidade principal do nosso projeto: a interação. Vamos adicionar o botão "Abrir Pacote", criar a lógica no servidor para sortear 4 cartas e, o mais importante, aprender como o navegador (**Cliente**) pode se comunicar com nosso servidor **Flask** (**Servidor**) sem recarregar a página.


## 3.1 Lógica por trás do "Booster Pack": Simulando a aleatoriedade


Lógica por trás da Simulação: Como um pacote de TCG funciona no mundo real? Você compra um pacote fechado e não sabe o que vem dentro. As cartas são aleatórias. Precisamos replicar essa lógica em nosso código.

Como vimos no Capítulo 6 de "Fundamentos de **Python** 1", o módulo embutido random do **Python** é a ferramenta perfeita para isso.

Temos uma lista com os 151 Pokémon carregados da **API**. Nosso objetivo é sortear 4 itens únicos dessa lista. Se usássemos random.choice() quatro vezes, poderíamos sortear o mesmo Pokémon duas vezes. A função ideal para este caso é random.sample().

random.sample(populacao, k): Retorna uma lista de k elementos únicos escolhidos da populacao (no nosso caso, a lista de 151 Pokémon).


## 3.2 Projeto (Parte 1): A Rota /abrir_pacote (API Interna)


Nossa primeira tarefa é ensinar ao nosso servidor **Flask** como "sortear 4 cartas" e entregar o resultado para quem pedir.

Até agora, nossa única rota (/) retorna uma página **HTML** renderizada. Agora, vamos criar uma rota diferente. Esta rota não retornará **HTML**. Ela será uma **API** interna, e sua única responsabilidade é processar a lógica do sorteio e retornar os dados brutos das cartas sorteadas em formato JSON.

Abra o app.py e adicione as importações de random e jsonify. jsonify é uma função do **Flask** que converte um dicionário **Python** em uma resposta JSON formatada corretamente para o navegador.

```python
from flask import **Flask**, render_template, jsonify # <-- Adicione jsonify

import requests

import random # <-- Adicione random

```

# ... (código do app, POKEAPI_URL, carregar_dados_pokemon) ...

Mova o carregamento dos Pokémon para o escopo global. No Capítulo 2, a função carregar_dados_pokemon() era chamada toda vez que o usuário visitava a index. Isso é lento. Como os 151 Pokémon não mudam, podemos carregar essa lista uma única vez quando o servidor inicia e armazená-la em uma variável global.

```python
```

# ... (importações) ...



app = **Flask**(__name__)

POKEAPI_URL = "https://pokeapi.co/api/v2/pokemon?limit=151"



def carregar_dados_pokemon():

    # ... (código da função exatamente como antes) ...

    # ... (retorna lista_pokemon) ...



# --- NOVO ---

# Carrega a lista UMA VEZ quando o app inicia

LISTA_GLOBAL_POKEMON = carregar_dados_pokemon()

# -----------



@app.route('/')

def index():

    nome_treinador = "Ash Ketchum"

    # Agora usamos a lista global, muito mais rápido!

    return render_template('index.html', 

                           treinador=nome_treinador,

                           lista_pokemon=LISTA_GLOBAL_POKEMON)



# ... (app.run) ...

(Obs: Isso é uma refatoração de performance. O app agora inicia um pouco mais devagar, mas a página index carrega instantaneamente.)

Crie a nova rota da **API**:

```python
```

# ... (código anterior) ...



# --- NOVA ROTA DE **API** ---

@app.route('/abrir_pacote', methods=['POST'])

def abrir_pacote():

    """Sorteia 4 cartas da lista global e as retorna como JSON."""

    try:

        if not LISTA_GLOBAL_POKEMON:

            # Caso a **API** tenha falhado na inicialização

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

Análise da Rota:

@app.route('/abrir_pacote', methods=['POST']): Definimos a URL e especificamos que ela SÓ aceita requisições do tipo POST. GET é para obter dados (como carregar uma página), POST é para enviar dados ou realizar uma ação. Abrir um pacote é uma ação.

random.sample(LISTA_GLOBAL_POKEMON, 4): O coração da nossa lógica.

jsonify({'cartas': cartas_sorteadas}): Nossa resposta. Não é **HTML**. É um objeto JSON.


## 3.3 Lógica por trás do JavaScript (Client-Side): O que o Flask não pode fazer


Lógica por trás da Interação Assíncrona (AJAX): Temos um problema fundamental. O **Python**/**Flask** vive no **Servidor**. O seu botão **HTML** vive no **Cliente** (navegador). Eles são computadores diferentes, possivelmente a milhares de quilômetros de distância.

Quando o usuário clica no botão "Abrir Pacote" no navegador, como podemos executar a função abrir_pacote() que está no app.py?

A Forma Antiga (Recarregando a Página): Poderíamos envolver o botão em um <form action="/abrir_pacote" method="POST">. Quando clicado, o navegador descartaria a página atual e pediria a URL /abrir_pacote. O **Flask** executaria e... retornaria um JSON. O usuário veria apenas um texto JSON feio na tela. Nós teríamos que fazer o **Flask** retornar uma nova página **HTML** inteira. Isso é lento e uma péssima experiência.

A Forma Moderna (AJAX): Queremos que a página permaneça onde está. Quando o usuário clica, queremos que o navegador, silenciosamente e em segundo plano, envie a requisição POST para o servidor. Isso é chamado de AJAX (Asynchronous **JavaScript** and XML).

A única linguagem que roda dentro do navegador do cliente é o **JavaScript**. Nossa arquitetura será:

**Cliente** (**HTML**): O usuário clica em <button id="btn-abrir">.

**Cliente** (**JavaScript**): Nosso script JS "ouve" esse clique.

**Cliente** (JS - AJAX): O JS envia a requisição POST /abrir_pacote para o servidor (**Flask**).

**Servidor** (**Flask**): A rota /abrir_pacote executa, sorteia as 4 cartas e retorna o JSON.

**Cliente** (JS): O JS recebe o JSON (as 4 cartas) em segundo plano.

**Cliente** (JS - DOM): O JS usa esses dados para manipular o **HTML** da página (sem recarregar!) e exibir as cartas em um pop-up (Modal).


## 3.4 Introdução ao fetch(): Chamando nossa API Flask


Para implementar o AJAX (Passo 3), usamos a função fetch() do **JavaScript**. fetch() é a ferramenta moderna para fazer requisições de rede.

Crie a pasta static: O **Flask** procura por arquivos estáticos (**CSS**, JS, Imagens) em uma pasta chamada static. Crie-a no mesmo nível de templates.

Crie o arquivo JS: Dentro de static, crie app.js.

Linke o arquivo JS no base.html: Precisamos que todas as nossas páginas carreguem esse script. Abra templates/base.html e adicione o <script> no final do <body>, logo após o script do **Bootstrap**.

```html


    <script src="{{ url_for('static', filename='app.js') }}"></script>

</body>

</html>

(Nota: url_for('static', ...) é a forma correta do **Jinja2** de gerar o caminho para arquivos estáticos.)

Adicione o botão em index.html: Coloque um botão e um lugar para exibir as cartas (usaremos um Modal do **Bootstrap**).

{% extends 'base.html' %}

{% block title %}Minha Coleção{% endblock %}

```

{% block content %}

    <div class="d-flex justify-content-between align-items-center mb-4">

        <h2>Minha Coleção ({{ treinador }})</h2>

        <button id="btn-abrir-pacote" class="btn btn-primary btn-lg">Abrir Pacote!</button>

    </div>



    <div class="modal fade" id="modal-cartas-reveladas" tabindex="-1" aria-hidden="true">

        <div class="modal-dialog modal-lg modal-dialog-centered">

            <div class="modal-content">

                <div class="modal-header">

                    <h5 class="modal-title">Cartas Reveladas!</h5>

                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>

                </div>

                <div class="modal-body" id="corpo-modal-cartas">

                    </div>

            </div>

        </div>

    </div>

{% endblock %}

Escreva o **JavaScript** em static/app.js:

```javascript
```

// Espera o **HTML** ser totalmente carregado antes de rodar o script

document.addEventListener('DOMContentLoaded', () => {



    // 1. Encontra o botão e o modal na página

    const btnAbrir = document.getElementById('btn-abrir-pacote');

    const modalElement = document.getElementById('modal-cartas-reveladas');



    // Se o botão não existir nesta página, não faz nada

    if (!btnAbrir || !modalElement) {

        return;

    }



    // Cria uma instância do Modal do **Bootstrap**

    const meuModal = new bootstrap.Modal(modalElement);



    // 2. Adiciona um "ouvinte" de clique no botão

    btnAbrir.addEventListener('click', () => {

        console.log("Botão clicado! Pedindo pacote ao servidor...");



        // Desabilita o botão para evitar cliques duplos

        btnAbrir.disabled = true;

        btnAbrir.textContent = "Sorteando...";



        // 3. Chama a função fetch()

        fetch('/abrir_pacote', {

            method: 'POST' // Especifica o método POST

        })

        .then(response => {

            if (!response.ok) {

                throw new Error('Falha na rede ou erro do servidor.');

            }

            return response.json(); // Converte a resposta do **Flask** em JSON

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

    // ... (lógica para criar o **HTML** das cartas) ...

}


## 3.5 Projeto (Parte 2): Exibindo as Cartas Reveladas


O último passo é implementar a função exibirCartasNoModal(). Ela receberá a lista de 4 cartas (JSON) e deverá construir o **HTML** dinamicamente.

Adicione a função em static/app.js:

```javascript
```

// ... (código do addEventListener) ...



function exibirCartasNoModal(cartas) {

    const corpoModal = document.getElementById('corpo-modal-cartas');



    // 1. Limpa o modal de cartas anteriores

    corpoModal.innerHTML = '';



    // 2. Cria a <div class="row"> do **Bootstrap**

    const row = document.createElement('div');

    row.className = 'row';



    // 3. Faz um loop nas 4 cartas recebidas

    cartas.forEach(carta => {

        // 4. Cria o **HTML** do Card para cada carta

        // (Usamos o mesmo padrão de classes do **Bootstrap** do index.html)



        const col = document.createElement('div');

        col.className = 'col-md-6 mb-3'; // 2 colunas por linha no modal



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



        // 5. Adiciona o **HTML** na coluna e a coluna na linha

        col.innerHTML = cardHTML;

        row.appendChild(col);

    });



    // 6. Adiciona a linha (com todas as 4 cartas) ao corpo do modal

    corpoModal.appendChild(row);

}

Rode seu app.py, recarregue a página e clique no botão "Abrir Pacote!". Você verá o botão ser desabilitado, a mensagem "Sorteando..." aparecer, e, em seguida, o modal surgirá com 4 cartas aleatórias!



## 3.6 Validação de Dados do Cliente

### Por que Validar Dados do Cliente?

Quando desenvolvemos aplicações web, é fundamental entender que **nunca devemos confiar cegamente nos dados enviados pelo cliente**. O navegador do usuário está completamente fora do nosso controle, e qualquer pessoa com conhecimentos básicos de desenvolvimento web pode:

- Modificar requisições HTTP usando ferramentas de desenvolvedor
- Enviar dados maliciosos através de scripts automatizados
- Manipular parâmetros de URL e formulários
- Injetar código malicioso em campos de entrada

### Validação no Servidor é Obrigatória

Embora possamos (e devamos) implementar validações no lado do cliente usando **JavaScript** para melhorar a experiência do usuário, essas validações são facilmente contornáveis. A **validação no servidor** é a única linha de defesa confiável.

### Exemplo Prático: Validando a Rota /abrir_pacote

Em nossa aplicação PokéBooster, a rota `/abrir_pacote` sorteia 4 cartas aleatórias. Mas e se um usuário malicioso tentar manipular essa requisição para obter vantagens? Vamos implementar validações adequadas:

```python
from flask import Flask, jsonify, request
import random

@app.route('/abrir_pacote', methods=['GET'])
def abrir_pacote():
    # 1. Verificar se o método HTTP é o esperado
    if request.method != 'GET':
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
        cartas_sorteadas = random.sample(lista_pokemon, 4)
        return jsonify({'cartas': cartas_sorteadas})
    except Exception as e:
        # Nunca expor detalhes do erro para o cliente
        return jsonify({'erro': 'Erro ao processar requisição'}), 500
```

### Princípios de Validação Segura

1. **Validação de Tipo**: Sempre verifique se os dados recebidos são do tipo esperado (string, número, etc.)

2. **Validação de Intervalo**: Para números, verifique se estão dentro de limites aceitáveis

3. **Sanitização de Entrada**: Remova ou escape caracteres especiais que possam ser usados em ataques

4. **Lista Branca vs Lista Negra**: Prefira validar contra uma lista de valores permitidos (whitelist) ao invés de bloquear valores específicos (blacklist)

5. **Mensagens de Erro Genéricas**: Não exponha detalhes internos do sistema nas mensagens de erro

### Validação de Dados JSON

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

### Proteção Contra Ataques de Negação de Serviço (DoS)

Validar também significa proteger contra requisições que possam sobrecarregar o servidor:

```python
# Limitar tamanho de requisições
app.config['MAX_CONTENT_LENGTH'] = 16 * 1024 * 1024  # 16 MB máximo

@app.route('/upload', methods=['POST'])
def upload():
    # Validar quantidade de itens em listas
    dados = request.get_json()
    if len(dados.get('cartas', [])) > 1000:
        return jsonify({'erro': 'Quantidade de cartas excede o limite'}), 400
    
    # Validar comprimento de strings
    if len(dados.get('nome', '')) > 100:
        return jsonify({'erro': 'Nome muito longo'}), 400
    
    return jsonify({'sucesso': True})
```

### Exercício Prático

Adicione validação à rota `/abrir_pacote` para:

1. Verificar se o usuário não está fazendo requisições muito frequentes (mais de 1 por hora)
2. Validar que a lista de Pokémon não está vazia antes de sortear
3. Retornar mensagens de erro apropriadas com códigos HTTP corretos

### Pontos-Chave

- **Nunca confie nos dados do cliente**: Sempre valide no servidor
- **Valide tipo, formato e intervalo**: Não assuma que os dados estão corretos
- **Use códigos HTTP apropriados**: 400 para erros do cliente, 500 para erros do servidor
- **Mensagens de erro genéricas**: Não exponha detalhes internos do sistema
- **Proteja contra DoS**: Limite tamanho e frequência de requisições

A validação adequada de dados é a primeira linha de defesa contra muitos tipos de ataques web. No próximo capítulo, veremos como proteger dados armazenados no navegador contra outros tipos de vulnerabilidades.




## 3.6 Validação de Dados do Cliente

### Por que Validar Dados do Cliente?

Quando desenvolvemos aplicações web, é fundamental entender que **nunca devemos confiar cegamente nos dados enviados pelo cliente**. O navegador do usuário está completamente fora do nosso controle, e qualquer pessoa com conhecimentos básicos de desenvolvimento web pode:

- Modificar requisições HTTP usando ferramentas de desenvolvedor
- Enviar dados maliciosos através de scripts automatizados
- Manipular parâmetros de URL e formulários
- Injetar código malicioso em campos de entrada

### Validação no Servidor é Obrigatória

Embora possamos (e devamos) implementar validações no lado do cliente usando **JavaScript** para melhorar a experiência do usuário, essas validações são facilmente contornáveis. A **validação no servidor** é a única linha de defesa confiável.

### Exemplo Prático: Validando a Rota /abrir_pacote

Em nossa aplicação PokéBooster, a rota `/abrir_pacote` sorteia 4 cartas aleatórias. Mas e se um usuário malicioso tentar manipular essa requisição para obter vantagens? Vamos implementar validações adequadas:

```python
from flask import Flask, jsonify, request
import random

@app.route('/abrir_pacote', methods=['GET'])
def abrir_pacote():
    # 1. Verificar se o método HTTP é o esperado
    if request.method != 'GET':
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
        cartas_sorteadas = random.sample(lista_pokemon, 4)
        return jsonify({'cartas': cartas_sorteadas})
    except Exception as e:
        # Nunca expor detalhes do erro para o cliente
        return jsonify({'erro': 'Erro ao processar requisição'}), 500
```

### Princípios de Validação Segura

1. **Validação de Tipo**: Sempre verifique se os dados recebidos são do tipo esperado (string, número, etc.)

2. **Validação de Intervalo**: Para números, verifique se estão dentro de limites aceitáveis

3. **Sanitização de Entrada**: Remova ou escape caracteres especiais que possam ser usados em ataques

4. **Lista Branca vs Lista Negra**: Prefira validar contra uma lista de valores permitidos (whitelist) ao invés de bloquear valores específicos (blacklist)

5. **Mensagens de Erro Genéricas**: Não exponha detalhes internos do sistema nas mensagens de erro

### Validação de Dados JSON

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

### Proteção Contra Ataques de Negação de Serviço (DoS)

Validar também significa proteger contra requisições que possam sobrecarregar o servidor:

```python
# Limitar tamanho de requisições
app.config['MAX_CONTENT_LENGTH'] = 16 * 1024 * 1024  # 16 MB máximo

@app.route('/upload', methods=['POST'])
def upload():
    # Validar quantidade de itens em listas
    dados = request.get_json()
    if len(dados.get('cartas', [])) > 1000:
        return jsonify({'erro': 'Quantidade de cartas excede o limite'}), 400
    
    # Validar comprimento de strings
    if len(dados.get('nome', '')) > 100:
        return jsonify({'erro': 'Nome muito longo'}), 400
    
    return jsonify({'sucesso': True})
```

### Exercício Prático

Adicione validação à rota `/abrir_pacote` para:

1. Verificar se o usuário não está fazendo requisições muito frequentes (mais de 1 por hora)
2. Validar que a lista de Pokémon não está vazia antes de sortear
3. Retornar mensagens de erro apropriadas com códigos HTTP corretos

### Pontos-Chave

- **Nunca confie nos dados do cliente**: Sempre valide no servidor
- **Valide tipo, formato e intervalo**: Não assuma que os dados estão corretos
- **Use códigos HTTP apropriados**: 400 para erros do cliente, 500 para erros do servidor
- **Mensagens de erro genéricas**: Não exponha detalhes internos do sistema
- **Proteja contra DoS**: Limite tamanho e frequência de requisições

A validação adequada de dados é a primeira linha de defesa contra muitos tipos de ataques web. No próximo capítulo, veremos como proteger dados armazenados no navegador contra outros tipos de vulnerabilidades.



## 3.7 Verifique seu Conhecimento


Qual função do **Flask** é usada para converter um dicionário **Python** em uma resposta JSON? a) render_template() b) json.dump() c) jsonify() d) return_json()

Qual módulo **Python** é usado para sortear itens únicos de uma lista? a) math.random b) random.sample() c) random.choice() d) 

list.sort(random=True)

Por que usamos **JavaScript** para lidar com o clique do botão? a) Porque **Python** não sabe lidar com cliques. b) Para permitir que o **Cliente** (navegador) se comunique com o **Servidor** (**Flask**) sem recarregar a página. c) Porque **JavaScript** é mais rápido que **Python**. d) Para estilizar o botão com cores.

O que é AJAX? a) Um framework **CSS** concorrente do **Bootstrap**. b) Uma técnica que usa **JavaScript** para enviar e receber dados do servidor em segundo plano. c) Um tipo de banco de dados do **Flask**. d) O nome da **API** do Pokémon.

Qual método **HTTP** é semanticamente correto para uma rota que realiza uma ação (como /abrir_pacote)? a) GET b) POST c) UPDATE d) DELETE

Qual é a função nativa do **JavaScript** usada para fazer requisições de rede (AJAX)? a) requests.get() b) ajax.call() c) fetch() d) http.send()

No **JavaScript**, o que o .then(response => response.json()) faz? a) Converte um objeto JSON do **JavaScript** em uma string. b) Pede ao **Flask** para enviar uma resposta em JSON. c) Pega a resposta bruta do servidor e a converte em um objeto JSON utilizável pelo **JavaScript**. d) Cria um arquivo chamado response.json.

Em qual pasta o **Flask** espera encontrar arquivos estáticos como app.js? a) templates b) static c) js d) No mesmo local do app.py.

Qual método do **JavaScript** usamos para alterar o **HTML** dentro de um <div>? a) element.innerHTML = "..." b. element.setHTML("...") c) element.text = "..." d) element.jinja = "..."

Qual a diferença entre random.sample(lista, 4) e random.choice(lista) chamado 4 vezes? a) Nenhuma, ambos fazem a mesma coisa. b) sample garante 4 itens únicos; choice pode repetir o mesmo item. c) sample é para números; choice é para strings. d) sample é mais lento que choice.


## 3.8 Exercícios Práticos


Melhorar Feedback Visual: No static/app.js, dentro da função addEventListener, o texto do botão muda para "Sorteando...". Use o **Bootstrap** para adicionar um spinner (ícone de carregamento) ao botão.

Dica: O **HTML** para um spinner é <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>. Você pode adicionar isso ao innerHTML do botão.

Exibir ID da Carta: Modifique o cardHTML dentro do app.js para que, abaixo do nome do Pokémon, ele exiba o ID da carta (ex: "ID: #25"). A variável carta.id já está disponível no loop.

Refatorar o JS: O código que gera o **HTML** do card em app.js é um bloco de string grande. Crie uma nova função **JavaScript** separada, chamada criarCardHTML(carta), que recebe um objeto carta e retorna a string **HTML** do card. Em seguida, chame essa função dentro do loop forEach. Isso torna o código mais limpo (princípio da modularização, que vimos no Capítulo 5 de "Fundamentos de **Python** 1" ).