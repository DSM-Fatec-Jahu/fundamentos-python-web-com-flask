# Capítulo 2: Renderizando o Visual (Templates com Jinja2 e Bootstrap)


## Sumário do Capítulo

- [2.1 O que são Templates? A separação do HTML e do Python
](#o-que-são-templates-a-separação-do-html-e-do-python)
- [2.2 Lógica por trás do Jinja2: Injetando dados do Python no HTML
](#lógica-por-trás-do-jinja2-injetando-dados-do-python-no-html)
- [2.3 Integrando o Bootstrap 5.3: Criando o base.html
](#integrando-o-bootstrap-53-criando-o-basehtml)
- [2.4 Projeto (Parte 1): Exibindo a "Pokédex" (Galeria de Cartas)
](#projeto-parte-1-exibindo-a-pokédex-galeria-de-cartas)
- [2.5 Lógica por trás da API Pokémon (PokeAPI)
](#lógica-por-trás-da-api-pokémon-pokeapi)
- [2.6 Projeto (Parte 2): Carregando os 151 Pokémon via API
](#projeto-parte-2-carregando-os-151-pokémon-via-api)
- [2.7 Verifique seu Conhecimento
](#verifique-seu-conhecimento)
- [2.8 Exercícios Práticos
](#exercícios-práticos)

---

No capítulo anterior, criamos nosso primeiro servidor web. No entanto, nossa função index() apenas retornava uma string de texto simples: return 'Olá, Mestre Pokémon!'. Isso não é uma página web; é apenas texto.

Para construir uma aplicação web real, precisamos enviar ao cliente (navegador) uma estrutura de documento complexa, escrita em **HTML**, estilizada com **CSS** e com interatividade de **JavaScript**. Neste capítulo, vamos aprender a fazer exatamente isso, separando nossa lógica **Python** da nossa interface visual.


## 2.1 O que são Templates? A separação do HTML e do Python


Poderíamos escrever todo o nosso **HTML** dentro de uma string no app.py:

```python
@app.route('/')

def index():

    html_gigante = """

    <html>

      <head><title>Meu App</title></head>

      <body><h1>Olá, Mundo!</h1></body>

    </html>

    """

    return html_gigante

Imagine um site com centenas de linhas de **HTML**. Manter isso dentro de um arquivo **Python** seria um pesadelo. Isso viola um princípio fundamental da programação: Separação de Preocupações (Separation of Concerns). A lógica (**Python**) deve ser separada da apresentação (**HTML**).

Templates são a solução. São arquivos .html puros que mantemos em uma pasta separada. O **Flask** então "renderiza" esses arquivos, ou seja, ele os lê e os envia como resposta.

Por convenção, o **Flask** procura por templates em uma pasta chamada templates, no mesmo diretório do seu app.py.

Vamos refatorar nosso projeto:

Crie uma pasta chamada templates dentro do seu diretório poke_tcg.

Dentro de templates, crie um arquivo: index.html.

Mova seu **HTML** para templates/index.html:

<!DOCTYPE html>

<html lang="pt-br">

<head>

    <meta charset="UTF-8">

    <title>PokéBooster TCG</title>

</head>

<body>

    <h1>Bem-vindo ao Simulador PokéBooster TCG!</h1>

    <p>Aqui você poderá abrir seus pacotes.</p>

</body>

</html>

Modifique seu app.py para usar a função render_template:

# app.py (Forma CORRETA)

```

# 1. Importe 'render_template'

from flask import **Flask**, render_template



app = **Flask**(__name__)



@app.route('/')

def index():

    # 2. Em vez de retornar uma string, renderize o arquivo

    return render_template('index.html')



if __name__ == '__main__':

    app.run(debug=True)

Agora, se você recarregar seu servidor (http://127.0.0.1:5000/), verá sua página **HTML** completa!


## 2.2 Lógica por trás do Jinja2: Injetando dados do Python no HTML


Lógica por trás do **Jinja2**: Nossas páginas ainda são estáticas. O arquivo index.html sempre exibirá o mesmo texto. Mas uma aplicação web é dinâmica; ela precisa mostrar informações que mudam (como o nome de um usuário logado, ou uma lista de Pokémon).

Como "passar" uma variável do app.py para o index.html?

O **Flask** usa um "motor de templates" chamado **Jinja2**. Ele permite escrevermos "pseudo-códigos" **Python** diretamente no nosso **HTML**. O **Jinja2** lê o arquivo **HTML**, executa esses códigos, e os substitui pelo resultado final.

O **Jinja2** tem duas sintaxes principais:

{{ ... }}: Usado para exibir/imprimir o valor de uma variável.

{% ... %}: Usado para lógica, como if, else e for.

Vamos tornar nossa página dinâmica:

Modifique o app.py para passar variáveis para o render_template:

```python
@app.route('/')

def index():

    # Podemos passar quantas variáveis quisermos

    nome_treinador = "Ash Ketchum"

    cidade_natal = "Pallet"



    return render_template('index.html', 

                           treinador=nome_treinador, 

                           cidade=cidade_natal)

Modifique o templates/index.html para usar essas variáveis:

<!DOCTYPE html>

<html lang="pt-br">

<head>

    <meta charset="UTF-8">

    <title>PokéBooster TCG</title>

</head>

<body>

    <h1>Bem-vindo, {{ treinador }}!</h1>

    <p>Saudações da cidade de {{ cidade }}.</p>

</body>

</html>

Recarregue a página. A saída agora será: "Bem-vindo, Ash Ketchum!" e "Saudações da cidade de Pallet.". O **Jinja2** substituiu as variáveis com sucesso.

```


## 2.3 Integrando o Bootstrap 5.3: Criando o base.html


Nossa página funciona, mas é visualmente pobre. Poderíamos escrever centenas de linhas de **CSS**, ou podemos usar um Framework **CSS** como o **Bootstrap**. O **Bootstrap** 5.3 nos dá componentes prontos (botões, menus, grids) para que nosso site pareça profissional com pouco esforço.

Herança de Templates (**Template** Inheritance):

Toda página do nosso site precisará dos mesmos links do **Bootstrap** no <head> e no fim do <body>. Seria terrível copiar e colar isso em todos os arquivos **HTML**.

Lógica por trás do base.html: A lógica aqui é o princípio DRY (Don't Repeat Yourself). Criamos um arquivo-mestre, o base.html, que contém toda a estrutura repetitiva (links do **Bootstrap**, nosso menu de navegação, o rodapé).

Este arquivo-mestre terá "blocos" vazios que os arquivos-filhos (como index.html) irão preencher.

Crie o arquivo templates/base.html:

```html
<html lang="pt-br">

<head>

    <meta charset="UTF-8">

    <meta name="viewport" content="width=device-width, initial-scale=1.0">



    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">



    <title>{% block title %}PokéBooster{% endblock %}</title>

</head>

<body>

    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">

        <div class="container">

            <a class="navbar-brand" href="/">PokéBooster TCG</a>

        </div>

    </nav>



    <main class="container mt-4">

        {% block content %}{% endblock %}

    </main>



    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>

</body>

</html>

Modifique o templates/index.html para "herdar" de base.html:

{% extends 'base.html' %}

```

{% block title %}Página Inicial - PokéBooster{% endblock %}



{% block content %}

    <div class="p-5 mb-4 bg-light rounded-3">

        <div class="container-fluid py-5">

            <h1 class="display-5 fw-bold">Bem-vindo, {{ treinador }}!</h1>

            <p class="col-md-8 fs-4">Saudações da cidade de {{ cidade }}.</p>

            <p>Pronto para ver sua coleção e abrir novos pacotes?</p>

        </div>

    </div>

{% endblock %}

Recarregue a página. Agora você verá seu conteúdo, mas com o menu e o estilo do **Bootstrap** aplicados!


## 2.4 Projeto (Parte 1): Exibindo a "Pokédex" (Galeria de Cartas)


Vamos usar a lógica de loop {% for ... %} do **Jinja2** e o sistema de "Grid" e "Cards" do **Bootstrap** para exibir a galeria de Pokémon.

Em app.py, vamos criar uma lista temporária (dummy) de Pokémon para enviar ao template.

```python
@app.route('/')

def index():

    nome_treinador = "Ash Ketchum"



    # Lista de dicionários: nossos dados temporários

    pokemons = [

        {'id': 1, 'nome': 'Bulbasaur', 'capturado': True},

        {'id': 4, 'nome': 'Charmander', 'capturado': False},

        {'id': 7, 'nome': 'Squirtle', 'capturado': True},

        {'id': 25, 'nome': 'Pikachu', 'capturado': True}

    ]



    return render_template('index.html', 

                           treinador=nome_treinador,

                           lista_pokemon=pokemons) # Passa a lista para o template

Em templates/index.html, vamos iterar sobre essa lista e criar os cards.

{% extends 'base.html' %}

{% block title %}Minha Coleção{% endblock %}

```

{% block content %}

    <h2 class="mb-4">Minha Coleção ({{ treinador }})</h2>



    <div class="row">



        {% for poke in lista_pokemon %}

            <div class="col-md-3 mb-4">

                <div class="card text-center">

                    <img src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/{{ poke.id }}.png" 

                         class="card-img-top p-3" 

                         alt="{{ poke.nome }}">

                    <div class="card-body">

                        <h5 class="card-title">{{ poke.nome }}</h5>



                        {% if poke.capturado %}

                            <span class="badge bg-success">Capturado</span>

                        {% else %}

                            <span class="badge bg-secondary">Não Capturado</span>

                        {% endif %}

                    </div>

                </div>

            </div>

        {% endfor %}



    </div>

{% endblock %}

Recarregue. Você verá uma galeria com 4 cards de Pokémon, cada um exibindo a imagem, o nome e o status de captura!


## 2.5 Lógica por trás da API Pokémon (PokeAPI)


Nossa lista de Pokémon temporária foi útil, mas não é escalável. Não vamos digitar os 151 Pokémon manualmente.

Lógica por trás da **API**: Assim como nosso app **Flask** é um servidor que responde com **HTML**, existem servidores na web especializados em responder apenas com dados. Estes são chamados de APIs (Application Programming Interfaces).

Nós (o cliente, que neste caso é o nosso servidor **Flask**) faremos um request para a **API** (o servidor de dados) e ela nos enviará um response com os dados em formato JSON.

Usaremos a PokeAPI (pokeapi.co), uma **API** pública e gratuita. Para fazer requisições **HTTP**, usaremos a biblioteca requests, que vimos no curso anterior.

Primeiro, instale o requests no seu **venv**:

```bash
```


## 2.6 Projeto (Parte 2): Carregando os 151 Pokémon via API


Vamos substituir nossa lista temporária por dados reais da PokeAPI.

Modifique o app.py para chamar a **API**:

```python
from flask import **Flask**, render_template

import requests # Importa a biblioteca requests

```

app = **Flask**(__name__)



# URL da PokeAPI para pegar os 151 primeiros Pokémon

POKEAPI_URL = "https://pokeapi.co/api/v2/pokemon?limit=151"



def carregar_dados_pokemon():

    """Busca os dados dos 151 Pokémon na PokeAPI."""

    try:

        response = requests.get(POKEAPI_URL)

        response.raise_for_status() # Lança um erro se a requisição falhar



        data = response.json() # Converte a resposta JSON em um dicionário **Python**

        resultados = data['results'] # Pega a lista de Pokémon



        lista_pokemon = []

        # Enumera os resultados para obter um índice (começando em 1)

        for i, poke in enumerate(resultados):

            pokemon_id = i + 1

            lista_pokemon.append({

                'id': pokemon_id,

                'nome': poke['name'].title(), # 'bulbasaur' -> 'Bulbasaur'

                'capturado': False # Por padrão, o usuário não tem nenhum

            })

        return lista_pokemon



    except requests.RequestException as e:

        print(f"Erro ao buscar dados da PokeAPI: {e}")

        return [] # Retorna lista vazia em caso de erro



@app.route('/')

def index():

    nome_treinador = "Ash Ketchum"



    # Substitui a lista temporária pela chamada da função

    pokemons = carregar_dados_pokemon()



    return render_template('index.html', 

                           treinador=nome_treinador,

                           lista_pokemon=pokemons)



if __name__ == '__main__':

    app.run(debug=True)

Nenhuma alteração é necessária no templates/index.html!

Essa é a mágica da separação de preocupações. Nosso template apenas espera uma variável lista_pokemon que seja uma lista de dicionários. Ele não se importa se essa lista veio de dados temporários ou de uma **API**. Nós mudamos a "fonte de dados" em app.py e a "apresentação" (index.html) continuou funcionando perfeitamente.

Recarregue sua página. Você verá agora a galeria completa com os 151 Pokémon!


## 2.7 Verifique seu Conhecimento


Qual é a principal vantagem de usar "Templates" em vez de strings de **HTML** no **Python**? a) O código roda mais rápido. b) Separação de Preocupações (lógica vs. apresentação). c) Usa menos memória. d) O **Flask** exige isso para funcionar.

Em qual pasta o **Flask** procura por templates por padrão? a) static b) html c) templates d) views

Qual sintaxe do **Jinja2** é usada para exibir o valor de uma variável? a) {{ minha_variavel }} b) {% minha_variavel %} c) [[ minha_variavel ]] d) ( minha_variavel )

Qual comando do **Jinja2** é usado para herdar de um template mestre? a) {% include 'base.html' %} b) {% extends 'base.html' %} c) {% import 'base.html' %} d) {% parent 'base.html' %}

O que é o **Bootstrap**? a) Um framework **Python** para servidores web. b) Um banco de dados para salvar Pokémon. c) Um framework **CSS**/JS para criar interfaces visuais profissionais. d) Uma **API** para buscar dados.

Qual é o propósito da biblioteca requests? a) Fazer requisições **HTTP** (chamar APIs) de forma simples. b) Renderizar templates **HTML**. c) Conectar-se a bancos de dados **MySQL**. d) Gerenciar ambientes virtuais.

O que é JSON? a) Uma linguagem de programação concorrente do **Python**. b) Um formato leve de intercâmbio de dados, fácil de ler para humanos e máquinas. c) Um tipo de banco de dados usado pelo **Flask**. d) Um arquivo de template do **Jinja2**.

Qual sintaxe do **Jinja2** é usada para criar um loop for? a) {{ for poke in lista }} b) {% for poke in lista %} ... {% endfor %} c) (for poke in lista) ... (endfor) d) <for poke in lista> ... </for>

Qual é o propósito dos blocos {% block content %} ... {% endblock %} em base.html? a) Definir o único conteúdo que a página pode ter. b) Criar um bloco de comentário que o **Jinja2** ignora. c) Definir uma "janela" que os templates filhos podem preencher. d) Importar a biblioteca de content.

No app.py, o que a função render_template() faz? a) Lê um arquivo **HTML**, processa o **Jinja2** e retorna o **HTML** final como uma string. b) Apenas imprime texto no terminal do servidor. c) Conecta-se ao banco de dados. d) Cria uma nova pasta templates.


## 2.8 Exercícios Práticos


Embeleze o Card: Nossa galeria está funcional, mas podemos melhorá-la. Usando a documentação oficial do **Bootstrap** (getbootstrap.com), adicione as seguintes classes **CSS** aos seus cards em index.html:

Faça o card ter uma leve sombra (shadow-sm).

Faça o texto do nome do Pokémon ficar centralizado (text-center).

Faça a imagem ter uma leve "borda" cinza (border-bottom).

Página de Detalhes do Pokémon: Crie uma nova página que mostre detalhes de um Pokémon. Para isso, você precisará:

Em app.py: Criar uma nova rota, @app.route('/pokemon/<int:poke_id>'), que aceita um ID na URL.

Em app.py: Na função dessa nova rota, chame a PokeAPI para buscar dados de um único Pokémon (ex: https://pokeapi.co/api/v2/pokemon/{poke_id}).

Em app.py: Extraia dados como name, id, height, weight e a lista de types.

Em templates/: Criar um novo template, detalhe.html, que também deve herdar de base.html.

Em templates/detalhe.html: Exibir os dados detalhados do Pokémon (nome, imagem, altura, peso, tipos).

(Bônus): Em index.html, transforme cada card em um link clicável que leva para sua respectiva página de detalhes (ex: <a href="/pokemon/{{ poke.id }}">...</a>).
