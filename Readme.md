CURSO PYTHON DJANGO e CRIAÇÃO DE REPOSITÓRIO GITHUB



criando um novo repositorio GITHUB

GERAR UMA CHAVE SSH (CASO NÃO TENHA GERADO AINDA)
ssh -keygen -t rsa -b 4096 -C

ENTRAR NA PASTA DO SSH
cd ~/.ssh/

utilizar o comando abaixo para pegar a chave ssh e colar no settings (add ssh) do github.
cat id_rsa.pub 

CRIANDO UM NOVO REPOSITÓRIO E FAZENDO O PUSH PARA O GITHUB

entrar na pasta desejada e dar os comandos:
git init

git add. (para adicionar todos os arquivos) 

git commit -m " comentario do commit"


UTILIZAR O COMANDO ABAIXO PARA CRIAR O REPOSITÓRIO REMOTO "ORIGIN" com o endereço do ssh do github

git remote add origin git@github.com:saulocost4/django-project.git

DAR O PUSH DO SEU MASTER PARA O NOVO REPOSITÓRIO "ORIGIN" CRIADO
git push -u origin master

Depois só atualizar ou acrescentar qualquer arquivo e seguir os passos padrões
git add "nome do arquivo"(git add. para adicionar tudo)
git commit -m "comentário"
git push 

lembrar de conferir o git status(indica sua situação atual no processo)
git log(ver os commmits ja feitos),etc .






DJANGO AULA 6

criar um app
python manage.py startapp "nome do app"

acrescentar no arquivo settings.py em installed apps
'users.apps.UsersConfig' onde users é o nome do app

acrescentar o codigo no views.py para utilizar a biblioteca fornecida pelo django e criar um template para registro
de usuários register.html

from django.contrib.auth.forms import UserCreationForm

def register(request):
    form = UserCreationForm()
    return render (request, 'users/register.html',{'form': form})

criou o template para registrar usuarios
{% extends "blog/base.html" %}
{% block content %}    
    <div class= "content-section">
        <form method="POST">
            {% csrf_token %}
            <fieldset class ="form-group">
                <legend class ="border-bottom mb-4">Join Today</legend>
                {{ form }}
            </fieldset>
            <div class ="form-group">
                <button class = "btn btn-outline-info" type="submit">Sign Up</button>
            </div>
        </form>
        <div class="border-top pt-3">
            <small class="text-muted">
                Already Have An Account? <a class="ml-2" href="#"> Sign In </a>
            </small>
        </div>
    </div>
{% endblock content %}

criar a rota para acessar o link de registro no urls.py do projeto

from users import views as users_views
path('register/', users_views.register, name ='register'),

modificar o views para criação do usuário, redirecionamento de pagina ao criar o usuario e mensagem de sucesso
ao criar o usuario

from django.shortcuts import render, redirect
from django.contrib.auth.forms import UserCreationForm
from django.contrib import messages

def register(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            username = form.cleaned_data.get('username')
            messages.success(request,f'Account created for {username}!')
            return redirect('Blog-Home')
    else:
        form = UserCreationForm()
    return render (request, 'users/register.html',{'form': form})

acrescentar no html base para mostrar mensagem de sucesso ao criar o usuário
 
{% if messages %}
    {% for message in messages %}
       <div class="alert alert-{{ message.tags }}">
         {{ message }}
       </div>
    {% endfor %}
{% endif %}

e salvar o usuário acrecetando em views.py de users (dentro do if que valida a criação)
form.save()

criar um arquivo forms.py para adicionar o campo do email, etc ao formulario de inscrição do usuário e substituir 
o usercreationform

from django import forms
from django.contrib.auth.models import User
from django.contrib.auth.forms import UserCreationForm


class UserRegisterForm(UserCreationForm):
    email = forms.EmailField()

    class Meta:
        model = User
        fields = ['username', 'email', 'password1', 'password2']

utilizar crispy forms para organizar melhor a página, pip install django-crispy-forms e acrescentar

{% load crispy_forms_tags %}

no register.html

lembrar de colocar 'crispy_forms' no installed apps (settings.py) e CRISPY_TEMPLATE_PACK = 'bootstrap4'
para usar o bootstrap 4 ao invés do default.





DJANGO AULA 7



importar a biblioteca (no arquivo urls.py do projeto)
do django.contrib.author com as views para criar o login e logout e criar o path
para as aplicações

from django.contrib import admin
from django.urls import path,include
from users import views as user_views
from django.contrib.auth import as auth_views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('register/', user_views.register, name ='register'),
    path('login/', auth_views.LoginView.as_view(template_name='users/login.html'), name ='login'),
    path('logout/', auth_views.LogoutView.as_view(template_name='users/logout.html'), name ='logout'),
    path('',include("blog.urls")),
]

direcionando para o template a ser criado (login/logout.html) no argumento.

criar template para login e logout e referenciar as urls de sign up e sign in (register.html)

redirecionar a pagina para  a home ao logar (settings.py)
LOGIN_REDIRECT_URL = 'Blog-Home'

acrescentar no base.htlm um if para se o usuário estiver ou não logado e com isso aparecer login ou logout
de acordo com a situação ( na barra de navegações)

criar um template (profile.html) e definir que apenas as pessoas logadas tem direito a acessar essa pagina
além de redirecionar quem não está logado para fazer o login
acrescentar o path no urls.py e a requisiçao def profile(request) no views.py do users


 



DJANGO AULA 8





criando o perfil para um usuário, modificando o arquivo models.py do users e o arquivo admin.py
admin.py

from .models import Profile

admin.site.register(Profile)


models.py

from django.db import models
from django.contrib.auth.models import User

class Profile (models.Model):
    user =models.OneToOneField(User, on_delete=models.CASCADE)
    image= models.ImageField(default= 'default.jpg', upload_to='profile_pics')

    def __str__(self):
        return f'{self.user.username} Profile'

fazer as migrações , antes disso baixar o pillow pip install pillow , python manage.py makemigrations,
python manage.py migrate

utilizar o python shell pra ver o que foi criado, manage.py shell
associar o usuário à variável user
user=User.objects.filter(username='saulo').first()
user.profile(perfil)
user.profile.image(diretorio da imagem)
user.profile.image.width

mudar o profile.html

criar um arquivo signals.py para ser criado um perfil sem contato com admingpage, automaticamente

from django.db.models.signals import post_save
from django.contrib.auth.models import User
from django.dispatch import receiver
from .models import Profile

@receiver(post_save, sender=User)
def create_profile(sender, instance, created, **kwargs):
    if created: 
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_profile(sender, instance, created, **kwargs):
    instance.profile.save()

quando cria-se um usuário (salvo) é enviado um sinal que é recebido (receiver) que é uma função de criar o perfil
ou salvar o perfil.

acrescentar em apps.py do users

def ready(self):
        import users.signals




