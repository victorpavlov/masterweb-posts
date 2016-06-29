Это перевод англоязычной статьи [How to lint your Sass/CSS properly with Stylelint](http://www.creativenightly.com/2016/02/How-to-lint-your-css-with-stylelint/) автора [Scotty Vernon](https://twitter.com/KingScooty). Stylesheet линтинг. Не многие люди делают это. Гораздо больше людей должны, особенно разнообразные команды, которые имеют кодовую базу доступную многим разработчикам. В этой статье я буду говорить о том, почему мы должны линтовать наши таблицы стилей, и как реализовать линтинг таблиц стилей в процессе нашей сборки, как для чистого CSS, та и для Sass. 

**Содержание:**

1.  [Вступление](#introduction)
    1.  [Что такое линтинг?](#what-is-linting)
    2.  [Почему мы должны линтовать наши CSS?](#why-should-we-lint-our-stylesheets)
    3.  [Представляем Stylelint](#introducing-stylelint)
2.  [Настройка](#setup)
    1.  [Конфигурационные файлы Stylelint](#config-stylelint)
    2.  [Как линтовать ваш CSS](#how-to-lint-your-css)
    3.  [Как линтовать ваш Sass](#how-to-lint-your-sass)
3.  [Расширяем Stylelint плагинами](#extending-stylelint-with-plugins)
    1.  [Реальный пример: линтинг на практике](#case-study-linting-in-practice)
4.  [Послесловие](#afterword)
    1.  [Установка в один клик пайпа для Sass в gulp](#a-one-click-install-gulp-build-pipeline-for-sass)

## <a name="introduction"></a>Вступление

### <a name="what-is-linting"></a>Что такое линтинг?

Линтинг — это процесс проверки исходного кода на программные и стилистические ошибки. Это самый полезный способ выявления распространенных, и не очень, ошибок допускаемых во время написания кода. Это, своего рода, проверка правописания для языков программирования. В то время как линтинг полез при самостоятельной работе, он становится незаменимым при работе в команде: когда много (неосторожных) рук касаются кода. Линт (или линтер) — это программа или инструмент поддерживающий линтинг (проверку качества кода). Они доступны для большинства языков, таких как C, Python, JavaScript, CSS и прочих.

### <a name="why-should-we-lint-our-stylesheets"></a>Почему мы должны линтовать наши CSS?

Есть много причин линтовать файлы стилей. Это поддерживает единообразие кода, показывает ошибки в кодовой базе, помогает уменьшить ненужный код, а также помогает избежать «ленивого» кодинга.

Давайте рассмотрим несколько примеров.

```css
.no-space-after-colon {
    display:block; /* Нет пробела после двоеточия. */
}

.no-semicolon {
    position: relative /* Нет точки с запятой в конце. */
}
```

Линтеры очень хороши для отлавливания стилистических ошибок, какие показаны выше. Настройка стилистических правил в линтинге не критическое требование, но они помогают содержать код однородным. Кроме того, не знаю как у вас, но две ошибки показанные выше, являются моим слабым местом.

```css
.invalid-hex {
    color: #FFF00G; /* Не правильно задан цвет в Hex. */
}
```

Они также очень хороши для отлавливания ошибок с неправильно заданными цветами в формате Hex, которые могут быть результатом ошибки. Ошибки данного типа могут сломать важные визуальные стили, если не будут отловлены.

```css
/* Ненужные префиксы. */
.unnecessary-prefixes {
    -webkit-border-radius: 5px;
    -moz-border-radius: 5px;
    border-radius: 5px;
}
```
На самом деле существует несколько CSS3 правил, которым не нужны префиксы, чтобы работать. Линтинг помогает выявить эти правила и дает возможность удалить ненужный и устаревший код. Линтинг префиксов особенно полезен в паре с [Autoprefixer](https://github.com/postcss/autoprefixer) — это позволит вам убрать все префиксы и добавить только те, которые вам нужны, учитывая вашу аудиторию.

```css
.duplicate-rule {
    display: block;
    transition: opacity .2s;
    color: #444;
    background-color: #eee;
    transition: background-color .4s; /* Дублирование правила. */
}
```

Повторяющиеся правила являются распространенной формой ошибочного кода. Что, если разработчик подразумевал для обеих свойств `opacity` и `background-colour` иметь эффект перехода? В этом случае эффект перехода для `opacity` будет утерян. Линтинг покажет эту ошибку.

Достаточно убедительно? Если нет, то продолжайте чтение.

### <a name="introducing-stylelint"></a>Представляем Stylelint

[Stylelint](http://stylelint.io/) — это супер-расширяемый и ненавязчивый CSS линтер написанный на JavaScript. Это самый новый и самый лучший линтер CSS. Он поддерживает последний CSS синтаксис, понимает схожий с CSS синтаксис, а также расширяется с помощью плагинов. Что еще, поскольку он разработан на JavaScript, а не на Ruby, он гораздо быстрее чем [scss-lint](https://github.com/brigade/scss-lint).

> Stylelint мощный, современный CSS линтер, который поможет вам обеспечить соблюдение согласованных правил и избежать ошибок в ваших таблицах стилей.

Линтер разработан с помощью [PostCSS](https://github.com/postcss/postcss), так что он понимает любой синтаксис, который может распарсить PostCSS, включая SCSS.

> PostCSS является инструментом для преобразования стилей с помощью плагинов JS. Эти плагины могут линтовать ваш CSS, поддерживать переменные и микшины, разбирать CSS синтаксис будущих спецификаций, встроенные изображения и многое другое.

Главное предназначение PostCSS — это делать одну вещь, и делать ее хорошо; все остальное делают плагины. Сейчас насчитывается более 200 плагинов для PostCSS и, поскольку все они написаны на JavaScript, они запускаются супер быстро!

PostCSS и Stylelint — это то, что мы будем использовать, чтобы линтовать наши стили в следующем разделе.

## <a name="setup"></a>Настройка

### <a name="config-stylelint"></a>Конфигурационные файлы Stylelint

Красота Stylelint в том, на сколько ненавязчивым он есть. Вы создаете свои наборы правил с нуля, так что он может быть настолько заносчивым и упрямым, насколько вы захотите — вам не придется тратить время на отключение не нужных правил, чтобы начать работу.

[Документация правил Stylelint](https://github.com/stylelint/stylelint/blob/master/docs/user-guide/rules.md) является хорошей отправной точкой. Она также предоставляет [стандартный конфигурационный файл Stylelint](https://github.com/stylelint/stylelint-config-standard/blob/master/index.js), который достаточно хорошо продуман, чтобы использовать его в ваших проектах.

Для начала нашей работы мы будем использовать хороший и компактный конфигурационный файл, который охватывает самое необходимое. Лично я считаю, что это лучше, чем стартовый конфиг, который предоставляет Stylelint, поскольку это дает вам место, чтобы добавить то, что вам нужно, вместо того, чтобы отключить правила, которые вам не нужны.

Он выглядит так:

```javascript
"rules": {
  "block-no-empty": true,
  "color-no-invalid-hex": true,
  "declaration-colon-space-after": "always",
  "declaration-colon-space-before": "never",
  "function-comma-space-after": "always",
  "function-url-quotes": "double",
  "media-feature-colon-space-after": "always",
  "media-feature-colon-space-before": "never",
  "media-feature-name-no-vendor-prefix": true,
  "max-empty-lines": 5,
  "number-leading-zero": "never",
  "number-no-trailing-zeros": true,
  "property-no-vendor-prefix": true,
  "rule-no-duplicate-properties": true,
  "declaration-block-no-single-line": true,
  "rule-trailing-semicolon": "always",
  "selector-list-comma-space-before": "never",
  "selector-list-comma-newline-after": "always",
  "selector-no-id": true,
  "string-quotes": "double",
  "value-no-vendor-prefix": true
}

```

Я рекомендую внимательно изучить [документацию по правилам Stylelint](https://github.com/stylelint/stylelint/blob/master/docs/user-guide/rules.md) и доработать его, создав ваш идеальный конфиг для линтера. Теперь давайте настроим нашу сборку, используя эти правила.

### <a name="how-to-lint-your-css"></a>Как линтовать ваш CSS

Давайте начнем с линтинга чистого CSS. Вы будете удивлены как легко это настроить! Инструменты, которые вам нужно установить: `gulp-postcss`, `postcss-reporter`, и `stylelint`. Давайте сделаем это.

```bash
npm install gulp-postcss postcss-reporter stylelint --save-dev
```
И вот файл для Gulp, чтобы собрать все это вместе:

```javascript
/**
 * Linting CSS stylesheets with Stylelint
 * http://www.creativenightly.com/2016/02/How-to-lint-your-css-with-stylelint/
 */

var gulp        = require('gulp');

var postcss     = require('gulp-postcss');
var reporter    = require('postcss-reporter');
var stylelint   = require('stylelint');

gulp.task("css-lint", function() {

  // Stylelint config rules
  var stylelintConfig = {
    "rules": {
      "block-no-empty": true,
      "color-no-invalid-hex": true,
      "declaration-colon-space-after": "always",
      "declaration-colon-space-before": "never",
      "function-comma-space-after": "always",
      "function-url-quotes": "double",
      "media-feature-colon-space-after": "always",
      "media-feature-colon-space-before": "never",
      "media-feature-name-no-vendor-prefix": true,
      "max-empty-lines": 5,
      "number-leading-zero": "never",
      "number-no-trailing-zeros": true,
      "property-no-vendor-prefix": true,
      "rule-no-duplicate-properties": true,
      "declaration-block-no-single-line": true,
      "rule-trailing-semicolon": "always",
      "selector-list-comma-space-before": "never",
      "selector-list-comma-newline-after": "always",
      "selector-no-id": true,
      "string-quotes": "double",
      "value-no-vendor-prefix": true
    }
  }

  var processors = [
    stylelint(stylelintConfig),
    // Pretty reporting config
    reporter({
      clearMessages: true,
      throwError: true
    })
  ];

  return gulp.src(
      // Stylesheet source:
      ['app/assets/css/**/*.css',
      // Ignore linting vendor assets:
      // (Useful if you have bower components)
      '!app/assets/css/vendor/**/*.css']
    )
    .pipe(postcss(processors));
});
```

Как легко это было? Я написал 50 строчек кода, включая привила линтера и импорты. Убедитесь что вы изменили пути расположения исходников, согласно вашего проекта!

А более прекрасно то, что только одна строчка кода должна быть изменена, чтобы включить поддержку Sass. Давайте сделаем это сейчас...

### <a name="#how-to-lint-your-sass"></a>Как линтовать ваш Sass

Линтовать Sass файлы супер просто с PostCSS. Единственное различие между линтингом CSS и SCSS в том, что вам нужно научить PostCSS понимать синтаксис `.scss`, и это так же просто, как установка `postcss-scss`, и изменение одной строки в задаче выше.

```bash
npm install postcss-scss --save-dev
```

```javascript
  //[...]

  return gulp.src(
      ['app/assets/css/**/*.scss',
      '!app/assets/css/vendor/**/*.scss']
    )
    .pipe(postcss(processors, {syntax: syntax_scss})); ⬅
});
```

Вот полный файл для Gulp, чтобы линтовать Sass файлы:

```bash
npm install gulp-postcss postcss-reporter stylelint postcss-scss --save-dev
```

```javascript
/**
 * Linting Sass stylesheets with Stylelint
 * http://www.creativenightly.com/2016/02/How-to-lint-your-css-with-stylelint/
 */

var gulp        = require('gulp');

var postcss     = require('gulp-postcss');
var reporter    = require('postcss-reporter');
var syntax_scss = require('postcss-scss');
var stylelint   = require('stylelint');

gulp.task("scss-lint", function() {

  // Stylelint config rules
  var stylelintConfig = {
    "rules": {
      "block-no-empty": true,
      "color-no-invalid-hex": true,
      "declaration-colon-space-after": "always",
      "declaration-colon-space-before": "never",
      "function-comma-space-after": "always",
      "function-url-quotes": "double",
      "media-feature-colon-space-after": "always",
      "media-feature-colon-space-before": "never",
      "media-feature-name-no-vendor-prefix": true,
      "max-empty-lines": 5,
      "number-leading-zero": "never",
      "number-no-trailing-zeros": true,
      "property-no-vendor-prefix": true,
      "rule-no-duplicate-properties": true,
      "declaration-block-no-single-line": true,
      "rule-trailing-semicolon": "always",
      "selector-list-comma-space-before": "never",
      "selector-list-comma-newline-after": "always",
      "selector-no-id": true,
      "string-quotes": "double",
      "value-no-vendor-prefix": true
    }
  }

  var processors = [
    stylelint(stylelintConfig),
    reporter({
      clearMessages: true,
      throwError: true
    })
  ];

  return gulp.src(
      ['app/assets/css/**/*.scss',
      // Ignore linting vendor assets
      // Useful if you have bower components
      '!app/assets/css/vendor/**/*.scss']
    )
    .pipe(postcss(processors, {syntax: syntax_scss}));
});
```
Так легко! И это все! Теперь вы можете линтовать и CSS, и Sass файлы.

Продолжайте читать, если вы хотите узнать о расширении Stylelint с помощью плагинов, и почему вы хотите сделать это, рассмотрев живой пример.

## <a name="extending-stylelint-with-plugins"></a>Расширяем Stylelint плагинами

Так же, как PostCSS, Stylelint расширяется с помощью плагинов, что на самом деле удивительно!

Давайте запустим быстрый сценарий, где линтинг поможет улучшить читаемость кода, а также поможет пнуть ленивых разработчиков, когда они попытаются в легкую обойти стандарты кодовой базы.

### <a name="case-study-linting-in-practice"></a>Реальный пример: линтинг на практике

#### Проект-менеджер, который любит код

Как насчет такого сценария. Проект-менеджер менеджер управляет проектом по созданию нового веб-приложения, которое находится в разработке, и, чтобы освободить какое-то время разработчикам, решает сам добавить какую-то фичу. Он решает добавить тень для бокса компонента на хувер, и также добавить состояние на хувер для ссылок дочернего компонента.

*Худшее, что может случится?*

![Твит высказывания проект-менеджера](images/stylelint-twitter-quote.jpg)

Вот код, который менеджер добавляет в проект:

```css
.component {
  position: relative;
  //[...]

  &:hover { ⬅
    box-shadow: 1px 1px 5px 0px rgba(0,0,0,0.75);

    .component__child { ⬅
      ul { ⬅
        li { ⬅
          a { ⬅
            &:hover { ⬅
              text-decoration: underline;
            }
          }
        }
      }
    }
  }
}
```
Ух!

#### Вложенность селекторов в Sass — это плохо! Используем линтер!

Вложенность селекторов является необходимым злом при разработке с использованием Sass; это действительно полезно при правильном использовании, и один из способов путешествие в ад [специфичности](https://css-tricks.com/specifics-on-css-specificity/), если ей злоупотреблять. Вложенность, как правило, является последствием ленивого кодинга и в результате получается коде, который трудно читаем, и плохо написан. Первое правило `&:hover{...}` может быть на 10 строк ниже определения родительского компонента, в результате сложно понять к чему оно принадлежит. Однако, более важно то, что вложенность здесь совершенно не нужна.

Вот CSS, в который будет скомпилировано правило выше:

```css
.component:hover .component_child ul li a:hover {}
/* Черт возьми, что это?! */
```

Если бы я работал в команде и кто-то предложил бы такой код, то он бы имел очень серьезный разговор со мной.

Следующий разработчик, который приходит и хочет переопределить это правило каскадирования будет иметь проблемы. Зная об этом, я бы посоветовал воздержаться от использования вложенности любой ценой, по крайней мере, если вы не знаете, что вы делаете.

На счастье, для этого есть плагин! С помощью Stylelint мы можем плагин с именем `stylelint-statement-max-nesting-depth`, и установить максимальный уровень вложенности, чтобы избежать излишней вложенности.

```bash
npm install stylelint-statement-max-nesting-depth --save-dev
```

И просто добавляем в наш файл для Gulp следующий код в таск `scss-lint `:

```javascript
gulp.task("scss-lint", function() {
  var stylelintConfig = {
    "plugins": [
      "stylelint-statement-max-nesting-depth"
    ],
    "rules": {
      //[...]
      "statement-max-nesting-depth": [3, { countAtRules: false }],
    }
  }

  //[..]
});
```

Для команд, которые знают, что они делают, я установил максимальный лимит вложенности в 3. *(Установите его ниже для неопытных команд)*

С максимальным лимитом вложенности равным 3, Stylelint заставит проект-менеджера исправить код выше. Проект-менеджер пойдет, подумает и вернется с таким кодом:

```css
.component:hover {
  box-shadow: 1px 1px 5px 0px rgba(0, 0, 0, 0.75);

  .component__child {
    ul {
      li {
        a:hover {
          text-decoration: underline;
        }
      }
    }
  }
}
```

Исправленная версия кода более читаемая, но все еще не подходит. Там все еще есть не нужная вложенность селекторов! Линтер знает это и заставляет проект-менеджера переосмыслить его реализацию, чтобы исправить сборку.

```css
.component:hover {
  box-shadow: 1px 1px 5px 0px rgba(0, 0, 0, 0.75);
}

.component__child {
  a:hover {
    text-decoration: underline;
  }
}
```

Вот, теперь мы уже пришли к чему-то! Теперь это будет принято линтером, и сборка пройдет. Код выше не лох, но он всегда может быть лучше! Если вы хотите быть действительно *суровым*, вы можете отключить вложенность вовсе, оставив только `@ правила` (например для `@at-root {}`, — примечание переводчика). Это заставит членов команды, включая проект-менеджеров думать серьезно о том, что они пишут.

```css
.component:hover {
  box-shadow: 1px 1px 5px 0px rgba(0, 0, 0, 0.75);
}

.component__link:hover {
  text-decoration: underline;
}
```

Мило! Без линтинга таблицы стилей при сборке, и побуждения к рефакторингу, этот тип ленивого кодинга никогда бы не был пойман, и кодовая база постепенно деградировала бы в качестве.

Надеюсь, теперь я убедил вас, что линтинг таблиц стилей является выгодным вложением. Линтинг является вашим другом. Инвестиция не дорогая, но это защищает команду от технической задолженности в виде плохо написанного кода.

В добрый путь, мой друг разработчик и дизайнер!

Иди и линтуй!

## <a name="afterword"></a>Послесловие

### <a name="a-one-click-install-gulp-build-pipeline-for-sass"></a>Установка в один клик пайпа для Sass в gulp

Хотели бы вы использовать мощь линтинга Sass, вместе с общим процессом Gulp сборки Sass бесплатно, просто установив небольшой пакет через `npm`? 

Я думаю да. У меня есть пакет с открытым исходным кодом под названием [Slushie](https://github.com/kingscooty/slushie), вы можете прочитать об этом в [моем блоге](http://www.creativenightly.com/2016/02/Slushie-the-pre-packaged-gulp-pipeline/).




> **Об авторе**
>
> [Scotty Vernon](https://twitter.com/KingScooty) (*Скотти Вернон*) является творческим разработчиком и директором Wildflame Studios. Он работал с такими проектами, как  BBC Sport, BBC R&D и другими. Он всегда не далеко от твиттера, любит немецкое пиво, игры, а иногда и писать в третьем лице.



