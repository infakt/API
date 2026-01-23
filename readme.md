# inFakt API v3 — Dokumentacja

## Wstęp

inFakt API oparty jest o architekturę REST i umożliwia dostęp do zasobów serwisu inFakt.pl za pomocą interfejsu JSON. Dzięki temu możliwe staje się stworzenie zewnętrznych aplikacji integrujących się z kontem użytkownika inFakt.pl.

Niniejsza dokumentacja odnosi się do API w wersji 3 (APIv3).

Pełna dokumentacja interaktywna: https://docs.infakt.pl

---

## Punkt dostępowy

| Środowisko | URL |
|---|---|
| **Produkcja** | `https://api.infakt.pl/api/v3` |
| **Sandbox** | `https://api.sandbox-infakt.pl/api/v3` |

Dostęp odbywa się szyfrowanym połączeniem HTTPS z wykorzystaniem protokołu TLS 1.2 oraz TLS 1.3.

---

## Uwierzytelnianie

Klucz API generuje się po zalogowaniu do aplikacji WWW: [Ustawienia konta → API](https://app.infakt.pl/app/ustawienia/inne_opcje/api).

Klucz przekazywany jest w nagłówku HTTP:

```
X-inFakt-ApiKey: TWÓJ_KLUCZ_API
```

### Przykładowe zapytanie

```bash
curl -H "X-inFakt-ApiKey: TWÓJ_KLUCZ_API" \
  -H "Content-Type: application/json" \
  https://api.infakt.pl/api/v3/invoices.json
```

---

## Zakresy uprawnień (Scopes)

| Scope | Opis |
|---|---|
| `api:invoices:read` | Odczyt faktur, klientów i produktów |
| `api:invoices:write` | Zarządzanie fakturami, klientami i produktami |
| `api:accounting:read` | Odczyt danych księgowych (ZUS, podatki) |
| `api:accounting:write` | Zarządzanie księgowością |
| `api:sensitive:bank_accounts:write` | Zarządzanie rachunkami bankowymi |
| `api:costs:read` | Odczyt kosztów |
| `api:costs:write` | Zarządzanie kosztami i skanami |
| `api:ksef:integration:write` | Zarządzanie integracją z KSeF |

---

## Sandbox

Testowy punkt dostępowy do darmowego przetestowania API:

```
https://api.sandbox-infakt.pl/api/v3
```

Rejestracja konta sandbox: https://konto.sandbox-infakt.pl/rejestracja

**Limity sandbox:**
- Do 2500 faktur miesięcznie
- Dane firmowe zanonimizowane
- API działa identycznie jak produkcja
- Dostęp tylko do sekcji przychodowej i ustawień

---

## Zasoby API

### Przychody

| Zasób | Endpoint | Operacje |
|---|---|---|
| Faktury VAT | `/invoices.json` | CRUD, wysyłka, PDF, KSeF |
| Faktury korygujące | `/corrective_invoices.json` | CRUD, wysyłka, KSeF |
| Faktury marża | `/margin_invoices.json` | CRUD, wysyłka, KSeF |
| Faktury zaliczkowe | `/advance_invoices.json` | CRUD, wysyłka, KSeF |
| Faktury końcowe | `/final_invoices.json` | CRUD, wysyłka, KSeF |
| Faktury OSS | `/oss_invoices.json` | CRUD |
| Raporty fiskalne | `/fiscal_reports.json` | CRUD |
| Utargi dzienne | `/daily_revenues.json` | CRUD |

### Pozostałe

| Zasób | Endpoint | Operacje |
|---|---|---|
| Klienci | `/clients.json` | CRUD |
| Produkty | `/products.json` | CRUD |
| Konta bankowe | `/bank_accounts.json` | CRUD |
| Koszty | `/costs.json` | CRUD |
| Skany dokumentów | `/document_scans.json` | Upload, listowanie |
| Składki ZUS | `/zus_contributions.json` | Odczyt |
| Podatek VAT/PIT | `/vat.json`, `/pit.json` | Odczyt |
| KSeF (e-Faktury) | `/ksef/...` | Integracja, wysyłka, import |
| Dane użytkownika | `/account.json` | Odczyt |

---

## Tworzenie faktur (asynchroniczne)

Tworzenie faktury odbywa się asynchronicznie:

```bash
curl -H "X-inFakt-ApiKey: KLUCZ" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"invoice":{
    "payment_method": "transfer",
    "client_company_name": "Firma Sp. z o.o.",
    "client_tax_code": "1234567890",
    "client_country": "PL",
    "services": [{
      "name": "Usługa programistyczna",
      "unit_net_price": 500000,
      "quantity": 1,
      "tax_symbol": "23"
    }]
  }}' \
  https://api.infakt.pl/api/v3/async/invoices.json
```

### Odpowiedź

```json
{
  "timestamps": {
    "task_created_at": "2024-01-15 10:21:25 +0100"
  },
  "invoice_task_reference_number": "34263ae5-4365-4ba1-a38d-c7952c5a5715",
  "processing_code": 100,
  "processing_description": "Zlecenie przyjęte"
}
```

### Statusy faktury przy tworzeniu

| Status | Opis |
|---|---|
| `draft` | Szkic (domyślnie) |
| `paid` | Od razu oznaczona jako zapłacona (wymaga `paid_date`) |
| `printed` | Wydrukowana i uwzględniona w księgowości |

### Sprawdzenie statusu tworzenia

```bash
GET /api/v3/async/invoices/status/{invoice_task_reference_number}.json
```

---

## Definicja faktury VAT

| Pole | Typ | Opis |
|---|---|---|
| `id` | integer | ID faktury (readonly) |
| `number` | string | Numer faktury (auto-generowany) |
| `currency` | string | Waluta (domyślnie PLN) |
| `kind` | string | `vat` lub `proforma` |
| `payment_method` | string | Metoda płatności |
| `invoice_date` | date | Data wystawienia (RRRR-MM-DD) |
| `sale_date` | date | Data sprzedaży |
| `payment_date` | date | Termin zapłaty |
| `paid_date` | date | Data opłacenia |
| `status` | string | `draft`, `sent`, `printed`, `paid` (readonly) |
| `net_price` | integer | Netto w groszach (readonly) |
| `tax_price` | integer | VAT w groszach (readonly) |
| `gross_price` | integer | Brutto w groszach (readonly) |
| `left_to_pay` | integer | Pozostało do zapłaty w groszach |
| `client_id` | integer | ID istniejącego klienta |
| `client_company_name` | string | Nazwa firmy (jeśli brak client_id) |
| `client_tax_code` | string | NIP klienta |
| `client_country` | string | Kraj klienta (Alpha-2) |
| `client_street` | string | Ulica klienta |
| `client_street_number` | string | Nr budynku |
| `client_city` | string | Miasto |
| `client_post_code` | string | Kod pocztowy |
| `client_business_activity_kind` | string | `private_person`, `self_employed`, `other_business` |
| `services` | array | Lista pozycji na fakturze (wymagane) |
| `bank_name` | string | Nazwa banku |
| `bank_account` | string | Numer konta bankowego |
| `swift` | string | Numer SWIFT |
| `split_payment_type` | string | `required` lub `optional` |
| `notes` | string | Uwagi |
| `invoice_date_kind` | string | `sale_date`, `service_date`, `cargo_date`, `continuous_date_end_on` |
| `continuous_service_start_on` | date | Początek usługi ciągłej |
| `continuous_service_end_on` | date | Koniec usługi ciągłej |
| `sale_type` | string | `service` lub `merchandise` (dla zagranicznych) |
| `vat_exemption_reason` | integer | ID podstawy zwolnienia z VAT |
| `bdo_code` | string | Numer rejestrowy BDO |
| `document_markings_ids` | array | ID oznaczeń dokumentów |
| `receipt_number` | string | Numer paragonu |
| `check_duplicate_number` | boolean | Sprawdzanie duplikacji numeru |
| `ksef_data` | object | Dane KSeF (readonly) |
| `created_at` | datetime | Data stworzenia (readonly) |

### Definicja pozycji (Service)

| Pole | Typ | Opis |
|---|---|---|
| `name` | string | Nazwa pozycji (wymagane) |
| `tax_symbol` | string | Stawka VAT |
| `unit` | string | Jednostka |
| `quantity` | number | Ilość |
| `unit_net_price` | integer | Cena netto/szt. w groszach |
| `net_price` | integer | Wartość netto w groszach |
| `gross_price` | integer | Wartość brutto w groszach |
| `tax_price` | integer | VAT w groszach (readonly) |
| `pkwiu` | string | PKWiU |
| `cn` | string | CN |
| `pkob` | string | PKOB |
| `gtu_id` | integer | ID kodu GTU |
| `discount` | integer | Rabat w procentach |
| `unit_net_price_before_discount` | integer | Cena przed rabatem w groszach |
| `flat_rate_tax_symbol` | string | Stawka ryczałtu |
| `vat_date_value` | string | Data dla celów VAT: `issue_date`, `sale_date`, `paid_date` |

> **Uwaga:** Aby wystawić fakturę od brutto, podaj `gross_price` bez `unit_net_price`/`net_price`. System wyliczy wartości netto.

---

## Metody płatności

```
transfer, cash, card, barter, check, bill_of_sale, delivery,
compensation, accredited, paypal, instalment_sale, payu, tpay,
przelewy24, dotpay, other
```

---

## Filtrowanie

Format: `/invoices.json?q[PARAMETR_modyfikator]=WARTOŚĆ`

### Modyfikatory tekstowe

| Modyfikator | Znaczenie |
|---|---|
| `_eq` | Równa się |
| `_cont` | Zawiera |

### Modyfikatory dat

| Modyfikator | Znaczenie |
|---|---|
| `_lt` | Mniejsze niż |
| `_gt` | Większe niż |
| `_lteq` | Mniejsze lub równe |
| `_gteq` | Większe lub równe |

### Inne

| Modyfikator | Znaczenie |
|---|---|
| `_null` | Pole jest puste (true/false) |

### Przykłady filtrowania

```bash
# Wyszukaj fakturę po numerze
GET /api/v3/invoices.json?q[number_eq]=1/09/2024

# Wyszukaj klienta po NIP
curl -H "X-inFakt-ApiKey: KLUCZ" \
  -H "Content-Type: application/json" \
  -X GET \
  -d '{"q": {"nip_eq": "1234567890"}}' \
  https://api.infakt.pl/api/v3/clients.json

# Faktury bez daty opłacenia
GET /api/v3/invoices.json?q[paid_date_null]=true

# Faktury wystawione po dacie
GET /api/v3/invoices.json?q[invoice_date_gteq]=2024-01-01
```

---

## Stronicowanie

```bash
GET /api/v3/clients.json?offset=10&limit=50
```

Maksymalna wartość `limit`: **100**

### Format odpowiedzi

```json
{
  "metainfo": {
    "count": 10,
    "total_count": 467,
    "next": "https://api.infakt.pl/api/v3/clients.json?offset=20&limit=10",
    "previous": "https://api.infakt.pl/api/v3/clients.json?offset=0&limit=10"
  },
  "entities": [...]
}
```

---

## Sortowanie

Format: `order=parametr typ_sortowania`

```bash
# Sortowanie rosnąco
GET /api/v3/products.json?order=name asc

# Sortowanie malejąco
GET /api/v3/invoices.json?order=invoice_date desc
```

---

## Zawężanie pól (Partial Response)

Parametr `fields` określa które pola zwrócić:

```bash
GET /api/v3/invoices.json?fields=number,services(name,tax_symbol)
GET /api/v3/bank_accounts.json?fields=bank_name
```

---

## Zarządzanie klientami

### Stwórz klienta

```bash
curl -H "X-inFakt-ApiKey: KLUCZ" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"client":{
    "company_name": "Nowa Firma Sp. z o.o.",
    "nip": "1234567890",
    "street": "Główna",
    "street_number": "10",
    "city": "Kraków",
    "postal_code": "30-001",
    "country": "PL",
    "payment_method": "transfer",
    "days_to_payment": 14
  }}' \
  https://api.infakt.pl/api/v3/clients.json
```

### Edytuj klienta

```bash
curl -H "X-inFakt-ApiKey: KLUCZ" \
  -H "Content-Type: application/json" \
  -X PUT \
  -d '{"client":{"company_name":"Zmieniona Nazwa Sp. z o.o."}}' \
  https://api.infakt.pl/api/v3/clients/1.json
```

### Definicja klienta

| Pole | Typ | Opis |
|---|---|---|
| `id` | integer | ID klienta (readonly) |
| `company_name` | string | Nazwa firmy |
| `first_name` | string | Imię |
| `last_name` | string | Nazwisko |
| `nip` | string | NIP |
| `street`, `street_number`, `flat_number` | string | Adres |
| `city`, `postal_code` | string | Miasto i kod (NN-NNN) |
| `country` | string | Kod Alpha-2 (wymagane) |
| `email` | string | Email |
| `phone_number` | string | Telefon |
| `web_site` | string | Strona WWW |
| `payment_method` | string | Domyślna metoda płatności |
| `days_to_payment` | integer | Termin płatności w dniach |
| `business_activity_kind` | string | `private_person`, `self_employed`, `other_business` |
| `receiver` | string | Odbierający dokument |
| `invoice_note` | string | Domyślne uwagi do faktur |
| `note` | string | Uwagi o kliencie |
| `same_forward_address` | boolean | Adres koresp. = firmowy (domyślnie true) |
| `mailing_company_name` | string | Nazwa firmy do korespondencji |
| `mailing_street` | string | Ulica korespondencyjna |
| `mailing_city` | string | Miasto korespondencyjne |
| `mailing_postal_code` | string | Kod korespondencyjny |

---

## Faktura korygująca

```bash
POST /api/v3/async/corrective_invoices.json
```

Dodatkowe pola:

| Pole | Typ | Opis |
|---|---|---|
| `corrected_invoice_number` | string | Numer faktury korygowanej |
| `corrected_invoice_date` | date | Data faktury korygowanej |
| `correction_reason` | string | `mistake` lub `other` |
| `confirmation` | boolean | Otrzymano podpisaną fakturę |
| `confirmation_date` | date | Data podpisania |

---

## Faktura marża

```bash
POST /api/v3/async/margin_invoices.json
```

| Pole | Typ | Opis |
|---|---|---|
| `margin_kind` | string | Procedura marży (wymagane) |
| `gross_price` | integer | Brutto łącznie z marżą w groszach |
| `margin_amount_price` | integer | Marża w groszach |

Procedury marży (`margin_kind`):
- `second_hand_goods` — towary używane
- `travel_agencies` — biura podróży
- `works_of_art` — dzieła sztuki
- `collectables_and_antiques` — przedmioty kolekcjonerskie

---

## Faktura zaliczkowa

```bash
POST /api/v3/async/advance_invoices.json
```

| Pole | Typ | Opis |
|---|---|---|
| `advance_date` | date | Data otrzymania zaliczki |
| `advance_price` | integer | Wpłacona zaliczka w groszach |
| `previous_advance_id` | integer | ID poprzedniej zaliczki |
| `previous_advances` | array | Lista poprzednich zaliczek (readonly) |

---

## Odbiorca JST (Jednostka Samorządu Terytorialnego)

Przy fakturach dla JST można podać adres odbiorcy:

```json
{
  "invoice": {
    "local_government_recipient_address": {
      "company_name": "Urząd Miejski Krakowa",
      "nip": "1060006024",
      "street": "Plac Wszystkich Świętych",
      "street_number": "3-4",
      "postal_code": "31-004",
      "city": "Kraków",
      "country": "PL"
    }
  }
}
```

---

## Kody błędów

| Kod | Opis |
|---|---|
| `200` | OK |
| `201` | Zasób utworzony |
| `202` | Przyjęto do przetworzenia |
| `204` | Usunięto |
| `400` | Nieprawidłowa wartość parametru |
| `401` | Brak autoryzacji |
| `402` | Przekroczono limit planu |
| `403` | Brak uprawnień / blokada IP |
| `404` | Nie znaleziono |
| `406` | Niepoprawny typ danych |
| `422` | Błędy walidacji |
| `423` | Zasób zablokowany |
| `429` | Przekroczono limit zapytań |
| `503` | Serwis niedostępny |

---

## Limity

| Limit | Wartość |
|---|---|
| GET z jednego IP | 300 zapytań / 60 s |
| POST/PUT/DELETE z jednego IP | 150 zapytań / 60 s |
| Rekordów na stronę | 100 |
| Wysyłek maili (płatne konto) | 3000 / dzień |
| Wysyłek maili (bez płatności) | 20 / dzień |

---

## Kodowanie

Wszystkie stringi muszą być kodowane w **UTF-8**.

Parametry POST/PATCH/PUT/DELETE kodowane jako JSON z nagłówkiem:
```
Content-Type: application/json
```

---

## Webhooki

inFakt obsługuje webhooki do powiadamiania o zdarzeniach (np. utworzenie faktury).

Konfiguracja: [Instrukcja dodania webhooka](https://pomoc.infakt.pl/hc/pl/articles/1460299772801)
Aktywacja: [Instrukcja aktywacji](https://pomoc.infakt.pl/hc/pl/articles/14603458812818)

---

## KSeF 2.0 (e-Faktury)

Pełna dokumentacja integracji z Krajowym Systemem e-Faktur dostępna w pliku **[ksef.md](ksef.md)**.

---

## Przydatne linki

- Dokumentacja API: https://docs.infakt.pl
- Sandbox rejestracja: https://konto.sandbox-infakt.pl/rejestracja
- Historia zmian: https://www.infakt.pl/historia-zmian-w-aplikacji-infakt/
- Regulamin: https://www.infakt.pl/regulamin/
- Pomoc inFakt: https://pomoc.infakt.pl
