AI: Claude Sonnet 5.5, 2026-10-08 (шинэ чат, өмнөх ярианы контекстгүй)

## Prompt
POST /transfer  { "from": ID, "to": ID, "amount": NUM, "currency": CODE }

Дүрмүүд:
- from данс байх ёстой, царцаагдаагүй (frozen биш) байх ёстой
- to данс байх ёстой, from-оос өөр байх ёстой
- amount > 0, дансны үлдэгдлээс хэтрэхгүй, өдрийн лимитээс хэтрэхгүй
- currency данснаас өөр бол ханшийн хөрвүүлэлт хийгдэнэ
- Амжилт: 201 {"result":"OK"} · Оролтын алдаа: 200 {"result":"ERROR_..."}
  ERROR_NO_FROM, ERROR_FROZEN, ERROR_NO_TO, ERROR_SAME_ACCOUNT,
  ERROR_BAD_AMOUNT (amount ≤ 0), ERROR_INSUFFICIENT, ERROR_LIMIT
- amount нь currency-гийн валютаар; үлдэгдэл, лимитийг хөрвүүлсний дараах дүнгээр шалгана

Дээрх шилжүүлгийн тодорхойлолтод category-partition аргаар сонголт, утга, хязгаарлалт гаргаад PICT модель бич.
