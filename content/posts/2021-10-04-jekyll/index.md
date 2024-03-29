---
title: Базовая настройка Jekyll для заметок
description: Использование Jekyll в качестве генератора статического сайта
date: 2021-10-04 00:27:00 +03:00
categories: blog
layout: post
tags: ['Markdown', 'Jekyll', 'Blog']
---

Для ведения заметок я решил использовать в первую очередь Markdown как наиболее простой и удобный формат. Заметок и всяческих записок сумасшедшего у меня очень много, но отдельные штуки хочется не только не потерять, но и другим показать. Значит надо публиковать где-то, где заметки не будут удалены в обозримом будущем. Например на GitHub Pages, которые бесплатны и привязаны к аккаунту GitHub. В качестве движка для конвертации заметок в статичный web-сайт сначала было решено использовать Jekyll, как это рекомендуется самим GitHub, если перейти в репозитории в разделе Settings -> Pages.

Для установки я использовал WSL, потому что в Linux проще с Ruby (на нём написан Jekyll).
```
$ sudo apt install ruby-dev
$ sudo gem install jekyll bundler
$ jekyll new <sitename>
$ cd <sitename>
$ bundle add webrick
$ bundle exec jekyll serve
```

Подробнее про установку в [официальной документации](https://jekyllrb.com/docs/#instructions).

Статьи (.md файлы) создаются в директории `_posts`

Файлы имеют вид `2021-10-03-let-the-story-begins.md`

Каждый файл начинается с  описания и даты, например
```markdown
---
title: "Выходные с Let's encrypt"
date: 2021-10-03 13:35:00 +03:00
categories: hosting
layout: post
---
```

Для улучшения читаемости я исправил тему по умолчанию, для этого пришлось скопировать её из установленного gem. Сделал регион чтения шире (теперь он занимает почти полэкрана если выровнять браузер по левой или правой половине экрана), шрифт сменил на более читаемый и сделал крупнее. Вроде и с телефона, и с читалки смотрится приемлемо.

Базовый конфиг `_config.yml`
```yaml
title: Ruslan Yakauley's notes
email: ruslan.yakauleu(at)gmail.com
description: >- # this means to ignore newlines until "baseurl:"
  Some notes, hints and cheatsheets which came in handy in my surrounding
baseurl: "" # the subpath of your site, e.g. /blog
url: "https://quazi.github.io" # the base hostname & protocol for your site, e.g. http://example.com
github_username:  QuAzI

theme: minima
plugins:
  - jekyll-feed
```

В директории `assets` кастомизация стилей
`assets/customization.scss`
```scss
$base-font-size: 18px !default;

$base-font-family: "PT Serif", serif;

.page-content {
  .wrapper {
    max-width: calc(900px - (30px * 2));
  }
}
```

`assets/main.scss`
```scss
$base-font-size: 22px !default;

@import "minima";

.page-content {
  .wrapper {
    max-width: calc(900px - (30px * 2));
  }
}
```

`assets/minima-social-icons.svg` скопирован из используемой темы

