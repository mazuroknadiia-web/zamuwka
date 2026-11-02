# Підбір 10 рішень для Zamówka (окрема інсталяція для малих гуртівень, PL)

Легенда: ✅ підтверджено джерелом у цій сесії · ⚠️ з моїх знань або джерела розходяться, перевірити до рішення.
Зусилля: S = дні, M = тижні, L = місяці.

## Context

**Задача.** До Zamówka (WooCommerce + власний плагін) додати готові або майже готові, безкоштовні чи недорогі рішення. Тоді мала гуртівня, чиї клієнти замовляють щодня або кілька разів на тиждень, веде весь облік в одному місці. Найважливіше: **виставляння фактур у KSeF** і **обмін із Subiekt (InsERT)**.

**Припущення, яке треба підтвердити:** «субєкт» означає Subiekt GT / nexo / nexo PRO.

**Що є на сайті зараз** (читання через конектор ✅). WordPress 7.1.4, PHP 8.3.36, WooCommerce 11.2.1, WooPayments, `plata by mono` (український платіжний плагін), Zamówka 0.1.0 (db 14, проблем нема). За журналом подій у Zamówka є прайс-листи, реєстрація клієнтів, імпорт каталогу, рекламації, розсилки, правила доставки. **Немає:** фактур і KSeF, складу, обміну з Subiekt, польських платіжних шлюзів. Git-репозиторій порожній, коду плагіна в ньому немає.

**Правове тло KSeF** (✅ кілька польських джерел):
- Обов'язковий з 01.02.2026 для великих платників (понад 200 млн zł), з 01.04.2026 для решти, а для найменших (до 10 000 zł брутто на місяць) з 01.01.2027.
- До 31.12.2026 штрафи за помилки не накладають.
- 20.10 Рада міністрів прийняла **проєкт** відтермінування штрафів до 01.01.2028. Це ще не закон (Bezprawnik, InFakt), і відтермінування стосується лише штрафів, а не самого обов'язку.
- Висновок: без KSeF продукт у Польщі не продати.

## Спершу одне рішення: де «живе» фактура

| | Варіант | Плюси | Мінуси |
|---|---|---|---|
| **A** | Zamówka сама формує FA(3) і шле в KSeF; Subiekt отримує документи імпортом | Нуль комісій, повний контроль, відповідає «весь облік у застосунку» | Більше розробки, відповідальність за offline24, корекції, UPO |
| **B** | Zamówka лише передає замовлення (ZK) у Subiekt; фактуру і KSeF робить Subiekt | Менше коду, бухгалтерія звична | Потрібен Subiekt (Windows, Sfera) у кожного клієнта |
| **C** | Проміжний хаб OpenLinker між WooCommerce, Subiekt, KSeF і кур'єрами | Багато готового | Версія alpha, Node + Postgres + Redis (потрібен VPS) |

**Рекомендація:** A як основа плюс адаптер «Експорт/імпорт у Subiekt». Так можна підтримати і B для клієнтів, які вже мають Subiekt. C оцінити як PoC. **Правило:** на кожній інсталяції лише одне джерело відправляє фактури в KSeF, інакше будуть дублі.

## 10 категорій

### 1. Фактури і KSeF (виставлення і отримання) — головна
| Варіант | Суть | Ліцензія / ціна | Зусилля |
|---|---|---|---|
| **ksef-php-client** (N1ebieski) ✅ | PHP-клієнт KSeF API 2.0: онлайн-сесії, відправка, UPO, QR, DTO для FA(3) → XML. PHP ≥ 8.1 у гілці 1.x. Гілка 2.x вимагає PHP 8.4, а на сайті 8.3.36, тому брати 1.x. | MIT, безкоштовно | M |
| Офіційні бібліотеки MF ✅ | `ksef-client-csharp`, `ksef-client-java`, `ksef-pdf-generator` (TS, PDF з XML) | MIT | S–M (sidecar або референс) |
| Готові плагіни WooCommerce ✅ | Flexible Invoices + KSeF Add-on (WP Desk, 700+ інсталяцій), FakturFlow (безкоштовний v0.3.0; авто-відправка й offline лише в Pro; тільки PLN і польські покупці), Polski for WooCommerce (70+) | ціни ⚠️ | S (зрілість низька, крім WP Desk) |
| SaaS через API ✅ | inFakt (KSeF у базовому тарифі), Fakturownia, wFirma, iFirma | ~5–79 zł/міс. на клієнта, джерела розходяться ⚠️ | S |
| Dolibarr + KSeF-модуль ✅ | Модуль InPoint-Automation: FA(3), online/offline, корекції, імпорт отриманих. Потрібен Dolibarr v22+. «У проді з лютого 2026». PHP/MySQL, як WP. | AGPL-3.0+ | M |
| Odoo ✅ | Нативного KSeF не підтверджено; є платні сторонні модулі (Pokutsoft; inbound ≈ $367) | платно | M |
| Безкоштовний резерв ✅ | MF «Aplikacja Podatnika KSeF 2.0» і мобільний застосунок: вручну, на випадок збою | безкоштовно | S |

### 2. Обмін із Subiekt (друга ключова вимога)
Підтверджено: у InsERT **немає публічного REST API**. Офіційний шлях це SDK і Sfera (✅ документація InsERT і форум). У nexo PRO Sfera входить у ціну, для GT це окремий платний додаток ⚠️. KSeF у Subiekt GT є (офіційні сторінки e-pomoc InsERT про генерацію й імпорт e-faktur ✅), ціна модуля в реселерів розходиться ⚠️.

| Варіант | Суть | Ліцензія / ціна | Зусилля |
|---|---|---|---|
| Файловий імпорт (Importer dokumentów XML/XLS) ✅ | Zamówka генерує XML замовлень чи фактур. Партнерський «Importer wielu zamówień, dokumentów, faktur» (ArtoIT) потребує активної Sfery. Бібліотеки `erpLinker-EDI-XML`, `PreEncje.net` (NuGet) формують XML. | 490 zł/рік (ArtoIT), ліцензія бібліотек не вказана | S–M, односторонній |
| **OpenLinker** ✅ | WooCommerce → Subiekt GT/nexo, KSeF, InFakt, InPost/DPD. Node 22 + NestJS + Postgres + Redis, Docker. Alpha: бракує повернень і тонких ролей. | Apache-2.0 | PoC S, продакшн M |
| Easy Nexo Integrator (NT.NET) ✅ | REST-міст до Subiekt nexo (через Bridge). Версії GT/PRO уточнити. | 900 zł (разово чи на рік ⚠️) | M, двосторонній |
| Власний конектор на Sfera/SDK (.NET) ✅ | Повна двосторонність: ZK → WZ → FS, залишки, ціни. Працює на ПК, де стоїть Subiekt. | Sfera: PRO входить, GT платно ⚠️ | L |
| Base.com (BaseLinker) ✅ | SaaS-інтеграція з Subiekt (ZK/FS, потрібна Sfera). Сумісність із WooCommerce і ціни не підтверджені. | ⚠️ | S |
| Файловий експорт для бухгалтера | EDI++/EPP. Приклад готового: Fakturownia «Eksport InsERT 1.11 (GT, nexo)» ✅ | — | S |

**Підводний камінь:** якщо фактуру в KSeF відправила Zamówka, Subiekt при імпорті не повинен відправляти її вдруге. Також треба узгодити номер KSeF і внутрішній номер (у Comarch Optima для цього є поле «Numer obcy» ✅). Для Subiekt це треба **перевірити на тестовому середовищі**.

### 3. Склад і документи WZ/PZ
Якщо в клієнта є Subiekt, він залишається «master» складу, а Zamówka читає залишки й ціни. Інакше:
| Варіант | Суть | Ліцензія | Зусилля |
|---|---|---|---|
| InvenTree ✅ | Партії, локації, штрихкоди, REST API; Python/Django. Шаблони WZ/PZ треба зробити самим. | MIT | M |
| Dolibarr (склад, відвантаження, прийом) ⚠️ | Тієї ж LAMP-екосистеми, що й WP; разом із KSeF-модулем з п.1 дає міні-ERP | GPL-3.0+ | M |
| WooCommerce stock + ATUM ⚠️ | Мінімум для старту | freemium | S |
| ERPNext / Odoo Community ⚠️ | Повні, але важкі (Python + Postgres). JPK_MAG в Odoo лише платними модулями (≈ від $499, заява вендора) | GPL-3.0 / LGPL-3.0 | L |

### 4. Платежі і розрахунки
Зараз стоять WooPayments і український `plata by mono`, для Польщі потрібні місцеві шлюзи.
| Варіант | Суть | Ліцензія / ціна | Зусилля |
|---|---|---|---|
| Przelewy24, Tpay, Autopay (Blue Media), PayU ⚠️ | BLIK і перекази, готові плагіни WooCommerce | комісія за договором | S |
| Імпорт виписок CAMT.053 (заміна MT940 ✅ mBank) з автозіставленням до номерів фактур | Референси: `ksefnik` (MIT, Node: звірка банку + клієнт KSeF ✅), `firmowid` (AGPL-3.0, Elixir: банківське зіставлення + GoCardless ✅) | безкоштовно | M |
| PSD2-агрегатори (GoCardless, Kontomatik, Salt Edge) ⚠️ | Пізніший етап | платно | M |

### 5. Доставка і логістика
Правила доставки в Zamówka вже є ✅.
| Варіант | Суть | Ліцензія / ціна | Зусилля |
|---|---|---|---|
| Furgonetka ✅ | Плагін (7000+ інсталяцій); також виписує фактури KSeF | вільний/платний ⚠️ | S |
| InPost, DPD ✅ | Адаптери в OpenLinker; окремі плагіни ⚠️ | — | S |
| VROOM + OSRM (самохостинг) | Планування розвозу на день власними бусами; VROOM ✅ BSD-2, OSRM ⚠️ BSD | безкоштовно | M |
| Fleetbase ✅ | Важка платформа логістики | AGPL-3.0 | L |

### 6. Комунікація з клієнтами
Розсилки в Zamówka вже є ✅. Додати канали й сценарії: підтвердження замовлення, нагадування «до котрої приймаємо», нагадування про оплату.
| Варіант | Суть | Ціна | Зусилля |
|---|---|---|---|
| SMSAPI, SerwerSMS ⚠️ | SMS у PL, API | оплата за SMS, ціни не перевірені | S |
| Brevo, Mailgun, Amazon SES, Postmark ⚠️ | Транзакційний e-mail | є безкоштовні ліміти | S |
| WhatsApp Cloud API, Telegram, Web Push (PWA) ⚠️ | Безкоштовні канали для постійних клієнтів | — | M |

### 7. Звіти і BI
| Варіант | Суть | Ліцензія | Зусилля |
|---|---|---|---|
| Звіти Zamówka ✅ | База | — | — |
| Metabase OSS / Apache Superset ⚠️ | Читання БД (read-only користувач): маржа, ABC, RFM клієнтів | AGPL / Apache-2.0 | S |
| Looker Studio ⚠️ | Безкоштовно, через CSV/Sheets | — | S |

### 8. Закупівельні фактури, документи, архів
| Варіант | Суть | Ліцензія | Зусилля |
|---|---|---|---|
| Отримання з KSeF як XML ✅ | OCR не потрібен; Subiekt сам перетворює e-faktury на закупівлі (e-pomoc InsERT ✅). У застосунку: ksef-php-client + ksef-pdf-generator. | MIT | M |
| KSeF_Monitor ✅ | Python: опитує KSeF, сповіщення, PDF; референс (ліцензію не вказано) | ? | S |
| Paperless-ngx ⚠️ | Архів і OCR для документів поза KSeF (накладні, договори) | GPL-3.0 | S |

### 9. CRM і торгові представники
У v1 окремої CRM не додавати: картки клієнтів і чернетки замовлень уже є в Zamówka ✅ (за інструментами конектора). Додати PWA для менеджерів у полі. Далі ⚠️: EspoCRM (AGPL-3.0, PHP, легка), SuiteCRM (AGPL-3.0, важча), Dolibarr CRM (якщо вибрано Dolibarr).

### 10. Дистрибуція «окремої інсталяції»
| Варіант | Суть | Зусилля |
|---|---|---|
| Ліцензії й оновлення ⚠️ | Freemius, Lemon Squeezy/Paddle, свій сервер оновлень (Plugin Update Checker), Keygen | S–M |
| Розгортання | Скрипт WP-CLI / Docker Compose / Ansible (на сайті `cli_tools` увімкнено ✅). KSeF-токен чи сертифікат кожної гуртівні зберігати зашифровано, мультитенантності не потрібно. | M |
| Резервні копії і моніторинг ⚠️ | UpdraftPlus або restic → S3; Uptime Kuma (MIT), Sentry / GlitchTip; перевірка черги відправки в KSeF | S |
| Ліцензування ⚠️ | Плагіни WordPress мають бути GPL-сумісні — уточнити в юриста | — |

## Рекомендований порядок (MVP)
1. **KSeF.** PoC на тестовому середовищі KSeF з `ksef-php-client` 1.x. Паралельно оцінити «купити замість будувати»: Flexible Invoices + KSeF Add-on або API inFakt.
2. **Subiekt.** Етап 1: генерація XML для Importer dokumentów. Етап 2: PoC OpenLinker і/або Sfera-конектор.
3. **Платежі.** Польські шлюзи + імпорт CAMT.053.
4. Решта (склад, доставка, SMS, BI, DMS, CRM) — за потребою перших клієнтів.

## Рішення, потрібні від вас
1. Архітектура A / B / C (де живе фактура).
2. Яка версія Subiekt у цільових клієнтів: GT, nexo чи nexo PRO.
3. Хостинг клієнтів: звичайний PHP-хостинг чи VPS з Docker (від цього залежить OpenLinker).
4. Чи замовлення Zamówka завжди є замовленнями WooCommerce (від цього залежить, чи годяться плагіни WooCommerce).

## Перевірка (як тестувати кандидатів)
- **KSeF:** тестовий платник і токен у тестовому середовищі KSeF. Відправити FA(3) через `ksef-php-client`, отримати UPO, перевірити корекцію і режим offline24.
- **Subiekt:** demo-ліцензії GT і nexo PRO (demo-ліцензії існують ✅). Імпортувати тестовий XML, перевірити нумерацію, дублікати та повторну відправку в KSeF.
- **OpenLinker:** `pnpm demo:up` (Docker) ✅ і підключення тестового WooCommerce.
- Усі пункти ⚠️ (ціни, ліцензії) перевірити в первинному джерелі до закупівлі.

## Обмеження дослідження
- Про Subiekt GT і nexo багато джерел це сторінки реселерів, які суперечать одне одному (наприклад, чи закінчилась підтримка GT, ціна модуля KSeF). Офіційне підтвердження InsERT щодо цього не знайдено.
- Ціни SMS, BaseLinker, платіжних шлюзів не перевірялись.
- Код Zamówka недоступний (репозиторій порожній), тому інтеграцію з її моделлю замовлень оцінено лише за назвами подій і інструментів конектора.

## Джерела
- KSeF терміни: [kompasfirmy.pl](https://kompasfirmy.pl/ksef-od-kiedy/), [LinTax](https://www.lintax.pl/aktualnosci/ksef-2026-konczy-sie-okres-przejsciowy-co-zmieni-sie-od-1-stycznia-2027-roku), [Bezprawnik](https://bezprawnik.pl/ksef-dla-najmniejszych-firm/), [InFakt](https://www.infakt.pl/blog/kary-w-ksef-od-2028-r-wazna-zmiana-dla-firm/)
- KSeF бібліотеки: [CIRFMF](https://github.com/CIRFMF), [ksef-php-client](https://github.com/N1ebieski/ksef-php-client), [GitHub topic ksef](https://github.com/topics/ksef)
- WooCommerce-плагіни: [FakturFlow](https://wordpress.org/plugins/fakturflow-ksef)
- Dolibarr: [модуль KSeF](https://github.com/InPoint-Automation/Dolibarr-KSeF-Module)
- Odoo: [l10n_pl_ksef_connector](https://apps.odoo.com/apps/modules/19.0/l10n_pl_ksef_connector), [l10n_pl_ksef_inbound](https://apps.odoo.com/apps/modules/19.0/l10n_pl_ksef_inbound)
- Безкоштовні інструменти MF: [Aplikacja Podatnika KSeF 2.0](https://ksef.podatki.gov.pl/aplikacja-podatnika-ksef-20/)
- Subiekt: [InsERT SDK](https://www.insert.com.pl/dla_uzytkownikow/e-pomoc_techniczna/4708,insert-nexo-co-to-jest-i-skad-pobrac-sdk-szczegolowa-dokumentacja-techniczna.html), [InsERT: KSeF w Subiekt GT](https://www.insert.com.pl/dla_uzytkownikow/e-pomoc_techniczna/11213,subiekt-gt-%E2%80%93-konfiguracja-parametrow-zwiazanych-z-generowaniem-i-importem-e-faktur-ksef.html), [ArtoIT importer](https://www.insert.com.pl/dla_uzytkownikow/subiekt_gt/dodatki_rozszerzenia/rozwiazania_partnerskie/2388.html), [NuGet artoit](https://www.nuget.org/profiles/artoit), [Easy Nexo Integrator](http://easynexointegrator.pl/), [Fakturownia: eksport InsERT](https://fakturownia.pl/apps/insert_export)
- Хаб: [OpenLinker](https://github.com/openlinker-project/openlinker), [firmowid](https://github.com/Alergeek-Ventures/firmowid)
- Логістика: [Fleetbase](https://github.com/fleetbase/fleetbase), [VROOM](https://awesome.ecosyste.ms/projects/github.com%2FVROOM-Project%2Fvroom)
- Банк: [mBank CAMT.053](https://www.mbank.pl/pdf/msp-korporacje/bankowosc-elektroniczna/opis-formatu-pliku-eksportu-wyciagow-dziennych-camt.053.001.08.pdf)
- Закупівельні фактури з KSeF: [fillup.pl](https://www.fillup.pl/kanalksiegowy24/cms/att/ebooks/poradniki-ksef/proces-obslugi-faktur.pdf), [Cairo ERP](https://pomoc.cairo.pl/books/ksef/page/import-okresowy-dokumentow-pz-z-ksef/export/pdf)
