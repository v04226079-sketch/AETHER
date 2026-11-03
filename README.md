# Aether

A blockchain written from scratch in C++. One person, ~9000 lines, 
no framework, no fork, no team.

**Status:** research / pre-release. Consensus works, but the 
project is not yet ready for real funds. See "What works" below.

---

## What this is

Aether is a proof-of-work blockchain built independently, without 
forking Bitcoin Core or any other codebase. It uses public 
cryptographic primitives and public consensus concepts, combined 
into a design of its own:

- **PoW:** Argon2d with 1 GiB working memory (RFC 9106), 
  memory-hard by design. Parameters chosen for ASIC resistance 
  on home computers, not for farm efficiency.
- **Model:** UTXO (Bitcoin-style).
- **Consensus:** nBits difficulty retarget, chain work, tiebreaker 
  by hash, orphan handling, median-time-past.
- **PoS:** a 210 000-block window activates after a defined 
  transition height. From then on, transaction fees go to the 
  validator selected deterministically from the parent hash.
- **Vesting:** a 60-year release schedule. Allocations unlock 
  in 720 monthly slices, 5% of a monthly tranche per month.
  At year 65, any unclaimed slice returns to the ecosystem fund.
  Claiming is done by self-send — an explicit action that 
  protects unlocked funds from forfeit.
- **Economics:** 21 000 000 AETH cap, halving every 210 000 
  blocks, a 6% premine with vesting.

Everything in the consensus path is written from scratch. 
The P2P protocol is Aether's own (`CHAIN`, `REQBLK`, `BLOCK`, `LOC`), 
not Bitcoin's `inv` / `getdata` / `headers`.

---

## ⚠️ Disclaimer about names

The genesis allocation includes addresses labeled `Elon` and 
`Bernhard Muller`.

**These are fictional labels.** They do not represent real 
people, partnerships, affiliations, endorsements, or financial 
involvement of any kind. Any coincidence with real individuals 
is unintended and not a claim of association.

The names are placeholders for distribution roles only, chosen 
for personal reasons at the time of writing. They should not be 
read as anything more than that.

---

## What works

Verified in practice, not only in design:

- SHA-256, AES, ECDSA secp256k1, BIP39
- Argon2d `pow_hash` — verified 5× independently, including 
  against `argon2-cffi`
- nBits difficulty and retarget
- Blockchain add, reorg, chain work
- UTXO set with transaction validation
- Wallet: BIP39, transaction creation, restore
- P2P: connect, broadcast, sync, IBD
- Orphan handling (bounded map, 128 blocks × 32 MiB, 8 children)
- Median-time-past validation
- PoS window with deterministic fee recipient
- Vesting: 720 slices, monthly unlock, claim by self-send, 
  one-time forfeit to ecosystem fund at block `780 × COOLING_BLOCKS`
- Genesis with 2162 outputs, verified against expected totals
  (founder 50 + vesting 209 950, Elon 105 000 vesting, 
  Muller 105 000 vesting, ecosystem 840 000 unlocked)
- Cross-node sync between two nodes with different code versions
  (bidirectional, no fork)

DOIS, Aegis, and AntiAbuse are logging systems only. They never 
reject blocks. Only objective rules enter consensus.

---

## What does not work yet, or is not verified

Honest list:

- Genesis hash is derived from a hardcoded string, not from the 
  UTXO set. Two binaries with different constants can produce 
  the same genesis hash but different UTXO state. A fix is 
  planned (hash from serialized outputs + hardcoded expected hash).
- No hardcoded `EXPECTED_GENESIS_HASH` check yet.
- No block checkpoints yet.
- `MATURITY` is set to `0` for testing. Before any real funds: 
  `MATURITY = 100`.
- Peer discovery is not implemented. Nodes connect only to 
  configured seed addresses.
- Test coverage is manual. No CI, no automated test suite yet.

This list will shrink. For now it's here so nobody assumes more 
than what exists.

---

## What this is not

- Not a fork of Bitcoin Core, Bitweb, RinCoin, NKL, or any other 
  codebase.
- Not a competitor to Bitcoin or to any existing project. 
  Different point in the design space.
- Not a claim that memory-hard PoW fully blocks GPUs. Argon2d 
  with 1 GiB slows GPUs significantly but does not make them 
  impossible. The only thing that truly blocks GPUs is 8+ GiB 
  per thread, which kills home machines — that trade-off is 
  discussed elsewhere, not hidden.
- Not a claim of inventing a new class of consensus. Memory-hard 
  PoW on UTXO already exists. This is one person's implementation 
  of a known design, not a new one.

---

## Credits

Aether uses the following public works. All are acknowledged 
here in full.

**Consensus and structure:**
- Bitcoin whitepaper (Satoshi Nakamoto, 2008) — UTXO model, 
  nBits difficulty, halving schedule, chain work, P2P sync 
  structure, orphan handling
- Peercoin (Sunny King, 2012) — proof-of-stake concept
- MTP paper (John Tromp, 2016) — large-memory PoW concept

**Cryptography:**
- RFC 9106 — Argon2d (Biryukov, Dinu, Khovratovich, 2015–2021)
- NIST FIPS 180-4 — SHA-256
- SEC 2 (Standards for Efficient Cryptography) — secp256k1 curve
- RIPEMD-160 (Dobbertin, Bosselaers, Preneel)
- BLAKE2b (Aumasson, Neves, Wilcox-O'Hearn, Winnerlein)
- BIP39 (Palatinus, Rusnak, Voisine, Bowe, 2013) — mnemonic 
  seed phrase format and English wordlist (MIT-licensed)

**Licenses of borrowed material:**
- BIP39 English wordlist: MIT
- All other listed works are cryptographic specifications or 
  whitepapers, not source code. Their algorithms are used, 
  not their reference implementations. No source code from 
  these projects is copied into Aether.

See `NOTICE` for the BIP39 wordlist attribution.

---

## Original to this codebase

Things written here, not taken from any other project:

- Full consensus implementation in C++ from scratch — no 
  forked repository, no copied files
- Custom P2P message protocol (`CHAIN` / `REQBLK` / `BLOCK` / `LOC`)
- Argon2d integration with 1 GiB memory, 2 MiB scratch, 
  65 536-step walk — parameters chosen for this project
- PoS window of 210 000 blocks with fees routed to the validator
- 720-slice vesting over 60 years (`COOLING_BLOCKS = 4380`)
- One-time forfeit at block `780 × COOLING_BLOCKS` (65 years), 
  only for unclaimed slices
- Claim-by-self-send mechanism to protect unlocked slices from 
  forfeit
- `genesis_lock` with ecosystem index computed from 
  `3 × VESTING_MONTHS + 1` (adaptable if slice count changes)
- Correct decimal index handling in UTXO keys (a bug found and 
  fixed during development)
- Reverse-compatible key generation across nodes with different 
  release schedules

None of the above claims to be a new cryptographic primitive, 
a new consensus rule, or a new economic model. They are 
engineering decisions made in one place, for one project.

---

## License

MIT. See `LICENSE`.

Code is open. Read it, compare it, judge for yourself.

---

## Why this exists

Not for a market cycle. Not for a treasury. Not for a team.

For the belief that things that are publicly available should 
stay available — a coin, a ledger, a way to hold what is 
yours — to people without lawyers, without funds, without 
a foundation behind them.

This was written alone, in spare hours. It is not finished. 
But it is honest about what it is and what it is not.

*Believe and participate.*
# Aether

Блокчейн, написанный с нуля на C++. Один человек, ~9000 строк, 
без фреймворков, без форка, без команды.

**Статус:** исследовательский / до-релиз. Консенсус работает, 
но проект пока не готов для реальных средств. См. «Что работает» 
ниже.

---

## Что это

Aether — блокчейн на proof-of-work, написанный независимо, 
без форка Bitcoin Core и любого другого кода. Использует 
публичные криптографические примитивы и публичные концепции 
консенсуса, собранные в собственную конструкцию:

- **PoW:** Argon2d с рабочей памятью 1 ГиБ (RFC 9106), 
  memory-hard по замыслу. Параметры выбраны для 
  ASIC-устойчивости на домашних компьютерах, а не для 
  эффективности ферм.
- **Модель:** UTXO (в стиле Bitcoin).
- **Консенсус:** nBits-ретаргет, chain work, tiebreaker по 
  хешу, обработка сирот, median-time-past.
- **PoS:** окно 210 000 блоков активируется после заданной 
  высоты перехода. С этого момента комиссии идут валидатору, 
  выбранному детерминированно по хешу родителя.
- **Вестинг:** 60-летний график. Доли открываются 720 
  месячными срезами, по 5% от месячного транша в месяц. 
  На 65-м году любой невостребованный срез возвращается 
  в эко-фонд. Востребование — самоперевод, явное действие, 
  которое защищает разблокированные средства от форфейта.
- **Экономика:** ограничение 21 000 000 AETH, халвинг каждые 
  210 000 блоков, премайн 6% с вестингом.

Всё в консенсусной части написано с нуля. Протокол P2P — 
собственный (`CHAIN`, `REQBLK`, `BLOCK`, `LOC`), не 
`inv` / `getdata` / `headers` от Bitcoin.

---

## ⚠️ Оговорка об именах

В генезис-распределении есть адреса с метками `Elon` и 
`Bernhard Muller`.

**Это вымышленные метки.** Они не обозначают реальных людей, 
партнёрства, аффилированности, одобрения или финансового 
участия. Любое совпадение с реальными лицами непреднамеренно 
и не является заявлением о связи.

Метки — это заполнители для ролей распределения, выбранные 
по личным причинам на момент написания. Не следует читать 
в них ничего большего.

---

## Что работает

Проверено на практике, не только в проекте:

- SHA-256, AES, ECDSA secp256k1, BIP39
- Argon2d `pow_hash` — верифицирован 5 раз независимо, 
  включая сверку с `argon2-cffi`
- nBits-сложность и ретаргет
- Добавление блоков, реорг, chain work
- UTXO-множество с валидацией транзакций
- Кошелёк: BIP39, создание транзакций, восстановление
- P2P: подключение, broadcast, синхронизация, IBD
- Обработка сирот (ограниченная карта, 128 блоков × 32 МиБ, 
  8 детей на родителя)
- Median-time-past
- PoS-окно с детерминированным получателем комиссий
- Вестинг: 720 срезов, помесячное открытие, claim самопереводом, 
  одноразовый форфейт в эко-фонд на блоке `780 × COOLING_BLOCKS`
- Генезис с 2162 выходами, проверен против ожидаемых сумм 
  (основатель 50 + вестинг 209 950, Elon 105 000 вестинг, 
  Muller 105 000 вестинг, эко-фонд 840 000 без замка)
- Двусторонняя синхронизация двух нод на разных версиях кода 
  (без форка)

DOIS, Aegis и AntiAbuse — только системы логов. Они никогда 
не отвергают блоки. В консенсус попадают только объективные 
правила.

---

## Что пока не работает или не проверено

Честный список:

- Хеш генезиса выводится из захардкоженной строки, а не из 
  UTXO-множества. Два бинарника с разными константами могут 
  дать одинаковый хеш генезиса, но разное состояние UTXO. 
  Исправление запланировано (хеш от сериализованных выходов 
  плюс захардкоженный ожидаемый хеш).
- Нет проверки `EXPECTED_GENESIS_HASH`.
- Нет чекпоинтов блоков.
- `MATURITY` равен `0` для тестов. До любых реальных средств: 
  `MATURITY = 100`.
- Поиск пиров не реализован. Ноды подключаются только 
  к заданным seed-адресам.
- Покрытие тестами ручное. Нет CI, нет автотестов.

Этот список будет сокращаться. Пока он здесь, чтобы никто 
не предполагал больше, чем есть.

---

## Чем это не является

- Не форк Bitcoin Core, Bitweb, RinCoin, NKL или любого другого 
  репозитория.
- Не конкурент Bitcoin или любому существующему проекту. 
  Другая точка в пространстве решений.
- Не заявление, что memory-hard PoW полностью блокирует GPU. 
  Argon2d с 1 ГиБ существенно замедляет GPU, но не делает их 
  невозможными. Полностью блокирует GPU только 8+ ГиБ на поток, 
  что убивает домашние машины — этот компромисс обсуждается 
  открыто, не скрывается.
- Не заявление об изобретении нового класса консенсуса. 
  Memory-hard PoW на UTXO уже существует. Это реализация 
  известной конструкции одним человеком, а не новая.

---

## Благодарности

Aether использует следующие публичные работы. Все они 
перечислены здесь полностью.

**Консенсус и структура:**
- Bitcoin whitepaper (Satoshi Nakamoto, 2008) — UTXO-модель, 
  nBits-сложность, халвинг, chain work, структура P2P-синхронизации, 
  обработка сирот
- Peercoin (Sunny King, 2012) — концепция proof-of-stake
- MTP paper (John Tromp, 2016) — концепция PoW с большим объёмом 
  памяти

**Криптография:**
- RFC 9106 — Argon2d (Biryukov, Dinu, Khovratovich, 2015–2021)
- NIST FIPS 180-4 — SHA-256
- SEC 2 (Standards for Efficient Cryptography) — кривая secp256k1
- RIPEMD-160 (Dobbertin, Bosselaers, Preneel)
- BLAKE2b (Aumasson, Neves, Wilcox-O'Hearn, Winnerlein)
- BIP39 (Palatinus, Rusnak, Voisine, Bowe, 2013) — формат 
  мнемонической фразы и английский список слов (MIT)

**Лицензии заимствованного материала:**
- Английский wordlist BIP39: MIT
- Остальные перечисленные работы — криптографические 
  спецификации или whitepaper'ы, не исходный код. Используются 
  алгоритмы, не референсные реализации. Никакой исходный код 
  из этих проектов не скопирован в Aether.

См. `NOTICE` для атрибуции wordlist BIP39.

---

## Оригинальное в этой кодовой базе

Написано здесь, не взято из другого проекта:

- Полная консенсусная реализация на C++ с нуля — без форка 
  репозитория, без копирования файлов
- Собственный протокол P2P (`CHAIN` / `REQBLK` / `BLOCK` / `LOC`)
- Интеграция Argon2d с памятью 1 ГиБ, scratch 2 МиБ, walk 
  65 536 шагов — параметры выбраны для этого проекта
- PoS-окно 210 000 блоков с направлением комиссий валидатору
- Вестинг 720 срезов на 60 лет (`COOLING_BLOCKS = 4380`)
- Одноразовый форфейт на блоке `780 × COOLING_BLOCKS` (65 лет), 
  только для невостребованных срезов
- Механизм claim самопереводом для защиты разблокированных 
  срезов от форфейта
- `genesis_lock` с индексом эко-фонда, вычисляемым из 
  `3 × VESTING_MONTHS + 1` (адаптируется при изменении числа 
  срезов)
- Корректная десятичная обработка индексов в ключах UTXO 
  (баг, найденный и исправленный в разработке)
- Обратно совместимая генерация ключей между нодами с разными 
  графиками вестинга

Ни один из пунктов выше не заявляется как новый 
криптографический примитив, новое правило консенсуса или 
новая экономическая модель. Это инженерные решения, принятые 
в одном месте, для одного проекта.

---

## Лицензия

MIT. См. `LICENSE`.

Код открыт. Читайте, сравнивайте, судите сами.

---

## Зачем это существует

Не для рыночного цикла. Не для казны. Не для команды.

Для веры в то, что публично доступное должно оставаться 
доступным — монета, реестр, способ держать своё — людям 
без юристов, без фондов, без организации за спиной.

Это писалось в одиночку, в свободные часы. Оно не 
закончено. Но оно честно о том, что есть и чего нет.

*Веруй и участвуй.*

MIT License

Copyright (c) 2026 [Vovka]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

This software includes the BIP39 English wordlist (2048 words),
originally published as part of the Bitcoin BIPs repository.

Source: https://github.com/bitcoin/bips/blob/master/bip-0039/english.txt
License: MIT
Original authors: Marek Palatinus, Pavol Rusnak, Aaron Voisine, Sean Bowe

The wordlist is included verbatim to support mnemonic seed phrase
generation and recovery. All other cryptographic algorithms used in
this project (SHA-256, secp256k1, Argon2d, BLAKE2b, RIPEMD-160) are
implemented from their published specifications; no reference
implementation source code is included.


