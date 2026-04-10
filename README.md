# VNK LAB · Claude Skills

Набор скиллов для Claude Code — инструменты для создания контента, монтажа Reels и SMM-стратегии в стиле @vnk_lab.

---

## Установка

```bash
# Клонировать в папку скиллов Claude Code
git clone git@github.com:vnklab/sm-montage-pack.git ~/.claude/skills-vnklab

# Или скопировать нужный скилл вручную
cp -r sm-montage-pack ~/.claude/skills/
```

После копирования скилл доступен в Claude Code как `/sm-montage-pack`.

---

## Скиллы

### `/sm-montage-pack`
**Remotion Reels Montage Pack — VNK LAB**

Генерирует полный пак монтажёра для Instagram Reels на Remotion:
- Карта сцен с таймингом
- Дизайн-система (токены, шрифты, spring-конфиг)
- Рецепты компонентов (Grid, Cam, In, TgSpinner, Liquid Glass)
- Скрипт каждой сцены с анимационными нотами
- Список ассетов
- Заметки редактора

**Вызов:** `/sm-montage-pack`

Claude спросит о продукте, ключевом сообщении, CTA и ассетах — и выдаст готовый пак.

---

## Дизайн-система VNK LAB (кратко)

| Параметр | Значение |
|----------|---------|
| Формат | 1080 × 1920 px, 30 fps |
| Фон | `#000000` |
| Акцент | `#0A84FF` (iOS Blue) |
| Hero-шрифт | Widock Bold |
| Вторичный шрифт | Manrope |
| Spring | `damping: 16, mass: 1.0, stiffness: 120` |

---

## Стек

- [Remotion](https://remotion.dev) 4.x
- React + TypeScript
- FontFace API для кастомных шрифтов
- SSH: `git@github.com:vnklab`
