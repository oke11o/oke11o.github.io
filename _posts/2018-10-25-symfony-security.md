---
layout: post
title:  "Symfony Security Сomponent"
date:   2018-10-27 12:20:50 +0300
categories: symfony tutor
---

# Symfony Security Сomponent

## 1. Security & the User Class

Security - это огромная тема. 

Можно аутентификации есть:
- login form
- token based API auth system
- two-factor auth
- across an API to a Single Sign-On server
- others

Для авторизации:
- roles
- access control
- others

И в symfony4 появиились новые фичи, которые тут рассмотрим.


#### Получить код и запустить

Надо скачать код для этого урока. Он немного отличается от того, на чем мы закончили в прошлый раз. 
Обновили symfony до 4.1. и изменили fixtures.

```php bin/console server:run```


#### Установка security и обновлени MakerBundle

Первая задача - создать authentication system. То есть, как пользователь будет логинитться. 
Не важно как это будет, логин форма или что-то другое. Сперва надо создать пользователя User class.

Для этого появилась новая фича. В версии maker-bundle 1.7 появилась команда `make:user`.

```php bin/console make:user```

Оу. Только сперва надо поставить `security`. 

```composer require security```


#### Создание User Class с `make:user`

```php bin/console make:user```

Будет задавать вопросы. Вопрос, про то, надо ли хранить данные в базе, 99% надо отвечать да, 
даже если вы получаете данные по LDAP или c single sign-on server'a. Нет надо отвечать, только если вы вообще не хотите хранить данные у себя.

Потом выбираем unique field. По умолчанию это email и нас устроит.

Потом указать, нужен ли пароль. Например, для API auth пароля нет вообще. 
Райн решил выбрать NO и добавить пароль позже самостоятельно.



## 2. Все про User class

Так как мы выбрали `yes` в вопросе хранения в базе, был создал UserRepository. И id другие свойства. Ничего особенного. Обычный Entity класс.

Единственное, он `class User implements UserInterface`

Такие методы как `getUsername()`, `getRoles()`, `getPassword()`, `getSalt()` и `eraseCredentials()`.

Так же изменился файл `config/packages/security.yaml`. Изменился security provider.

```yaml
security:
    providers:
        app_user_provider:
            entity:
                class: App\Entity\User
                property: email
```

User provider нужен каждому классу User.



## 3. Настройка User Entity

#### Добавим полей

Добавим поля в User.

```php bin/console make:entity```

firstName

#### Настроим Doctrine server_version

Стоит обратить внимание на поле `roles`. Это json. И в MySQL5.7 и в PostreSQL есть тип JSON. 
В других его может не быть. Не важно. Doctrine может сделать json_encode() из простого TEXT типа.
Но в `config/packages/doctrine.yaml` есть ключ `server_version`. По умолчанию оно стоит в 5.7.
Можно попробовать изменить. Райн это сделал. Я не хочу.

#### Создание миграций

```bash
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

#### Создание фикстур

```php bin/console make:fixtures```

Райн любит свою заготовку BaseFixture. 
Он там добавил новую фичу `groupName`

```php bin/console doctrine:fixtures:load```


## 4. The Login Form

Необходимо 2 шага для построения Login Form. Создание визуальной части. И части логикии отправки формы - 
поиск пользователя, проверка пароля и механизм Login.
Первая часть - это обычная скучная форма.

Кстати, в планах - создание команды `make` для построения такой формы. Пока ее нетт, поэтому создадим ее вручную.


#### Создание контроллера и шаблона

Создадим контроллер. ```php bin/console make:controller``` с именем `SecurityController`. Создан класс и шаблон.

Заменим `index()` action на `login()`. Почистим шаблон


#### Заполним логику безопастности в LoginForm

Обратимся к документации https://symfony.com/doc/current/security/form_login_setup.html

Возьмем оттуда тело экшна. Обратим внимание, что нам нужен `class Symfony\Component\Security\Http\Authentication\AuthenticationUtils`.
Так же возьмем из документации шаблон формы.

Из формы можем убрать `action="{{ '{{' }} path('app_login') }}"`, чтобы форма отправлялась на тот же url, на котором была отображена.

Добавим в шаблон переменные из контроллера.

Для красоты заменим шаблон на bootstrap.

#### Добавим login.css

И добавим css файл. Круто! Мы сделали это без encore. Если что, можно его добавить.


#### Поля LoginForm

Это обычная html форма. Даже не symfony form. Тут можно активировать флажок Запомни меня, но сделаем это позже.

Единственное - это блок вывода ошибок. 


#### Добавление ссыки на логин


## 5. Firewalls & Authenticator

Мы создали традиционную форму и так как форма отправляется на тот же url, можно пологать, что форма будет обрабатываться в этот экшне.

#### What are Authentication Listeners / Authenticators?

На самом деле нет. Тут работает Security Component. В начале каждого запроса Симфони вызывает "authentication listeners" или "authenticators".
Их работа - смотреть запрос и проверять, есть ли авторизационная информация. Логин/пароль или может API auth token.
И если они есть, попробовать аутентицироваться.

#### Понимание Firewalls

Откроем конфиг `config/packages/security.yaml`. Главная секция там `firewalls`. Что же такое firewall на языке Симфони.
Вернемся к теории. Есть 2 главных части: authentication and authorization.
Authentication - все про поиск, кто же ты и как доказать, что это вы. То есть login процесс.
Authorization выполняется после Authentication и определяет, есть ли у вас доступ к ресурсу.

Вся работа firewall'a - authentication. И как правило имеет смысл только один firewall, даже если вы хотите, чтобы 
у вас было много возможностей авторизоваться. И через логин форму и через API.

Но по умолчанию в симфони указано 2 firewall'a. В начале каждого запроса Симфони проверяет, какой firewall соответсвует текущему запросу.
Он сравнивает URL с pattern в настройках firewall. Первый firewall - фейковый. Он для `pattern: ^/(_(profiler|wdt)|css|images|js)/`
У него указано `security: false`. То есть не требует авторизации.

И реальный firewall - `main`. Так как у него не настроен `pattern` - он используется для всех запросов за исключением указанных выше.

Кстати, имя firewall'a (dev, main) - бессмысленны. Не имеют значения.

Большая часть настроек firewall'a - это добавление authentication listeners.

Обратим внимание на `anonymous: true`. Оставить это, чтобы пользователи могли получить доступ к public страницам сайта.
Даже если таких страниц нет, лучше это оставить, а запрет делать в спец месте `access_control`.


#### Creating the Authentication with make:auth

```php bin/console make:auth``` и назовем класс `LoginFormAuthenticator`.

Это крутой класс. Он наследуется от `AbstractGuardAuthenticator`. 

Но можно сделать чуть меньше работы наследуясь от `AbstractFormLoginAuthenticator`. 
И можно удалить методы `onAuthenticationFailure()`, `start()` и `supportsRememberMe()`. Они уже реализованы.
Для аутентикатора API токен мы эти методы будем использовать, но это позже.
И надо добавить метод `getLoginUrl()`.


#### Activating the Authenticator in security.yaml

К сожалению, данная команда не активирует аутентикатор автоматически. Придется его настроить вручную.
```yaml
security:
    firewalls:
        main:
            guard:
                authenticators:
                    - App\Security\LoginFormAuthenticator
```

Вся система аутентикации проходит из компонента безопастности симфони под названием guard. Отсюда и название.
Когда мы это добавили, все запросы будут вызывать метод `support()` каждого аутентикатора.

Можно это проверить, добавив `die()` в этот метод. 

Да. Так и есть.


## 6. Login Form Authenticator

#### The supports() Method

Этот метод вызывается на каждом запросе для каждого зарегистрированного listener'a.

```php 
return $request->attributes->get('_route') === 'app_login' && $request->isMethod('POST');
```


Если метод возвращает false, это означает, что аутентикатор не подходит и дальше он ничего делать не будет.

Если он возвращает true, это означает, что данный аутентикатор может обработать данный запрос и попытаться аутентифицировать пользователя.
Он вызовет `getCredentials()`

Попробуем. Выведем в нем `dump($request->request->all()); die();` или тоже самое `dd($request->request->all());`

#### The getCredentials() Method

Задача метода, получить из Request данные для аутентикации. В нашем случае, это email and password. В случае API токена - API token))

Вернем массив 

```php
public function getCredentials(Request $request)
{
    return [
        'email' => $request->request->get('email'),
        'password' => $request->request->get('password'),
    ];
}
```

Ответ метода `getCredentials(Request $request)` будет передан в метод `getUser($credentials, UserProviderInterface $userProvider)`.

Можно проверить `dd($credentials)`.


#### The getUser() Method

Этот метод должен возвращать User or null. Так как мы будет брать из базы, нам нужен UserRepository. Заинжектим его в __construct.

```php
public function __construct(UserRepository $userRepository)
{
    $this->userRepository = $userRepository;
}
...
public function getUser($credentials, UserProviderInterface $userProvider)
{
    return $this->userRepository->findOneBy(['email' => $credentials['email']]);
}
```

Если getUser() возвращает User, то сразу вызывается метод `checkCredentials($credentials, UserInterface $user)`.


#### The checkCredentials() Method

Так как у наc пока нет password, будем всегда возвращать true.

Если метод checkCredentials() возвращает true, то происходит вывоз метода `onAuthenticationSuccess()`.
Если false - метода `onAuthenticationFailure()`, 
который определен у нашего родительского `class \Symfony\Component\Security\Guard\Authenticator\AbstractFormLoginAuthenticator`.

Супер! Мы прошлись почти по всем шагам аутентикации.


## 7. Redirecting on Success & the User Provider

Если `checkCredentials()` вернул true, пользователь авторизовывается. Но что же делать дальше. 
Дальше вызывается метод `onAuthenticationSuccess()`
Для форм логин - ответ, надо куда-то редиректить.
Для API token системы - ничего.

Если мы на onAuthenticationSuccess() вернем Response. Он будет сразу отправлен, как ответ.
Если же мы ничего не вернем, то запрос пойдет дальше к контролеру.

#### Redirecting on Success

В Симфони для редиректа надо вернуть `class RedirectResponse`.

Нам придется добавить RouterInterface, чтобы верну указывать, куда мы хотим редиректить пользователя.

`php bin/console debug:autowiring`

Добавим редирект `return new RedirectResponse($this->router->generate('app_homepage'));`

Все. Теперь успешная авторизация работает полностью. Можно увидеть в DebugToolbar, что мы авторизовались как пользователь `spacebar1@example.ru`

#### Authentication & the Session: User Provider

После того, как мы авторизовались, информация о авторизованном пользователе хранится в сессии. 
Поэтому, когда на LoginFormAuthenticator не срабатывает, мы все равно остаемся авторизованными.

А точнее в `class \Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorageInterface`.

Помните, в security.yaml мы указывали UserProvider. Как раз он и помогает достать пользователя из сессии.

Это немного запутано, но очень важно. Сам объект User хранится в сессии, но он должен проверить, не устарела ли эта информация в сравнении с базой.

Это и есть работа UserProvider'a. Брать User'a по id из базы для fresh User.


## 8. Authentication Errors

Но мы еще не сделали обработку ошибок.
Если сейчас указать неверный email, нам выдаст ошибку `Cannot redirect to an empty URL.`.

Эта ошибка приходит из `\Symfony\Component\Security\Guard\Authenticator\AbstractFormLoginAuthenticator::onAuthenticationFailure()`
Который в свою очередь вызывает `\App\Security\LoginFormAuthenticator::getLoginUrl`.


#### Filling in getLogUrl()

`return $this->router->generate('app_login');`

Отлично, теперь мы видим ошибку авторизации в шаблоне `Username could not be found.`. То что надо. 
А если мы укажем в `checkCredentials()` return false, и укажем верный email, то ошибка будет `Invalid credentials.`.


#### How Authentication Errors are Stored

Если посмотреть `\App\Controller\SecurityController::login()` можно увидеть, 
что ошибку мы берем из метода `\Symfony\Component\Security\Http\Authentication\AuthenticationUtils::getLastAuthenticationError()`,
который в свою очередь смотрит `$request->attributes->has(Security::AUTHENTICATION_ERROR)` и `$session->has(Security::AUTHENTICATION_ERROR)`

То есть по специальному ключу в сессии или в запросе (`Security::AUTHENTICATION_ERROR`).

Ага. И можно увидеть, что в сессию ошибка устанавливается в `\Symfony\Component\Security\Guard\Authenticator\AbstractFormLoginAuthenticator::onAuthenticationFailure()`


#### Filling in the Last Email

Когда мы указали неверный email, нас средиректило обратно на `app_login`. Но мы потеряли данные, какой же email мы ввели.
Неплохо было бы это пофиксить.

Хм. Так в контроллере мы уже берем `\Symfony\Component\Security\Http\Authentication\AuthenticationUtils::getLastUsername()`.
Блин, забыли вывести в шаблоне.

Но Симфони сама не сохраняет последний username. Так как просто не знает, что же у нас им является.
Придется указать это нам самим.

```php
public function getCredentials(Request $request)
{
    $credentials = [
        'email' => $request->request->get('email'),
        'password' => $request->request->get('password'),
    ];

    $request->getSession()->set(
        Security::LAST_USERNAME,
        $credentials['email']
    );

    return $credentials;
}
```

Теперь осталось кастомизировать сообщения об ошибке и создать logout.


## 9. Customizing Errors & Logout

#### Customizing Error Messages

Есть 2 способа.

1й - создать свой собственный exception class `CustomUserMessageAuthenticationException`. 
Мы сделаем это, когда будем делать API authenticator.

2й - использую translator. 
Обратим внимание, в шаблоне `{{ '{{' }} error.messageKey|trans(error.messageData, 'security') }}`
В debug toolbar можно посмотреть, какие. На пункте меню Translations.
Создадим файл `translations/security.en.yaml` и там укажем перевод

```yaml
Username could not be found.: Oh no! It doesn't look like that email exists!
Invalid credentials.: Invalid credentials!
```

Не забуйте сбросить кэш.


#### Logging Out

Создадим роут в 
```php
/**
 * @Route("/logout", name="app_logout")
 */
public function logout() { throw new \Exception('Will be intercepted before getting here!'); }
```

А в `config/packages/security.yaml` укажем.

```yaml
security:
    firewalls:
        main:
            logout:
                path: app_logout
```

То есть наш файрвол свяжет данный роут со своей системой разлогирования. Можно проверить, перейдя на url `/logout`.

Там можно настроить, куда редиректить после логаута. См документацию.

Теперь надо поговорить о CSRF защите и функцииональности "Запомни меня".


## 10. CSRF Protection

Для защиты от CSRF атак надо добавить на форму логина CSRF token. 
По умолчанию в Симфони эта защита влючена.
Но мы не использовали стандартную форму Симфони.
Придется сделать это вручную.


#### Adding the CSRF Input Field

```html
<input type="hidden" name="_csrf_token" value="{{ '{{' }} csrf_token('authenticate') }}">
```

#### Verifying the CSRF Token

```php
public function getCredentials(Request $request)
{
    $credentials = [
        'email' => $request->request->get('email'),
        'password' => $request->request->get('password'),
        'csrf_token' => $request->request->get('_csrf_token'),
    ];

    $request->getSession()->set(
        Security::LAST_USERNAME,
        $credentials['email']
    );

    return $credentials;
}
```

Токен будем проверять в `getUser()` так как хотим, чтобы он был проверен до того, как мы сделаем запрос в базу.

Заинжекрим сервис в наш аунетикатор. `php bin/console debug:autowiring csrf` покажет нам, что подходит класс
`Symfony\Component\Security\Csrf\CsrfTokenManagerInterface`

```php
public function getUser($credentials, UserProviderInterface $userProvider)
{
    $token = new CsrfToken('authenticate', $credentials['csrf_token']);
    if (!$this->csrfTokenManager->isTokenValid($token)) {
        throw new InvalidCsrfTokenException();
    }

    return $this->userRepository->findOneBy(['email' => $credentials['email']]);
}
``` 

Обратить внимание, что строку, которую мы передали в шаблоне, ту же строку надо проверять на isTokenValid().
В нашем случае это 'authenticate'.

Теперь "Запомни меня"


## 11. Adding Remember Me

Сделаем его.

Добавим в шаблоне для этого input'a атрибут и значение.

```html
<input type="checkbox" name="_remember_me" value="true">
```

Значение проверяется в `\Symfony\Component\Security\Http\RememberMe\AbstractRememberMeServices::isRememberMeRequested()`
И должно быть

```php
return 'true' === $parameter || 'on' === $parameter || '1' === $parameter || 'yes' === $parameter || true === $parameter;
``` 


Это магическое поля для Симфони.

И в `security.yaml` добавим секцию:

```yaml
security:
    firewalls:
        main:
            remember_me:
                secret: "%kernel.secret%"
                lifetime: 2592000 # 30 days. Default is one year
```

#### More about Parameters

Если пользователь устанавливает поле `_remember_me`, при успешной авторизации ему устанавливается кука. 
И он автоматически авторизуется в течение этого времени.
Мы тут использует спец криптографический параметр `"%kernel.secret%"`. 

Можно увидеть его в:

```php bin/console debug:container --parameters```

```kernel.secret  %env(APP_SECRET)% ```

Ага, берется из ENV.

#### Watch the Cookie Save Login!

Проверим в инспекторе. Разлогинимася, и посмотрим, что в куках. `Inspector -> Application -> Storage -> Cookies -> http://127.0.0.1:8000`
В начале только PHPSESSID... иии разные счетчики от Гугла. Сука!!

И когда мы укажем галочку, то появится новая куку `REMEMBERME`

И теперь, если удалить куку PHPSESSID (где и хранится инфа о сессии, а соответственно о авторизованном пользователе),
то создастся новая сессиия, а пользователь будет авторизован, так как данные возьмутся из этой куки `REMEMBERME`.

Пора разобраться с паролем.



## 12. Adding & Checking the User's Password

#### Adding the password Field to User

```php bin/console make:entity``` -> User -> password -> string -> 255 -> no

```bash
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

#### Updating the User Class

Теперь у нас есть password в классе пользователя и в базе.
Но maker подсказал, что метод `getPassword()` already exists. 
Да. Но он был пустой. Исправим его

```php
public function getPassword()
{
    return $this->password;
}
```

Стоит обратить внимание, что `password` это не `plainPassword`. Это salted and encoded string. (Посоленная и закодированая).

Про соль. Хорошая новость. Все современные энкодеры используют эту рандомную соль уже внитри своих алгоритмах и так же хранять.
То есть если закодировать ими одну и туже строку, соль будет примешана внутри и результат будет разных. Хотя декодироваться будет одинаково.
Поэтому оставим `getSalt()` пустым и просто изменим комментарий.

```// not needed when using bcrypt or argon```


#### Configuring the Encoder

Симфони сама заботится о кодировании. Но ей надо указать, для какого класса, какое использовать.
Добавим:

```yaml
security:
    encoders:
        App\Entity\User:
            algorithm: bcrypt
            # cost: 30
```

Есть еще `argon2i`. Он покруче `bcrypt`, но использует PHP 7.2 или ему нужно php расширение Sodium.

К сожалению, менять алгоритм кодирования, когда пользователи уже забили свои пароли - это боль(((

Для обоих алгоритмов еще есть опция `cost`. Чем выше число - тем защищеннее шифрование. Но и больше нагрузка на CPU. 
Если вам это важно - читаейте доки по алгоритмам шифрования и выбирайте подходящий. Я оставлю значение по умолчанию.

Теперь Симфони может брать чистый пароль и сравнивать его с закодированным паролем пользователя.

#### Encoding Passwords

Сперва укажем наш пароль в Fixtures.

```bash
php bin/console debug:autowiring | grep assword
```

Видим 

```
 Symfony\Component\Security\Core\Encoder\UserPasswordEncoderInterface                
      alias to security.user_password_encoder.generic 
```

Заинжектим этот сервис в фикстуру `UserFixture`. И в них жестко зашьем пароль `engage`

```php
public function __construct(UserPasswordEncoderInterface $passwordEncoder)
{
    $this->passwordEncoder = $passwordEncoder;
}
protected function loadData(ObjectManager $manager)
{
    // ...
        $user->setPassword($this->passwordEncoder->encodePassword($user, 'engage'));
    // ...
}
```

Выполним фикстуру и проверим, что у нас появилось это поле в базе.

```bash
php bin/console doctrine:fixtures:load
php bin/console doctrine:query:sql 'SELECT * FROM user'
```


#### Checking the Password

Заинжектим в наш `\App\Security\LoginFormAuthenticator` encoder `Symfony\Component\Security\Core\Encoder\UserPasswordEncoderInterface`


```php
public function __construct(
    UserRepository $userRepository,
    RouterInterface $router,
    CsrfTokenManagerInterface $csrfTokenManager,
    UserPasswordEncoderInterface $passwordEncoder
) {
    $this->userRepository = $userRepository;
    $this->router = $router;
    $this->csrfTokenManager = $csrfTokenManager;
    $this->passwordEncoder = $passwordEncoder;
}

public function checkCredentials($credentials, UserInterface $user)
{
    return $this->passwordEncoder->isPasswordValid($user, $credentials['password']);
}
```

Теперь можно проверить. Сперв в форме попробуем залогиниться с `foo` паролем. Не проходит. Потом указываем верный пароль `engage`.

Работает!!!

Пора разобраться в `access_control`.


## 13. access_control Authorization & Roles

Все что было до этого касалось аутентификации.
Пора разобраться с авторизаций. То есть права. Может ли пользователь видеть какой-то блок или страницу.

Есть 2 способа авторизации. access_control и управление доступом в контроллере.

#### access_control in security.yaml

Раскомментируем в `security.yaml`:

```yaml
security:
    access_control:
        - { path: ^/admin, roles: ROLE_ADMIN }
```

Тут `path` - это регулярка.
То есть тут сказано, что все url'ы начинающиеся на `/admin`, для них необходима роль `ROLE_ADMIN`

```php bin/console debug:router```

Ага, у нас есть 2 роута `/admin/article/new ` и `/admin/comment`

Попробуем перейти на `/admin/comment`. Меня перебрасывает на `/login`. После авторизации я вижу `Access denied`.


#### Roles!

Система проверяет у пользователя роли через метод `getRoles()`. Обратить внимание, что у нас сейчас в этом методе добавлен код

```php
public function getRoles(): array
{
    $roles = $this->roles;
    // guarantee every user at least has ROLE_USER
    $roles[] = 'ROLE_USER';

    return array_unique($roles);
}
```

То есть мы гарантируем, что у юзера как минимум есть роль `ROLE_USER`. 


#### Only One access_control Matches per Page

Стоит понимать одну важную вещь. В `access_control` можно указывать множество правил. Но они работают так же как роуты.
То есть идут сверху вниз, и как только находят нужное правило, используют его. И дальше не идут.
На одной странице используется только одно правило из `access_control`.


## 14. Target Path: Redirecting an Anonymous User

#### Customizing the Error Page

В dev окружении на страницах ошибок мы видим Stack trace и дебаг панель. В prod окружении мы увидим скучную страницу ошибки.
Но ее можно задизайнить. См доку https://symfony.com/doc/current/controller/error_pages.html


#### Redirecting Anonymous Users: Entry Point

Как я писал выше, если я не авторизован и пытаюсь зайти на страницу, для которой настроен access_control, меня перекидывет на `login`.
За это поведение отвечает как раз наш `LoginFormAuthenticator` а точнее метод его родителя
`\Symfony\Component\Security\Guard\Authenticator\AbstractFormLoginAuthenticator::start())`

#### Redirecting Back on Success

После того, как нас с `/admin/comment` перекинуло на `/login` и мы авторизовались, нас должно перекинуть обратно на `/admin/comment`.
Но сейчас кидает на `homepage`.

За это отвечает `\App\Security\LoginFormAuthenticator::onAuthenticationSuccess()`. Там сейчас жестко зашит роут, куда редиректить после успешной аутентикации.

У симфони для решения этой пробемы есть спец помощник. Перед тем, как с `/admin/comment` перекинуть на `/login` Симфони сохраняет
этот `/admin/comment` роут в сессию по спец ключу. 

И можно его взять. Для этого используем trait `use \Symfony\Component\Security\Http\Util\TargetPathTrait;`. И добавляем

```php
// class LoginFormAuthenticator extends AbstractFormLoginAuthenticator

use \Symfony\Component\Security\Http\Util\TargetPathTrait;
//...
public function onAuthenticationSuccess(Request $request, TokenInterface $token, $providerKey)
{
    if ($targetPath = $this->getTargetPath($request->getSession(), $providerKey)) {
        return new RedirectResponse(($targetPath));
    }

    return new RedirectResponse($this->router->generate('app_homepage'));
}
```

Класс. Теперь работает как надо. На редиректит на тот урл, с которого мы попали на `/login`.

Давайте посмотрим, как управлять доступом из контроллеров.


## 15. Deny Access in the Controller

Установим доступы через контроллер. Закомментируем `access_control` в `security.yaml`. А в контроллере
`\App\Controller\CommentAdminController::index()` укажем `$this->denyAccessUnlessGranted('ROLE_ADMIN');`

Проверим. Да. Все так же видим `Access denied`.

#### IsGranted Annotation

Но это можно делать и через аннотации.

```php
use Sensio\Bundle\FrameworkExtraBundle\Configuration\IsGranted;
//...
class CommentAdminController extends Controller
{
    /**
     *  //...
     * @IsGranted("ROLE_ADMIN")
     */
    public function index(CommentRepository $repository, Request $request, PaginatorInterface $paginator)
    {
        //...
    }
}
```

Сообщение чуть другое, но смысл тот же `Access Denied by controller annotation @IsGranted("ROLE_ADMIN")`

#### Protecting an Entire Controller Class

Аннотацию можно указывать на весь класс. Это очень круто. То есть все роуты контроллера будут проверять это одно правило.



## 16. Dynamic Roles

Создадим новый контроллер для User Account.

```php bin/console make:controller```

Поправим экшн и шаблон для него. Просто почистим от лишнего.

#### Check if the User Is Logged In

Разлогинимся и зайдем на `/account`. Ага заходит. 
Как же нам проверить, что пользователь просто залогинен.

Самое простое - добавить `@IsGranted("ROLE_USER")`. Мы же точно знаем, что все пользователи имеют эту роль.


#### Adding Admin Users

```php
class UserFixture extends BaseFixture
{
    // ...
    protected function loadData(ObjectManager $manager)
    {
        $this->createMany(10, 'main_users', function($i) {
            // ...
        });
        
        $this->createMany(3, 'admin_users', function($i) {
            $user = new User();
            $user->setEmail(sprintf('admin%d@example.com', $i));
            $user->setFirstName($this->faker->firstName);
            $user->setPassword($this->passwordEncoder->encodePassword($user, 'engage'));
            $user->setRoles(['ROLE_ADMIN']);

            return $user;
        });
        
        $manager->flush();
    }
}
```

```php bin/console doctrine:fixtures:load```

Теперь можно залогиниться под `admin1@example.com` и убедиться, что `/admin/comment` разрешены для нашего пользователя.
Как и `/account` (так как `ROLE_USER` устанавливается всем)


#### Checking for Roles in Twig

Просто. Например в `base.html.twig`.

```twig
<ul class="navbar-nav ml-auto">
{% if is_granted('ROLE_USER') %}
    <li class="nav-item dropdown" style="margin-right: 75px;">
        <a class="nav-link dropdown-toggle" href="http://example.com" id="navbarDropdownMenuLink" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
          <img class="nav-profile-img rounded-circle" src="{{ '{{' }} asset('images/astronaut-profile.png') }}">
        </a>
        <div class="dropdown-menu" aria-labelledby="navbarDropdownMenuLink">
            <a class="dropdown-item" href="{{ '{{' }} path('app_account') }}">Profile</a>
            {% if is_granted('ROLE_ADMIN') %}
                <a class="dropdown-item" href="{{ '{{' }} path('admin_article_new') }}">Create Post</a>
            {% endif %}
            <a class="dropdown-item" href="{{ '{{' }} path('app_logout') }}">Logout</a>
        </div>
    </li>
{% else %}
    <li class="nav-item">
        <a style="color: #fff;" class="nav-link" href="{{ '{{' }} path('app_login') }}">Login</a>
    </li>
{% endif %}
</ul>
```

Обратите внимание, что ссылка на создание статьи доступна только ROLE_ADMIN. И в контроллере мы так же разрешили ее только им.



## 17. IS_AUTHENTICATED_ & Protecting All URLs

Есть 2 способа проверить, авторизован ли пользователь.
1. Проверить is_granted("ROLE_USER");
2. IS_AUTHENTICATED_FULLY

```yaml
security:
    access_control:
        - { path: ^/account, roles: IS_AUTHENTICATED_FULLY }
#        - { path: ^/account, roles: ROLE_USER } # или так
```

#### Web Debug Toolbar & Access Control Checks

Можно зайти в дебаг панель и посмотреть в раздел `Access decision log`. Там показано все, какие права у нас спрашивали на этой странице.
http://joxi.ru/krDOZnpTKMJ5eA

1 раз IS_AUTHENTICATED_FULLY, потом 2 раза ROLE_USER, и 1 раз ROLE_ADMIN (для показа ссылки на create article)


#### Requiring Login on Every Page

Если мы хотим, чтобы у нас спрашивали авторизацию на каждой странице.

```yaml
security:
    access_control:
        # if you wanted to force EVERY URL to be protected
        - { path: ^/, roles: IS_AUTHENTICATED_FULLY }
```

Попробуем разлогиниться. Сайт сломался. `Too many redirects`


#### Allowing the Login Page: IS_AUTHENTICATED_ANONYMOUSLY

Понятно, что, если мы не авторизованы, он пытается редиректить нас на страницу `login`, которая тоже требует быть авторизованным. FIX:

```yaml
security:
    access_control:
        # but, definitely allow /login to be accessible anonymously
        - { path: ^/login, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        # if you wanted to force EVERY URL to be protected
        - { path: ^/, roles: IS_AUTHENTICATED_FULLY }
```

**Замечание**: Так как это регулярка, можно указать жестко `path: ^/login$`, то есть не пропускать такие url `/login/foo`
(символ $ - окончание строки)

Так же стоит помнить, что access_control идет сверху и применяется только одно правило. Поэтому, когда применилось правило
для `/login` проверка будет завершена и дальше не будем проверять.
Собственно нам это и надо.

Работает.


#### IS_AUTHENTICATED_REMEMBERED

```yaml
security:
    access_control:
        # but, definitely allow /login to be accessible anonymously
        - { path: ^/login, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        # require the user to fully login to change password
        - { path: ^/change-password, roles: IS_AUTHENTICATED_FULLY }
        # if you wanted to force EVERY URL to be protected
        - { path: ^/, roles: IS_AUTHENTICATED_REMEMBERED }
```

IS_AUTHENTICATED_FULLY - означает именно в ЭТОЙ сессии. То есть этим защищаем чувствительные страницы, типа смены пароля.

IS_AUTHENTICATED_REMEMBERED - проверяет так же и REMEMBER_ME функциональность.

Всю эту функциональность можно использовать и в контроллере и в шаблоне Twig.

Так как нам все таки надо сделать сайт открытым, закомментим этот access_control в самом проекте.


## 18. Fetch the User Object

Мы можем проверять, есть ли у пользователя права. И вообще залогинен ли пользователь.

Как получить пользователя? В контроллере можно просто вызвать ```$this->getUser();```


#### Using the User Object

Мы можем залогинить email пользователя:

```php

public function index(LoggerInterface $logger)
{
    $logger->debug('Checking account page for '.$this->getUser()->getEmail());
    //...
}
```

#### Base Controller: Auto-complete $this->getUser()

Но у нас не будет работать auto-comlete в PhpStorm. Можно посмотреть в `\Symfony\Bundle\FrameworkBundle\Controller\ControllerTrait::getUser()`

```php
/**
 * Get a user from the Security Token Storage.
 *
 * @return mixed
 *
 * @throws \LogicException If SecurityBundle is not available
 *
 * @see TokenInterface::getUser()
 *
 * @final
 */
protected function getUser()
{
    if (!$this->container->has('security.token_storage')) {
        throw new \LogicException('The SecurityBundle is not registered in your application. Try running "composer require symfony/security-bundle".');
    }

    if (null === $token = $this->container->get('security.token_storage')->getToken()) {
        return;
    }

    if (!\is_object($user = $token->getUser())) {
        // e.g. anonymous authentication
        return;
    }

    return $user;
}
```

Симфони не знает, какой класс у нас отвечает за пользователя.

Создадим нас базовый контроллер:

```php
abstract class BaseController extends AbstractController
{
    protected function getUser(): User
    {
        return parent::getUser();
    }
}
```

И теперь наследуемся от нашего базового контроллера.
Теперь auto-complete работает.


#### Fetching the User in Twig

```twig
<h1>Manage Your Account {{ '{{' }} app.user.firstName }}</h1>
```

У twig'a в симфони есть глобальная переменная `app`. И через нее можно обращаться к разным другим переменным. Типа `app.user` или `app.session`.


#### Making the Account Page Pretty

Сделали страницу красивой.

Стоит обратить внимание на аватар. https://robohash.org/{{ '{{' }} app.user.email }}


## 19. Custom User Method

Создадим новое поле у User.  `twitterUsername` `Nullable=true`

```bash
sf make:entity
sf make:migration
sf doctrine:migrations:migrate
```
И в фикстуры его тоже добавим.

```php
if ($this->faker->boolean) {
    $user->setTwitterUsername($this->faker->userName);
}
```

```php bin/console doctrine:fixtures:load```

И в шаблоне добавим условие

```twig
{% if app.user.twitterUsername %}
    <h4 class="white"><i class="fa fa-twitter"></i> {{ '{{' }} app.user.twitterUsername }}</h4>
{% endif %}
```

Можно проверить для разных пользователей.


#### Custom User Method for RoboHash

Добавили в иконку в шапке. Но. Хотим, это немного улучшить.
Создадим метод у User:

```php
public function getAvatarUrl(string $size = null): string
{
    $url = 'https://robohash.org/'.$this->getEmail();
    if ($size) {
        $url .= sprintf('?size=%dx%d', $size, $size);
    }
        
    return $url;
}
```

И используем его в шаблонах. Получим такие ссылки `https://robohash.org/admin0@example.com?size=100x100` `https://robohash.org/admin0@example.com`


## 20. Fetching the User In a Service

Мы знаем, как получить текущего пользователя в шаблоне (`{{ '{{' }} app.user }}`). И в контроллере `$this->getUser()`.

Но как его получить в сервисах? Например, в нашем `MarkdownHelper` есть метод логирования, когда в url есть слово bacon.
И мы хотим залогировать, какой именно юзер вызвал этот метод.

#### The Security Service

`\Symfony\Component\Security\Core\Security $security`

Он имеет 3 публичных метода. getToken(), getUser(), isGranted(). Похоже, то, что надо.

```php
$this->logger->info(
    'They are talking about bacon again!',
    [
        'user' => $this->security->getUser(),
    ]
);
```


## 21. Role Hierarchy

Часто достаточно 2х типов пользователей. Admin and User.
Но есть надо организовать сложную систему, лучше использовать Role hierarchy.

#### Role Naming

Как же правильно сделать? Можно называть роли по типу пользователя. Например, `ROLE_EDITOR`, `ROLE_HUMAN_RESOURCES` or `ROLE_THE_PERSON_THAT_OWNS_THE_COMPANY`
Но это не очень удобно. Непонятно, что какие на самом деле права имеет `ROLE_EDITOR`.

Лучше называть роли специализировано. Что дает эта роль. `ROLE_ADMIN_ARTICLE` и `ROLE_ADMIN_COMMENT`. Для этих контроллеров `ArticleAdminController` и `CommentAdminController`.


#### role_hierarchy

Все классно. Но теперь я, как админ, не имею прав на эту ROLE. Да. Можно добавить всем админам в базе эти роли. Но это не очень удобно.
При появлении новой роли, придется каждый раз дергать базу.

Конечно решение простое)):

```yaml
security:
    role_hierarchy:
        ROLE_ADMIN: [ROLE_ADMIN_COMMENT, ROLE_ADMIN_ARTICLE]
```

Собственно так же можно создать роль `ROLE_EDITOR`, пользователю, который может только редактировать ну и тп.


## 22. Impersonation (switch_user)

Это крутая фишка для быстрого дебага. Когда можно быстро переключаться между юзерами. Не разлогиниваясь.

```yaml
security:
    firewalls:
        main:
            switch_user: true
```

Но надо дать на это права только тем пользователям, которым это можно (`ROLE_ALLOWED_TO_SWITCH`)

```yaml
security:
    role_hierarchy:
        ROLE_ADMIN: [ROLE_ADMIN_COMMENT, ROLE_ADMIN_ARTICLE, ROLE_ALLOWED_TO_SWITCH]
```

После этого можно в url добавлять `?_switch_user=spacebar1@example.com`. И мы будем авторизованны под этим пользователем.


#### User Provider & _switch_user

Кстати, да. В _switch_user надо передавать email. Потому что у нас так настроен наш UserProvider:

```yaml
security:
    providers:
        app_user_provider:
            entity:
                class: App\Entity\User
                property: email
```

Этот провайдер используется для того, чтобы получать данные из сессии. Но и так же таким функционалом как remember_me and switch_user.
Если там поменяем на id, то и передавать в switch_user надо будет id.


#### Adding a Banner when you are Impersonating

`?_switch_user=_exit` - для выхода.

Есть такая фигня, что мы можем быть авторизованы под своим пользователем. Потом сделали switch_user. Забыли про это и начали что-то делать из-под другого пользователя.
Стоит напоминать о том, что мы в этом режиме. Добавим в base.html.twig большой баннер об этом.

```
{{ '{%' }} if is_granted('ROLE_PREVIOUS_ADMIN') %}
    <div class="alert alert-warning" style="margin-bottom: 0;">
        You are currently switched to this user.
        <a href="{{ '{{' }} path('app_homepage', {'_switch_user': '_exit'}) }}">Exit Impersonation</a>
    </div>
{{ '{%' }} endif %}
```

Класс!!


## 23. Serializer & API Endpoint

#### Creating the API Endpoint

Добавим в `AccountController` метод для API.

```php
/**
 * @Route("/api/account", name="api_account")
 */
public function accountApi()
{
    $user = $this->getUser();
    
    return $this->json($user);
}
```

ИИИ. По url `/api/account` видим пустоту. Почему? Классный метод контроллера `$this->json($data)` работает так. 
Если есть serializer, то он сериализует $data при помощи его.
Если нет. То передает объект в JsonResponse. Там смотрят public свойства, и их отдают. У нашего User нет публичных свойств. 
А serializer мы не ставили.
Ставим.

```composer require serializer```


#### Serialization Groups

Блин, мы видим все поля. Надо решить это с помощью `use Symfony\Component\Serializer\Annotation\Groups;` в классе `User`.

```php
/**
 * @Groups("main")
 */
private $email;
```

А в контроллере

```php
return $this->json($user, 200, [], [
    'groups' => ['main'],
]);
```

Класс. Теперь можно и к API token приступить.



## 24. API Auth: Do you Need it? And its Parts

Перед тем как кодить, надо разобраться с разными вещами. Люди склонны все усложнять. Но тут все проще.

#### Do you Need API Authentication

Для начала задайте вопрос, вам нужна система аутентификации через API token?

Если у вас есть api endpoints, но вы грузите js c вашего сайта и используете его только на этом сайте. То нет. Вам не нужен API token auth.

Достаточно только LoginFormAuthenticator.

Можно сделать красивые LoginForm с AJAX. LoginFormAuthenticator останется прежним. Ну только за исключением, что вам надо будет слать
success json вместо redirect при успешной аутентификации.
Ну и надо переопределить методы `onAuthenticationError()` и `start()`, для того же. Позже об этом тоже будет.

Конечно, даже если ваш JS - единственный, кто пользует ваш API, вы все равно можете захотеть создать систему API token auth.
Возможно для каких-то других вещей.


#### Two Sides of API Token Authentication

Если все таки остановились на этом. Есть еще 2 несвязанные части.
1. Как приложение обрабатывает api-token'ы и регистрирует пользователя.
2. Как создаются и дистрибьютятся токены.

#### Part 1: Processing an API Token

Token - это просто строка, которая связана с User'ом. Не важно как ее создали. Клиент в заголовке отправляет эту строку
и становится авторизованным, как этот пользователь. Существуют разные токены, для scopes and permissions. Но это основная идея.

То, как токен связан с пользователем можно реализовать разными путями. Например, хранить токен в таблице User. Мы так и сделаем.

Другой вариат - это JSON web token. В этом случае токен - это какая-то зашифрованная подпись. И приложение расшифровывает подписть и получает идентификатор.

В любом случае - первая часть - это прочитать токен из заголовка, и как-то получить по нему пользователя.

#### Part 2: Creating & Distributing API Tokens

Как создавать и распространять токены.

Есть 3 примера, как это делать и GITHUB использует все 3.

Первый, позволять создать токен через веб интерфейс. Пользователь просто вручную создает себе токены и сохраняет их в своем аккаунте.

Второй, у вас есть API endpoint, который создает токен. Например, вы отправляете на него логин/пасс. А он создает и возвращает токен.
Минус этого подхода, что его нельзя использовать с third-party apps. Так как пользователю придется вводить свой пароль от вашего приложения в другом приложении.
Это очень небезопасно.

И наконец, третье. OAuth2. Единственный минус - это просто сложнее.

Вообще, это вторая часть - это не часть аутентификации. Это именно про использование.

Мы построим как раз первую часть. Истинную часть аутентификация.


## 25. ApiToken Entity

Создадим сущность ApiToken.

```
ApiToken
token string 255 no nullable
expiredAt datetime no nullable
user relation User ManyToOne no nullable add field to User and yes for orphanRemoval
```

Создадим и выполним миграции.


#### How are Tokens Created?

Да как угодно.
Сейчас создадим через фикстуры.


#### Making the ApiToken Class Awesome

Так как для каждого токена нужен User. Добавим User в `ApiToken::__construct()`

```php
public function __construct(User $user)
{
    $this->token = bin2hex(random_bytes(60));
    $this->user = $user;
    $this->expiredAt = new \DateTime('+1 day');
}
```
    
И удалим все сеттеры. То есть сделали класс Immatable.

Если мы захотим изменять expiredAt время. В будущем можно добавить методы `setExpiredAt(\DateTime $date)`
или лучше его назвать `renewExpiredAt()`.


#### Adding ApiTokens to the Fixtures

Вообще, можно было создать фикстуру ApiTokenFixture. Но мы просто добавим по 2 токена в фикстуры UserFixtures.

```php
$apiToken1 = new ApiToken($user);
$manager->persist($apiToken1);
```

Загрузим фикстуру и проверим базу:

```bash
php bin/console doctrine:fixtures:load
php bin/console doctrine:query:sql 'SELECT * FROM api_token'
```



## 26. Entry Point: Helping Users Authenticate

#### make:auth ApiTokenAuthenticator

Создадим второй аутентикатор `ApiTokenAuthenticator`

```bash
php bin/console make:auth
```

Пока просто пустой класс. Добавим его в `config/packages/security.yaml`

```yaml
security:
    firewalls:
        main:
            guard:
                authenticators:
                    - App\Security\LoginFormAuthenticator
                    - App\Security\ApiTokenAuthenticator
```

Мы сделаем этот аутентикатор supports каждый запрос.
Но сейчас мы получили ошибку

```
Because you have multiple guard configurators, you need to set the "guard.entry_point" key 
to one of your configurators (App\Security\LoginFormAuthenticator, App\Security\ApiTokenAuthenticator)
```

Так как у нас получилось несколько аутентикаторов, надо установить ключ "guard.entry_point" для одного из аутентикаторов.


#### What is an Entry Point?

Вообще есть новая версия `make:auth`. Которая заботится об этом параметре. Но нам надо понимать, что это.
Добавим

```yaml

security:
    firewalls:
        main:
            guard:
                entry_point: App\Security\LoginFormAuthenticator
```

Наш firewall должен иметь один entry_point. Его задача - определить, что надо делать, когда анонимный пользователь 
пытается получить доступ к защищенной странице. Если пользователь не анонимный, мы проверяем права. 
А когда анонимный? Например, сейчас мы при попытке зайти на `/admin/comments` редиректились на `/login`.

Но где этот код живет. Да именно в нашем LoginFormAuthenticator. 
Зайдем в него, перейдем на родителя AbstractFormLoginAuthenticator. У каждого аутентикатора есть метод start().
Это он и есть. Выглядит так:

```php
public function start(Request $request, AuthenticationException $authException = null)
{
    $url = $this->getLoginUrl();

    return new RedirectResponse($url);
}
```

ААА. Поэтому нам и нужен только один entry_point. Ведь когда пользователь анонимный, мы не знаем какой аутентикатор использовать
и соответственно не знаем откуда брать User'a и Credentials.

Сейчас это простой редирект. Но мы можем сделать этот метод умнее. Например, когда роут является api мы может отдавать какой-то json.


## 27. API Token Authenticator

Будем использовать Postman для тестирования api. Вообще, лучше использовать функциональные тесты. Но это отдельная тема.

Будем отправлят GET запрос на `http://localhost:8000/api/account`.

Как мы будем отправлять токен? Вообще можно как угодно, но обычным стандартом является отправка в HEADER'e.
У Postman выберем Autorization Bearer Token. И вставим наш реальный токен пользователя из базы.

```php bin/console doctrine:query:sql 'SELECT * FROM api_token'```


#### Authorization: Bearer

Обновим токен и увидим, что в HEADERS появился параметр `Authorization: Bearer T_O_K_E_N`

Bearer - значит носит. Это просто принятый стандрарт. Профессионалов. Access token - это тоже bearer token.
Хотя мы можем придумывать свои стандарты.

#### supports()

```php
public function supports(Request $request)
{
    return $request->headers->has('Authorization')
        && 0 === strpos($request->headers->get('Authorization'), 'Bearer');
}
```

#### getCredentials()

```php
public function getCredentials(Request $request)
{
    $authorizationHeader = $request->headers->get('Authorization');

    return substr($authorizationHeader, 7); // skip "Bearer "
}

public function getUser($credentials, UserProviderInterface $userProvider)
{
    dd($credentials);
}
```

В LoginFormAuthenticator переменная $credentials была массивом. В нашем новом ApiTokenAuthenticator это строка.
Ничего страшного. Интерфейс не определяет тип этой переменной. А получается она от нашего же метода getCredentials().
Поэтому аутентикатор сам решает, какой тип Credentials ему нужен. Мы можем так же под это и объект создать.

#### getUser()

Заинжектим в аутентикатор `ApiTokenRepository $apiTokenRepo`

```php
public function __construct(ApiTokenRepository $apiTokenRepo)
{
    $this->apiTokenRepo = $apiTokenRepo;
}

//...

public function getUser($credentials, UserProviderInterface $userProvider)
{
    $token = $this->apiTokenRepo->findOneBy(['token' => $credentials]);
    
    if (!$token) {
        return null;
    }
    
    return $token->getUser();
}

public function checkCredentials($credentials, UserInterface $user)
{
    dd('checking credentials');
}
```

Но пока мы не проверяем, не протух ли токен. И надо подумать, как отдавать error response.


## 28. API Token Authenticator Part 2!

Что же будет, когда токен неверный?
Попробуем. Нас редиректит на /login.

За обработку отвечает onAuthenticationFailure(). Сейчас он пустой. То есть ничего не происходит и систему аутентификации продолжает работать по своей логике.
Проверяет права, а так как мы анонимны - нас отправляет на entry_point - /login.


#### onAuthenticationError()

```php
public function onAuthenticationFailure(Request $request, AuthenticationException $exception)
{
    return new JsonResponse([
        'message' => $exception->getMessageKey(),
    ], 401);
}
```

#### Custom Error Messages with CustomUserMessageAuthenticationException

Класс!! Теперь мы видим сообщение об ошибке. Но оно немного странное `Username could not be found`

Когда мы делали LoginAuthenticator - мы просто использовали translator. См файл `translations/security.en.yaml`.
В принципе можно сделать также. Но мы пойдем другим путем.
Как еще можно кастомизировать сообщение об ошибке?

Просто кинем наше кастомное исключение

```php
public function getUser($credentials, UserProviderInterface $userProvider)
{
    $token = $this->apiTokenRepo->findOneBy(['token' => $credentials]);

    if (!$token) {
        throw new CustomUserMessageAuthenticationException('Invalid API token');
    }

    return $token->getUser();
}
```

#### Checking Token Expiration

Теперь проверим, не протух ли токен? Добавим у токена метод `isExpired()`

```php
public function isExpired()
{
    return $this->getExpiredAt() <= new \DateTime();
}
```

Честно, я бы этого не делал в ApiToken. А проверял бы в аутентикаторе.

```php
public function getUser($credentials, UserProviderInterface $userProvider)
{
    $token = $this->apiTokenRepo->findOneBy(['token' => $credentials]);

    if (!$token) {
        throw new CustomUserMessageAuthenticationException('Invalid API token');
    }

    if ($token->isExpired()) {
        throw new CustomUserMessageAuthenticationException('Token Expired');
    }

    return $token->getUser();
}
```

Почему мы сделали это в getUser  а не в checkCredintials()? Нет причин. Можно было и там. Но тогда нам надо было бы опять доставать токен. А в getUser он уже есть.

В checkCredentials() просто return true;

#### onAuthenticationSuccess()

Хм. На LoginForm  мы редиректили. А тут? Ничего. Просто продолжаем жизнь запроса.


#### start() & supportsRememberMe()

Так как у нас entry_point - это LoginFormAuth. То на это аутентикаторе start() не будет вызван никогда. Кинем там исклчение

```php
throw new \Exception("Not used: entry_point from other authenticator is used.");
```

supportsRememberMe() return false; Так как если да, то система будут следить за чекбоксом _remember_me . Но это бессмысленно.

Отлично. Теперь у нас есть 2 способа аутентификации. А дальше мы просто работаем с пользователем, и не важно, каким способом он зарегался.


## 29. Создание регистрации

#### Создание формы регистрации

```php
class SecurityController extends AbstractController
{
    /**
     * @Route("/register", name="app_register")
     */
    public function register()
    {
        return $this->render('security/register.html.twig');
    }
}
```

Для шаблона login.html.twig. Переименуем все. И создадим ссылку на register.


#### Обработка отправки формы

```php
public function register(Request $request, UserPasswordEncoderInterface $passwordEncoder)
{
    // TODO - use Symfony forms & validation
    if ($request->isMethod('POST')) {
        $user = new User();
        $user->setEmail($request->request->get('email'));
        $user->setFirstName('Mystery');
        $user->setPassword($passwordEncoder->encodePassword(
            $user,
            $request->request->get('password')
        ));
        
        $em = $this->getDoctrine()->getManager();
        $em->persist($user);
        $em->flush();

        return $this->redirectToRoute('app_account');
    }
    
    return $this->render('security/register.html.twig');
}
```

Класс! После этого мы регистрируемся. 

#### Авторизация сразу после регистрации

Но после авторизации остаемся анонимными. Надо сразу авторизовываться.

Но тут есть проблема. Представьте, у нас интернет магазин и мы добавляем в корзину товары. Но на /checkout нас требуют залогинится.
И нас редиректит на /login. Там мы узнаем, что мы не зарегистрированы и отправляемся на /register. После успешной регистрации, нас авторизует.
Но куда нас редиректнет после этого?

/checkout -> /login -> /register -> ??? (/checkout)


```php
/**
 * @Route("/register", name="app_register")
 */
public function register(
    Request $request,
    UserPasswordEncoderInterface $passwordEncoder,
    GuardAuthenticatorHandler $guardHandler,
    LoginFormAuthenticator $formAuthenticator
) {
    // TODO - use Symfony forms & validation
    if ($request->isMethod('POST')) {
        $user = new User();
        $user->setEmail($request->request->get('email'));
        $user->setFirstName('Mystery');
        $user->setPassword(
            $passwordEncoder->encodePassword(
                $user,
                $request->request->get('password')
            )
        );

        $em = $this->getDoctrine()->getManager();
        $em->persist($user);
        $em->flush();

        return $guardHandler->authenticateUserAndHandleSuccess(
            $user,
            $request,
            $formAuthenticator,
            'main'
        );
    }

    return $this->render('security/register.html.twig');
}
```


## 30. Связь автора и статей (один-ко-многим)

Сейчас автор - это строка. А надо сделать, чтобы это была связь в базе данных с пользователями.
Для чего? Во-первых, это правильно. Во-вторых, потом покажем про AccessDenied по этому полю.

Удалим private $author; У Article. Создадим и выполним миграции.

#### Добавим связь

Проще через `sf make:entity`. Главное, что не может быть null.

Создадим еще одну миграцию. 

ИИИИ. Ошибка миграции. Не может быть NULL.

Что ж. Придется снести базу.

```php
sf doctrine:schema:drop --full-database --force
sf doc:mi:mi
```

#### Пофиксим фикстуры

```php
$article->setAuthor($this->getRandomReference('main_users'));
```

И не забудьте зависимости фикстур. `implements DependentFixtureInterface`


## 31. Админка статей и низкоуровневый контроль доступа

У нас тут ошибка `Catchable Fatal Error: Object of class Proxies\__CG__\App\Entity\User could not be converted to string`

Ну у нас изменился метод getAuthor(). Раньше отдавал string, теперь User. Добавим у User метод __toString()

```php
public function __toString()
{
    return (string) $this->getFirstName();
}
```

#### Добавим страницу редактирования

```php
class ArticleAdminController extends AbstractController
{
    /**
     * @Route("/admin/article/{id}/edit")
     */
    public function edit(Article $article)
    {
    }
}
```

Провирим, какие у нас есть id статей. 

```bash
php bin/console doctrine:query:sql 'SELECT * FROM article'
```

Перейдем сюда `/admin/article/9/edit`. Попросит авторизоваться. Ок. Видим страницу редактирования.


#### План на контроль доступа

Мы должны давать права на данную страницу если у пользователя права ROLE_ADMIN_ARTICLE или если он является автором данной статьи.

#### Ручное ограничение прав

Уберем `@IsGranted("ROLE_ADMIN_ARTICLE")` и установим его только для `new()`.

А для edit сделаем:

```php
/**
 * @Route("/admin/article/{id}/edit")
 */
public function edit(Article $article)
{
    if ($article->getAuthor() != $this->getUser() && !$this->isGranted('ROLE_ADMIN_ARTICLE')) {
        throw $this->createAccessDeniedException('No access!');
    }

    dd($article);
}
```

Теперь, если мы авторизуемся под простым пользователем, то там будет проверка, является ли он автором данной статьи.

Но это не очень прикольно. Писать эту логику в контроллере. Придется дублировать код для других экшнов.
И в шаблоне, если надо ограничить доступ на показ кнопки.
Надо сделать лучше.


## 32. Voters

Заменим в контроллере

```php
// if ($article->getAuthor() != $this->getUser() && !$this->isGranted('ROLE_ADMIN_ARTICLE')) {
if (!$this->isGranted('MANAGE', $article)) {
```

Обратите внимание, что мы передаем и $article.

И мы видим, что Access Denied.


#### Объяснение Voter system

Когда вызывается isGranted() denyAccessUnlessGranted(), (Как и в аннотациях) Симфони спрашивает каждый Воутер, он умеет решать,
как обрабатывать строку `ROLE_ADMIN_ARTICLE`, `MANAGE` ?

В симфони есть 2 воутера, `RoleVoter` и `AuthenticatedVoter`.

Когда передается любая строка начинающаяся с ROLE_, их сразу хватает RoleVoter.
Он проверяет, есть ли у пользователя эта роль. Другие воутеры игнорятся ("abstains"). То есть решает только этот.

Когда передается `IS_AUTHENTICATED_` например `IS_AUTHENTICATED_FULLY`, `IS_AUTHENTICATED_ANONYMOUSLY`, `IS_AUTHENTICATED_REMEMBERED`.
За них берется `AuthenticatedVoter`.


#### Добавление собственного Воутера

Когда мы отправили MANAGE. Ни один из дефолтных Воутеров не подхватил это.

На самом деле MANAGE - это permission attribute. Вообще можно писать все что угодно. Например, EDIT, SHOW, DELETE.
И наш воутер должен проверять этот атрибут и объект, который мы передаем. В нашем случае - Article. 


## 33. Добавиление собственного воутера

```sh
php bin/console make:voter
```

ArticleVoter

#### supports()

```php
class ArticleVoter extends Voter
{
    protected function supports($attribute, $subject)
    {
        return 'MANAGE' === $attribute
            && $subject instanceof Article;
    }
    
    protected function voteOnAttribute($attribute, $subject, TokenInterface $token)
    {
        /** @var Article $subject */
        $user = $token->getUser();

        // if the user is anonymous, do not grant access
        if (!$user instanceof UserInterface) {
            return false;
        }

        switch ($attribute) {
            case 'MANAGE':
                if ($subject->getAuthor() == $user) {
                    return true;
                }
                
                if ($this->security->isGranted('ROLE_ADMIN_ARTICLE')) {
                    return true;
                }
                
                return false;
        }
    }
}
```


#### Проверка роли в воутере

```php
use Symfony\Component\Security\Core\Security;

private $security;

public function __construct(Security $security)
{
    $this->security = $security;
}

```

#### @IsGranted с объектами

```php
class ArticleAdminController extends AbstractController
{
    public function edit(Article $article)
    {
        $this->denyAccessUnlessGranted('MANAGE', $article);
        
        dd($article);
    }
}
```

Но можно просто добавить аннотацию

```php
     * @IsGranted("MANAGE", subject="article")
```

Ну вот и все про Security!!!