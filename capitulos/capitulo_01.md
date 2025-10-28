# Capítulo 1: Preparando o Terreno: O Projeto e o Flask


## Sumário do Capítulo

- [1.1 Do Console para a Web: O Próximo Passo
](#do-console-para-a-web-o-próximo-passo)
- [1.2 Lógica por trás do Flask: O que é um Micro-Framework?
](#lógica-por-trás-do-flask-o-que-é-um-micro-framework)
- [1.3 Nosso Projeto: O "PokéBooster" TCG Simulator
](#nosso-projeto-o-pokébooster-tcg-simulator)
- [1.4 Lógica por trás dos Ambientes Virtuais (venv)
](#lógica-por-trás-dos-ambientes-virtuais-venv)
- [1.5 Configurando o Ambiente do Mestre Pokémon
](#configurando-o-ambiente-do-mestre-pokémon)
- [1.6 O "Olá, Mundo!" do Flask: Nosso primeiro App
](#o-olá-mundo-do-flask-nosso-primeiro-app)
- [1.7 Verifique seu Conhecimento
](#verifique-seu-conhecimento)
- [1.8 Exercícios Práticos
](#exercícios-práticos)

---

Seja bem-vindo de volta, desenvolvedor! No módulo anterior, "Fundamentos de **Python** 1", você dominou a lógica de programação, desde variáveis e loops até funções e manipulação de arquivos. Você aprendeu a criar programas que rodam no seu terminal.

Agora, vamos dar o próximo passo: levar seus programas para a web. Neste curso, construiremos um projeto completo, do zero, aplicando seus conhecimentos de **Python** em um contexto de desenvolvimento web usando o framework **Flask** e o framework visual **Bootstrap** 5.3.


## 1.1 Do Console para a Web: O Próximo Passo


Até agora, seus programas usavam input() para receber dados e print() para mostrar resultados. Na web, a arquitetura é diferente. Ela se baseia no modelo **Cliente**-**Servidor**:

**Cliente** (Client): É o navegador do usuário (Chrome, Firefox, etc.). Ele é responsável por pedir páginas (fazer requests) e exibir o que recebe (renderizar **HTML**, **CSS**, JS).

**Servidor** (Server): É o seu programa **Python** rodando com **Flask**. Ele fica escutando por pedidos (requests), processa a lógica necessária (consultar um banco de dados, calcular algo) e envia uma resposta (response) de volta para o cliente, geralmente na forma de uma página **HTML**.


## 1.2 Lógica por trás do Flask: O que é um Micro-Framework?


Você poderia usar **Python** para escutar pedidos de rede e construir respostas **HTML** manualmente, mas isso seria extremamente complexo e repetitivo. Um framework web cuida de todo o "trabalho sujo" para você.

O **Flask** é chamado de micro-framework não por ser limitado, mas por ser intencionalmente pequeno e extensível. Ele fornece o essencial:

Roteamento: Associa uma URL (como /contatos) a uma função **Python** específica.

Motor de Templates (**Jinja2**): Permite injetar dados do **Python** dentro de arquivos **HTML**.

**Servidor** de Desenvolvimento: Um servidor embutido para testar seu aplicativo localmente.

A lógica do **Flask** é: "Eu cuido da parte web (**HTTP**, rotas), você cuida da lógica da sua aplicação". Isso o torna perfeito para aprender e para construir projetos de qualquer tamanho, adicionando extensões (como banco de dados) conforme a necessidade.


## 1.3 Nosso Projeto: O "PokéBooster" TCG Simulator


Para aplicar esses conceitos, construiremos um aplicativo web divertido: um simulador de abertura de pacotes de TCG (Trading Card Game) dos 151 Pokémon iniciais.

Requisitos do Projeto:

Pokédex Visual: O usuário deve ver uma galeria com todas as 151 cartas. Cartas que ele possui devem parecer diferentes das que ele não possui.

Abrir Pacote: O usuário deve ter um botão para "Abrir Pacote". Ao clicar, o servidor deve sortear 4 cartas aleatórias e revelá-las.

Timer: O usuário só pode abrir um novo pacote a cada 1 hora.

Persistência (Nível 1): A coleção de cartas do usuário e o timer devem ser salvos no Local Storage do navegador, para que ele não perca seu progresso ao fechar a aba.

Persistência (Nível 2 - Opcional): O usuário deve ter a opção de "Salvar na Nuvem", migrando sua coleção do Local Storage para um banco de dados **MySQL** no servidor.


## 1.4 Lógica por trás dos Ambientes Virtuais (venv)


Como vimos no Capítulo 6 de "Fundamentos de **Python** 1", ambientes virtuais são essenciais. No desenvolvimento web, isso é ainda mais crítico.

A lógica é o isolamento. Seu projeto **Flask** dependerá de bibliotecas (**Flask**, **Flask**-**SQLAlchemy**, mysqlclient). Outro projeto seu (talvez com Django) usará outras versões dessas bibliotecas. Se você instalar tudo globalmente, um projeto quebrará o outro.

O **venv** (Ambiente Virtual) cria uma "bolha" para seu projeto, com sua própria instalação do **Python** e suas próprias bibliotecas.

Regra de Ouro: Sempre crie um **venv** antes de instalar qualquer pacote para um novo projeto.


## 1.5 Configurando o Ambiente do Mestre Pokémon


Vamos criar a estrutura do nosso projeto. Abra seu terminal.

Crie a pasta do projeto:

```bash
cd poke_tcg

Crie o ambiente virtual:

python -m **venv** **venv**

Saída: (Uma nova pasta chamada **venv** será criada)

Ative o ambiente virtual:

No Windows (CMD):

**venv**\Scripts\activate

No Linux/macOS:

source **venv**/bin/activate

Saída: Seu prompt do terminal agora deve ter (**venv**) no início.

Instale o **Flask**: Com o **venv** ativo, instale o **Flask**:

pip install flask

Saída: (O pip fará o download e instalação do **Flask** e suas dependências).

```


## 1.6 O "Olá, Mundo!" do Flask: Nosso primeiro App


Agora que o ambiente está pronto, vamos criar nosso servidor.

Crie um arquivo chamado app.py na pasta poke_tcg.

Adicione o seguinte código:

```python
from flask import **Flask**

```

# 2. Cria uma instância do aplicativo **Flask**

# __name__ informa ao **Flask** onde procurar por arquivos (como templates)

app = **Flask**(__name__)



# 3. Define uma rota (Roteamento)

# @app.route('/') diz ao **Flask**: "Quando alguém visitar a URL raiz ('/')...

@app.route('/')

def index():

    # 4. ...execute esta função e retorne a resposta."

    return 'Olá, Mestre Pokémon! Bem-vindo ao PokéBooster TCG!'



# 5. Garante que o servidor só rode quando executarmos o script diretamente

if __name__ == '__main__':

    # app.run() inicia o servidor de desenvolvimento

    # debug=True reinicia o servidor automaticamente quando salvamos o arquivo

    app.run(debug=True)

Executando o **Servidor**:

No seu terminal (com o **venv** ainda ativo), execute o arquivo:

```bash
Saída:

 * Serving **Flask** app 'app' (lazy loading)

 * Environment: development

 * Debug mode: on

 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

 * Restarting with stat

Abra seu navegador e acesse a URL http://127.0.0.1:5000/. Você deverá ver a mensagem: "Olá, Mestre Pokémon! Bem-vindo ao PokéBooster TCG!".

Parabéns! Você acabou de criar seu primeiro servidor web.

```


## 1.7 Verifique seu Conhecimento


Qual é a função do "**Cliente**" no modelo **Cliente**-**Servidor**? a) Executar a lógica **Python** e acessar o banco de dados. b) Fazer pedidos (requests) e renderizar o **HTML** recebido. c) Armazenar as senhas de todos os usuários. d) Enviar e-mails.

O que o **Flask** não fornece em sua instalação básica (como micro-framework)? a) Roteamento de URLs. b) Um servidor de desenvolvimento. c) Um sistema completo de autenticação de usuários e painel de admin. d) Um motor de templates (**Jinja2**).

Qual comando instala o **Flask** em um ambiente virtual ativo? a) python install flask b) pip install flask c) **venv** install flask d) flask.run()

No **Flask**, o que o decorador @app.route('/perfil') faz? a) Cria um arquivo **HTML** chamado perfil.html. b) Imprime a palavra "perfil" no terminal. c) Associa a URL /perfil à função definida logo abaixo dele. d) Importa a biblioteca de perfis de usuário.

Por que a opção debug=True é útil em app.run()? a) Para tornar o site mais rápido para o usuário final. b) Para reiniciar automaticamente o servidor sempre que o código é salvo. c) Para criptografar os dados do usuário. d) Para instalar novas bibliotecas automaticamente.


## 1.8 Exercícios Práticos


Nova Rota: Adicione uma nova rota ao app.py. Se o usuário visitar http://127.0.0.1:5000/pokemon-favorito, a página deve exibir o texto "Meu Pokémon favorito é o [Seu Pokémon Favorito]".

Rota Dinâmica: (Desafio) Pesquise sobre "rotas dinâmicas" no **Flask**. Crie uma rota /pokemon/<nome_do_pokemon> que exiba "Você escolheu o Pokémon: [nome_do_pokemon]". (Ex: /pokemon/pikachu deve exibir "Você escolheu o Pokémon: pikachu").
