# Aether

Монета для обычного компьютера. Потолок выпуска 21 000 000 AETH, дальше только комиссии. Без ASIC-гонок, пулов и допечатки.

> ⚠️ **Прототип.** Сеть ещё не запущена, код не аудирован, монеты тестовые — не стоят ничего. Не отправляйте сюда настоящие деньги.

---

## Зачем

Биткоин остался рабочим, но майнинг ушёл к тем, у кого ферма и дешёвое электричество. Aether пишется с другой ставкой: блок должен закрывать домашний процессор, правила выпуска нельзя раздуть голосованием, а код протокола не должен тихо меняться каждый сезон.

Девиз: **«Веруй и участвуй»**.

---

## Правила, которые не двигаются

| Параметр | Значение |
|---|---|
| Потолок | **21 000 000 AETH**, навсегда |
| Единица | 1 AETH = 100 000 000 минимальных единиц |
| Награда | 50 AETH, халвинг каждые 210 000 блоков |
| Цель по времени | 10 минут |
| Первые годы | доказательство работы |
| После первого халвинга | к работе добавляется доля. Валидатор живёт с комиссий, награда блока ему не отдаётся |
| Комиссия | за байт, не фиксированная «цена в долларах» |
| Управление | порог 85% только на аварийный выключатель защиты, не на выпуск |

Резерв основателя и экофонд лежат **внутри** потолка, а не сверху. Доля основателя — 1%. Экофонд — 840 000 AETH, на развитие, не на кредиты и не на частичное резервирование. Невостребованное по правилам возвращается в экофонд.

Иммунитет основателя действует **до первого халвинга** и не даёт преимущества в майнинге: сложность и награда те же, что у всех.

---

## Как устроено

Свой клиент на **C++17**, один `main.cpp`, без OpenSSL, Boost и libsecp256k1.

- Учёт выходов, как в биткоине: выход тратится **один раз**.
- Подпись **secp256k1**, написана в коде. Одноразовое число подписи считается по **RFC 6979**, не из случайного генератора системы.
- Блок: заголовок, дерево транзакций, награждение за блок, мемпул.
- Сеть: свои сообщения `CHAIN`, `REQBLK`, `BLOCK`. Узел сверяет генезис и не принимает чужую цепь.
- Адрес начинается с `AETH_`.

**Защита DOIS** задумана как тихий фильтр, а не как бан по доле блоков. Доля блоков за окно для этого не годится: ферма делит адреса и остаётся, крупный честный майнер вылетает. В протоколе этого запрета нет. От ферм и ASIC должен защищать сам алгоритм работы, тяжёлый по памяти. Сейчас стоит простое доказательство работы для отладки. Постоянный алгоритм ещё не включён.

**Обновления** не бесплатные и не бесконечные. После отделения кода допускается не больше **18 патчей**, каждый со своим ожиданием. **Конституция из 13 статей** зафиксирована в клиенте: выпуск, халвинг, вестинг, DOIS, патчи, порог 85%, доля после халвинга, иммунитет основателя, запрет цензуры, экофонд, невостребованные суммы, кулдаун патчей и сама конституция.

---

## Чего ещё нет

- **Постоянного поиска узлов по миру.** Сейчас адрес пишется вручную. Для открытой сети в клиент войдут несколько имён-семян, не адреса майнеров.
- **Алгоритма работы, который бесполезен для ASIC и видеокарт.**
- **Зашитого генезиса.** Пока цепь копируют файлом, узлы можно развести.
- **Аудита** подписей, отката цепи и комиссий сторонним криптографом.

Пока этого нет — **релиз для посторонних людей не выпускается**.

---

## Сборка

**Windows**, MinGW, C++17. Исходник один: `main.cpp`.

### Dev-C++

Открыть `Makefile.win`, нажать **F12** (Compile & Run).

### Командная строка

Если `g++` уже в `PATH`:

```bash
g++ -std=c++17 -O2 -o aether.exe main.cpp ^
    -lws2_32 -lcrypt32 -ladvapi32 -luser32 -lkernel32 ^
    -static -static-libgcc -static-libstdc++
```

Для **статического** `.exe` (без DLL рядом):

```bash
g++ -std=c++17 -O2 -o aether.exe main.cpp ^
    -static -static-libwinpthread ^
    -lws2_32 -lcrypt32 -ladvapi32 -luser32 -lkernel32 ^
    -static-libgcc -static-libstdc++
```

### Запуск

```bash
./aether.exe
```

При первом запуске создаётся папка `aether_chain/` — там цепь, ключи и настройки. **Не удаляйте её** — там ваши монеты и сид-фраза.

---

## Статус

| Компонент | Статус |
|---|---|
| Криптография (SHA-256, AES, ECDSA, BIP39) | ✅ |
| Подпись secp256k1 + RFC 6979 | ✅ |
| Учёт выходов (UTXO) | ✅ |
| Блоки, мемпул, награда | ✅ |
| Сеть P2P (`CHAIN`, `REQBLK`, `BLOCK`) | ✅ |
| Синхронизация (IBD, reorg по work) | ✅ |
| Доказательство работы (Argon2d, 1 ГиБ) | ⚠️ Для отладки |
| DOIS-журнал | ⚠️ Только лог, не консенсус |
| PoS после первого халвинга | ⏳ Спецификация |
| Генезис зашит | ❌ Нет |
| Поиск узлов | ❌ Нет (вручную) |
| Внешний аудит | ❌ Нет |

---

## Лицензия

**MIT** — см. файл [`LICENSE`](LICENSE).

---

## Дисклеймер

Экспериментальный прототип. Не для реальных денег. Используйте на свой риск. Автор не несёт ответственности за потерю средств, данных или времени.

Если хотите помочь — откройте issue, пришлите pull request или просто запустите ноду и напишите, что не работает.

[🇷🇺 Русский](README.md) |

**🇬🇧 English**

# Aether

A coin for the ordinary computer. Hard cap 21,000,000 AETH — after that, only fees. No ASIC races, no pools, no inflation.

> ⚠️ **Prototype.** Network not launched, code not audited, test coins worth nothing. Do not send real money here.

---

## Why

Bitcoin still works, but mining has gone to those with farms and cheap electricity. Aether takes a different bet: a block should be mineable on a home CPU, issuance rules should not be inflatable by vote, and protocol code should not silently change every season.

Motto: **"Believe and Participate."**

---

## Rules that don't move

| Parameter | Value |
|---|---|
| Supply cap | **21,000,000 AETH**, forever |
| Unit | 1 AETH = 100,000,000 base units |
| Block reward | 50 AETH, halving every 210,000 blocks |
| Block time target | 10 minutes |
| Early years | Proof of Work |
| After first halving | PoS layer added. Validators live on fees, block reward stays with miner |
| Fee | per byte, not a fixed "dollar price" |
| Governance | 85% threshold only for emergency kill-switch, not for issuance |

Founder reserve and ecosystem fund sit **inside** the cap, not on top. Founder share is 1%. Ecosystem fund — 840,000 AETH, for development, not for loans or fractional reserve. Unclaimed amounts return to the fund by rule.

Founder immunity is active **until first halving** and gives no mining advantage: same difficulty, same reward as everyone else.

---

## How it's built

Custom client in **C++17**, single `main.cpp`, no OpenSSL, Boost or libsecp256k1.

- UTXO accounting, as in Bitcoin: an output is spent once.
- **secp256k1** signature, written in-house. Nonce per **RFC 6979**, not from system RNG.
- Block: header, transaction tree, block reward, mempool.
- Network: custom messages `CHAIN`, `REQBLK`, `BLOCK`. Node validates genesis and rejects foreign chains.
- Address starts with `AETH_`.

**DOIS** is designed as a quiet filter, not a ban-by-block-share mechanism. Block share over a window is unsuitable: a farm splits addresses and survives while a large honest miner gets kicked. There is no such ban in the protocol. Defense against farms and ASIC should come from the work algorithm itself, memory-heavy. Currently a simple proof-of-work is in place for debugging. The permanent algorithm is not enabled yet.

**Updates** are not free and not unlimited. After code separation, no more than **18 patches** are allowed, each with its own cooldown. **Constitution of 13 articles** is fixed in the client: supply, halving, vesting, DOIS, patches, 85% threshold, post-halving share, founder immunity, anti-censorship, ecosystem fund, unclaimed funds, patch cooldown, and the constitution itself.

---

## What's missing

- **Global peer discovery.** Currently manual. For an open network, several seed names will ship in the client — not miner addresses.
- **A work algorithm unusable for ASIC and GPUs.**
- **Hardcoded genesis.** Currently chain is copied as a file; nodes can drift.
- **Third-party audit** of signatures, chain rollback and fees.

Until these are done — **no public release.**

---

## Build

**Windows**, MinGW, C++17. Single source file: `main.cpp`.

### Dev-C++

Open `Makefile.win`, press **F12** (Compile & Run).

### Command line

If `g++` is already in `PATH`:

```bash
g++ -std=c++17 -O2 -o aether.exe main.cpp ^
    -lws2_32 -lcrypt32 -ladvapi32 -luser32 -lkernel32 ^
    -static -static-libgcc -static-libstdc++
```

For a **static** `.exe` (no DLLs needed):

```bash
g++ -std=c++17 -O2 -o aether.exe main.cpp ^
    -static -static-libwinpthread ^
    -lws2_32 -lcrypt32 -ladvapi32 -luser32 -lkernel32 ^
    -static-libgcc -static-libstdc++
```

### Run

```bash
./aether.exe
```

On first run, a directory `aether_chain/` is created — chain, keys, settings. **Do not delete it** — your coins and seed phrase live there.

---

## Status

| Component | Status |
|---|---|
| Crypto (SHA-256, AES, ECDSA, BIP39) | ✅ |
| secp256k1 + RFC 6979 signing | ✅ |
| UTXO accounting | ✅ |
| Blocks, mempool, reward | ✅ |
| P2P (`CHAIN`, `REQBLK`, `BLOCK`) | ✅ |
| Sync (IBD, reorg by work) | ✅ |
| Proof-of-Work (Argon2d, 1 GiB) | ⚠️ Debug only |
| DOIS journal | ⚠️ Log only, not consensus |
| PoS after first halving | ⏳ Spec |
| Hardcoded genesis | ❌ No |
| Peer discovery | ❌ No (manual) |
| External audit | ❌ No |

---

## License

**MIT** — see [`LICENSE`](LICENSE).

---

## Disclaimer

Experimental prototype. Not for real money. Use at your own risk. Author is not responsible for loss of funds, data, or time.

If you want to help — open an issue, send a pull request, or just run a node and report what's broken.
