---
title: Вёрстка страниц под е-книги
description: Вёрстка страниц с учётом особенностей электронных читалок
date: 2022-11-07 20:15:00 +03:00
categories: dev
layout: post
tags: ['e-books', 'readers']
---

У меня есть электронная читалка, периодически я пытаюсь засесть читать на ней не только книги, но и что-то из веба. 
И далеко не всегда это смотрится адекватно.

Решил и свой недобложик "подтянуть" под читалку, заодно разобраться, что ж там такого особенного. Требования, в основном, такие же, как для печати на ЧБ-принтере на нормальной бумаге.
Но если принтер можно отличить через медиа-запрос `@media print`, то с читалкой всё сложнее. Если читалка чёрно-белая, то впринципе, должно подойти `@media monochrome`. Для некоторых читалок Kindle есть свои медиазапросы, ни с кем не совместимые и не интересные. Но у меня цветной PocketBook 740 с 4096 цветами. С медиазапросами учитывающими поддерживаемое количество цветов не прокатило и я стал разбираться, а что вообще рассказывает о себе браузер моей читалки. Очень выручил сайт https://mediaqueriestest.com/ , на котором видно, какие медиа-фичи "умеет" браузер (либо притворяется, что умеет).

Посмотрев по фичам сначала не нашёл ничего интересного, но потом сравнил вывод с телефоном и ПК. Выяснилось что книга не умеет "hover" (что логично), и наглухо не умеет точное позиционирование при клике (pointer). Дополнительно ограничив эту "выборку" по размерам книги в горизонтальном положении получил вот такой комплексный медиа-запрос.

```css
@media print, monochrome, (hover: none) and (any-pointer: none) {
}
```

Внутри цвета подтягиваются к естественным для печати на бумаге, сносятся лишние элементы (шапка, менюшки и т.д.).