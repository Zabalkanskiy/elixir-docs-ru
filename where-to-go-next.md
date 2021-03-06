---
title: Куда двигаться дальше
---

# {{ page.title }}

Хотите изучить Эликсир глубже? Продолжайте читать!

## Сделайте свой первый проект на Эликсире

Для того, чтобы начать ваш первый проект, в поставке Эликсира есть инструмент сборки Mix. Вы можете начать новый проект запустив:

```bash
$ mix new path/to/new/project
```

Мы написали руководство, которое объясняет, как сделать приложение на Эликсире, с его собственным деревом супервизора, конфигурацией, тестами и прочим. Приложение работает с распределенным хранилищем ключ-значение, в котором мы объединяем пары ключ-значение в "корзины" (buckets) и распределяем эти "корзины" между несколькими нодами:

* [Mix и OTP](/getting-started/mix-otp/introduction-to-mix.html)

## Мета-программирование

Эликсир - расширяемый и глубоко кастомизируемый язык программирования, благодаря поддержке мета-программирования. Большая часть мета-программирования в Эликсире основана на макросах, которые очень полезны в некоторых ситуациях, особенно при написании DSL. Мы написали небольшое руководство, которое объясняет основы работые макросов, показывает, как писать макросы, и как их использовать для создания DSL:

* [Мета-программирование в Эликсире](/getting-started/meta/quote-and-unquote.html)

## Сообщество и другие ресурсы

У нас есть раздел ["Обучение"](/learning.html), где мы рекомендуем книги, скринкасты и другие ресурсы для изучения Эликсира и его экосистемы. Есть и другие ресурсы, например, конференции, проекты с открытым исходным кодом и прочие материалы, предоставляемые сообществом.

Не забывайте и о возможности взглянуть на [исходный код самого Эликсира](https://github.com/elixir-lang/elixir), Который написан в основном на Эликсире (главным образом директория `lib`), или [исследовать документацию](/docs.html).

## Немного Эрланга

Эликсир работает на виртуальной машине Эрланга, и, рано или поздно, Эликсир разработчики захотят работать с существующими библиотеками Эрланга. Мы приводим список ресурсов, которые раскрывают основы Эрланга и его более продвинутые возможности:

* [Erlang Syntax: A Crash Course](/crash-course.html) - введение в синтаксис Эрланга. Каждый сниппет сопровождается эквивалентным кодом на Эликсире. Это возможность не только разобраться с синтаксисом Эрланга, но и лучше понять некоторые вещи, которые изучались в данном руководстве.

* На официальном сайте Эрланга есть короткое [руководство](http://www.erlang.org/course/concurrent_programming.html) с картинками, которое кратко объясняет основы параллельного программирования в Эрланге.

* [Learn You Some Erlang for Great Good!](http://learnyousomeerlang.com/) - прекрасное введение в Эрланг, его структуру, стандартную библиотеку, передовые практики и многое другое. Если вы уже прочитали Crash Course, упомянутый выше, можете свободно пропустить первые пару глав этой книги, которые в основном посвящены синтаксису. Когда вы дойдёте до главы [The Hitchhiker's Guide to Concurrency](http://learnyousomeerlang.com/the-hitchhikers-guide-to-concurrency), это будет то место, где начинается всё самое интересное.
