# Лаборатори №6 — Category-Partition ба PICT

**Хичээл:** F.CSA313 — Программ хангамжийн чанарын баталгаа ба тест  
**Оюутан:** Ө. Анар  
**Оюутны код:** B242270129  

---

## A. Category-Partition — Шилжүүлгийн функц

Тестлэгдэх функц:

```text
POST /transfer
{
  "from": ID,
  "to": ID,
  "amount": NUM,
  "currency": CODE
}
```

### A1. Сонголт ба төлөөлөх утгууд

| Category | Choice | Төлөөлөх утга | Тайлбар |
|---|---|---|---|
| From | Valid | `ACC100` | Байгаа, frozen биш данс |
| From | Frozen | `ACC200` | Байгаа боловч царцаагдсан данс |
| From | Missing | `ACC999` | Системд байхгүй данс |
| To | Valid | `ACC300` | Байгаа бөгөөд from-оос өөр данс |
| To | Missing | `ACC998` | Системд байхгүй данс |
| To | SameAsFrom | `ACC100` | from болон to ижил |
| Amount | NonPositive | `0` | `amount <= 0` |
| Amount | Normal | `100` | Үлдэгдэл болон өдрийн лимитийн дотор |
| Amount | OverBalance | `1100` | Дансны үлдэгдлээс их |
| Amount | OverLimit | `600` | Өдрийн үлдсэн лимитээс их боловч дансны үлдэгдлээс бага |
| Currency | Same | `USD` | Дансны currency-тай ижил |
| Currency | Different | `EUR` | Дансны currency-аас өөр, хөрвүүлэлт хийгдэнэ |

### Далд орчны төлөв

`amount`-ийн ангиллыг тодорхой болгохын тулд тестийн суурь setup-д дараах төлөвийг ашиглана:

- from дансны үлдэгдэл: `1000` нэгж
- тухайн өдрийн үлдсэн лимит: `500` нэгж
- currency өөр үед amount, balance, limit-ийг хөрвүүлсний дараах утгаар ангилна

Иймээс:

- `100` → хэвийн дүн
- `600` → үлдэгдэлд багтах боловч өдрийн лимитээс хэтэрсэн
- `1100` → дансны үлдэгдлээс хэтэрсэн

### Олон алдаа давхцах үеийн давуу эрэмбэ

Энэ лабораторид дараах давуу эрэмбийг ашиглана:

```text
from → to → amount validity → balance → daily limit
```

Эхлээд эх үүсвэр данс хүчинтэй эсэхийг шалгахгүйгээр бусад нөхцөлийг найдвартай үнэлэх боломжгүй гэж үзсэн. Дараа нь хүлээн авагч дансны төлөвийг шалгана. Үүний дараа amount-ийн утга, дансны үлдэгдэл болон өдрийн лимитийг дарааллаар шалгана.

### Бүрэн олонлог

PICT моделийн үндсэн category-ууд:

```text
From:     Valid, Frozen, Missing
To:       Valid, Missing, SameAsFrom
Amount:   NonPositive, Normal, OverBalance, OverLimit
Currency: Same, Different
```

Бүрэн хослолын тоо:

```text
3 × 3 × 4 × 2 = 72
```

Иймээс хязгаарлалт тавихаас өмнөх бүрэн олонлог:

```text
N = 72
```

---

## A2. Хязгаарлалтууд

Category-Partition аргаар алдаатай болон тусгай утгуудыг `[ERROR]`, `[SINGLE]`, `[IF/property]` тэмдэглэгээгээр хязгаарлав.

| Category | Choice | Тэмдэглэгээ | Үндэслэл |
|---|---|---|---|
| From | Valid | `[property FromValid]` | Дараагийн `to`, `amount` нөхцлүүдийг хэвийн шалгах боломжтой |
| From | Frozen | `[ERROR][SINGLE]` | Царцаагдсан данс тул `ERROR_FROZEN` |
| From | Missing | `[ERROR][SINGLE]` | Эх данс байхгүй тул `ERROR_NO_FROM` |
| To | Valid | `[IF FromValid]` | Хүчинтэй эх дансны үед хэвийн хүлээн авагчийг шалгана |
| To | Missing | `[ERROR][SINGLE]` | Хүлээн авагч байхгүй тул `ERROR_NO_TO` |
| To | SameAsFrom | `[ERROR][SINGLE]` | Ижил данс тул `ERROR_SAME_ACCOUNT` |
| Amount | NonPositive | `[ERROR][SINGLE]` | `amount <= 0` тул `ERROR_BAD_AMOUNT` |
| Amount | Normal | `[IF FromValid]` | Хэвийн шилжүүлгийн үндсэн утга |
| Amount | OverBalance | `[ERROR][SINGLE]` | Үлдэгдлээс хэтэрсэн тул `ERROR_INSUFFICIENT` |
| Amount | OverLimit | `[ERROR][SINGLE]` | Өдрийн лимитээс хэтэрсэн тул `ERROR_LIMIT` |
| Currency | Same | — | Хөрвүүлэлт хийх шаардлагагүй |
| Currency | Different | — | Ханшийн хөрвүүлэлтийн замыг шалгана |

### Хязгаарлалтын үндэслэл

`[ERROR]` утгууд нь өөрсдөө тодорхой алдааны үр дүнтэй тул бусад бүх category-тай Cartesian product үүсгэх шаардлагагүй. `[SINGLE]` ашигласнаар тухайн алдааг нэг төлөөлөх тестээр шалгана. `From = Valid` үед `FromValid` property үүсгэж, хэвийн `To` болон `Amount` сонголтуудыг түүнтэй холбоно.

### Бүрэн олонлог ба хязгаарласны дараах тоо

Хязгаарлалтгүй үед:

```text
3 × 3 × 4 × 2 = 72
```

SINGLE алдааны тестүүд:

```text
From: Frozen, Missing                         → 2
To: Missing, SameAsFrom                       → 2
Amount: NonPositive, OverBalance, OverLimit   → 3

Нийт SINGLE = 7
```

Хэвийн нөхцөлд currency-ийн хоёр хувилбар үлдэнэ:

```text
Valid + Valid + Normal + Same
Valid + Valid + Normal + Different
```

Иймээс:

```text
72 → 9
```

Өөрөөр хэлбэл 72 боломжит бүрэн хослолыг 7 тусгаар алдааны тест болон 2 хэвийн тестээр төлөөлүүлэн шалгана.

### Давхар алдааны бодлого

`[SINGLE]` нь үндсэн Category-Partition олонлогийг багасгах зорилготой боловч A3 хэсэгт алдааны давуу эрэмбийг шалгахын тулд давхар алдаатай тестийг зориудаар нэмнэ.

```text
from → to → amount validity → balance → daily limit
```

Жишээлбэл `from` данс байхгүй бөгөөд `to` данс мөн байхгүй бол `ERROR_NO_FROM`-ийг хүлээгдэж буй үр дүн гэж үзнэ.


---

## A3. Спецификацийн хүснэгт

Доорх спецификациуд нь Category-Partition аргаар сонгосон төлөөлөх утгууд болон A2 хэсэгт тогтоосон хязгаарлалтыг ашиглана. Хүлээгдэх үр дүнг дараах давуу эрэмбээр тодорхойлов:

```text
from → to → amount validity → balance → daily limit
```

| ID | From | To | Amount | Currency | Төрөл | Хүлээгдэх статус | Хүлээгдэх result |
|---|---|---|---|---|---|---:|---|
| TC01 | Valid | Valid | Normal | Same | Happy path | 201 | `OK` |
| TC02 | Valid | Valid | Normal | Different | Happy path / currency conversion | 201 | `OK` |
| TC03 | Missing | Valid | Normal | Same | `[ERROR][SINGLE]` | 200 | `ERROR_NO_FROM` |
| TC04 | Frozen | Valid | Normal | Same | `[ERROR][SINGLE]` | 200 | `ERROR_FROZEN` |
| TC05 | Valid | Missing | Normal | Same | `[ERROR][SINGLE]` | 200 | `ERROR_NO_TO` |
| TC06 | Valid | SameAsFrom | Normal | Same | `[ERROR][SINGLE]` | 200 | `ERROR_SAME_ACCOUNT` |
| TC07 | Valid | Valid | NonPositive | Same | `[ERROR][SINGLE]` | 200 | `ERROR_BAD_AMOUNT` |
| TC08 | Valid | Valid | OverBalance | Same | `[ERROR][SINGLE]` | 200 | `ERROR_INSUFFICIENT` |
| TC09 | Valid | Valid | OverLimit | Same | `[ERROR][SINGLE]` | 200 | `ERROR_LIMIT` |
| TC10 | Missing | Missing | Normal | Same | Давхар алдаа | 200 | `ERROR_NO_FROM` |
| TC11 | Frozen | Missing | Normal | Same | Давхар алдаа | 200 | `ERROR_FROZEN` |
| TC12 | Valid | Valid | OverLimit | Different | Currency conversion + limit | 200 | `ERROR_LIMIT` |

### Спецификацийн тайлбар

- **TC01** нь бүх үндсэн нөхцөл хүчинтэй happy path бөгөөд шилжүүлэг амжилттай хийгдэх ёстой.
- **TC02** нь currency өөр үед ханшийн хөрвүүлэлт хийгдсэн ч бусад нөхцөл хангагдсан бол шилжүүлэг амжилттай болохыг шалгана.
- **TC03–TC09** нь тодорхойлолтод өгөгдсөн алдааны анги бүрийг тус бүр нэг төлөөлөх тестээр шалгана.
- **TC10** дээр `from` болон `to` хоёулаа байхгүй боловч бидний тогтоосон давуу эрэмбээр `from` эхэлж шалгагдах тул `ERROR_NO_FROM` хүлээнэ.
- **TC11** дээр эх данс frozen бөгөөд `to` байхгүй үед мөн `from` талын алдаа түрүүлж, `ERROR_FROZEN` хүлээнэ.
- **TC12** нь өөр currency-тэй үед хөрвүүлсний дараах дүн өдрийн лимитээс хэтэрвэл `ERROR_LIMIT` гарах ёстойг шалгана.


---

## B. PICT — Pairwise covering array

### B1. Browser model

`browser.model`:

```text
Content:   Allow, Restrict, Block
Cookies:   Allow, Restrict, Block
Popups:    Yes, No
Addons:    Yes, No
Attacks:   Yes, No
Forgeries: Yes, No
```

PICT 2-way ажиллуулалт:

```bash
pict browser.model | tee results/browser-2way.txt
```

PICT 3-way ажиллуулалт:

```bash
pict browser.model /o:3 > results/browser-3way.txt
```

Мөрийн тоо:

```text
2-way = 10
3-way = 20
```

Browser model-ийн бүрэн олонлог:

```text
3 × 3 × 2 × 2 × 2 × 2 = 144
```

| Хувилбар | Мөрийн тоо | Бүрэн олонлогтой харьцуулалт |
|---|---:|---:|
| 2-way | 10 | 10 / 144 |
| 3-way | 20 | 20 / 144 |
| Бүрэн олонлог | 144 | 144 / 144 |

Лекц дээр 2-way хамралтыг гараар 9 мөрөнд багтаасан жишээтэй харьцуулахад PICT 10 мөр гаргасан. Энэ нь PICT greedy алгоритм ашигладаг бөгөөд хамгийн бага боломжит мөрийн тоог заавал олно гэсэн баталгаа байхгүйтэй холбоотой. Харин PICT нь pairwise хамралтыг автоматаар бүрдүүлж, гараар хийх үед алдах боломжтой хослолуудыг системтэйгээр хамруулдаг.

3-way үед 20 мөр гарсан бөгөөд энэ нь 2-way-ийн 10 мөрөөс их боловч бүрэн 144 хослолоос хавьгүй бага байна. Interaction strength өсөх тусам илүү олон параметрийн хослолыг нэгэн зэрэг хамруулах шаардлагатай тул тестийн мөрийн тоо өссөн.

Гаралтууд:

```text
results/browser-2way.txt
results/browser-3way.txt
```

### B2. Transfer model

Хэсэг A-д тодорхойлсон сонголтуудыг PICT model хэл рүү буулгаж, гурван хувилбарыг ажиллуулав.

#### 1. Хязгаарлалтгүй model — `transfer-nc.model`

```text
From: Valid, Frozen, Missing
To: Valid, Missing, SameAsFrom
Amount: NonPositive, Normal, OverBalance, OverLimit
Currency: Same, Different
```

Ажиллуулсан команд:

```bash
pict transfer-nc.model > results/transfer-nc.txt
```

Гарсан мөрийн тоо:

```text
13
```

Энэ model-д алдаатай утгуудыг энгийн утгатай адил авч үзсэн тул нэг мөрөнд хэд хэдэн алдаа зэрэг орсон. Жишээлбэл:

```text
Missing  SameAsFrom  NonPositive  Same
```

гэсэн мөрөнд `From=Missing`, `To=SameAsFrom`, `Amount=NonPositive` гэсэн **3 алдаа** зэрэг орсон. Ийм олон алдаатай мөр нь error masking үүсгэж, яг аль алдааг шалгаж байгааг тодорхой бус болгоно.

`transfer-nc` гаралтад happy path мөр байна:

```text
Valid  Valid  Normal  Different
```

#### 2. Хязгаарлалттай model — `transfer.model`

A2 хэсэгт тогтоосон `[SINGLE]` санааг PICT-ийн `IF` constraint-уудаар буулгав. Алдаатай choice сонгогдсон үед бусад параметрүүдийг хэвийн утгад тогтоосон.

Ажиллуулсан команд:

```bash
pict transfer.model > results/transfer.txt
```

Гарсан мөрийн тоо:

```text
8
```

Гаралт:

```text
Missing Valid Normal Same
Frozen Valid Normal Same
Valid Valid NonPositive Same
Valid SameAsFrom Normal Same
Valid Valid OverLimit Same
Valid Missing Normal Same
Valid Valid OverBalance Same
Valid Valid Normal Different
```

Энд нэг мөрөнд хамгийн ихдээ **1 алдаатай choice** байна. Ингэснээр `ERROR_NO_FROM`, `ERROR_FROZEN`, `ERROR_NO_TO`, `ERROR_SAME_ACCOUNT`, `ERROR_BAD_AMOUNT`, `ERROR_INSUFFICIENT`, `ERROR_LIMIT` гэсэн алдааны ангиудыг тус тусад нь тайлбарлах боломжтой болсон.

Happy path:

```text
Valid  Valid  Normal  Different
```

> A2 дахь `72 → 9` нь Category-Partition аргаар гаргасан төлөөлөх спецификацийн тооцоо. Харин PICT нь pairwise coverage-ийг greedy алгоритмаар бүрдүүлдэг тул яг 9 мөр заавал үүсгэхгүй; энэ constrained model дээр 8 мөр үүссэн.

#### 3. Negative values model — `transfer-neg.model`

Category-Partition-ийн `[ERROR]` утгуудыг PICT-д `~` negative value хэлбэрээр тэмдэглэв.

```text
From: Valid, ~Frozen, ~Missing
To: Valid, ~Missing, ~SameAsFrom
Amount: Normal, ~NonPositive, ~OverBalance, ~OverLimit
Currency: Same, Different
```

Ажиллуулсан команд:

```bash
pict transfer-neg.model > results/transfer-neg.txt
```

Гарсан мөрийн тоо:
Хэсэг A-д тодорхойлсон сонголтуудыг PICT model хэл рүү буулгаж, гурван хувилбарыг ажиллуулав.

#### 1. Хязгаарлалтгүй model — `transfer-nc.model`

```text
From: Valid, Frozen, Missing
To: Valid, Missing, SameAsFrom
Amount: NonPositive, Normal, OverBalance, OverLimit
Currency: Same, Different
```

Ажиллуулсан команд:

```bash
pict transfer-nc.model > results/transfer-nc.txt
```

Гарсан мөрийн тоо:

```text
13
```

Энэ model-д алдаатай утгуудыг энгийн утгатай адил авч үзсэн тул нэг мөрөнд хэд хэдэн алдаа зэрэг орсон. Жишээлбэл:

```text
Missing  SameAsFrom  NonPositive  Same
```

гэсэн мөрөнд `From=Missing`, `To=SameAsFrom`, `Amount=NonPositive` гэсэн **3 алдаа** зэрэг орсон. Ийм олон алдаатай мөр нь error masking үүсгэж, яг аль алдааг шалгаж байгааг тодорхой бус болгоно.

`transfer-nc` гаралтад happy path мөр байна:

```text
Valid  Valid  Normal  Different
```

#### 2. Хязгаарлалттай model — `transfer.model`

A2 хэсэгт тогтоосон `[SINGLE]` санааг PICT-ийн `IF` constraint-уудаар буулгав. Алдаатай choice сонгогдсон үед бусад параметрүүдийг хэвийн утгад тогтоосон.

Ажиллуулсан команд:

```bash
pict transfer.model > results/transfer.txt
```

Гарсан мөрийн тоо:

```text
8
```

Гаралт:

```text
Missing Valid Normal Same
Frozen Valid Normal Same
Valid Valid NonPositive Same
Valid SameAsFrom Normal Same
Valid Valid OverLimit Same
Valid Missing Normal Same
Valid Valid OverBalance Same
Valid Valid Normal Different
```

Энд нэг мөрөнд хамгийн ихдээ **1 алдаатай choice** байна. Ингэснээр `ERROR_NO_FROM`, `ERROR_FROZEN`, `ERROR_NO_TO`, `ERROR_SAME_ACCOUNT`, `ERROR_BAD_AMOUNT`, `ERROR_INSUFFICIENT`, `ERROR_LIMIT` гэсэн алдааны ангиудыг тус тусад нь тайлбарлах боломжтой болсон.

Happy path:

```text
Valid  Valid  Normal  Different
```

> A2 дахь `72 → 9` нь Category-Partition аргаар гаргасан төлөөлөх спецификацийн тооцоо. Харин PICT нь pairwise coverage-ийг greedy алгоритмаар бүрдүүлдэг тул яг 9 мөр заавал үүсгэхгүй; энэ constrained model дээр 8 мөр үүссэн.

#### 3. Negative values model — `transfer-neg.model`

Category-Partition-ийн `[ERROR]` утгуудыг PICT-д `~` negative value хэлбэрээр тэмдэглэв.

```text
From: Valid, ~Frozen, ~Missing
To: Valid, ~Missing, ~SameAsFrom
Amount: Normal, ~NonPositive, ~OverBalance, ~OverLimit
Currency: Same, Different
```

Ажиллуулсан команд:

```bash
pict transfer-neg.model > results/transfer-neg.txt
```

Гарсан мөрийн тоо:

```text
16
```

PICT нь negative value ашиглахад нэг мөрөнд хамгийн ихдээ **нэг negative утга** оруулсан. Иймээс хязгаарлалтгүй model дээр гарсан 2–3 алдаа зэрэг давхцах асуудал арилсан.

Happy path хоёр мөр гарсан:

```text
Valid  Valid  Normal  Same
Valid  Valid  Normal  Different
```

Мөн negative мөрийн жишээ:

```text
Valid  Valid  ~OverBalance  Different
Valid  ~Missing  Normal  Different
~Frozen  Valid  Normal  Same
```

Эдгээр мөр бүрт зөвхөн нэг сөрөг утга байгаа тул oracle тодорхой байна.

### Гурван model-ийн харьцуулалт

| Model | Мөрийн тоо | Happy path мөр | Нэг мөр дэх хамгийн их алдаа | Тайлбар |
|---|---:|---:|---:|---|
| `transfer-nc.model` | 13 | 1 | 3 | Constraint байхгүй тул олон алдаа нэг мөрөнд давхцсан |
| `transfer.model` | 8 | 1 | 1 | A2-ын SINGLE санааг IF constraint-аар тусгаарласан |
| `transfer-neg.model` | 16 | 2 | 1 negative | `~` нь нэг мөрөнд нэг negative утгыг зөвшөөрсөн |

`transfer-neg.model` дээр `~` ашигласнаар алдаатай утгууд хоорондоо нэг мөрөнд давхцахгүй болсон. Иймээс A2-ын алдааг тусгаарлах олон `IF` constraint шаардлагагүй болж байгаа боловч бизнесийн логикийн хувьд боломжгүй эсвэл утгагүй хослолыг хасах шаардлага гарвал constraint ашиглах хэрэгцээ хэвээр үлдэнэ.

### PICT гаралтаас бодит тест кейс болгон хөрвүүлсэн 3 мөр

| Test | Setup | Оролт | Oracle |
|---|---|---|---|
| PICT-TC01 Happy path | `ACC100` valid, frozen биш, balance=1000, remaining daily limit=500; `ACC300` valid | `{"from":"ACC100","to":"ACC300","amount":100,"currency":"EUR"}` | HTTP `201`, `result="OK"` |
| PICT-TC02 Frozen account | `ACC200` frozen; `ACC300` valid | `{"from":"ACC200","to":"ACC300","amount":100,"currency":"USD"}` | HTTP `200`, `result="ERROR_FROZEN"` |
| PICT-TC03 Over balance | `ACC100` valid, balance=1000, limit хангалттай; `ACC300` valid | `{"from":"ACC100","to":"ACC300","amount":1100,"currency":"EUR"}` | HTTP `200`, `result="ERROR_INSUFFICIENT"` |

PICT-TC01 нь бүх сонголт хүчинтэй happy path. PICT-TC02 нь `transfer-neg.model`-ийн `~Frozen` нэг алдаатай мөрийг бодит тест болгон хөрвүүлсэн. PICT-TC03 нь `~OverBalance` мөрийг бодит amount утгаар илэрхийлж, үлдэгдэл хүрэлцэхгүй нөхцөлийн oracle-ийг тодорхойлсон.

Гаралтын файлууд:

```text
results/transfer-nc.txt
results/transfer.txt
results/transfer-neg.txt
```

### B3. Category-Partition ба PICT харьцуулалт



---

## Дүгнэлт


