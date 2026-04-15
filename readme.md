# inFakt API v3 — Dokumentacja

## Wstęp

inFakt API oparty jest o architekturę REST i umożliwia dostęp do zasobów serwisu inFakt.pl za pomocą interfejsu JSON. Dzięki temu możliwe staje się stworzenie zewnętrznych aplikacji integrujących się z kontem użytkownika inFakt.pl.

Niniejsza dokumentacja odnosi się do API w wersji 3 (APIv3).

Pełna dokumentacja interaktywna: https://docs.infakt.pl

---

## Spis treści

- [Punkt dostępowy](#punkt-dostępowy)
- [Uwierzytelnianie](#uwierzytelnianie)
- [Zakresy uprawnień (Scopes)](#zakresy-uprawnień-scopes)
- [Sandbox](#sandbox)
- [Przychody](#przychody)
  - [Faktury VAT](#faktury-vat)
    - [Typy faktur](#typy-faktur)
    - [Tworzenie faktury (asynchroniczne)](#tworzenie-faktury-asynchroniczne)
    - [Tworzenie faktury (synchroniczne)](#tworzenie-faktury-synchroniczne)
    - [Sprawdzenie statusu tworzenia](#sprawdzenie-statusu-tworzenia)
    - [Listowanie faktur](#listowanie-faktur)
    - [Podgląd faktury](#podgląd-faktury)
    - [Edycja faktury](#edycja-faktury)
    - [Usuwanie faktury](#usuwanie-faktury)
    - [Pobranie PDF](#pobranie-pdf)
    - [Wysyłka emailem](#wysyłka-emailem)
    - [Oznaczenie jako zapłacona](#oznaczenie-jako-zapłacona)
    - [Oznaczenie jako zaksięgowana](#oznaczenie-jako-zaksięgowana)
    - [Następny numer faktury](#następny-numer-faktury)
    - [Załączniki](#załączniki)
    - [Link do udostępniania](#link-do-udostępniania)
    - [Szybkie płatności](#szybkie-płatności)
    - [Definicja faktury VAT](#definicja-faktury-vat)
    - [Definicja pozycji (Service)](#definicja-pozycji-service)
    - [Odbiorca JST](#odbiorca-jst-jednostka-samorządu-terytorialnego)
  - [Faktury korygujące VAT](#faktura-korygująca)
  - [Faktury marża](#faktura-marża)
  - [Faktury zaliczkowe](#faktura-zaliczkowa)
  - [Faktury końcowe](#faktura-końcowa)
  - [Faktury OSS](#faktury-oss)
  - [Faktury korygujące OSS](#faktury-korygujące-oss)
  - [Faktury wewnętrzne](#faktury-wewnętrzne)
  - [Raporty fiskalne](#raporty-fiskalne)
  - [Dowody wewnętrzne](#dowody-wewnętrzne)
  - [Utarg dzienny](#utarg-dzienny)
- [Klienci](#klienci)
  - [Listowanie klientów](#listowanie-klientów)
  - [Podgląd klienta](#podgląd-klienta)
  - [Tworzenie klienta](#tworzenie-klienta)
  - [Edycja klienta](#edycja-klienta)
  - [Usuwanie klienta](#usuwanie-klienta)
  - [Definicja klienta](#definicja-klienta)
- [Produkty](#produkty)
  - [Listowanie produktów](#listowanie-produktów)
  - [Podgląd produktu](#podgląd-produktu)
  - [Tworzenie produktu](#tworzenie-produktu)
  - [Edycja produktu](#edycja-produktu)
  - [Usuwanie produktu](#usuwanie-produktu)
  - [Definicja produktu](#definicja-produktu)
- [Konta bankowe](#konta-bankowe)
- [Koszty](#koszty)
- [Księgowość](#księgowość)
  - [JPK V7](#jpk-v7)
  - [Podatek VAT-UE](#podatek-vat-ue)
  - [Podatek dochodowy](#podatek-dochodowy)
  - [Księga przychodów i rozchodów](#księga-przychodów-i-rozchodów)
  - [Składki ZUS](#składki-zus)
  - [Koszyk płatności za podatki](#koszyk-płatności-za-podatki)
- [Dane referencyjne](#dane-referencyjne)
  - [Stawki VAT](#stawki-vat)
  - [Stawki VAT dla OSS](#stawki-vat-dla-oss)
  - [Podstawy zwolnień z VAT](#podstawy-zwolnień-z-vat)
  - [Stawki ryczałtu ewidencjonowanego](#stawki-ryczałtu-ewidencjonowanego)
  - [Kody GTU](#kody-gtu)
  - [Oznaczenia dokumentów przychodowych](#oznaczenia-dokumentów-przychodowych)
  - [Rodzaje transakcji](#rodzaje-transakcji)
  - [Kraje](#kraje)
- [Dane użytkownika](#dane-użytkownika)
- [Metody płatności](#metody-płatności)
- [Filtrowanie](#filtrowanie)
- [Stronicowanie](#stronicowanie)
- [Sortowanie](#sortowanie)
- [Zawężanie pól (Partial Response)](#zawężanie-pól-partial-response)
- [Kody błędów](#kody-błędów)
- [Limity](#limity)
- [Kodowanie](#kodowanie)
- [Webhooki](#webhooki)
- [KSeF (e-Faktury)](#ksef-e-faktury)
- [Przydatne linki](#przydatne-linki)

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

## Przychody

### Faktury VAT

#### Typy faktur

| Typ | Endpoint | Opis |
|---|---|---|
| Faktury VAT | `/invoices.json` | Standardowe faktury VAT i proforma |
| Faktury korygujące VAT | `/corrective_invoices.json` | Korekty do faktur VAT |
| Faktury marża | `/margin_invoices.json` | Faktury w procedurze marży |
| Faktury zaliczkowe | `/advance_invoices.json` | Faktury zaliczkowe |
| Faktury końcowe | `/final_invoices.json` | Faktury końcowe (rozliczające zaliczki) |
| Faktury OSS | `/oss_invoices.json` | Faktury w procedurze OSS |
| Faktury korygujące OSS | `/corrective_oss_invoices.json` | Korekty do faktur OSS |
| Faktury wewnętrzne | `/internal_invoices.json` | Faktury wewnętrzne |
| Raporty fiskalne | `/fiscal_reports.json` | Raporty z kasy fiskalnej |
| Dowody wewnętrzne | `/internal_evidences.json` | Dowody wewnętrzne |
| Utargi dzienne | `/daily_revenues.json` | Ewidencja utargów |

Wspólne operacje dla typów faktur: listowanie, podgląd, tworzenie (async), edycja, usuwanie, PDF, wysyłka emailem, oznaczenie jako zapłacona, następny numer, wysyłka do KSeF, pobranie XML KSeF. Szczegóły dostępnych operacji różnią się w zależności od typu — patrz dokumentacja interaktywna.

### Tworzenie faktury (asynchroniczne)

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

**Odpowiedź (202):**

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

Faktura zawsze tworzy się w statusie `draft`. Po utworzeniu można oznaczyć ją jako zapłaconą, wydrukowaną lub wysłaną.

### Sprawdzenie statusu tworzenia

```bash
GET /api/v3/async/invoices/status/{invoice_task_reference_number}.json
```

### Listowanie faktur

```bash
GET /api/v3/invoices.json
```

Obsługuje filtrowanie, sortowanie, stronicowanie i zawężanie pól.

### Podgląd faktury

```bash
GET /api/v3/invoices/{invoice_uuid}.json
```

### Edycja faktury

```bash
PUT /api/v3/invoices/{invoice_uuid}.json
```

```bash
curl -H "X-inFakt-ApiKey: KLUCZ" \
  -H "Content-Type: application/json" \
  -X PUT \
  -d '{"invoice":{"notes":"Dodatkowe uwagi"}}' \
  https://api.infakt.pl/api/v3/invoices/{invoice_uuid}.json
```

### Usuwanie faktury

```bash
DELETE /api/v3/invoices/{invoice_uuid}.json
```

### Pobranie PDF

```bash
GET /api/v3/invoices/{invoice_uuid}/pdf.json?document_type=original&locale=pl
```

**Parametry:**

| Parametr | Wymagany | Opis |
|---|---|---|
| `document_type` | Tak | `original_copy`, `original`, `copy`, `original_duplicate`, `copy_duplicate`, `duplicate`, `regular`, `double_regular` |
| `locale` | Nie | `pl` – Polski, `en` – Angielski, `pe` – polsko-angielski |

> Pobranie PDF zmienia status faktury na „Wydrukowano".

### Wysyłka emailem

```bash
POST /api/v3/invoices/{invoice_uuid}/deliver_via_email.json
```

```json
{
  "print_type": "original",
  "locale": "pl",
  "recipient": "klient@example.com",
  "send_copy": false
}
```

| Parametr | Wymagany | Opis |
|---|---|---|
| `print_type` | Tak | `original`, `copy`, `original_duplicate`, `copy_duplicate`, `duplicate`, `regular` |
| `locale` | Nie | `pl`, `en`, `pe` |
| `recipient` | Nie | Email odbiorcy (domyślnie email klienta) |
| `send_copy` | Nie | Czy wysłać kopię do właściciela konta |

> Wysłanie emailem zmienia status faktury na „Wysłano".

### Oznaczenie jako zapłacona

```bash
POST /api/v3/async/invoices/{invoice_uuid}/paid.json
```

Operacja asynchroniczna. Kod 201 oznacza przyjęcie zlecenia, nie opłacenie.

| Parametr | Wymagany | Opis |
|---|---|---|
| `paid_date` | Nie | Data opłacenia (RRRR-MM-DD), nie wcześniejsza niż data wystawienia |

W ciele zapytania można podać `allow_correction: true` — umożliwia opłacenie faktury powodujące korekty księgowe.

### Oznaczenie jako zaksięgowana

```bash
PUT /api/v3/invoices/{invoice_uuid}/mark_as_accounted.json
```

Uwzględnia fakturę w szkicu w księgowości.

### Następny numer faktury

```bash
GET /api/v3/invoices/next_number.json?kind=vat&date=2024-01-15
```

| Parametr | Wymagany | Opis |
|---|---|---|
| `kind` | Nie | `vat`, `proforma`, `advance`, `final`, `margin` |
| `date` | Nie | Data wystawienia (RRRR-MM-DD) |

**Odpowiedź:**

```json
{
  "invoice_date": "2024-01-15",
  "invoice_kind": "vat",
  "next_number": "5/01/2024"
}
```

### Załączniki

**Listowanie załączników:**

```bash
GET /api/v3/invoices/{invoice_uuid}/attachments.json
```

**Pobranie załącznika:**

```bash
GET /api/v3/invoices/{invoice_uuid}/attachments/{attachment_id}.json
```

Zwraca obiekt z polami: `id`, `name`, `content_type`, `download_link` (ważny 10 minut).

### Link do udostępniania

**Pobierz link:**

```bash
GET /api/v3/invoices/{invoice_uuid}/share_links.json
```

**Stwórz link:**

```bash
POST /api/v3/invoices/{invoice_uuid}/share_links.json
```

**Przedłuż ważność (o 30 dni):**

```bash
POST /api/v3/invoices/{invoice_uuid}/share_links/prolong.json
```

**Usuń link:**

```bash
DELETE /api/v3/invoices/{invoice_uuid}/share_links.json
```

**Odpowiedź:**

```json
{
  "share_link": "https://app.infakt.pl/app/twoja-faktura/f431d987-...",
  "expiration_date": "2024-02-15"
}
```

Link umożliwia podgląd faktury, drukowanie, import jako koszt i opłacenie przez szybkie płatności.

### Definicja faktury VAT

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

> **Uwaga:** Wszystkie kwoty podawane są w **groszach** (1 PLN = 100 groszy).

### Faktura korygująca

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

### Faktura marża

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

### Faktura zaliczkowa

```bash
POST /api/v3/async/advance_invoices.json
```

| Pole | Typ | Opis |
|---|---|---|
| `advance_date` | date | Data otrzymania zaliczki |
| `advance_price` | integer | Wpłacona zaliczka w groszach |
| `previous_advance_id` | integer | ID poprzedniej zaliczki |
| `previous_advances` | array | Lista poprzednich zaliczek (readonly) |

### Odbiorca JST (Jednostka Samorządu Terytorialnego)

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

## Klienci

### Listowanie klientów

```bash
GET /api/v3/clients.json
```

Obsługuje filtrowanie, sortowanie, stronicowanie i zawężanie pól.

```bash
# Wyszukaj klienta po NIP
GET /api/v3/clients.json?q[nip_eq]=1234567890

# Wyszukaj po nazwie firmy
GET /api/v3/clients.json?q[company_name_cont]=Firma
```

### Podgląd klienta

```bash
GET /api/v3/clients/{client_id}.json
```

### Tworzenie klienta

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

### Edycja klienta

```bash
curl -H "X-inFakt-ApiKey: KLUCZ" \
  -H "Content-Type: application/json" \
  -X PUT \
  -d '{"client":{"company_name":"Zmieniona Nazwa Sp. z o.o."}}' \
  https://api.infakt.pl/api/v3/clients/{client_id}.json
```

### Usuwanie klienta

```bash
DELETE /api/v3/clients/{client_id}.json
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

## Produkty

### Listowanie produktów

```bash
GET /api/v3/products.json
```

Obsługuje filtrowanie, sortowanie, stronicowanie i zawężanie pól.

```bash
# Wyszukaj produkt po nazwie
GET /api/v3/products.json?q[name_eq]=Usługa programistyczna

# Sortowanie po nazwie
GET /api/v3/products.json?order=name asc

# Tylko wybrane pola
GET /api/v3/products.json?fields=name,unit_net_price,tax_symbol
```

### Podgląd produktu

```bash
GET /api/v3/products/{product_id}.json
```

### Tworzenie produktu

```bash
curl -H "X-inFakt-ApiKey: KLUCZ" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"product":{
    "name": "Usługa programistyczna",
    "unit_net_price": 50000,
    "tax_symbol": "23",
    "unit": "godz.",
    "pkwiu": "62.01.11"
  }}' \
  https://api.infakt.pl/api/v3/products.json
```

**Odpowiedź (201):**

```json
{
  "id": 34935,
  "name": "Usługa programistyczna",
  "unit_net_price": 50000,
  "net_price": 50000,
  "tax_price": 11500,
  "gross_price": 61500,
  "tax_symbol": "23",
  "unit": "godz.",
  "quantity": 1,
  "pkwiu": "62.01.11",
  "cn": null,
  "pkob": null,
  "gtu_id": null,
  "discount": "0.0",
  "flat_rate_tax_symbol": "",
  "unit_net_price_before_discount": 50000,
  "purchase_unit_net_price": 0,
  "purchase_unit_gross_price": 0,
  "symbol": "62.01.11 / - / -"
}
```

### Edycja produktu

```bash
curl -H "X-inFakt-ApiKey: KLUCZ" \
  -H "Content-Type: application/json" \
  -X PUT \
  -d '{"product":{"unit_net_price": 60000}}' \
  https://api.infakt.pl/api/v3/products/{product_id}.json
```

### Usuwanie produktu

```bash
DELETE /api/v3/products/{product_id}.json
```

### Definicja produktu

| Pole | Typ | Opis |
|---|---|---|
| `id` | integer | ID produktu (readonly) |
| `name` | string | Nazwa produktu (wymagane) |
| `unit_net_price` | integer | Cena netto/szt. w groszach |
| `net_price` | integer | Wartość netto w groszach |
| `tax_price` | integer | VAT w groszach (readonly) |
| `gross_price` | integer | Wartość brutto w groszach |
| `tax_symbol` | string | Stawka VAT (np. `23`, `8`, `5`, `0`, `zw`, `np`) |
| `unit` | string | Jednostka (np. `szt.`, `godz.`, `usł.`) |
| `quantity` | number | Domyślna ilość |
| `pkwiu` | string | Kod PKWiU |
| `cn` | string | Kod CN |
| `pkob` | string | Kod PKOB |
| `gtu_id` | integer | ID kodu GTU |
| `discount` | string | Rabat w procentach |
| `flat_rate_tax_symbol` | string | Stawka ryczałtu |
| `unit_net_price_before_discount` | integer | Cena przed rabatem w groszach |
| `purchase_unit_net_price` | integer | Cena zakupu netto/szt. w groszach |
| `purchase_unit_gross_price` | integer | Cena zakupu brutto/szt. w groszach |
| `symbol` | string | Symbol (readonly, generowany z pkwiu/cn/pkob) |

---

## Konta bankowe

**Scope:** `api:sensitive:bank_accounts:write`

> Przed użyciem numeru konta na fakturze, konto musi być dodane i zweryfikowane w aplikacji.

### Operacje

```bash
# Listowanie
GET /api/v3/bank_accounts.json

# Podgląd
GET /api/v3/bank_accounts/{bank_account_id}.json

# Tworzenie
POST /api/v3/bank_accounts.json

# Edycja
PUT /api/v3/bank_accounts/{bank_account_id}.json

# Usuwanie
DELETE /api/v3/bank_accounts/{bank_account_id}.json
```

### Tworzenie konta

```bash
curl -H "X-inFakt-ApiKey: KLUCZ" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"bank_account":{
    "account_number": "PL61109010140000071219812874",
    "bank_name": "Santander Bank Polska"
  }}' \
  https://api.infakt.pl/api/v3/bank_accounts.json
```

### Definicja konta bankowego

| Pole | Typ | Opis |
|---|---|---|
| `id` | integer | ID konta (readonly) |
| `account_number` | string | Numer konta (IBAN) |
| `bank_name` | string | Nazwa banku |
| `swift` | string | Kod SWIFT |
| `currency` | string | Waluta konta |
| `custom_name` | string | Nazwa własna |
| `default` | boolean | Czy konto domyślne |

---

## Koszty

**Scope:** `api:costs:read`, `api:costs:write`

| Operacja | Metoda | Endpoint |
|---|---|---|
| Upload kosztu | `POST` | `/documents/costs/upload.json` |
| Listowanie kosztów | `GET` | `/documents/costs.json` |
| Podgląd kosztu | `GET` | `/documents/costs/{uuid}.json` |
| Pobranie wielu kosztów (ZIP) | `GET` | `/documents/costs/download_many.json` |
| Oznacz jako zapłacone (wiele) | `PUT` | `/documents/costs/paid_many.json` |
| Oznacz jako niezapłacone (wiele) | `PUT` | `/documents/costs/unpaid_many.json` |
| Zmień nazwę pliku (wiele) | `PUT` | `/documents/costs/update_file_name_many.json` |
| Przypisz kategorię kosztową (wiele) | `POST` | `/documents/costs/assign_cost_category_many.json` |
| Dodaj notatkę (wiele) | `POST` | `/documents/costs/create_note_many.json` |
| Usuń wiele kosztów | `DELETE` | `/documents/costs/destroy_many.json` |

---

## Księgowość

### JPK V7

**Scope:** `api:accounting:read`, `api:accounting:write`

| Operacja | Metoda | Endpoint |
|---|---|---|
| Listowanie | `GET` | `/saf_v7_files.json` |
| Podgląd | `GET` | `/saf_v7_files/{id}.json` |
| Oznacz jako zapłacony | `POST` | `/saf_v7_files/{id}/paid.json` |

### Podatek VAT-UE

**Scope:** `api:accounting:read`

| Operacja | Metoda | Endpoint |
|---|---|---|
| Listowanie | `GET` | `/vat_eu_taxes.json` |
| Podgląd | `GET` | `/vat_eu_taxes/{id}.json` |

### Podatek dochodowy

**Scope:** `api:accounting:read`, `api:accounting:write`

| Operacja | Metoda | Endpoint |
|---|---|---|
| Listowanie | `GET` | `/income_taxes.json` |
| Podgląd | `GET` | `/income_taxes/{id}.json` |
| Oznacz jako zapłacony | `POST` | `/income_taxes/{id}/paid.json` |

### Księga przychodów i rozchodów

**Scope:** `api:accounting:read`

| Operacja | Metoda | Endpoint |
|---|---|---|
| Listowanie | `GET` | `/books.json` |
| Podgląd | `GET` | `/books/{id}.json` |

### Składki ZUS

**Scope:** `api:accounting:read`, `api:accounting:write`

| Operacja | Metoda | Endpoint |
|---|---|---|
| Listowanie | `GET` | `/insurance_fees.json` |
| Podgląd | `GET` | `/insurance_fees/{id}.json` |
| Oznacz jako zapłacone | `POST` | `/insurance_fees/{id}/paid.json` |

### Koszyk płatności za podatki

**Scope:** `api:accounting:write`

| Operacja | Metoda | Endpoint |
|---|---|---|
| Wygeneruj link do płatności | `POST` | `/payments/document_requests.json` |

---

## Dane referencyjne

| Zasób | Endpoint | Scope | Operacje |
|---|---|---|---|
| Stawki VAT | `/vat_rates.json` | `api:invoices:read` | `GET` — listowanie |
| Stawki VAT dla OSS | `/moss_vat_rates.json` | `api:invoices:read` | `GET` — listowanie |
| Podstawy zwolnień z VAT | `/vat_exemptions.json`, `/{id}.json`, `/selected.json` | `api:invoices:read` | `GET` — listowanie, podgląd, wybrane |
| Stawki ryczałtu ewidencjonowanego | `/flat_rates.json` | `api:invoices:read` | `GET` — listowanie |
| Kody GTU | `/gtus.json`, `/gtus/{id}.json`, `/gtus/selected.json` | `api:invoices:read` | `GET` — listowanie, podgląd, wybrane |
| Oznaczenia dokumentów przychodowych | `/documents_markings/incomes.json`, `/{id}.json`, `/selected.json` | `api:invoices:read` | `GET` — listowanie, podgląd, wybrane |
| Rodzaje transakcji | `/transaction_kinds.json` | `api:invoices:read` | `GET` — listowanie |
| Kraje | `/countries.json` | `api:invoices:read` | `GET` — listowanie |

---

## Dane użytkownika

| Operacja | Metoda | Endpoint |
|---|---|---|
| Szczegóły konta | `GET` | `/account/details.json` |
| Historia zdarzeń | `GET` | `/account/activities.json` |

---

## KSeF (e-Faktury)

Pełna dokumentacja integracji z Krajowym Systemem e-Faktur dostępna w pliku **[ksef.md](ksef.md)**.

**Scopes:** `api:ksef:integration:write`, `api:invoices:write`, `api:invoices:read`

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
GET /api/v3/clients.json?q[nip_eq]=1234567890

# Faktury bez daty opłacenia
GET /api/v3/invoices.json?q[paid_date_null]=true

# Faktury wystawione po dacie
GET /api/v3/invoices.json?q[invoice_date_gteq]=2024-01-01

# Produkt po nazwie
GET /api/v3/products.json?q[name_eq]=Usługa

# Konto bankowe po numerze
GET /api/v3/bank_accounts.json?q[account_number_eq]=PL61109010140000071219812874
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
GET /api/v3/bank_accounts.json?fields=bank_name,account_number
GET /api/v3/products.json?fields=name,unit_net_price
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

## Przydatne linki

- Dokumentacja API: https://docs.infakt.pl
- Sandbox rejestracja: https://konto.sandbox-infakt.pl/rejestracja
- Historia zmian: https://www.infakt.pl/historia-zmian-w-aplikacji-infakt/
- Regulamin: https://www.infakt.pl/regulamin/
- Pomoc inFakt: https://pomoc.infakt.pl
