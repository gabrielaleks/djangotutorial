Esse repositório possui o resultado final do [tutorial do Django](https://docs.djangoproject.com/en/3.1/intro/tutorial01/) + um resumo desse tutorial com explicação dos aspectos mais importantes.

# Parte 1

# Parte 2
O objetivo dessa parte é criar o banco de dados, criar o primeiro modelo e fazer uma introdução ao site de administração gerado automaticamente.

## Setup do banco de dados
1) Abra o mysite/settings.py. Esse módulo possui variáveis que representam as configurações do Django. Setar a TIME_ZONE para a local

Observação: O Django usa, por padrão, o SQLite. Ele é incluído no python, logo não é preciso instalar nada. 

2) O array INSTALLED_APPS possui o nome de todas as aplicações (apps) ativadas do Django. Os apps podem ser usados em múltiplos projetos. Alguns apps já vem com o Django:

```
django.contrib.admin
django.contrib.auth
django.contrib.contenttypes
django.contrib.sessions
django.contrib.messages
django.contrib.staticfiles
```

Essas aplicações usam tabelas do banco de dados, portanto as tabelas precisam ser criadas antes que as aplicações possam ser usadas. Para isso, rodar:

```
$ python manage.py migrate
```
Uma mensagem vai aparecer mostrando as operações feitas. O comando 'migrate' olha para o array INSTALLED_APPS e cria as tabelas necessárias de acordo com as configurações do banco de dados presentes no mysite/settings.py e para os database migrations feitos no app.

## Criando models
Model é o layout do banco de dados. Ele contém apenas os campos essenciais dos dados que você vai armazenar. Os comandos de migrate olham atuam sobre os arquivos model.

1) Criar dois models: Question e Choice. Question tem uma pergunta e uma data de publicação. Choice tem o texto da escolha e a contagem de votos.

Para criar os models, editar o arquivo polls/models.py da seguinte forma:

```
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

Cada modelo é representado por uma classe que chama django.db.models.Model. O modelo tem um número de variáveis, cada uma representando um campo do banco de dados. 

Cada campo é representado por uma instância da classe Field, como CharField e DateTimeField. O nome de cada uma dessas instâncias (question_text, pub_date) será o nome do campo.

A classe ForeignKey é usada para determinar uma relação entre Choice e Question: cada Choice é relacionada a apenas uma Question.

## Ativando models
1) O código em polls/models.py é utilizado pelo Django para:
- criar um database schema para o app (statement CREATE TABLE).
- criar uma api de acesso ao banco de dados para acessar os objetos Question e Choice. 

2) Antes de usar esse app, precisamos indicar que ele foi instalado. Para incluí-lo no projeto, é necessário fazer uma referência da sua classe de configuração no array INSTALLED_APPS em mysite/settings.py. A classe de configuração fica em polls/apps.py e se chama PollsConfig. Portanto, seu caminho é polls.apps.PollsConfig. Adicione isso no mysite/settings.py:

```
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
3) Temos que incluir o app polls. Execute o seguinte comando:

```
$ python manage.py makemigrations polls
```
Uma mensagem de criação dos modelos deve aparecer.

O comando makemigrations diz que mudanças foram feitas e que elas devem ser armazenadas como uma migration.

A migration é a forma como o Django armazena as mudanças nos modelos - ou seja, mudanças no database schema. Quando você executa makemigrations, uma migration é criada na forma de um arquivo. O arquivo criado pode ser encontrado em polls/migrations/0001_initial.py.

4) Para ver qual o código SQL que a migration vai rodar, execute:

```
$ python manage.py sqlmigrate polls 0001
```
O output é:

```
BEGIN;
--
-- Create model Question
--
CREATE TABLE "polls_question" (
    "id" serial NOT NULL PRIMARY KEY,
    "question_text" varchar(200) NOT NULL,
    "pub_date" timestamp with time zone NOT NULL
);
--
-- Create model Choice
--
CREATE TABLE "polls_choice" (
    "id" serial NOT NULL PRIMARY KEY,
    "choice_text" varchar(200) NOT NULL,
    "votes" integer NOT NULL,
    "question_id" integer NOT NULL
);
ALTER TABLE "polls_choice"
  ADD CONSTRAINT "polls_choice_question_id_c5b4b260_fk_polls_question_id"
    FOREIGN KEY ("question_id")
    REFERENCES "polls_question" ("id")
    DEFERRABLE INITIALLY DEFERRED;
CREATE INDEX "polls_choice_question_id_c5b4b260" ON "polls_choice" ("question_id");

COMMIT;
```

Observações:
- Nomes de tabelas são gerados combinando o nome do app (polls) com o nome do modelo (question e choice).
- Primary keys são adicionadas automaticamente.
- "_id" é adicionado no fim do nome da foreign key.
- O código SQL não foi executado, só mostrado.

4) Caso queira, o comando 

```
python manage.py check
```

pode ser executado. Sua função é checar se há algum problema no projeto. Todos os apps são checados.

5) Para criar as tabelas dos models no banco de dados (rodar o código SQL) execute o seguinte comando:

```
python manage.py migrate
```

O comando migrate executa todas as migrations que ainda não foram executadas. Ou seja, as mudanças feitas nos models são aplicadas e adicionadas ao banco de dados.

Portanto, as 3 etapas de interação com o banco de dados são:

- Mudar os models no models.py
- Criar migrations (pré-código SQL) com python manage.py makemigrations
- Aplicar mudanças no banco de dados (executar o código SQL) com python manage.py migrate

