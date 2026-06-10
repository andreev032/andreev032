---
name: andreev-personal-site
description: Скилл для создания и редактирования личного сайта andreevkirill.com. Знает всю архитектуру, дизайн-систему, как деплоить на GitHub Pages, как работает видео на мобиле и десктопе, и все правила редактирования. Запуск — команда `/site`.
tags: [personal-site, github-pages, html, css, video, deploy, andreev032]
version: 1.0
language: ru
---

# Personal Site Skill — andreevkirill.com

## Архитектура проекта

| Что | Где |
|---|---|
| Репозиторий | github.com/andreev032/andreev032 |
| Хостинг | GitHub Pages |
| Домен | andreevkirill.com (Namecheap) |
| Файл | index.html (один файл, ~2MB) |
| Видео | hero.mp4 (в корне репозитория) |
| Деплой | Пуш в main → автодеплой ~2 мин |

## Дизайн-система

- Цвета: белый #ffffff / тёмно-синий #0a1628 / оранжевый #ff5500
- Шрифты: Playfair Display (заголовки) + DM Sans (текст)
- Стиль: Apple-минимализм, чистые линии

## Hero-секция (финальный вид)

- Eyebrow: "Путешественник · Предприниматель · Тревелблогер"
- Headline: Кирилл / Андреев / 032
- Sub: "Строю бизнес и личный бренд в путешествиях. Один рюкзак. Без ограничений."
- Статы: 42 Стран / 20 Лет в бизнесе / Самый счастливый (большой) + человек на свете (маленький, не жирный)
- Видео: справа на десктопе (52% ширины) / весь экран на мобиле

## Видео — правила

### Десктоп (@media min-width: 769px)
```css
.hero-video-wrap { position:absolute; top:0; right:0; width:52%; height:100%; }
.hero-video { object-fit:cover; object-position:center center; opacity:.85; }
/* Градиент слева для читаемости текста */
.hero-video-wrap::before { background: linear-gradient(to right, var(--white) 0%, transparent 45%),
  linear-gradient(to top, var(--white) 0%, transparent 25%); }
```

### Мобиль (@media max-width: 768px)
```css
.hero-video-wrap { position:absolute; top:0; left:0; width:100%; height:100%; }
.hero-video { object-fit:cover; object-position:center center; opacity:.65; }
/* Только нижний градиент — БЕЗ белых полос по бокам */
.hero-video-wrap::before { background: linear-gradient(to top, rgba(255,255,255,0.85) 0%, transparent 30%); }
```

### iOS loop fix (обязательно)
```js
(function(){
  var v=document.getElementById('heroVideo');if(!v)return;v.muted=true;
  function tryPlay(){var p=v.play();if(p!==undefined)p.catch(function(){});}
  v.addEventListener('timeupdate',function(){if(v.duration&&v.currentTime>=v.duration-0.3){v.currentTime=0;tryPlay();}});
  v.addEventListener('ended',function(){v.currentTime=0;tryPlay();});
  setInterval(function(){if(v.paused||v.ended){v.currentTime=0;tryPlay();}},2000);
  tryPlay();
})();
```

### Размеры видео
| Версия | Размер | Качество |
|---|---|---|
| Десктоп | 4.69MB | 900p crf29 |
| Мобиль (оптимум) | ~1.3MB | 540p crf33 |

```bash
# Конвертация для мобиля:
ffmpeg -i input.mov -vcodec libx264 -crf 33 -preset fast -vf "scale=540:-2" -an -movflags +faststart -y hero_mobile.mp4
```

## Критические правила

1. ВСЕГДА спрашивать подтверждение перед любым изменением
2. НИКОГДА не трогать мобильные стили при работе с десктопом и наоборот
3. Десктоп = @media(min-width:769px) / Мобиль = @media(max-width:768px)
4. Всегда брать свежую копию index.html перед правками
5. Батчить все изменения — один пуш, не несколько

## Деплой через GitHub API

```python
import base64, json, urllib.request

# TOKEN брать у Кирилла — не хранить в файлах!
REPO = "andreev032/andreev032"

def push_file(token, path, local_path, message):
    req = urllib.request.Request(
        f"https://api.github.com/repos/{REPO}/contents/{path}",
        headers={"Authorization": f"token {token}", "Accept": "application/vnd.github.v3+json"}
    )
    with urllib.request.urlopen(req) as resp:
        sha = json.loads(resp.read())["sha"]
    with open(local_path, "rb") as f:
        content = base64.b64encode(f.read()).decode("utf-8")
    payload = json.dumps({"message": message, "content": content, "sha": sha}).encode("utf-8")
    req = urllib.request.Request(
        f"https://api.github.com/repos/{REPO}/contents/{path}",
        data=payload, method="PUT",
        headers={"Authorization": f"token {token}", "Accept": "application/vnd.github.v3+json", "Content-Type": "application/json"}
    )
    with urllib.request.urlopen(req) as resp:
        return json.loads(resp.read())["commit"]["sha"]
```

## Уроки из работы

1. `display:none` на базовый класс — сломает и мобиль и десктоп
2. `object-position: top center` — если обрезает голову; `center center` — середина кадра
3. iOS видео без loop fix останавливается после первого просмотра
4. GitHub Pages кеш до 5 минут — открывать в новой вкладке
5. Градиент слева на десктопе для текста; на мобиле только нижний
6. Токены НИКОГДА не писать в файлы — только передавать в чате
