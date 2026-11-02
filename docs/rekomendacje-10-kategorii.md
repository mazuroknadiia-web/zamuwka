# Zamówka: готові рішення для окремих інсталяцій гуртівень (v2, WordPress-first)

Легенда: ✅ підтверджено джерелом у цій сесії · ⚠️ з моїх знань або джерела розходяться, перевірити до рішення.
Зусилля: S = дні, M = тижні, L = місяці.

## Вихідні умови (підтверджено власником)

- Усе працює на хостингу, який купив власник продукту. Кожна гуртівня має **власну WP-інсталяцію зі своїм субдоменом**.
- Zamówka це плагін WordPress над WooCommerce і може користуватися всіма його функціями. **Центр усього тут.**
- Різним клієнтам потрібні різні рішення.

Що з цього випливає:
1. **Лише PHP/WP-рішення на самому хостингу.** Docker, Node, Python і окремі сервери (OpenLinker, InvenTree, Metabase, Paperless-ngx, VROOM/OSRM, Dolibarr) не годяться як основа. Їх можна лишити як опцію для клієнтів із VPS. Поточне середовище (✅ конектор): WordPress 7.1.4, PHP 8.3.36, WooCommerce 11.2.1, Zamówka 0.1.0.
2. **Модульність.** Ядро (Zamówka + WooCommerce) плюс змінні «драйвери» на кожну категорію, які вмикаються під клієнта.
3. **Subiekt стоїть на ПК клієнта (Windows)**, а не на хостингу. Прямий виклик Sfera з WordPress неможливий. Обмін іде через файли/FTP або через міст на ПК клієнта.

**KSeF, правове тло** (✅ кілька польських джерел). Обов'язковий з 01.04.2026, для найменших платників (до 10 000 zł брутто на місяць) з 01.01.2027. До 31.12.2026 штрафів за помилки немає. 20.10 Рада міністрів прийняла **проєкт** відтермінування штрафів до 01.01.2028, це ще не закон (Bezprawnik, InFakt).

## Профілі клієнтів → набір драйверів

| Профіль клієнта | Фактури / KSeF | Обмін з обліком | Примітка |
|---|---|---|---|
| **P1. Без ERP**, усе у WP | Модуль KSeF у Zamówka або Flexible Invoices + KSeF Add-on | CSV/XML-експорт для бухгалтера | Найпростіший і найдешевший варіант |
| **P2. Subiekt GT** | Фактури в Zamówka, або фактуру і KSeF робить Subiekt GT (модуль KSeF є ✅) | **LEO Synchronizator Platform Internetowych**: XML через FTP, від 69 zł/міс. ✅ | Потрібна Sfera для GT (платно ⚠️) |
| **P3. Subiekt nexo PRO** | Фактуру і KSeF робить nexo (за реселерами ⚠️), або Zamówka | **LEO Synchronizator WooCommerce**: ZK → фактура/paragon + PDF клієнту, від 89 zł/міс. ✅. Альтернатива: Easy Nexo Integrator (REST, 900 zł ✅) | Синхронізацію залишків і цін LEO для nexo не описує, уточнити |
| **P4. SaaS-бухгалтерія** (inFakt, Fakturownia, wFirma, iFirma) | Фактура і KSeF у сервісі (KSeF у inFakt без доплати ✅) | API-драйвер | Щомісячна плата клієнта (~5–79 zł, джерела розходяться ⚠️) |
| **P5. Інший ERP** (Optima, Symfonia, enova…) | Залежить від ERP | Загальний файловий експорт | Не досліджено |

**Правило:** на кожній інсталяції рівно одне джерело відправляє фактури в KSeF (Zamówka, Subiekt або SaaS), інакше будуть дублі.

## 10 категорій (WordPress-first)

### 1. Фактури і KSeF — головна
| Варіант | Суть | Ціна / ліцензія | Зусилля |
|---|---|---|---|
| **Flexible Invoices + KSeF Add-on** (WP Desk) ✅ | Add-on v2.5.4 від 28.10.2026: KSeF API v2, FA(3), токен або сертифікат, масова відправка, колонка статусу, корекції, повтор кожні 8 год. 700+ інсталяцій, перевірений з WP 7.1.4. UPO та offline у описі не згадані. Потребує базового плагіна Flexible Invoices ≥ 6.2.0. Корекції, проформа й автовипуск належать до PRO. | Add-on на WP.org без ціни. PRO: All Plugins Bundle $99/рік на 25 сайтів (валюта в джерелах різна) ⚠️. Чи дозволяє ліцензія клієнтські інсталяції, перевірити. | S |
| **`ksef-php-client`** (власний модуль у Zamówka) ✅ | PHP-клієнт KSeF API 2.0 (MIT): онлайн-сесії, відправка, UPO, QR. Гілка **1.x** (1.10.0) працює на PHP ≥ 8.1 і отримує лише критичні виправлення KSeF. Гілка 2.x поки лише `2.0.0-rc.1` і вимагає PHP 8.4. Розширення: dom, json, libxml, mbstring, openssl, zip. | MIT, безкоштовно | M |
| FakturFlow ✅ | Безкоштовний v0.3.0: FA(3) з замовлень WooCommerce, ручна відправка. Автовідправка й offline лише в Pro. Лише PLN і польські покупці. <10 інсталяцій. | free / Pro | S (зрілість низька) |
| SaaS через API ✅ | inFakt, Fakturownia, wFirma, iFirma | ~5–79 zł/міс. ⚠️ | S |
| Безкоштовний резерв ✅ | MF «Aplikacja Podatnika KSeF 2.0» і мобільний застосунок: вручну, на випадок збою | безкоштовно | S |

**Рекомендація:** для швидкого старту Flexible Invoices + Add-on, а як ціль власний модуль на `ksef-php-client` 1.x. Він дає нульову плату за ліцензії і повний контроль (UPO, offline24, один код для всіх клієнтів). Обидва варіанти порівняти в PoC (див. нижче).

### 2. Обмін із Subiekt
Підтверджено: публічного REST API в InsERT немає, офіційний шлях це SDK і Sfera ✅. У nexo PRO Sfera входить у ціну, для GT вона окремо ⚠️.

| Варіант | Суть | Ціна | Зусилля |
|---|---|---|---|
| **LEO Synchronizator Platform Internetowych** (Subiekt GT) ✅ | Програма на ПК клієнта обмінюється XML через локальні теки або FTP. Імпорт: ZK, фактури, WZ, PZ, paragony. Експорт: товари, залишки, ціни. Вимоги: Subiekt GT ≥ 1.49, Sfera, Windows, SQL Server. Назви платформ (WooCommerce) не вказано. | від 69 zł/міс., є пробний період | M (на боці WP треба генерувати й читати XML за схемою LEO) |
| **LEO Synchronizator WooCommerce** (nexo PRO) ✅ | Замовлення з WooCommerce стають ZK, далі фактурою або paragon, PDF іде клієнту. Вимоги: nexo PRO ≥ 40.0.0, WooCommerce «5.8.0» (чи це мінімум, не сказано), Windows, SQL Server. | від 89 zł/міс., є пробний період | S |
| Easy Nexo Integrator (NT.NET) ✅ | REST API до Subiekt nexo через міст на ПК клієнта. Документація Swagger. Схоже на хмарний API, з якого WP може читати й писати ⚠️. Версії GT/PRO уточнити. | 900 zł (разово чи на рік, не сказано) | M |
| Інтегратори через Base.com ✅ | SaaS-посередник; потрібен Sfera | 1 380–4 900 zł, джерела розходяться ⚠️ | S |
| Власний агент на Sfera (.NET) ✅ | Windows-служба на ПК клієнта опитує REST Zamówka, створює документи, повертає номери KSeF і залишки | розробка | L |
| Файловий експорт для бухгалтера | EDI++/EPP; приклад: Fakturownia «Eksport InsERT» ✅ | — | S |

**Підводний камінь:** якщо фактуру в KSeF відправила Zamówka, Subiekt при імпорті не має відправити її вдруге, а номер KSeF і внутрішній номер треба узгодити (приклад «Numer obcy» в Optima ✅). Для Subiekt це перевірити на тестовому середовищі.

### 3. Склад і документи WZ/PZ
- Базовий склад це залишки WooCommerce. Окремого плагіна для WZ/PZ в одному пошуку **не знайшов** ✅. Варіант: друкована форма «Dowód wydania/WZ» як функція Zamówka.
- Якщо в клієнта є Subiekt, він залишається «master» складу (залишки приходять через LEO-експорт).
- ATUM та інші розширені склади для Woo ⚠️ (не перевірялись).

### 4. Платежі і розрахунки
| Варіант | Суть | Зусилля |
|---|---|---|
| Шлюзи для WooCommerce ✅ | Przelewy24 (власний модуль, у переліку Woo 9.x–10.x, оновлено 20.04.2026, BLIK; сумісність з Woo 11.2.1 перевірити), Autopay (плагін WP Desk і офіційний GitHub). PayU і Tpay ⚠️. | S |
| Імпорт виписок ✅ | `genkgo/camt` (MIT; CAMT.052/053/054, включно з 053.001.08, у такому форматі mBank видає виписки) і `kingsquare/php-mt940` (MIT). Автозіставлення з номерами фактур. | M |
| Ліміт кредиту і термін платежу | Готового польського плагіна не знайшов ✅ (знайшов лише сервіс Resolve, прив'язаний до стороннього кредитора). Реалізувати в Zamówka на рівні клієнта. | M |

### 5. Доставка
| Варіант | Суть | Зусилля |
|---|---|---|
| Етикетки кур'єрів ✅ | Furgonetka (плагін 7000+ інсталяцій, ще й виписує KSeF-фактури), InPost/DPD ⚠️ | S |
| Планування розвозу | OpenRouteService optimization (на базі VROOM). Публічні ліміти не підтверджені: 500 запитів/добу (2020), за форумом до 50 точок і 3 авто ⚠️. Власний VROOM лише на VPS. | M |
| Правила доставки | Уже є в Zamówka ✅ | — |

### 6. Комунікація з клієнтами
- Розсилки в Zamówka вже є ✅.
- WooCommerce SMSAPI (SMSAPI і WP Desk) **застарів**: останнє оновлення 2021 року, не тестувався з трьома останніми версіями WP ✅. Краще викликати REST API SMSAPI напряму з Zamówka (S) ⚠️.
- E-mail через стандартні листи WooCommerce плюс SMTP/Brevo/Mailgun ⚠️.

### 7. Звіти
WooCommerce Analytics і звіти Zamówka ✅ плюс експорт CSV/Google Sheets/Looker Studio ⚠️. Metabase лише на окремому VPS.

### 8. Архів і закупівельні фактури
- Отримання закупівельних фактур з KSeF як XML не потребує OCR ✅. Subiekt сам перетворює e-faktury на закупівлі ✅.
- Архів XML і UPO: захищена тека, поза webroot, плюс копія в S3-сумісне сховище ⚠️.
- PDF з XML: офіційний `ksef-pdf-generator` (MIT) написаний на TypeScript ✅ і на звичайному PHP-хостингу не запускається. Потрібен власний шаблон (mPDF/Dompdf) або клієнтський JS ⚠️.

### 9. CRM
У v1 окремої CRM не додавати: картки клієнтів і чернетки замовлень вже є в Zamówka ✅. Надалі, за потреби, WP-плагіни FluentCRM / Jetpack CRM / WP ERP ⚠️ (не перевірялись).

### 10. Дистрибуція, оновлення, експлуатація
| Варіант | Суть | Зусилля |
|---|---|---|
| **MainWP** ✅ | Безкоштовний (GPL-3.0) самохостинговий дашборд для багатьох WP-сайтів: масові оновлення, резервні копії, моніторинг доступності, звіти. Що з цього безкоштовне, а що платні розширення, уточнити; відгук скаржиться на слабкий інтерфейс бекапів. | S |
| WP-CLI для розгортання | `cli_tools` на сайті увімкнено ✅. Скрипт «створити субдомен → встановити WP/Woo/Zamówka → майстер налаштувань» | M |
| Ліцензії й оновлення плагіна ⚠️ | Freemius або власний сервер оновлень | S–M |
| Секрети | Токен/сертифікат KSeF кожної гуртівні зберігати зашифровано (окремий для кожної інсталяції) | S |

## Ризики
- **Дублі фактур.** Рівно одне джерело відправляє в KSeF (див. правило вище).
- **Схема XML у LEO** не перевірена (сторінка вендора віддала 403). Попросити документацію й пробну ліцензію.
- **Безпека FTP.** XML із даними клієнтів, тому окремий обмежений SFTP/FTPS-акаунт на кожну інсталяцію.
- **Cron.** Черга KSeF (повтори, UPO) потребує справжнього cron, а не лише WP-Cron. Тип хостингу невідомий.
- **Залежності.** Flexible Invoices PRO (ліцензія для клієнтських сайтів), `ksef-php-client` 1.x (лише критичні правки; 2.x ще rc), модулі InsERT (версії GT/nexo).
- **Ліцензування** плагіна WordPress і сторонніх GPL-компонентів: уточнити в юриста ⚠️.

## Наступні кроки (пропозиція)
1. Обрати 2–3 «еталонні» профілі клієнтів із таблиці вище.
2. PoC KSeF на тестовому середовищі: варіант А (Flexible Invoices + Add-on) і варіант Б (міні-модуль на `ksef-php-client` 1.10) на тестовому субдомені. Порівняти на реальному потоці замовлень.
3. Запит до LEO: схема XML, ціна, пробна ліцензія, чи є синхронізація залишків і цін для nexo PRO.
4. Підтвердити тип хостингу (shared чи VPS, панель, cron, ліміти PHP) і версію Subiekt у перших клієнтів.

## Джерела
- KSeF терміни: [kompasfirmy.pl](https://kompasfirmy.pl/ksef-od-kiedy/), [LinTax](https://www.lintax.pl/aktualnosci/ksef-2026-konczy-sie-okres-przejsciowy-co-zmieni-sie-od-1-stycznia-2027-roku), [Bezprawnik](https://bezprawnik.pl/ksef-dla-najmniejszych-firm/), [InFakt](https://www.infakt.pl/blog/kary-w-ksef-od-2028-r-wazna-zmiana-dla-firm/)
- KSeF у WordPress: [Flexible Invoices KSeF Add-on](https://wordpress.org/plugins/ksef-for-flexible-invoices/), [Flexible Invoices](https://wordpress.org/plugins/flexible-invoices/), [WP Desk](https://wpdesk.net/flexible-invoices-woocommerce), [FakturFlow](https://wordpress.org/plugins/fakturflow-ksef)
- KSeF бібліотеки: [ksef-php-client на GitHub](https://github.com/N1ebieski/ksef-php-client), [ksef-php-client на Packagist](https://packagist.org/packages/n1ebieski/ksef-php-client), [CIRFMF](https://github.com/CIRFMF)
- Subiekt: [InsERT SDK](https://www.insert.com.pl/dla_uzytkownikow/e-pomoc_techniczna/4708,insert-nexo-co-to-jest-i-skad-pobrac-sdk-szczegolowa-dokumentacja-techniczna.html), [InsERT: KSeF w Subiekt GT](https://www.insert.com.pl/dla_uzytkownikow/e-pomoc_techniczna/11213,subiekt-gt-%E2%80%93-konfiguracja-parametrow-zwiazanych-z-generowaniem-i-importem-e-faktur-ksef.html), [LEO Synchronizator Platform Internetowych](https://www.insert.com.pl/dla_uzytkownikow/rozwiazania_partnerskie/1723.html), [LEO Synchronizator WooCommerce](https://www.insert.com.pl/dla_uzytkownikow/rozwiazania_partnerskie/2397.html), [Easy Nexo Integrator](http://easynexointegrator.pl/), [Fakturownia: eksport InsERT](https://fakturownia.pl/apps/insert_export)
- Платежі: [Przelewy24 WooCommerce](https://www.przelewy24.pl/wtyczki/woocommerce), [Autopay](https://github.com/bluepayment-plugin/autopay-payments), [genkgo/camt](https://github.com/genkgo/camt), [mBank CAMT.053](https://www.mbank.pl/pdf/msp-korporacje/bankowosc-elektroniczna/opis-formatu-pliku-eksportu-wyciagow-dziennych-camt.053.001.08.pdf)
- Доставка і SMS: [OpenRouteService optimization](https://heigit.org/quota-for-openrouteservice-multi-vehicle-optimization-increased-to-support-logistics-during-corona-crisis/), [WooCommerce SMSAPI](https://is.wordpress.org/plugins/woo-smsapi-pl/)
- Експлуатація: [MainWP](https://mainwp.com/features)
- Безкоштовні інструменти MF: [Aplikacja Podatnika KSeF 2.0](https://ksef.podatki.gov.pl/aplikacja-podatnika-ksef-20/)
