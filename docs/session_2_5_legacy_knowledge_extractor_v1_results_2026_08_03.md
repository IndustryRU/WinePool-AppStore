# Session 2.5 — Legacy knowledge extractor v1: dry-run results

Дата: 03.08.2026

Статус: код и production read-only dry-run завершены; database materialization
не выполнялась.

Extractor: `legacy-knowledge-extractor.v1`.

## 1. Реализованный безопасный контур

Добавлены:

- `tool/legacy_knowledge_profile/legacy_knowledge_profile.py`;
- unit/regression tests;
- PowerShell runner с production snapshot внутри `READ ONLY`-транзакции;
- локальные JSON/JSONL/Markdown dry-run artifacts в Git-ignored output.

Extractor:

- не содержит database connection или canonical write path;
- не вызывает AI и web search;
- читает wines, Atlas terms/aliases и immutable v3 reports как JSONL;
- разделяет aroma, pairing, method, eco/style и v3 knowledge;
- учитывает отрицания и explicit release year;
- подавляет часть broad/specific дублей;
- remap-ит известные wrong-kind v3 values;
- объединяет повтор одного semantic v3 candidate в пределах report;
- формирует только preview proposals и clusters.

## 2. Финальный production dry-run

Локальный run:
`tool/legacy_knowledge_profile/output/20260803_201208/`.

| Метрика | Значение |
|---|---:|
| Прочитано wine descriptions | 1 042 |
| Прочитано Atlas terms | 1 |
| Прочитано v3 reports | 52 |
| Preview proposals | 2 867 |
| Из legacy descriptions | 2 631 |
| Из v3 reports | 236 |
| Term assignments preview | 2 854 |
| Awards preview | 13 |
| Candidate clusters | 204 |
| Review | 2 854 |
| Conflicts | 13 |
| Safe candidates | 0 |

Legacy extractor нашёл хотя бы один поддержанный текущими правилами candidate у
695 вин. Это 66,7% всех описанных вин. Остальной текст не потерян: v1 намеренно
консервативен и не превращает любое существительное из narrative в term.

V3 adapter повторно использовал 52 отчёта, связанных с 46 винами, и сократил
сырой поток повторяющихся находок до 236 preview proposals без нового research.

## 3. Распределение term assignments

| Kind | Preview assignments |
|---|---:|
| Aroma | 2 098 |
| Pairing | 454 |
| Method | 147 |
| Region | 50 |
| Appellation | 35 |
| Feature | 26 |
| Eco | 19 |
| Approach | 11 |
| Certification | 11 |
| Style | 3 |

## 4. Наиболее сильные legacy clusters

### 4.1. Ароматы

| Term candidate | Вин |
|---|---:|
| Вишня | 363 |
| Чёрная смородина | 173 |
| Ежевика | 145 |
| Тёрн | 124 |
| Слива | 103 |
| Черника | 94 |
| Пряности | 94 |
| Шелковица | 93 |
| Чернослив | 78 |
| Лакрица | 74 |
| Малина | 73 |
| Гранат | 59 |
| Табак | 48 |
| Брусника | 47 |
| Ваниль | 47 |
| Шоколад | 46 |
| Красная смородина | 43 |
| Клюква | 36 |
| Фиалка | 33 |
| Яблоко | 31 |
| Чёрная слива | 31 |
| Чёрный перец | 29 |
| Груша | 21 |
| Цитрусовые | 20 |
| Кофе | 20 |
| Мёд | 16 |
| Персик | 15 |
| Грейпфрут | 13 |
| Лимон | 12 |
| Роза | 12 |
| Зелёное яблоко | 10 |
| Белые цветы | 10 |
| Кедр | 9 |

### 4.2. Pairing

| Term candidate | Вин |
|---|---:|
| Закуски | 55 |
| Говядина/стейк | 47 |
| Сыр | 45 |
| Овощи | 34 |
| Баранина | 31 |
| Выдержанный сыр | 28 |
| Морепродукты | 25 |
| Утка | 24 |
| Паста | 24 |
| Птица | 24 |
| Мясо на гриле | 23 |
| Рыба | 21 |
| Грибы | 18 |
| Пицца | 13 |
| Дичь | 12 |
| Свинина | 11 |
| Десерты | 10 |

### 4.3. Методы и сигналы

| Term candidate | Вин |
|---|---:|
| Выдержка в дубе | 32 |
| Традиционный метод | 16 |
| Выдержка на осадке | 15 |
| Ручной сбор | 8 |
| Винификация в стали | 7 |
| Метод Шарма | 6 |
| Ферментация в бутылке | 3 |
| Мацерация | 3 |

## 5. QA и найденные ошибки до materialization

Автоматические regression cases покрывают:

- aroma/pairing/production extraction;
- negative claim;
- release year scope;
- broad/specific suppression;
- wrong-kind v3 remap;
- semantic v3 dedup;
- локальную генерацию всех artifacts;
- запрет трактовать «ароматические ноты сочетаются» как pairing;
- запрет трактовать «розовый грейпфрут» как аромат розы.

Все 9 тестов проходят.

Ручная выборка top clusters показала корректные evidence fragments для вишни,
сливы, чёрной сливы, зелёного яблока, цитрусов, мёда, белых цветов, кожи, кофе,
выдержанного сыра, птицы, мяса на гриле и дубовой выдержки. Два найденных
ложноположительных класса (`розовый грейпфрут`, aroma `сочетается`) исправлены и
закреплены тестами.

## 6. Важная Atlas-проблема

Единственный текущий published term `Чёрная слива` содержит alias `Слива`.
Поэтому:

- 31 точное упоминание `чёрная слива` корректно match-ится как exact;
- 103 обычных упоминания `слива` match-ятся через слишком широкий alias;
- обычная слива не должна автоматически становиться чёрной сливой.

До materialization необходимо удалить/сузить alias `Слива` у `Чёрной сливы` и
создать отдельный canonical term `Слива`, если продуктово нужны оба уровня.
Extractor правильно оставил все alias matches в review, поэтому public data не
затронута.

## 7. Почему safe candidates равно нулю

Это ожидаемый результат, а не ошибка:

- Atlas содержит только один term;
- legacy descriptions имеют неизвестный per-row provenance и начинаются как
  `source_claimed`;
- v3 candidates почти не могут exact-match пустой taxonomy;
- wrong-kind и unknown release scope требуют review;
- текущий срез намеренно не создаёт Atlas terms автоматически.

Safe candidates появятся после утверждённого taxonomy seed и повторного
идемпотентного dry-run.

## 8. Scope

Все 2 631 legacy proposals получили wine scope: в legacy descriptions не нашлось
однозначной пары «явный год + существующий release» для текущих 25 release rows.
Это подтверждает, что нельзя искусственно привязывать старые описания к релизам.

Для v3:

- 8 award proposals имеют release hint;
- 228 proposals пока имеют unknown scope и требуют resolution/moderation.

## 9. Рекомендуемый следующий шаг

До unified proposal materialization нужен короткий product checkpoint:

1. утвердить первую волну canonical Atlas seed на основе top clusters;
2. определить family/specific policy: например, `Цитрусовые` против
   `Лимон/Лайм/Грейпфрут`, `Сыр` против `Выдержанный сыр`;
3. исправить alias `Слива` у `Чёрной сливы`;
4. связать seed terms с существующими icon keys, где assets уже есть;
5. повторить v1 dry-run и получить exact/alias coverage;
6. после этого реализовать unified proposals schema и materialization.

Без этого checkpoint создание сразу 204 Atlas terms приведёт к загрязнению
справочника одноразовыми v3-формулировками и пересекающимися категориями.
