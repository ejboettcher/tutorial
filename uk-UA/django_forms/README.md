# Django форми

Остання річ, яку б ми хотіли зробити з нашим сайтом це створити зручний спосіб додавати і редагувати пости. Django адміністратор це звісно круто, але водночас він є досить складним для налаштування. За допомогою форм ми матимемо абсолютний контроль над нашим інтерфейсом - ми можемо робити майже все, що тільки можемо собі уявити!

Що тішить стосовно Django форм так це те, що останні можна визначати як з нуля так і створювати засобами `ModelForm`, що зберігають результат форми всередині моделі.

Це саме те, що нам потрібно: створимо форму для нашої моделі `Post`.

Як і кожна важлива складова Django, форми мають свій власний файл: `forms.py`.

Потрібно створити файл із заданим ім'ям в папці `blog`.

    blog
       └── forms.py
    

Гаразд, давайте відкриємо його в редакторі коду і наберемо такий код:

{% filename %}blog/forms.py{% endfilename %}

```python
from django import forms

from .models import Post

class PostForm(forms.ModelForm):

    class Meta:
        model = Post
        fields = ('title', 'text',)
```

Спочатку нам потрібно імпортувати форми Django (`from django import forms`) та нашу модель (`from .models import Post`).

`PostForm`, як ви можливо зазначили, ім'я нашої форми. Нам потрібно сказати Django, що ця форма є `ModelForm` ( тому Django зробить для нас трішки магії ) – ` і рядок.ModelForm` відповідає за це.

Далі, у нас є `class Meta`, де ми вказуємо для Django котра з моделей має бути використана для створення форми (`model = Post`).

Зрештою, можемо зазначити якими полями має закінчуватись наша форма. У цьому сценарії ми хочемо опублікувати такі параметри як заголовок `title` і зміст `text`, дані про автора `author`, що автоматично встановлюються як дані особи, яка наразі увійшла в систему (тобто ви!), а також дата створення `created_date`, яка автоматично встановлюється при створенні публікації (тобто в коді), вірно ж?

І це все! Все що нам потрібно тепер зробити це використати форму в блоці *view* і відобразити її в шаблоні.

То ж ще раз ми створимо посилання на сторінку, URL- адресу, вид і шаблон.

## Посилання на сторінку з формою

Час відкрити `blog/templates/blog/base.html` у редакторі коду. Додаємо посилання всередину блоку `div` з ім’ям `page-header`.

{% filename %}blog/templates/blog/base.html{% endfilename %}

```html
<a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
```

Зверніть увагу, що ми хочемо викликати наше нове подання `post_new`. забезпечується темою bootstrap, яку ми використовуємо, і відображатиме для нас знак плюса.

Після додавання відповідного рядка, ваш файл HTML повинен виглядати таким чином:

{% filename %}blog/templates/blog/base.html{% endfilename %}

```html
{% load static %}
<html>
    <head>
        <title>Django Girls blog</title>
        <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
        <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css">
        <link href='//fonts.googleapis.com/css?family=Lobster&subset=latin,latin-ext' rel='stylesheet' type='text/css'>
        <link rel="stylesheet" href="{% static 'css/blog.css' %}">
    </head>
    <body>
        <div class="page-header">
            <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
            <h1><a href="/">Django Girls Blog</a></h1>
        </div>
        <div class="content container">
            <div class="row">
                <div class="col-md-8">
                    {% block content %}
                    {% endblock %}
                </div>
            </div>
        </div>
    </body>
</html>
```

Після зберігання та оновлення сторінки http://127.0.0.1:8000 ви побачите знайому помилку `NoReverseMatch` error. Правда ж? Добре! 

## URL

Відкриваємо `blog/urls.py` і додаємо відповідний рядок: 

{% filename %}blog/urls.py{% endfilename %}

```python
шлях('post/new/', views.post_new, name='post_new'),
```

Остаточно код буде виглядати так:

{% filename %}blog/urls.py{% endfilename %}

```python
from django.urls import path 
from . import views

urlpatterns = [
    path('', views.post_list, name='post_list'),
    path('post/<int:pk>/', views.post_detail, name='post_detail'),
    path('post/new/', views.post_new, name='post_new'),
]
```

Після того як ми оновимо сайт, ми побачимо помилку `AttributeError`, оскільки вид `post_new` не є реалізованим. Давайте додамо цей вид прямо зараз.

## post_new view

Тепер відкриємо файл `blog/views.py` і додамо наступні рядки:

{% filename %}blog/views.py{% endfilename %}

```python
from .forms import PostForm
```

А далі *view*:

{% filename %}blog/views.py{% endfilename %}

```python
def post_new(request):
    form = PostForm()
    return render(request, 'blog/post_edit.html', {'form': form})
```

Щоб створити нову форму `Post`, нам потрібно викликати функцію `PostForm()` і передати її до шаблону. Повернемось пізніше до цього відображення, а зараз ми створимо шаблон для форми.

## Шаблон

Нам потрібно створити файл `post_edit.html` у папці `blog/templates/blog`. Щоб створити форму нам потрібно декілька речей:

* Ми маємо проявити форму. Можемо це зробити, наприклад, за допомогою `{{ form.as_p }}`. 
* Верхній рядок має бути розміщений всередині тега HTML для форми: `<form method="POST">...</form>`.
* Нам знадобиться кнопка Зберегти - `Save`. Створимо її в HTML за допомогою: `<button type="submit">Save</button>`.
* І нарешті зразу після відкриття тегу для форми `<form ...>` потрібно додати `{% csrf_token %}`. Це дуже важливо з міркувань безпеки вашої форми! Якщо ви раптом забудете про цей біт, Django поскаржиться, коли будете намагатися зберегти форму:

![CSFR Forbidden page](images/csrf2.png)

Добре, отож давайте розглянемо як маж виглядати HTML код в `post_edit.html`:

{% filename %}blog/templates/blog/post_edit.html{% endfilename %}

```html
{% extends 'blog/base.html' %}

{% block content %}
    <h2>New post</h2>
    <form method="POST" class="post-form">{% csrf_token %}
        {{ form.as_p }}
        <button type="submit" class="save btn btn-default">Save</button>
    </form>
{% endblock %}
```

Час перезавантажитись! Йой! Ваша форма відображується!

![New form](images/new_form2.png)

Одну хвилинку! Коли ви набираєте щось в полях `title` і `text` і намагайтесь зберегти зміни - що ж станеться тоді?

Нічого! Ми знову опинимось на цій ж самій сторінці і наш текст зникне... і жодного нового поста. Отож, що ж пішло не так?

Відповідь: нічого. Мусимо проробити трохи більше роботи з нашим блоком *view*.

## Зберігання форми

Відкрийте `blog/views.py` знову. Наразі все що ми маємо в `post_new` це:

{% filename %}blog/views.py{% endfilename %}

```python
def post_new(request):
    form = PostForm()
    return render(request, 'blog/post_edit.html', {'form': form})
```

Коли ми відправляємо форму, ми потрапляємо в той самий view, але цього разу ми маємо більше даних в `request`, а якщо бути точним - в `request.POST` ( ця назва не є пов’язаною з блогерським терміном "пост", вона пов’язана з тим, що ми "постимо" дані. Запам’ятайте, що у файлі HTML наше визначення `<form>` містило деяку зміну `method="POST"`? Усі поля форми на даний момент знаходитимуться в `request.POST`. Ви не повинні перейменовувати `POST` на щось інше (лише єдине альтернативне прийнятне значення для атрибуту `method` це `GET`, але ми не маємо часу пояснювати в чому різниця).

Отож в нашому * режимі перегляду * ми маємо дві окремих ситуацій, що слід виконувати: спочатку, коли ми отримуємо доступ до сторінки вперше і хочемо отримати порожню форму, і вдруге, коли ми повертаємось до * виду * з усіма даними формами, які ми щойно вказали. Таким чином, маємо додати умову (використаємо для цього `if`):

{% filename %}blog/views.py{% endfilename %}

```python
if request.method == "POST":
    [...]
else:
    form = PostForm()
```

Час заповнювати точки `[...]`. Якщо ` метод ` `POST` тоді ми хочемо створити `PostForm` з даними нам формами, правда ж? Ми можемо зробити наступним чином:

{% filename %}blog/views.py{% endfilename %}

```python
form = PostForm(request.POST)
```

Наступний крок - перевірити правильність форми ( встановлено усі обов’язкові поля і не відбулося зберігання неправильних значень ). Ми зробимо це за допомогою `form.is_valid()`.

Ми перевіряємо чи є форма прийнятною і якщо це так, то можемо зберегти її!

{% filename %}blog/views.py{% endfilename %}

```python
if form.is_valid():
    post = form.save(commit=False)
    post.author = request.user
    post.published_date = timezone.now()
    post.save()
```

В основі, маємо тут два моменти: зберігаємо форму за допомогою `form.save` і додаємо автора (оскільки поле автор `author` було відсутнє в `PostForm` і це поле є необхідним). `commit=False` це означає, що ми не хочемо поки зберігати модель `Post` - ми хочемо спочатку додати ім’я автора. В більшості випадків ви будете використовувати `form.save()` без `commit=False`, але в даному випадку нам слід це зробити. `post.save()` це захистить зміни, які зробили ( додавання автора ) і получиться новий блог пост!

Нарешті, було б чудово, якби ми могли негайно переходити на сторінку `post_detail` для нашого нещодавнього створеного допису в блозі, правда ж? Для цього нам потрібен ще один імпорт: 

{% filename %}blog/views.py{% endfilename %}

```python
from django.shortcuts import redirect
```

Додайте його на самому початку вашого файлу. І тепер ми можемо сказати: перейдіть на сторінку `post_detail` для новоствореної публікації:

{% filename %}blog/views.py{% endfilename %}

```python
return redirect('post_detail', pk=post.pk)
```

`post_detail` ім’я виду, яке ми хочемо розглянути. Пам'ятаєте, що цей вид потребує змінної `pk`? Щоб передати вище згаданої змінної, використаємо `pk=post.pk`, where `post` є новоствореним постом!

Що ж, добре, забагато розмови, напевне ми хочемо побачити як відображається даний вид в цілому, чи не так?

{% filename %}blog/views.py{% endfilename %}

```python
def post_new(request):
    if request.method == "POST":
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.published_date = timezone.now()
            post.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm()
    return render(request, 'blog/post_edit.html', {'form': form})
```

Подивимося, чи усе працює. Перейдіть на сторінку http://127.0.0.1:8000/post/new/, і додайте `title` і `text`, не забудьте зберегти! Новий блог пост додано і нас перенаправило на сторінку `post_detail` !

Ви напевне замітили, що ми встановлюємо дату публікації перед збереженням поста. Згодом, ми введемо * кнопку публікації * в ** Django Girls Tutorial: Extensions**.

Просто чудово!

> Оскільки нещодавно ми працювали з інтерфейсом Django адміністратора, система вважає, що ми залогінились. Є кілька ситуацій, які могли призвести до того, що ми вийшли з системи ( закриття браузера, перезавантаження комп’ютера або бази даних, т.д. ) Якщо під час створення допису ви помітите, що у вас з’являються помилки, пов’язані з відсутністю зареєстрованого користувача, будь ласка перейдіть на сторінку адміністратора http://127.0.0.1:8000/admin і увійдіть знову. Проблему тимчасово буде вирішено. Безповоротне розв'язання проблеми чекає на вас у розділі **Домашня робота: безпека вашого сайту!** після освоєння головної програми.

![Logged in error](images/post_create_error.png)

## Form validation

А тепер, ми покажемо вам наскільки круто використовувати Django форми. Блог пост повинен мати такі поля як заголовок - `title` і зміст - `text`. В нашій моделі `Post` ми не вказали ( на відміну від дати публікації - `published_date`), що ці поля є обов’язковими, отже Django, за умовчуванням, очікує що вони будуть заповненими.

Намагайтесь зберегти форму без атрибутів `title` і `text`. Напевне здогадуємось що відбудеться!

![Form validation](images/form_validation2.png)

Django дбає про те, щоб перевірити правильність усіх полів у нашій формі. Хіба це не приголомшливо?

## Edit form

Тепер ми знаємо як додавати нову форму. Та якщо ми раптом захочемо відредагувати вже існуючу? Це дуже схоже на те, що ми щойно зробили. Давайте швиденько створимо деякі важливі речі. ( Якщо ви чогось не розумієте, вам слід запитати у свого тренера або переглянути попередні параграфи, оскільки всі ці кроки ми вже розглянули ).

Відкрийте `blog/templates/blog/post_detail.html` і додайте наступний рядок

{% filename %}blog/templates/blog/post_detail.html{% endfilename %}

```html
<a class="btn btn-default" href="{% url 'post_edit' pk=post.pk %}"><span class="glyphicon glyphicon-pencil"></span></a>
```

тому, шаблон буде виглядати таким чином:

{% filename %}blog/templates/blog/post_detail.html{% endfilename %}

```html
{% extends 'blog/base.html' %}

{% block content %}
    <div class="post">
        {% if post.published_date %}
            <div class="date">
                {{ post.published_date }}
            </div>
        {% endif %}
        <a class="btn btn-default" href="{% url 'post_edit' pk=post.pk %}"><span class="glyphicon glyphicon-pencil"></span></a>
        <h2>{{ post.title }}</h2>
        <p>{{ post.text|linebreaksbr }}</p>
    </div>
{% endblock %}
```

Відкрийте `blog/urls.py` і додаємо відповідний рядок:

{% filename %}blog/urls.py{% endfilename %}

```python
    path('post/<int:pk>/edit/', views.post_edit, name='post_edit'),
```

Повторюємо шаблон і використовуємо `blog/templates/blog/post_edit.html`, отож остання річ, яка залишила це відображення.

Давайте відкриємо `blog/views.py` і додамо це в самому кінці файлу:

{% filename %}blog/views.py{% endfilename %}

```python
def post_edit(request, pk):
    post = get_object_or_404(Post, pk=pk)
    if request.method == "POST":
        form = PostForm(request.POST, instance=post)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.published_date = timezone.now()
            post.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = PostForm(instance=post)
    return render(request, 'blog/post_edit.html', {'form': form})
```

Виглядає майже точно так як і наш вид `post_new`, чи не так? Але не зовсім. Спочатку: ми передамо додатковий параметр `pk` з адресного рядку. Наступне: ми отримуємо модель `Post`, яку хотіли б редагувати як `get_object_or_404(Post, pk=pk)` і потім, при створенні форми ми передаємо цей пост як приклад `instance`, коли зберігаємо форму...

{% filename %}blog/views.py{% endfilename %}

```python
form = PostForm(request.POST, instance=post)
```

...а коли ми просто відкриємо форму з цим постом для редагування:

{% filename %}blog/views.py{% endfilename %}

```python
form = PostForm(instance=post)
```

Добре, давайте подивимось чи все працює! Перейдімо на сторінку `post_detail`. У верхньому правому кутку має з’явитися кнопка редагування:

![Edit button](images/edit_button2.png)

Коли ви натиснете на неї, то побачите форму з нашим блог постом:

![Edit form](images/edit_form2.png)

Ви можете вільно змінити заголовок або текст і зберегти зміни!

Вітаємо! Ваш додаток стає все більше і більше завершеним!

Якщо ви хочете дізнатися більше інформації про Django форми прошу ознайомитись із документацією: https://docs.djangoproject.com/en/2.2/topics/forms/

## Безпека

Створювати нові публікації можна можна просто клацнувши на посилання - це ж класно! Але зараз, будь хто відвідує може відвідати ваш сайт, бути здатним опублікувати пост, а ви цього напевне не хочете. Давайте зробимо так, що кнопка показує для вас, але не показує для інших.

В `blog/templates/blog/base.html`, знайдіть наш `page-header` `div` і тег посилання який ви додали туди раніше. Вони повинні виглядати якось так:

{% filename %}blog/templates/blog/base.html{% endfilename %}

```html
<a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
```

Ми хочемо додати інший тег `{% if %}` який буде показувати посилання лише тим користувачам, які ввійшли від імені адміністратора. На даний момент, це лише ви! Змініть тег `<a>` щоб він виглядав так:

{% filename %}blog/templates/blog/base.html{% endfilename %}

```html
{% if user.is_authenticated %}
    <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
{% endif %}
```

Цей `{% if %}` зробить так, що посилання будуть відправлятись браузеру лише тоді, коли користувач, що запитував сторінку, зареєстрований. Це не захищає від створення нових публікацій анонімами повністю, але це непоганий перший крок. Ми розглянемо більше питань безпеки в розширених уроках.

Пам'ятаєте піктограму редагування, яку ми щойно додали на нашу сторінку деталей? Ми також хочемо додати ту саму зміну, щоб інші люди не мали змоги редагувати наявні публікації.

Відкрийте `blog/templates/blog/post_detail.html` і додайте наступний рядок:

{% filename %}blog/templates/blog/post_detail.html{% endfilename %}

```html
<a class="btn btn-default" href="{% url 'post_edit' pk=post.pk %}"><span class="glyphicon glyphicon-pencil"></span></a>
```

Змініть його на це:

{% filename %}blog/templates/blog/post_detail.html{% endfilename %}

```html
{% if user.is_authenticated %}
     <a class="btn btn-default" href="{% url 'post_edit' pk=post.pk %}"><span class="glyphicon glyphicon-pencil"></span></a>
{% endif %}
```

Оскільки ви швидше за все увійшли в систему, якщо ви оновите сторінку, ви не побачите нічого зміненого. Завантажте сторінку в інший браузер або вікно анонімного перегляду (що називається "InPrivate" у Windows Edge), і ви побачите, що посилання не відображається, і піктограма також не відображається!

## Ще одне: розгортання!

Давайте поглянемо чи це все працює на PythonAnywhere. Час для ще одного деплою!

* Спочатку зафіксуйте свій новий код і зробіть його на GitHub:

{% filename %}command-line{% endfilename %}

    $ git status
    $ git add .
    $ git status
    $ git commit -m "Додано подання для створення / редагування публікації в блозі сайту."
    $ git push
    

* Тоді, в [bash консолі PythonAnywhere](https://www.pythonanywhere.com/consoles/):

{% filename %}PythonAnywhere command-line{% endfilename %}

    $ cd ~/<your-pythonanywhere-domain>.pythonanywhere.com
    $ git pull
    [...]
    

(Не забудьте замінити `<your-pythonanywhere-domain>` вашим фактичним субдоменом PythonAnywhere, без кутових дужок.)

* Нарешті, перейти на сторінку ["Web"](https://www.pythonanywhere.com/web_app_setup/) ( використати кнопку меню у верхньому правому куті консолю ) та натисніть **Перезавантажити**. Оновіть свій https://subdomain.pythonanywhere.com блог, щоб почати зміни.

І на цьому все! Вітання вас :)