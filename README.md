# Лаборатори №6 — Category-Partition ба PICT

**Оюутан:** Nyamod Nyamsuren · **Код:** B222270805 · F.CSA313 (2026)

## 0. PICT-ийн бүтээлт

PICT-ийг `~/tools/pict`-д эх кодоос нь cmake-ээр бүтээсэн (репод ороогүй). Нотолгоо: `results/pict-build.txt`
(commit hash + хэрэглээний заавар).

---

# Хэсэг A — Category-Partition (POST /transfer)

## A1. Сонголт ба төлөөлөх утгууд

| Параметр | Сонголт | Төлөөлөх утга (setup) |
|---|---|---|
| **from** | Valid | ACC-A: үлдэгдэл 1 000 000 MNT, царцаагдаагүй |
| | Frozen | ACC-F: frozen = true |
| | Missing | ACC-XXXX (системд байхгүй ID) |
| **to** | Valid | ACC-B: байгаа данс, ACC-A-аас өөр |
| | Missing | ACC-YYYY (байхгүй ID) |
| | SameAsFrom | to = from (ACC-A) |
| **amount** | Normal | 10 000 (үлдэгдэл, лимитийн дотор) |
| | ExactBalance | 1 000 000 (яг үлдэгдэлтэй тэнцүү — хязгаарын утга) |
| | Zero | 0 |
| | Negative | -500 |
| | OverBalance | 1 500 000 (үлдэгдлээс их, лимитээс бага) |
| | OverLimit | 6 000 000 (үлдэгдэл 10 000 000, өдрийн лимит 5 000 000 үед) |
| **currency** | Same | данстай ижил валют (MNT) |
| | Different | USD (тестийн орчны ханш: 1 USD = 3 400 MNT гэж тохирсон) |
| **Далд орчин: өдрийн лимит** | Fresh / NearLimit | өнөөдрийн шилжүүлгийн дүн; OverLimit утгыг бодит болгох setup (тусдаа параметр болгоогүй — Amount-ийн OverLimit дотор шингээсэн) |

Ажиглалт: тодорхойлолтод тодорхойгүй зүйл — үл мэдэгдэх currency код (ERROR_ байхгүй), давхар алдааны эрэмбэ.
Давхар алдааны эрэмбийг доорх A2-д өөрөө тогтоосон.

## A2. Хязгаарлалтууд ба "N → M"

| Утга | Тэмдэглэгээ | Үндэслэл |
|---|---|---|
| from=Frozen | [ERROR] | Эхний алдаа нь бусдыг нуух (ERROR_FROZEN) тул нэг л удаа шалгана |
| from=Missing | [ERROR] | ERROR_NO_FROM |
| to=Missing | [ERROR] | ERROR_NO_TO |
| to=SameAsFrom | [ERROR] | ERROR_SAME_ACCOUNT |
| amount=Zero, Negative | [ERROR] | ERROR_BAD_AMOUNT (amount ≤ 0) |
| amount=OverBalance | [ERROR] | ERROR_INSUFFICIENT |
| amount=OverLimit | [ERROR] | ERROR_LIMIT |
| amount=ExactBalance | [SINGLE] | Хязгаарын утга; амжилттай (201) болохыг нэг удаа шалгахад хангалттай |
| to=SameAsFrom | [IF from≠Missing] | "from-той ижил" гэдэг нь from байх үед л утгатай |
| amount≠Normal | [IF from≠Missing] | from байхгүй бол үлдэгдэл/лимит тодорхойгүй тул дүнгийн сонголт утгагүй |

**Давуу эрэмб (олон алдаа зэрэг гарвал):** from → to → amount (NO_FROM → FROZEN → NO_TO → SAME_ACCOUNT →
BAD_AMOUNT → INSUFFICIENT → LIMIT). *Үндэслэл:* [ТАНЫ ӨӨРИЙН ҮНДЭСЛЭЛИЙГ ЭНД НЭГ ӨГҮҮЛБЭРЭЭР БИЧ — жишээ нь: "Эхлээд оролцогч
дансуудыг шалгаад, дараа нь дүнгийн бизнес дүрмийг шалгах нь API-ийн ердийн дараалалтай нийцнэ."]

**Тооцоо:** бүрэн олонлог = 3 (from) × 3 (to) × 6 (amount) × 2 (currency) = **108**.
Хязгаарлалтын дараа: хүчинтэй бүх утгын хослол 1×1×1×2 = 2, [SINGLE] 1, [ERROR] 8 (2+2+4) → **108 → 11**.

## A3. Спецификацийн хүснэгт

| # | from | to | amount | currency | Хүлээгдэх үр дүн |
|---|---|---|---|---|---|
| 1 | Valid | Valid | Normal | Same | 201 `OK` (happy path) |
| 2 | Valid | Valid | Normal | Different | 201 `OK` (ханшийн хөрвүүлэлттэй) |
| 3 | Valid | Valid | ExactBalance [SINGLE] | Same | 201 `OK` (үлдэгдэлтэй тэнцүү зөвшөөрөгдөнө) |
| 4 | Frozen | Valid | Normal | Same | 200 `ERROR_FROZEN` |
| 5 | Missing | Valid | Normal | Same | 200 `ERROR_NO_FROM` |
| 6 | Valid | Missing | Normal | Same | 200 `ERROR_NO_TO` |
| 7 | Valid | SameAsFrom | Normal | Same | 200 `ERROR_SAME_ACCOUNT` |
| 8 | Valid | Valid | Zero | Same | 200 `ERROR_BAD_AMOUNT` |
| 9 | Valid | Valid | Negative | Same | 200 `ERROR_BAD_AMOUNT` |
| 10 | Valid | Valid | OverBalance | Same | 200 `ERROR_INSUFFICIENT` |
| 11 | Valid | Valid | OverLimit | Same | 200 `ERROR_LIMIT` |
| 12 | Frozen | Missing | Normal | Same | 200 `ERROR_FROZEN` (давхар: from нь to-оос өмнө) |
| 13 | Valid | SameAsFrom | OverBalance | Same | 200 `ERROR_SAME_ACCOUNT` (давхар: to нь amount-аас өмнө) |
| 14 | Valid | Valid | үлдэгдэл БА лимитээс их | Same | 200 `ERROR_INSUFFICIENT` (давхар: INSUFFICIENT нь LIMIT-ээс өмнө) |
| 15 | Valid | Valid | 400 USD (= 1 360 000 MNT > үлдэгдэл 1 000 000) | Different | 200 `ERROR_INSUFFICIENT` (тоо нь үлдэгдлээс бага ч ХӨРВҮҮЛСНИЙ дараа хэтэрнэ) |

1–11 нь M = 11 спецификаци; 12–15 нь давхар нөхцөл/хөрвүүлэлтийн нэмэлт.

---

# Хэсэг B — PICT

## B1. Браузерын жишээ

| Хувилбар | Мөрийн тоо | Файл |
|---|---|---|
| Бүрэн олонлог | 144 | (3·3·2·2·2·2) |
| 2-way | <<browser-2way тоо>> | results/browser-2way.txt |
| 3-way | <<browser-3way тоо>> | results/browser-3way.txt |

**Greedy тайлбар:** [2-3 өгүүлбэр — лекц дээр гараар 9 мөр болсон. PICT <<тоо>> мөр өгсөн. PICT нь greedy алгоритмтай тул хос бүрийг
хамарсан байхыг баталгаажуулдаг ч хамгийн цөөн мөрийг ЗААВАЛ өгдөггүй (доод хязгаар 3×3=9). 3-way нь <<тоо>> мөр — бүрэн 144-ийн <<хувь>>%.]

## B2. Шилжүүлгийн модель

| Model | Мөрийн тоо | Happy path мөр | 1 алдаатай | 2+ алдаатай | Нэг мөрөнд дээд алдаа |
|---|---|---|---|---|---|
| transfer-nc (хязгаарлалтгүй) | <<>> | <<>> | <<>> | <<>> | <<>> |
| transfer (A2 хязгаарлалттай) | <<>> | <<>> | <<>> | <<>> | <<>> |
| transfer-neg (`~` сөрөг утга) | <<>> | <<>> | <<>> | <<>> | <<>> |

*(Хүснэгтийг `python3 ~/tools/analyze.py`-ийн гаралтаас хуулсан.)*

[Тайлбар: happy path гарсан уу, олон алдаатай мөр юуг шалгаж байна, ~ хэрэглэхэд A2-ын хязгаарлалт хэрэгтэй хэвээр үү?
Хариулт: transfer-neg-д нэг мөрөнд ≤1 сөрөг утга тул алдаа бие биенээ нуухаас өөрөө хамгаалагдсан, мөн From=Missing + SameAsFrom гэх мэт
боломжгүй хослол үүсэхгүй → IF хязгаарлалт хэрэггүй болсон.]

## B2. Бодит тест кейс (3+)

[`python3 ~/tools/analyze.py case <файл> <мөрийн дугаар>`-ийн гаралтыг хуулж, setup нэм. Доорх бүтэцтэй:]

**Кейс 1 — happy path (transfer-neg.txt мөр #1)**
- Setup: ACC-A (1 000 000 MNT, царцаагдаагүй), ACC-B байгаа, лимит 5 000 000, өнөөдөр 0
- Оролт: `POST /transfer {"from":"ACC-A","to":"ACC-B","amount":10000,"currency":"MNT"}`
- Oracle: HTTP 201 `{"result":"OK"}`

**Кейс 2 — transfer-neg-ийн нэг алдаатай мөр (мөр #__)**
- Setup / Оролт / Oracle: ...

**Кейс 3 — олон алдаатай мөр (transfer.txt мөр #__)**
- Setup / Оролт / Oracle: давуу эрэмбээр үндэслэсэн ...

## B3. Дүгнэлт (8-10 өгүүлбэр)

[Өөрийн үгээр бич: гараар хийсэн A3 нь алдааны семантикийг (ямар алдаа аль нь давуу) барьдаг, харин PICT нь хосын хамралтыг
баталгаажуулдаг, гэхдээ oracle-ийг өөрөө бодох ёстой; аль алхам хамгийн их бодол шаардсан (давуу эрэмб, хязгаарлалт) гэх мэт.]
