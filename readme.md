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



---

## A3. Спецификацийн хүснэгт



---

## B. PICT — Pairwise covering array

### B1. Browser model



### B2. Transfer model



### B3. Category-Partition ба PICT харьцуулалт



---

## Дүгнэлт


