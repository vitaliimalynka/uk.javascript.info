# Файли cookies, document.cookie

Файли cookies -- це невеликі рядки з даними, які зберігаються безпосередньо в браузері. Вони є частиною HTTP-протоколу, визначеного специфікацією [RFC 6265](https://tools.ietf.org/html/rfc6265).

Файли cookies зазвичай встановлюються вебсервером за допомогою HTTP-заголовка `Set-Cookie`. Потім браузер автоматично додаватиме їх при (майже) кожному запиті до відповідного домену використовуючи HTTP-заголовок `Cookie`.

Одним з найбільш поширених випадків використання є аутентифікація:

1. При вході в систему, сервер використовує відповідь отриману з  HTTP-заголовка `Set-Cookie`, щоб додати в файл cookie унікальний "ідентифікатор сесії" .
2. Наступного разу, коли на той самий домен буде відправлено запит, браузер надішле файл cookie використовуючи HTTP-заголовок `Cookie`.
3. Таким чином, сервер знає, хто зробив запит.

Також ми маємо доступ до файлів cookies з браузера, використовуючи властивість `document.cookie`.

Файли cookies мають багато тонкощів та особливостей. В цьому розділі ми детально їх розглянемо.

## Зчитування з document.cookie

```online
Чи зберігає браузер файли cookie з цього сайту? Перевіримо:
```

```offline
Уявімо, що ви зайшли на вебсайт, наступний код дає можливість побачити його файли cookies:
```

```js run
// На javascript.info, ми використовуємо Google Analytics для збору статистики,
// тому тут мають бути деякі файли cookies
alert( document.cookie ); // cookie1=value1; cookie2=value2;...
```


Значення `document.cookie` складається з пар `name=value` розділених `; `. Кожна пара -- це окремий файл cookie.

Щоб знайти певний файл cookie потрібно розділити значення `document.cookie` за допомогою `; `, а потім знайти потрібний ключ за його назвою.  Для цього можна використовувати як регулярні вирази так і функції для роботи з масивами.

Залишимо це завдання для самостійної роботи читача. Окрім того, в кінці розділу ви знайдете допоміжні функції для роботи з файлами cookies.

## Запис в document.cookie

Ми можемо записувати в `document.cookie`. Але це не просто властивості даних, це [аксесори (гетери/сетери)](info:property-accessors). Присвоєння цих властивостей обробляється особливим чином.

**Запис в `document.cookie` оновлює лише вказані файли cookies, та не чіпатиме решту.**

Наприклад, цей виклик встановить файл cookie з іменем `user` та значенням `John`:

```js run
document.cookie = "user=John"; // оновити лише файл cookie під назвою 'user'
alert(document.cookie); // показати всі файли cookies
```

Якщо ви запустите код вище, то напевно побачите декілька файлів cookies. Це тому що операція `document.cookie=` оновлює не всі файли cookies, а лише вказані файли з іменем `user`.

Технічно, ім’я та значення можуть містити будь-які символи. Щоб зберегти правильне форматування, вони повинні бути кодовані за допомогою вбудованої функції `encodeURIComponent`.

```js run
// спеціальні символи (пробіли, кирилиця), потребують кодування
let name = "my name";
let value = "John Smith";

// кодується як my%20name=John%20Smith
document.cookie = encodeURIComponent(name) + '=' + encodeURIComponent(value);

alert(document.cookie); // ...; my%20name=John%20Smith
```


```warn header="Обмеження"
Є декілька обмежень:
- Пара `name=value`, після кодування `encodeURIComponent`, не повинна перевищувати 4KB. Тому ми не можемо зберігати великий обʼєм даних у файлах cookie.
- Дозволена сумарна кількість файлів cookie на один домен приблизно 20+, точний ліміт залежить від браузера.
```

Файли cookies мають ряд параметрів, деякі з них важливі і повинні бути задані.

Параметри перераховуються після пари `key=value` та розділяються `;`, як показано нижче:

```js run
document.cookie = "user=John; path=/; expires=Tue, 19 Jan 2038 03:14:07 GMT"
```

## path

- **`path=/mypath`**

URL-префікс адреси повинен бути абсолютним. Для того щоб файли cookie були доступними зі сторінок за цією адресою. За замовчуванням, це поточна сторінка.

Якщо в файлі cookie задано `path=/admin`, то він видимий на сторінках `/admin` та `/admin/something`, але не на `/home` або `/adminpage`.

Зазвичай, в якості `path` вказують кореневу сторінку: `path=/` для того щоб зробити файли cookie доступними з будь-якої сторінки вебсайту.

## domain

- **`domain=site.com`**

Домен визначає звідки будуть доступні файли cookie. Проте, на практиці, існують деякі обмеження -- ми не можемо вказати будь-який домен.

**Не існує способу зробити файли cookie доступними з іншого домену 2-го рівня, тому `other.com` ніколи не отримає файл cookie заданий на `site.com`.**

Це обмеження задля безпеки, воно дозволяє нам зберігати конфіденційні дані в файлах cookie, які будуть доступними лише на одному сайті.

За замовчуванням, файли cookie доступні лише на тому домені на якому були встановлені.

Зверніть увагу, що за замовчуванням файли cookie також не доступні і на піддомені, такому як `forum.site.com`.

```js
// якщо задати файл cookie на вебсайті site.com ...
document.cookie = "user=John"

// ...ми не побачимо їх на forum.site.com
alert(document.cookie); // користувача немає
```

...Проте цю поведінку можна змінити. Якщо ми хочемо дозволити піддоменам на кшталт `forum.site.com` отримувати файли cookie задані на `site.com` -- це також можливо.

Для цього потрібно, задаючи файл cookie на `site.com`, явно вказати в `domain` кореневий домен: `domain=site.com`. Тоді файли cookie будуть видимі з усіх піддоменів.

Наприклад:

```js
// на site.com
// зробити файл cookie доступним на будь-якому піддомені *.site.com:
document.cookie = "user=John; *!*domain=site.com*/!*"

// пізніше

// на forum.site.com
alert(document.cookie); // файл cookie user=John існує
```

Історично склалося так що, `domain=.site.com`(з крапкою перед `site.com`) спрацює так само, надаючи доступ до файлів cookie з піддоменів. Цей застарілий синтаксис варто використовувати лише якщо потрібна підтримка дуже старих браузерів.

Отже, параметр `domain` робить файли cookie доступними на піддоменах.

## expires, max-age

За замовчуванням, якщо файл cookie не має одного з цих параметрів, то файл зникає при закриванні браузера. Такі файли cookies називаються сесійними ("session cookies").

Щоб файли cookies могли "пережити" закривання браузера, можна встановити значення одного з параметрів `expires` або `max-age`.

- **`expires=Tue, 19 Jan 2038 03:14:07 GMT`**

Термін придатності файлу cookie визначає час, коли браузер автоматично видалить його.

Дата має бути вказана саме в такому форматі, в часовому поясі GMT. Щоб отримати правильну дату, можна скористатися `date.toUTCString`. Наприклад, можемо встановити, що термін придатності файлу cookie закінчується через 1 день:

```js
// +1 день від сьогодні
let date = new Date(Date.now() + 86400e3);
date = date.toUTCString();
document.cookie = "user=John; expires=" + date;
```

Якщо встановити  значенням параметру `expires` дату з минулого, то файл cookie видалиться.

-  **`max-age=3600`**

Це альтернатива `expires`, який вказує термін придатності cookie в секундах з поточного моменту.

Як встановлено 0 або від’ємне значення, то файл cookie буде видалено:

```js
// файл cookie буде видалено через 1 годину
document.cookie = "user=John; max-age=3600";

// видалити файл cookie (термін придатності закінчується прямо зараз)
document.cookie = "user=John; max-age=0";
```

## secure

- **`secure`**

Файл cookie повинен передаватися виключно по HTTPS-протоколу.

**За замовчуванням, якщо ми створимо файл cookie на `http://site.com`, тоді він автоматично з’явиться на `https://site.com` та навпаки.**

Тобто файли cookie базуються на домені, вони не залежать від протоколів.

В разі якщо цей параметр задано, то файл cookie створений на `https://site.com` не буде доступний на тому ж вебсайті з HTTP-протоколом `http://site.com`. Тому якщо файли cookie містять конфіденційні дані, які в жодному разі не мають бути відправлені по незашифрованому HTTP-протоколу, тоді параметр `secure` це правильнйи вибір.

```js
// припустимо, що зараз ми на https://
// створимо захищений файл cookie (доступний лише по HTTPS)
document.cookie = "user=John; secure";
```

## samesite

Це ще один атрибут безпеки. Він створений щоб захищати від так званих XSRF-атак (міжсайтова підміна запиту).

Щоб зрозуміти як він працює та в яких випадках може бути корисним, давайте детальніше розглянемо поняття XSRF-атак.

### XSRF-атаки

Уявіть, що ви увійшли в свій обліковий запис на сайті `bank.com`. Тобто: у вас є файли cookie з даними аутентифікації. Ваш браузер відправляє їх вебсайту `bank.com` при кожному запиті для того щоб той розпізнав вас та виконав всі конфіденційні фінансові операції.

Тепер, переглядаючи вебсторінки в іншому вікні, ви випадково потрапили на сайт `evil.com`. Цей сайт містить код JavaScript, який відправляє форму `<form action="https://bank.com/pay">` на `bank.com` з заповненими полями, які ініціюють транзакцію на рахунок хакера.

Браузер відправляє файл cookies щоразу як ви відвідуєте сайт `bank.com`, навіть якщо форма була відпралена з `evil.com`. Тому банк "впізнає" вас та дійсно виконає платіж.

![](cookie-xsrf.svg)

Така атака називається міжсайтова підміна запиту (Cross-Site Request Forgery, XSRF).

Звісно, справжні банкові системи захищені від цього. У всіх згенерованих сайтом `bank.com` формах є спеціальне поле, так взваний "токен захисту від XSRF", який не може бути ані згенерований зловмисним сайтом, ані вилучений з віддаленої сторінки. Зловмисник може спробувати відправити форму туди, але не може отримати дані назад. Сайт `bank.com` перевіряє кожну отриману форму на наявність даного токену.

Проте такий захист вимагає більше часу на реалізацію. Нам потрібно переконатися, що кожна форма несе в собі обов’язкове поле з токеном, окрім того потрібно перевіряти всі запити.

### Параметр samesite

Параметр `samesite` пропонує ще один спосіб захисту від атак, який (в теорії) не вимагає "токен захисту від xsrf".

В нього є два можливі значення:

- **`samesite=strict` (теж саме що `samesite` без заданого значення)**

Файли cookie з параметром `samesite=strict` ніколи не відправляються якщо корстувач прийшов ззовні (з іншого сайту).

Іншими словами, якщо користувач перейшов за посиланням зі свого електронного листа або відправим форму з `evil.com`, або виконав яку завгодно операцію, що походить з іншого домену, файли cookie не відправляться.

Якщо авторизаційний файл cookie має параметр `samesite`, тоді у XSRF-атаки немає шансу на успіх, тому що запит з сайту `evil.com` надходить без файлу cookie. Таким чином, `bank.com` не ідентифікує користувача та не здійснить платіж.

Захист доволі надійний. Файли cookie з параметром `samesite` відправлятимуться тільки якщо операції надходять з `bank.com`, наприклад відправка форми з іншої сторінки домену `bank.com`.

Хоча, у цього рішення є дрібний недолік.

Коли користувач переходить за легітимним посилання до `bank.com`, як то зі своїх нотаток, то він буде неприємно здивований коли `bank.com` не "впізнає" його. Дійсно, в такому випадку файл cookie з парамером `samesite=strict` не відправляється.

Цю проблему можна вирішити використанням двох файлів cookie: один для загальної ідентифакації, лише для того щоб сказати: "Привіт, Іван", а інший з параметром `samesite=strict` для операцій з даними. Тоді особа, яка перейшла за посиланням поза межами сайту побачить привітання, проте для того щоб другий файл cookie відправився, платіжні операції повинні бути ініційовані з сайту банку.

- **`samesite=lax`**

Спрощене рішення яке так само захищає від XSRF-атак, але не руйнує очікування користувача.

Режим `lax`, так само як `strict`, забороняє браузеру відправляти файл cookies коли запит приходить не з сайту, але є одне виключення.

Файл cookie з параметром `samesite=lax` відправляється лише тоді, коли виконуються обидві умови:
1. Обраний HTTP-метод безпечний (наприклад GET, але не POST)

    Повний список безпечних HTTP-методів зібрано в [специфікацї RFC7231](https://tools.ietf.org/html/rfc7231). По суті, безпечними вважаються методи які використовуються для читаня, а не для запису даних. Вони не повинні виконувати жодних операції по редагуванню даних. Перехід за посиланням це завжди метод GET, тобто безпечний.

2. Операція виконує навігацію вищого рівня (змінює URL в адресному полі браузера)

    Як правило, ця умова виконуєтся, проте якщо навігація відбувається в `<iframe>` то це не вищий рівень. Також, методи JavaScript для мережевих запитів не виконують навігації, тому вони не підходять.

Тому насправді, все що робить параметр `samesite=lax` --  це дозволяє найбільш поширеним операціям (таким як перехід по URL) передавати файли cookie. Наприклад, відкривання вебсайту за посиланям зі своїх нотаток, що повністю задовольняє умовам.

Але будь-яка більш складна операція, як то мережевий запит з іншого сайту чи відправка форми, втрачає файли cookies.

Якщо вас це влаштовує, тоді використання параметру `samesite=lax` скоріш за все не зіпсує враженя користувача від взаємодії з вашим сайтом та захистить дані.

В цілому, `samesite` -- це чудовий параметр.

Є лише один недолік:

- `samesite` ігнорується  (не підтримується) старими версіями браузерів, до 2017 року і раніше.

**Тому, якщо для забезпечення захисту, покладатися виключно на параметр `samesite`, то старі браузери залишаться вразливими .**

Звісно, ми можемо використовувати `samesite` разом з іншими засобами безпеки, такими як "токен захисту від xsrf", щоб додати ше один рівень захисту, а потім в майбутньому, коли старі браузери вимруть, ми зможемо відмовитися від токенів.

## httpOnly

Цей параметр не має нічого спільного з JavaScript, але ми повинні згадати його заради повноти картини.

Вебсервер використовує заголовок `Set-Cookie` щоб задати файли cookie. Також він може встановити параметр `httpOnly`.

Даний параметр забороняє будь-який доступ до файлів cookie з JavaScript. Ми не можемо бачити чи маніпулювати файлами cookie користуючись `document.cookie`.

Його використовують як запобіжний метод, щоб захистити від деяких атак, коли хакер вживляє на сторінку свій власний код JavaScript та чекає поки користувач зайде на таку сторінку. За ідеальних умов це взагалі неможливо, хакери не повинні мати можливості проникнути своїм кодом на наш сайт, але в коді можуть бути помилки, які все ж дозволяють їм це робити.


Зазвичай, якщо таке трапляється, і користувач все ж зайшов на вебсторінку с хакерським кодом JavaScript, тоді код виконується та отримує доступ до команди `document.cookie`, яка містить аутентифікаційну інформацію. Це погано.

Проте якщо файл cookie має параметр `httpOnly`, то в такому разі `document.cookie` не бачить його, тому файл в безпеці.

## Додаток: Функції файлів cookie

Тут наведено невелику підбірку функцій для роботи з файлами cookie, більш зручних аніж ручна модифікація  `document.cookie`.

Існує безліч бібліотек для роботи з файлами cookie, тому ці функції тут скоріше в демонстраційних цілях. Але повністю робочі.

### getCookie(name)

Найкоротший шлях щоб отримати доступ до файлів cookie це використання [регулярних виразів](info:regular-expressions).

Функція `getCookie(name)` повертає файл cookie з заданим іменем `name`:

```js
// повертає файл cookie з заданим іменем
// або undefined якщо нічого не знайдено
function getCookie(name) {
  let matches = document.cookie.match(new RegExp(
    "(?:^|; )" + name.replace(/([\.$?*|{}\(\)\[\]\\\/\+^])/g, '\\$1') + "=([^;]*)"
  ));
  return matches ? decodeURIComponent(matches[1]) : undefined;
}
```

Тут динамічно генерується `new RegExp` який задовольняє `; name=<value>`.

Зверніть увагу, що значення файлу cookie закодоване, тому `getCookie` використовує вмонтовану функцію `decodeURIComponent` щоб розшифрувати його.

### setCookie(name, value, options)

Встановлює файл cookie з заданим ім’ям `name`, значенням `value`, та типовим параметром `path=/` (можна редагувати, щоб додати інші типові значення):

```js run
function setCookie(name, value, options = {}) {

  options = {
    path: '/',
    // за потреби додайте інші типові значення 
    ...options
  };

  if (options.expires instanceof Date) {
    options.expires = options.expires.toUTCString();
  }

  let updatedCookie = encodeURIComponent(name) + "=" + encodeURIComponent(value);

  for (let optionKey in options) {
    updatedCookie += "; " + optionKey;
    let optionValue = options[optionKey];
    if (optionValue !== true) {
      updatedCookie += "=" + optionValue;
    }
  }

  document.cookie = updatedCookie;
}

// Приклад використання:
setCookie('user', 'John', {secure: true, 'max-age': 3600});
```

### deleteCookie(name)

Щоб видалили файли cookie, ми можем викликати функцію вказавши від’ємне значення в параметрі `max-age`:

```js
function deleteCookie(name) {
  setCookie(name, "", {
    'max-age': -1
  })
}
```

```warn header="Операції оновлення або видалення повинні використовувати ті самі адресу та домен."
Зауважте: коли ми оновлюємо чи видаляємо файли cookie, ми повинні використовувати ті самі адресу та доменне ім’я, що і при встановленні.
```

Все разом: [cookie.js](cookie.js).


## Додаток: Сторонні файли cookies

Файли cookie називаються "сторонніми", якщо вони розміщені з домену який відрізняється від того, до якого належить поточна сторінка.

Наприклад:
1. Сторінка розміщена на `site.com` завантажує банер з іншого сайту: `<img src="https://ads.com/banner.png">`.
2. Разом з банером, віддалений сервер на `ads.com` може встановити заголовок `Set-Cookie` зі значенням `id=1234` взятим з файла cookie. Створені файли cookie походять з домену `ads.com` та будуть видимі лише з `ads.com`:

    ![](cookie-third-party.svg)

3. Наступного разу коли `ads.com` відправить запит на доступ, віддалений сервер отримає `id` з файлу cookie та розпізнає користувача:

    ![](cookie-third-party-2.svg)

4. Іще важливіше те, що коли користувач переходить з `site.com` до іншого сайту `other.com`, на якому також є банер, тоді `ads.com` отримує файли cookie, так наче вони насправді належать `ads.com` і таким чином розпізнає відвідувача та відстежить його, коли той переміщається між сайтами:

    ![](cookie-third-party-3.svg)


Сторонні файли cookie традиційно використовуються для відслідковування статистики відвідувань та показу реклами. Вони прив’язані до початкового домену, тому `ads.com` може відслідковувати одного й того ж користувача при переході між сайтами, якщо всі вони мають до нього доступ.

Природньо, що деякі люди не хочуть щоб за ними стежили, тому браузери дозволяють відключити такі файли cookie.

Крім того, деякі сучасні браузери запроваджують спеціальну політику для таких файлів cookie:
- Safari взагалі не дозволяє сторонні файли cookie.
- У Firefox є "чорний список" сторонніх доменів, з яких він блокує надходження сторонніх файлів cookie.


```smart
Якщо ми завантажимо скрипт з стороннього домену, такого як `<script src="https://google-analytics.com/analytics.js">`, і цей скрипт використовує `document.cookie` щоб встановити файли cookie, то вони не вважатимуться сторонніми.

Якщо скрипт встановлює файл cookie, тоді неважливо звідки завантажений цей скрипт -- файл cookie належить домену поточного web-сайту.
```

## Додаток: GDPR

Ця тема взагалі не пов’язана з JavaScript, це просто потрібно пам’ятати, налаштовуючи файли cookie.

У Європі існує законодавство під назвою GDPR, яке встановлює набір правил для вебсайтів щодо поваги до конфіденційності користувачів. Одне з цих правил — вимагати від користувача явного дозволу на відстеження файлів cookie.

Зауважте, що йдеться лише про відстеження/ідентифікацію/авторизацію файлів cookies.

Отже, якщо ми встановлюємо файл cookie, який лише зберігає деяку інформацію, але не відстежує та не ідентифікує користувача, ми можемо це зробити.

Але якщо ми збираємося встановити файл cookie з даними аутентифікації або ідентифікатором відстеження, користувач повинен дозволити це.

Зазвичай, виконання GDPR web-сайтами реалізується в двох варіантах. Ви, напевно, вже бачили їх обидва в мережі інтернет: 

1. Якщо вебсайт хоче встановити відслідковуючі файли cookie лише для аутентифікованих користувачів.

     Для цього в реєстраційній формі має бути прапорець на кшталт "прийняти політику конфіденційності" (яка описує, як використовуються файли cookie), користувач повинен відмітити його, лише тоді вебсайт може встановлювати файли cookie для авторизації.

2. Якщо вебсайт хоче встановити відстежуючі файли cookies для всіх.

     Щоб зробити це законно, вебсайт показує спливаюче модальне вікно для нових користувачів і вимагає від них погодитися на файли cookie. Тоді web-сайт може встановлювати їх і дозволити людям бачити контент. Але це може турбувати нових відвідувачів. Ніхто не любить бачити такі спливаючі модальні вікна, які вимагають взаємодії, замість перегляду контенту. Але GDPR вимагає чіткої угоди.


GDPR стосується не лише файлів cookie, а й інших питань, пов’язаних із конфіденційністю, але це виходить за рамки даного розділу.


## Підсумки

`document.cookie` забезпечує доступ до файлів cookies
- операції запису змінюють лише вказані файли cookies.
- ім’я та значення файлу cookie повинні бути закодовані.
- один файл cookie не повинен перевищувати 4KB, 20+ файлів на один сайт (залежно від браузера).

Параметри:
- `path=/`, встановлює поточну адресу як типове значеня, робить файли cookie видимими лише за вказаною адресою.
- `domain=site.com`, за замовчуванням файли cookie видимі лише на поточному домені. Якщо домен вказано явно, файли cookie стають видимі на сторінках піддомену.
- `expires` або `max-age` задають кінцевий термін придатності файлів cookie. Без них файли cookie помирають відразу після закритя вікна браузера.
- `secure` робить файли cookie доступними лише по HTTPS (HTTPS-only).
- `samesite` забороняє браузеру відправляти файли cookie у відповідь на запити які надходять з зовнішнього сайту. Це допомагає запобігти XSRF-атакам.

Додатково:
- Сторонні файли cookie можуть бути відхилені браузером, наприклад Safari робить це за замовчуванням.
- Встановлюючи відстежуючий файл cookie для громадян ЄС, GDPR вимагає запитувати дозвіл.
