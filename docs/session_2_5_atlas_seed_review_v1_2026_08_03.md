# Session 2.5 — Atlas seed review v1

Дата: 03.08.2026  
Обновлено: 04.08.2026

Статус: taxonomy и первые contact sheets утверждены владельцем; production
seed и изменение aliases ещё не выполнялись; individual icon production идёт
сериями.

Основание:

- production baseline из 1 042 descriptions;
- dry-run `legacy-knowledge-extractor.v1`;
- 2 867 preview proposals и 204 raw clusters;
- утверждённая владельцем иерархия general → specific;
- утверждённое разделение `Слива` и `Чёрная слива`.

## 1. Правила реестра

1. General и specific terms могут одновременно существовать в Atlas.
2. В карточке specific term подавляет своего general parent, если оба получены
   из одного evidence fragment.
3. General term остаётся назначаемым, когда источник не даёт уточнения.
4. Синонимы и словоформы — aliases, а не новые canonical terms.
5. Иконка привязывается к canonical term через стабильный `icon_key`.
6. Все public-facing aroma и pairing terms получают индивидуальную иконку.
7. Parent/family term также получает иконку, если может отображаться в карточке.
8. Region/appellation/certification не требуют уникальной предметной картинки
   на каждый term: для них используется отдельная визуальная политика.

## 2. Aroma taxonomy

Обозначения assets:

- `готова` — production PNG уже существует;
- `новая` — нужна новая иконка в утверждённой gold/amber стилистике;
- `family` — общий назначаемый parent, также нуждается в иконке.

### 2.1. Назначаемые family terms

| Canonical term | Slug | Parent | Назначение | Asset |
|---|---|---|---|---|
| Красные ягоды | `red-berries` | Ягодные | общий descriptor | готова: `aroma_family_red_berries_72.png` |
| Чёрные ягоды | `black-berries` | Ягодные | общий descriptor | готова: `aroma_family_black_berries_72.png` |
| Косточковые фрукты | `stone-fruit` | Фруктовые | общий descriptor | готова: `aroma_family_stone_fruit_72.png` |
| Садовые фрукты | `orchard-fruit` | Фруктовые | общий descriptor | готова: `aroma_family_orchard_fruit_72.png` |
| Цитрусовые | `citrus` | Фруктовые | общий descriptor и найденный cluster | готова: `aroma_family_citrus_72.png` |
| Тропические фрукты | `tropical-fruit` | Фруктовые | общий descriptor и найденный cluster | готова: `aroma_family_tropical_fruit_72.png` |
| Цветочные ноты | `floral` | — | общий descriptor | готова: `aroma_family_floral_72.png` |
| Пряности | `spices` | — | общий descriptor и найденный cluster | готова: `aroma_family_spices_72.png` |

`Ягодные` и `Фруктовые` используются как навигационные Atlas groups и не
назначаются напрямую, пока источник не использует именно такую формулировку.

### 2.2. Specific aroma terms

| Canonical term | Slug | Parent | Вин в dry-run | Asset |
|---|---|---|---:|---|
| Вишня | `cherry` | Красные ягоды / косточковые | 363 | готова: `aroma_cherry_sour_72.png` |
| Чёрная смородина | `black-currant` | Чёрные ягоды | 173 | готова: `aroma_black_currant_72.png` |
| Ежевика | `blackberry` | Чёрные ягоды | 145 | готова: `aroma_blackberry_72.png` |
| Тёрн | `sloe` | Косточковые фрукты | 124 | готова: `aroma_sloe_72.png` |
| Слива | `plum` | Косточковые фрукты | 103 | готова: `aroma_plum_72.png` |
| Черника / голубика | `blueberry` | Чёрные ягоды | 94 | готова: `aroma_blueberry_72.png` |
| Шелковица | `mulberry` | Чёрные ягоды | 93 | готова: `aroma_mulberry_72.png` |
| Чернослив | `prune` | Сухофрукты | 78 | готова: `aroma_prune_72.png` |
| Лакрица / солодка | `licorice` | Пряности | 74 | готова: `aroma_licorice_72.png` |
| Малина | `raspberry` | Красные ягоды | 73 | готова: `aroma_raspberry_72.png` |
| Гранат | `pomegranate` | Фруктовые | 59 | готова: `aroma_pomegranate_72.png` |
| Табак | `tobacco` | Выдержка/третичные | 48 | готова: `aroma_tobacco_72.png` |
| Брусника | `lingonberry` | Красные ягоды | 47 | готова: `aroma_lingonberry_72.png` |
| Ваниль | `vanilla` | Пряности | 47 | готова: `aroma_vanilla_72.png` |
| Шоколад | `chocolate` | Выдержка/третичные | 46 | готова: `aroma_cacao_dark_chocolate_72.png` |
| Красная смородина | `red-currant` | Красные ягоды | 43 | готова: `aroma_red_currant_72.png` |
| Смородина | `currant` | Ягодные | 37 | готова: `aroma_currant_72.png` |
| Клюква | `cranberry` | Красные ягоды | 36 | готова: `aroma_cranberry_72.png` |
| Фиалка | `violet` | Цветочные ноты | 33 | готова: `aroma_violet_72.png` |
| Яблоко | `apple` | Садовые фрукты | 31 | готова: `aroma_apple_72.png` |
| Чёрная слива | `black-plum` | Слива | 31 | готова: `aroma_plum_black_72.png` |
| Чёрный перец | `black-pepper` | Пряности | 29 | готова: `aroma_black_pepper_72.png` |
| Груша | `pear` | Садовые фрукты | 21 | готова: `aroma_pear_72.png` |
| Кофе | `coffee` | Выдержка/третичные | 20 | готова: `aroma_coffee_72.png` |
| Мёд | `honey` | — | 16 | готова: `aroma_honey_72.png` |
| Персик | `peach` | Косточковые фрукты | 15 | готова: `aroma_peach_72.png` |
| Грейпфрут | `grapefruit` | Цитрусовые | 13 | готова: `aroma_grapefruit_72.png` |
| Лимон | `lemon` | Цитрусовые | 12 | готова: `aroma_lemon_72.png` |
| Роза | `rose` | Цветочные ноты | 12 | готова: `aroma_rose_72.png` |
| Зелёное яблоко | `green-apple` | Яблоко | 10 | готова: `aroma_green_apple_72.png` |
| Белые цветы | `white-flowers` | Цветочные ноты | 10 | готова: `aroma_white_flowers_72.png` |
| Кедр | `cedar` | Выдержка/третичные | 9 | готова: `aroma_cedar_72.png` |
| Лайм | `lime` | Цитрусовые | 8 | готова: `aroma_lime_72.png` |
| Дуб | `oak` | Выдержка/третичные | 7 | готова: `aroma_oak_72.png` |
| Апельсин | `orange` | Цитрусовые | 7 | готова: `aroma_orange_72.png` |
| Абрикос | `apricot` | Косточковые фрукты | 6 | готова: `aroma_apricot_72.png` |
| Кожа | `leather` | Выдержка/третичные | 6 | готова: `aroma_leather_72.png` |

Дополнительные strategic terms для extractor/Atlas, даже если в v1 corpus у
них менее пяти совпадений: `Ананас`, `Манго`, `Клубника`, `Красные цветы`,
`Сухофрукты`, `Дым`, `Минеральность`, `Бриошь/тост`. Их включать в первую
иконографическую волну только после отдельного sample check, чтобы не смешать
ароматы, вкус и производственные признаки.

### 2.3. Утверждённое исправление сливы

- создать canonical `Слива` (`plum`);
- сохранить canonical `Чёрная слива` (`black-plum`);
- удалить alias `Слива` и `Plum` у `Чёрная слива`;
- оставить aliases `Чёрная слива`, `Черная слива`, `Black plum`;
- словоформы обычной сливы направлять в `Слива`;
- specific `Чёрная слива` подавляет `Слива` в одном evidence fragment.

## 3. Pairing taxonomy

| Canonical term | Slug | Parent | Вин | Asset |
|---|---|---|---:|---|
| Закуски | `appetizers` | — | 55 | готова: `pairing_appetizers.png` |
| Говядина | `beef` | Мясные блюда | 47* | готова: `pairing_beef.png` |
| Стейк | `steak` | Мясные блюда | 47* | готова: `pairing_steak.png` |
| Сыр | `cheese` | — | 45 | готова: `pairing_cheese.png` |
| Овощи | `vegetables` | Растительные блюда | 34 | готова: `pairing_vegetables.png` |
| Баранина | `lamb` | Мясные блюда | 31 | готова: `pairing_lamb.png` |
| Выдержанный сыр | `aged-cheese` | Сыр | 28 | готова: `pairing_aged_hard_cheese.png` |
| Морепродукты | `seafood` | Рыба и морепродукты | 25 | готова: `pairing_seafood.png` |
| Утка | `duck` | Птица | 24 | готова: `pairing_duck.png` |
| Паста | `pasta` | — | 24 | готова: `pairing_tomato_pasta.png` (визуально используется как pasta) |
| Птица | `poultry` | — | 24 | готова: `pairing_poultry.png` |
| Мясо на гриле | `grilled-meat` | Мясные блюда | 23 | готова: `pairing_grilled_red_meat.png` |
| Рыба | `fish` | Рыба и морепродукты | 21 | готова: `pairing_fish.png` |
| Грибы | `mushrooms` | Растительные блюда | 18 | готова: `pairing_mushrooms.png` |
| Пицца | `pizza` | — | 13 | готова: `pairing_pizza.png` |
| Дичь | `game` | Мясные блюда | 12 | готова: `pairing_game.png` |
| Свинина | `pork` | Мясные блюда | 11 | готова: `pairing_pork.png` |
| Десерты | `desserts` | — | 10 | готова: `pairing_desserts.png` |
| Тёмный шоколад | `dark-chocolate` | Десерты | 9 | готова: `pairing_dark_chocolate.png` |
| Мясные блюда | `meat-dishes` | — | family | готова: `pairing_family_meat_dishes.png` |
| Рыба и морепродукты | `fish-and-seafood` | — | family | готова: `pairing_family_fish_seafood.png` |
| Растительные блюда | `plant-dishes` | — | family | готова: `pairing_family_plant_dishes.png` |

`47*` — общий raw cluster `говядина/стейк` в dry-run. Перед записью assignments
extractor обязан разделить его на два canonical term по конкретной
формулировке; значения строк не суммируются.

Назначаемые general parents получили отдельные public-facing icons:
`Мясные блюда`, `Рыба и морепродукты`, `Растительные блюда`; для `Пасты`
используется существующая композиция `pairing_tomato_pasta.png`.

Правила подавления:

- `Выдержанный сыр` подавляет `Сыр` из одного fragment;
- будущая specific `Паста с томатным соусом` сможет подавлять general `Паста`;
- specific мясо не обязано подавлять `Мясо на гриле`: баранина на гриле может
  корректно дать оба разных пользовательских сигнала;
- `Тёмный шоколад` подавляет `Десерты`, только если general dessert не указан
  отдельно.

## 4. Production, eco и style normalization

### 4.1. Первая public wave

| Canonical term | Kind | Raw merge | Asset |
|---|---|---|---|
| Выдержка в дубе | method | oak/barrel/barrique/partial oak aging | готова: `production_oak_barrel.svg` |
| Ручной сбор | method | manual/hand harvest, feature variants | готова: `production_hand_harvest.svg` |
| Традиционный метод | method | traditional, classic, champenoise, metodo classico | placeholder: `production_traditional_method.svg` |
| Метод Шарма | method | charmat variants | placeholder: `production_charmat_method.svg` |
| Выдержка на осадке | method | lees aging, sur lie, duration in value | placeholder: `production_lees_aging.svg` |
| Винификация в стали | method | stainless fermentation/vinification/vats | placeholder: `production_stainless_steel.svg` |
| Ферментация в бутылке | method | bottle fermentation | placeholder: `production_bottle_fermentation.svg` |
| Мацерация | method | maceration variants | placeholder: `production_maceration.svg` |
| Спонтанная ферментация | method | natural/spontaneous fermentation | placeholder: `production_spontaneous_fermentation.svg` |
| Выдержка под флором | method | flor aging variants | placeholder: `production_flor_aging.svg` |
| En rama | method | en rama variants | placeholder: `style_en_rama.svg` |
| Col Fondo | method/style | col fondo | placeholder: `style_col_fondo.svg` |
| Органика | eco | organic product claim | готова: `eco_organic.svg` |
| Биодинамика | eco | biodynamic product claim | готова: `eco_biodynamic.svg` |
| Натуральное вино | eco/style | only explicit natural claim | готова: `style_natural.svg` |
| Pet-Nat | style | Pet-Nat explicit claim | готова: `style_pet_nat.svg` |
| Нефильтрованное | feature | unfiltered | placeholder: `feature_unfiltered.svg` |
| Неосветлённое | feature | unfined | placeholder: `feature_unfined.svg` |
| Низкий SO₂ | feature | low SO2 | placeholder: `feature_low_so2.svg` |

Числовые уточнения (`12 месяцев`, `19 месяцев`) хранятся в assignment value, а
не создают отдельные Atlas terms.

### 4.2. Не создавать как method без исправления

| Raw value | Решение |
|---|---|
| `ordinary` | категория/качество, не method |
| `red-still` | базовый type/color, не method |
| `spatlese` | classification/style, отдельная проработка |
| `young-wine` | style, не method |
| `unaged` | style/feature, не method |
| `light-sparkling-style` | style, не method |
| `Vinho Verde style` | region/appellation/style review |
| длинные compound methods | разложить на несколько assignments + values |

## 5. Certifications

Первая нормализация:

- `EU Organic Bio` + `EU organic agriculture` → `EU Organic`;
- `Demeter` остаётся отдельной сильной certification;
- `USDA Organic` отдельно;
- `BRC`, `IFS`, `IFS Food` относятся к food-safety certification и не должны
  визуально выдаваться за экологический признак вина;
- `SQNPI` и `Код национальной устойчивости Чили` требуют article/source review;
- generic `Органическая сертификация` остаётся review до определения issuer;
- `AOC` не certification, а appellation.

Для certification используется знак/логотип только при подтверждённом праве на
использование. Иначе — единая premium fallback icon certification.

## 6. Regions и appellations

Raw v3 clusters содержат дубли и смешение уровней:

- `Крым` / `Krym`;
- `Кубань` / `Kub`;
- три варианта `Краснодарский край`;
- `ЗГУ Кубань`, `Кубань. Таманский полуостров`, `ЗГУ «Кубань. Таманский
  полуостров»`;
- region `Шампань` и appellation `Шампань AOC`;
- generic `AOC`, `DOC`, `ЗГУ`, `VSQ` без конкретной зоны.

Они не входят в первую предметную icon wave. Region использует map/region
visual, appellation — существующий `origin_appellation.svg`. Перед seed они
должны match-иться с canonical region tables и не создавать второй справочник
географии.

## 7. Иконографическая оценка первой волны

На 04.08.2026 готовы:

- 45 aroma PNG: обновлённые исходные 8, 8 family assets и production-серии `black-currant`, `blackberry`,
  `sloe`, `plum`, `blueberry`, `mulberry`, `prune`, `licorice`, `raspberry`,
  `pomegranate`, `lingonberry`, `red-currant`, `currant`, `cranberry`,
  `violet`, `apple`, `pear`, `coffee`, `honey`, `peach`, `grapefruit`,
  `lemon`, `rose`, `green-apple`, `white-flowers`, `lime`, `oak`, `orange`,
  `apricot`, а также все назначаемые family terms;
- 22 pairing PNG: 8 исходных specific assets, 11 новых specific compositions
  и 3 family compositions;
- 2 production SVG;
- organic, biodynamic, natural и Pet-Nat SVG;
- award fallback medals и appellation fallback.

После утверждения этого реестра ориентировочно нужны:

- обязательных aroma icons по утверждённому реестру больше не осталось;
- обязательных pairing icons по утверждённому реестру больше не осталось;
- production/feature, certification и generic Atlas slots закрыты временными
  SVG-заглушками под окончательными именами; заменить нужно только содержимое
  файлов, не API и не пути.

Иконки создаются сериями и сначала показываются contact sheets. Только после
визуального утверждения отдельные transparent assets попадают в production.

### 7.1. Визуальное выравнивание ранних aroma assets

После сопоставления с production-сериями от 04.08.2026 восемь самых ранних
иконок были признаны менее объёмными и насыщенными и перерисованы двумя
сериями по четыре с сохранением имён файлов и resolver:

1. `cherry`, `cedar`, `tobacco`, `vanilla`;
2. `black-pepper`, `chocolate`, `leather`, `black-plum`.

`violet` дополнительно проверена на 72 px и обновлена светлым аметистовым
вариантом: золотой контур и тёмная сердцевина сохранены, лепестки лучше читаются
на тёмном фоне карточки.

### 7.2. Утверждённые назначаемые family icons

Групповая иконка строится как компактная композиция из двух-трёх узнаваемых
представителей, а не как увеличенная копия одного specific aroma. Это визуально
отделяет Atlas family от конкретного назначенного descriptor.

| Family | Композиция | Файл |
|---|---|---|
| Красные ягоды | малина, красная смородина и вишня | `aroma_family_red_berries_72.png` |
| Чёрные ягоды | ежевика, чёрная смородина и голубика | `aroma_family_black_berries_72.png` |
| Косточковые фрукты | слива, абрикос и косточка в центре | `aroma_family_stone_fruit_72.png` |
| Садовые фрукты | яблоко и груша на общей ветви | `aroma_family_orchard_fruit_72.png` |
| Цитрусовые | три веерных дольки апельсина, лимона и лайма | `aroma_family_citrus_72.png` |
| Тропические фрукты | манго, срез маракуйи и лист ананаса | `aroma_family_tropical_fruit_72.png` |
| Цветочные ноты | небольшая роза, фиалка и белый цветок | `aroma_family_floral_72.png` |
| Пряности | крупный бадьян, кардамон, гвоздика, перец и одна короткая корица | `aroma_family_spices_72.png` |

Все family compositions сохраняют воздух между элементами и более эмблемную
геометрию, чтобы оставаться читаемыми на 56 px.

### 7.3. Техническое хранение hierarchy

Hierarchy является DAG, а не простым деревом. Например, `Вишня` может иметь
parents `Красные ягоды` и `Косточковые фрукты`. Поэтому связи хранятся в
`atlas_term_relations`, а не в одном `parent_id` или невалидируемом metadata.
Один child может иметь несколько parents; циклы и связь разных kinds
запрещаются admin RPC.

## 8. Предлагаемый порядок подтверждения

1. Aroma names и hierarchy.
2. Pairing names и hierarchy.
3. Public production/eco/style wave.
4. Icon contact sheet первой aroma wave.
5. Icon contact sheet pairing wave.
6. SVG production/features.
7. Seed dry-run и exact/alias coverage report.

До прохождения пунктов 1–3 Atlas seed в production не применяется.
