# Иконки WinePool Atlas: что осталось дорисовать

Дата: 05.08.2026, дополнено 06.08.2026 волной 6
Кому: Codex
Основание: `session_2_5_atlas_seed_review_v1_2026_08_03.md` §7, реестр §1.5–1.7

Термины уже опубликованы и назначаются винам. Иконки нужны, чтобы заменить
нейтральный глиф, которым сейчас рисуется термин без ассета. Ничего не сломано:
факт показывается, просто без предметной картинки.

## 1. Как устроена связь файла и термина

Три вещи должны совпасть, иначе иконка не подхватится:

1. **Файл** кладётся в свою папку с точным именем.
2. **`icon_key`** термина в Atlas собирается из имени файла.
3. **Элемент enum** в Dart, потому что резолвер ищет ассет по имени файла.

| Вид | Папка | Имя файла | `icon_key` | Enum |
|---|---|---|---|---|
| Ароматы | `assets/icons/wine_card/aromas/` | `aroma_<name>_72.png` | `aroma.<name>` | `WineAromaAsset` |
| Сочетания | `assets/icons/wine_card/pairings/` | `pairing_<name>.png` | `pairing.<name>` | `WinePairingAsset` |
| Прочее | `assets/icons/wine_card/` | `<name>.svg` | `card.<name>` | `WineCardIconAsset` |

Пример: файл `aroma_cherry_sour_72.png` → `icon_key` = `aroma.cherry_sour` →
элемент `sourCherry('aroma_cherry_sour_72.png')`.

**Codex делает только файлы.** Элемент enum и `icon_key` в базе проставляются
отдельной правкой — иначе тест `every seeded Atlas aroma icon_key has a
production asset` покажет расхождение.

## 2. Приоритет 1: ароматы без иконок

Термины уже назначены винам, глиф нейтральный. Порядок — по числу вин.

| Термин | Вин | Файл | `icon_key` |
|---|---:|---|---|
| Клубника | 30 | `aroma_strawberry_72.png` | `aroma.strawberry` |
| Дым | 17 | `aroma_smoke_72.png` | `aroma.smoke` |
| Сухофрукты | 16 | `aroma_family_dried_fruits_72.png` | `aroma.family_dried_fruits` |
| Минеральность | 14 | `aroma_minerality_72.png` | `aroma.minerality` |
| Бриошь и тосты | 6 | `aroma_brioche_72.png` | `aroma.brioche` |
| Ананас | 1 | `aroma_pineapple_72.png` | `aroma.pineapple` |
| Манго | 1 | `aroma_mango_72.png` | `aroma.mango` |

Замечания по содержанию:

- **Сухофрукты** — назначаемое семейство, поэтому по правилу §7.2 реестра это
  композиция из двух-трёх представителей (курага, изюм, чернослив), а не
  увеличенная копия одного плода.
- **Минеральность** и **Дым** — не предметы. Нужен узнаваемый абстрактный
  образ: кремень и скол камня для минеральности, струйка дыма для дыма.
  Здесь важнее читаемость на 56 px, чем детализация.
- **Бриошь и тосты** — выпечка, а не тост-«пожелание».

Стилистика — утверждённая gold/amber серия, как у существующих 45 ароматов.

## 2.1. Приоритет 1б: 31 термин волны 6

Заведены 06.08.2026 миграцией `202608060200_seed_atlas_aroma_wave6.sql`. У всех
`icon_key` пустой, поэтому в карточке они уже рисуются нейтральным глифом —
ничего не сломано, но и предметной картинки нет.

Отбор по данным: каждый термин назван минимум в 7 винах каталога. Число вин —
это и есть приоритет: рисовать сверху вниз.

| Термин | Вин | Файл | `icon_key` |
|---|---:|---|---|
| Черноплодная рябина | 71 | `aroma_chokeberry_72.png` | `aroma.chokeberry` |
| Травы | 62 | `aroma_herbs_72.png` | `aroma.herbs` |
| Кизил | 56 | `aroma_cornel_72.png` | `aroma.cornel` |
| Черешня | 54 | `aroma_sweet_cherry_72.png` | `aroma.sweet_cherry` |
| Бальзамические тона | 36 | `aroma_balsamic_72.png` | `aroma.balsamic` |
| Бузина | 35 | `aroma_elderberry_72.png` | `aroma.elderberry` |
| Лавр | 31 | `aroma_bay_leaf_72.png` | `aroma.bay_leaf` |
| Изюм | 28 | `aroma_raisin_72.png` | `aroma.raisin` |
| Шиповник | 26 | `aroma_rosehip_72.png` | `aroma.rosehip` |
| Животные тона | 26 | `aroma_animal_72.png` | `aroma.animal` |
| Сливочные тона | 26 | `aroma_cream_72.png` | `aroma.cream` |
| Болгарский перец | 24 | `aroma_bell_pepper_72.png` | `aroma.bell_pepper` |
| Паслён | 23 | `aroma_nightshade_72.png` | `aroma.nightshade` |
| Эвкалипт | 23 | `aroma_eucalyptus_72.png` | `aroma.eucalyptus` |
| Инжир | 22 | `aroma_fig_72.png` | `aroma.fig` |
| Сухая листва | 20 | `aroma_dry_leaves_72.png` | `aroma.dry_leaves` |
| Можжевельник | 16 | `aroma_juniper_72.png` | `aroma.juniper` |
| Мята | 16 | `aroma_mint_72.png` | `aroma.mint` |
| Анис | 14 | `aroma_anise_72.png` | `aroma.anise` |
| Полевые цветы | 13 | `aroma_meadow_flowers_72.png` | `aroma.meadow_flowers` |
| Айва | 13 | `aroma_quince_72.png` | `aroma.quince` |
| Чай | 13 | `aroma_tea_72.png` | `aroma.tea` |
| Корица | 13 | `aroma_cinnamon_72.png` | `aroma.cinnamon` |
| Барбарис | 11 | `aroma_barberry_72.png` | `aroma.barberry` |
| Чабрец | 11 | `aroma_thyme_72.png` | `aroma.thyme` |
| Грибы | 9 | `aroma_mushroom_72.png` | `aroma.mushroom` |
| Гвоздика | 9 | `aroma_clove_72.png` | `aroma.clove` |
| Акация | 8 | `aroma_acacia_72.png` | `aroma.acacia` |
| Карамель | 8 | `aroma_caramel_72.png` | `aroma.caramel` |
| Розмарин | 8 | `aroma_rosemary_72.png` | `aroma.rosemary` |
| Миндаль | 7 | `aroma_almond_72.png` | `aroma.almond` |

Замечания по содержанию:

- **Травы** — назначаемое семейство (дети: Лавр, Эвкалипт, Мята, Можжевельник,
  Чабрец, Розмарин). По правилу §7.2 реестра это композиция из двух-трёх
  силуэтов, а не один увеличенный лист.
- **Бальзамические тона**, **Животные тона**, **Сухая листва** — не предметы.
  Нужен узнаваемый абстрактный образ; читаемость на 56 px важнее детализации.
- **Сливочные тона** — молочная текстура, не корова и не бутылка молока.
- **Болгарский перец** и **Паслён** — зелёные тона недозрелого винограда. Их
  часто показывают рядом, поэтому силуэты должны различаться с первого взгляда.
- **Черешня** обязана отличаться от уже существующей **Вишни**: черешня
  крупнее, светлее и на длинной плодоножке.
- **Черноплодная рябина** — гроздь мелких чёрных ягод, не одна ягода.

Стилистика — та же gold/amber серия.

## 3. Приоритет 2: 15 SVG-заглушек

Файлы существуют под окончательными именами и содержат одинаковую временную
заглушку. Менять нужно **только содержимое**, имена и пути трогать нельзя —
`icon_key` уже проставлены в базе, код уже их резолвит.

Порядок — по числу предложений, ожидающих проверки:

| Термин | Предложений | Файл |
|---|---:|---|
| Традиционный метод | 29 | `production_traditional_method.svg` |
| Выдержка на осадке | 17 | `production_lees_aging.svg` |
| Винификация в стали | 10 | `production_stainless_steel.svg` |
| Метод Шарма | 8 | `production_charmat_method.svg` |
| En rama | 4 | `style_en_rama.svg` |
| Мацерация | 4 | `production_maceration.svg` |
| Нефильтрованное | 4 | `feature_unfiltered.svg` |
| Спонтанная ферментация | 3 | `production_spontaneous_fermentation.svg` |
| Ферментация в бутылке | 3 | `production_bottle_fermentation.svg` |
| Col Fondo | 1 | `style_col_fondo.svg` |
| Выдержка под флором | 1 | `production_flor_aging.svg` |
| Неосветлённое | 1 | `feature_unfined.svg` |
| Низкий SO₂ | 1 | `feature_low_so2.svg` |
| — | — | `certification_fallback.svg` |
| — | — | `atlas_term_fallback.svg` |

Два последних — общие запасные иконки. `certification_fallback` используется
всеми сертификациями, потому что логотип нельзя показывать без подтверждённого
права. `atlas_term_fallback` — для опубликованного термина, у которого своей
иконки нет.

«Выдержка в дубе», «Ручной сбор», «Органика», «Биодинамика», «Натуральное
вино» и «Pet-Nat» уже имеют готовые ассеты, перерисовывать не нужно.

## 4. Чего рисовать не нужно

- **Ягодные**, **Фруктовые**, **Выдержка и третичные тона** — навигационные
  группы Atlas. Они не назначаются винам и в карточке не показываются.
- **Красные цветы** — термин забракован: все 87 совпадений в корпусе оказались
  «вино красного цвета».
- **Регионы и апелласьоны** — по §6 реестра у них отдельная визуальная
  политика, предметные иконки не нужны.

## 5. Как проверить свою работу

После добавления файлов:

```powershell
flutter test test/features/wines/presentation/wine_features_taste_pairings_test.dart
```

Тесты `every seeded Atlas aroma icon_key has a production asset` и
`every seeded production icon_key has a shipped asset` покажут, если файл, ключ
и enum разошлись.

Проверка размера: иконка должна читаться на **56 px** — на карточке она
показывается именно так, а не в исходном разрешении.
