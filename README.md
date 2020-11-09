Esse repositório possui o resultado final do [tutorial do Django](https://docs.djangoproject.com/en/3.1/intro/tutorial01/) + um resumo desse tutorial com explicação dos aspectos mais importantes.

# Parte 1
O objetivo dessa parte é criar um projeto e uma aplicação.

## Criando um projeto
1) Um projeto é um pacote python que contém todas as configurações necessárias para uma instância Django. Isso inclui uma configuração do banco de dados, opções específicas do Django e configurações específicas de aplicações. 

Vá para o diretório onde quer criar o projeto e digite:

```
$ django-admin startproject mysite
```

Isso vai criar uma pasta mysite com a seguinte estrutura:

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```

Resumindo a função de cada um desses arquivos/pastas:
- mysite/ (externo): Contâiner do projeto.
- manage.py: Acessar esse arquivo via linha de comando te permite interagir com o projeto de várias formas.
- mysite/ (interno): Pacote python do projeto. 
- mysite/__init__.py: Arquivo vazio que diz ao python que esse diretório deve ser considerado um pacote python.
- mysite/settings.py: Nesse arquivo você configura o seu projeto Django.
- mysite/urls.py: Nesse arquivo você define as URLs do seu projeto.
- mysite/asgi.py: Um ponto de entrada para servidores web ASGI-compatíveis.
- mysite/wsgi.py: Um ponto de entrada para servidores web WSGI-compatíveis.

2) Para rodar o servidor, vá ao diretório mysite externo e rode o seguinte comando:

```
$ python manage.py runserver
```

Um servidor de desenvolvimento será gerado no localhost (http://127.0.0.1:8000/).

## Criando a aplicação Polls
1) Cada aplicação consiste em um pacote python que segue uma determinada convenção. A aplicação é criada dentro do projeto.

Observação: Uma aplicação (app) é uma aplicação web que realiza alguma função. Um projeto é uma coleção de apps para um determinado site. Um projeto pode conter múltiplos apps. Um app pode ser utilizado em múltiplos projetos. 

2) Para criar o app, vá para o mesmo diretório de manage.py e execute o seguinte comando:

```
$ python manage.py startapp polls
```

A estrutura criada vai conter a aplicação:

```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```

3) Vá para o polls/views.py e mude o código:

```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

Para chamar a view, é necessário atribuir uma URL. Para isso, crie um arquivo chamado urls.py dentro do diretório polls. Dentro desse arquivo escreva o seguinte código:

```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

3) Agora é necessário apontar esse módulo polls.urls no arquivo de configuração de URL geral, o URLconf. Isso é feito no arquivo mysite/urls.py da seguinte forma:

```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

Veja que no array urlpatterns nós adicionamos o item correspondente ao arquivo polls/urls.py. 

A função include() permite referenciar outras URLconfs. 

A função path() recebe 4 argumentos. Dois deles são necessários: route e view. Os outros são kwargs e name.

- route: É a string com a URL. Quando processando um request, o Django começa no primeiro padrão de url patterns e vai descendo a lista, comparando a URLS requisitada com cada padrão até encontrar algum que dê match.
- view: Quando o Django encontra um padrão, ele chama a função view específica com um objeto de HttpRequest como primeiro argumento e qualquer valor capturado pela rota como argumentos. 
- kwargs: "keyword arguments". 
- name: É possível dar nome à URL e chamá-la em qualquer lugar no Django, especialmente dentro de templates.

# Parte 2
O objetivo dessa parte é criar o banco de dados, criar o primeiro modelo e fazer uma introdução ao site de administração gerado automaticamente.

## Setup do banco de dados
1) Abra o mysite/settings.py. Esse módulo possui variáveis que representam as configurações do Django. Setar a TIME_ZONE para a local.

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
$ python manage.py check
```

pode ser executado. Sua função é checar se há algum problema no projeto. Todos os apps são checados.

5) Para criar as tabelas dos models no banco de dados (rodar o código SQL) execute o seguinte comando:

```
$ python manage.py migrate
```

O comando migrate executa todas as migrations que ainda não foram executadas. Ou seja, as mudanças feitas nos models são aplicadas e adicionadas ao banco de dados.

Portanto, as 3 etapas de interação com o banco de dados são:

- Mudar os models no models.py
- Criar migrations (pré-código SQL) com python manage.py makemigrations
- Aplicar mudanças no banco de dados (executar o código SQL) com python manage.py migrate

## Brincando com a API
Para abrir a shell do python:

´´´
$ python manage.py shell
´´´

Na shell, escreva:

```
# Importa os models que fizemos.
>>> from polls.models import Choice, Question

# Não há nenhuma questão no sistema ainda.
>>> Question.objects.all()
<QuerySet []>

# Criar uma nova questão, determinar seu texto e atribuir um horário a ela.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Salvar objeto no banco de dados.
>>> q.save()

# Agora a questão tem um id.
>>> q.id
1

# Acessar campos válidos do model via atributos do python.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

# Mudar valores e depois salvar.
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() mostra todas as questões do banco de dados.
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```

Para mudar essa mensagem final, vá para polls/models.py e adicione o método __str__() nos models Question e Choice:

```
from django.db import models

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```

Adicione também o seguinte método:

```
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```

Volte ao terminal:

```
$ from polls manage.py shell

# Mostre que __str__() foi adicionado
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# Podemos relacionar as questions via primary key. Usar pk=1 vai mostrar a question de primary key igual a 1.
>>> Question.objects.get(pk=1)
<Question: What's up?>

# Mostre que o método adicionado no model Question funciona.
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# Mostre as choices existentes
>>> q.choice_set.all()
<QuerySet []>

# Crie três choices.
>>> q.choice_set.create(choice_text='Not much', votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

# Mostre as choices
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# Apague uma choice
>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()
```

## Introdução ao Django Admin 
1) Crie um superusuário para entrar na página de admin com o seguinte comando:

```
$ python manage.py createsuperuser
```

Insira o nome de usuário, email e senha para completar o processo.

2) Execute "$ python manage.py runserver" para abrir o servidor e acesse /admin para entrar na página de login. Ao fazer o login com o nome e senha definidos anteriormente, você verá a página de administração de usuários e grupos.

3) Para fazer com que os objetos Question tenham interface na página de admin, abra polls/admin.py e adicione o seguinte:

```
from django.contrib import admin

from .models import Question

admin.site.register(Question)

```

# Parte 3
O objetivo desta parte é continuar o app de polls com foco na criação de uma interface pública - "views"

## Visão geral
1) Uma view é um tipo de página web na aplicação que serve a um propósito e tem um template específico.

Nossa aplicação poll terá essas 4 views: 
- Página "index" de Question: Mostra as últimas questions.
- Página "detail" de Question: Mostra um texto de question sem resultados mas com um formulário de votação.
- Página "results" de Question: Mostra os resultados para uma question.
- Vote action: Lida com a votação de uma choice específica de uma question específica.

2) No Django, as views lidam com web pages e outros conteúdos. Cada view é representada por uma função/método python. A view é determinada através da URL que é requisitada. O padrão URL é a forma geral do URL. Exemplo: /newsarchive/+year+/+month+/.

3) Adicione novas views no polls/views.py:

```
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

Essas views são diferentes porque levam um argumento "question_id".

4) Vá a polls/urls.py e adicione novos urlpatterns:

```
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

Agora o site apresenta respostas para polls/algumnumero, polls/algumnumero/results e polls/algumnumero/vote. 

Lembre-se: a view é determinada pela url (e não o contrário!). Acessar o página /polls/10 vai executar o método detail() utilizando 10 como question_id.

Quando alguém requisita uma página do site - por exemplo, "polls/34/", o Django carrega o módulo mysite.urls porque ele é apontado pela configuração ROOT_URLCONF (presente em mysite/settings.py). Lá ele encontra a variável urlpatterns e lê os padrões um por um. Quando encontra "polls/", ele corta a parte da string igual ("polls/") e manda a string restante ("34/") para o URLconf de polls.url para procurar lá. Lá ele encontra 

```
'<int:question_id/>'
```

e chama o método detail(). Usar os sinais "maior de" e "menor de" permite capturar parte da URL e usá-la como argumento da função da view. A parte ":question_id>" da string define o nome do padrão da URL. A parte "int:" é um converter que determina quais padrões devem ser iguais à URL.

## Escreva views que fazem alguma coisa
Cada view faz duas coisas: retorna um objeto do tipo HttpResponse com o conteúdo para a página requisitada ou apresenta uma exception (tipo Http404).

1) Coloque o código abaixo na view. Ele mostra as últimas 5 questions do sistema separadas por vírgula e de acordo com a data de publicação:

```
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

# não mexer nas outras views (detail, results, vote)
```

2) O design da página está hard-coded (se você quiser mudar a forma como a página se apresenta, terá que mudar o código). Para mudar isso, vamos usar o sistema de template do Django.

Crie um diretório chamado templates dentro do diretório polls. É aí que o Django vai buscar por templates. 

A configuração TEMPLATES (em mysite/settings.py) descreve a forma em que o Django carrega e renderiza os templates. A configuração default tem um DjangoTemplates cuja opção APP_DIRS é setada como True. Por convenção, DjangoTemplates procura por subdiretórios "templates" em cada um dos INSTALLED_APPS.

3) Dentro do diretório templates crie outro diretório chamado polls. Dentro dele, crie um arquivo chamado index.html. Embora o caminho desse arquivo seja polls/templates/polls/index.html, o template loader do Django te permite referenciar esse arquivo simplesmente como polls/index.html.

Coloque o código abaixo no arquivo html:

```
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

Veja que se latest_question_list (últimas 5 questions ordenadas por ordem de publicação) existir, uma lista com o texto de cada uma das questions onde cada question aponta para sua própria página é gerada. Se não, o texto "No polls are available" aparece.

4) Agora vá para polls/views.py e atualize a view index:

```
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

Note que usamos render(), portanto é necessário fazer o import. Usar render() economiza código porque não precisamos declarar uma variável template associada ao arquivo html para depois usá-la como argumento no HttpResponse (ficaria HttpResponse(template.render(context, request))).

## Gerando uma página para erro 404
1) Coloque esse código na view detail em polls/views.py:

```
from django.http import Http404
from django.shortcuts import render

from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```

e crie um arquivo polls/templates/polls/detail.html com 

```
{{ question }}
```

para ver o código funcionando.

2) É prática comum usar a função get_object_or_404() no caso em que o objeto não existe. Veja a view detail reescrita:

```
from django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

Agora uma linha substitui as quatro de antes. A função get_object_or_404 recebe o model como primeiro argumento e algo arbitrário como segundo. Ela passa esses argumentos para a função get() do model manager e, caso o objeto não exista, a função Http404 é chamada.

## Removendo URL hardcoded do template
Havíamos escrito o seguinte no arquivo polls/index.html:

```
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```

Torna-se problemático mudar URLs em projetos com muitos templates. Entretanto, como definimos o argumento "name" nas funções path em polls.urls, podemos usar a tag {% url %} do template para não precisar hardcodar a URL.

```
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

Se quiséssemos mudar a URL da view details, ao invés de ter que mudar no(s) template(s) em que ela aparece, basta mudar em polls/urls.py. Por exemplo, se quisermos que a URL comece com 'specifics':

```
...
# added the word 'specifics'
path('specifics/<int:question_id>/', views.detail, name='detail'),
...
```

## Usando namespace em URLs
Projetos reais de Django tem muitos apps. O Django precisa diferenciar os nomes de URLs desses apps. O app polls tem uma view detail, e outro app também poderia ter.

Para resolver esse problema nós usamos namespaces no URLconf. Vá ao arquivo polls/urls.py e adicione um app_name:

```
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

Agora mude o arquivo polls/index.html. Coloque "polls:" antes de detail dentro da tag url:

```
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

# Parte 4
O objetivo dessa parte é o processamento de um formulário e a diminuição do código.
