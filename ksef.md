# KSeF (e-Faktury) — Integracja z inFakt API v3

## Wstęp

Krajowy System e-Faktur (KSeF) to obowiązkowy system elektronicznego fakturowania w Polsce. inFakt API v3 zapewnia integrację z KSeF przez dwa namespace'y:

- **`/ksef/`** — pełne zarządzanie integracją (tworzenie, usuwanie, sprawdzanie statusu) oraz wysyłka/import faktur
- **`/ksef2/`** — endpointy KSeF 2.0 (onboarding przez kis-app, tylko odczyt statusu integracji, wysyłka/import faktur, anulowanie eksportu)

Możliwości API:
- Wysyłanie faktur do KSeF
- Sprawdzanie statusu przetwarzania
- Pobieranie faktur w formacie XML/PDF/HTML
- Import faktur przychodowych i kosztowych z KSeF
- Zarządzanie integracją (token autoryzacyjny — tylko `/ksef/`)

**Wymagane scopes:**
- Zarządzanie integracją (`/ksef/`): `api:ksef:integration:write`
- Status integracji (`/ksef2/`): `api:invoices:read`
- Wysyłka faktur do KSeF: `api:invoices:write`
- Pobieranie XML / import: `api:invoices:read`

---

## Spis treści

1. [Integracja — zarządzanie połączeniem](#integracja)
2. [KSeF 2.0 — różnice](#ksef-20--różnice)
3. [Wysyłka faktur do KSeF](#wysyłka-faktur)
4. [Sprawdzanie statusu](#sprawdzanie-statusu)
5. [Pobieranie XML faktury](#pobieranie-xml)
6. [Import faktur z KSeF](#import-faktur)
7. [Obiekt ksef_data](#obiekt-ksef_data)
8. [Obsługiwane typy faktur](#obsługiwane-typy-faktur)
9. [Wysyłka przez endpoint zasobu](#wysyłka-przez-endpoint-zasobu)
10. [Obsługa błędów](#obsługa-błędów)
11. [Migracja do KSeF](#migracja-do-ksef)

---

## Integracja

Zarządzanie połączeniem z KSeF odbywa się przez endpoint `/ksef/integration.json`.

### Sprawdź status integracji

```bash
GET /api/v3/ksef/integration.json
```

**Odpowiedź (integracja aktywna):**

```json
{
  "active": true,
  "incomes_last_fetched_at": "2024-12-01T10:00:00+01:00",
  "costs_last_fetched_at": "2024-12-01T10:00:00+01:00"
}
```

**Odpowiedź (brak integracji):**

```json
{
  "active": false,
  "incomes_last_fetched_at": null,
  "costs_last_fetched_at": null
}
```

| Atrybut | Typ | Opis |
|---|---|---|
| `active` | boolean | Czy integracja aktywna |
| `incomes_last_fetched_at` | datetime | Data ostatniego pobrania faktur przychodowych |
| `costs_last_fetched_at` | datetime | Data ostatniego pobrania faktur kosztowych |

### Utwórz integrację

```bash
curl -H "X-inFakt-ApiKey: KLUCZ" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"access_token": "TOKEN_Z_KSEF"}' \
  https://api.infakt.pl/api/v3/ksef/integration.json
```

**Token** generuje się w aplikacji Ministerstwa Finansów ([ksef.mf.gov.pl](https://ksef.mf.gov.pl/web/login)) — logowanie profilem zaufanym, podpisem kwalifikowanym lub pieczęcią kwalifikowaną. Następnie należy przejść do opcji wygenerowania tokena autoryzacyjnego i nadać uprawnienia do odczytu i zapisu faktur.

Token musi być wygenerowany na ten sam NIP, który jest wpisany w danych firmowych na koncie inFakt. inFakt wykona testowe zapytanie do KSeF — jeżeli zakończy się sukcesem, integracja jest potwierdzona.

> Jeżeli konto jest już zintegrowane z KSeF, należy najpierw usunąć bieżącą integrację przed wprowadzeniem nowej (w przeciwnym razie zwracany jest błąd 422).

**Odpowiedź sukces (201):**

```json
{
  "active": true,
  "changed_at": "2024-01-15 10:00:00 +0100"
}
```

**Odpowiedź błąd (422):**

```json
{
  "error": "Nie udało się ukończyć procesu integracji. Upewnij się, że token jest poprawny i zgodny z NIP w danych firmowych w inFakt"
}
```

### Usuń integrację

```bash
DELETE /api/v3/ksef/integration.json
```

> Usunięcie integracji następuje tylko po stronie inFakt. Token dalej pozostaje ważny w aplikacji MF. Aby trwale usunąć token, zaloguj się w aplikacji MF → Tokeny → usuń wybrany token.

> **Uwaga:** Endpointy `POST` i `DELETE` dla `/ksef/integration.json` dotyczą wyłącznie namespace'u `/ksef/` (legacy). W `/ksef2/` onboarding integracji odbywa się przez kis-app, a nie przez API.

---

## KSeF 2.0 — różnice

Namespace `/ksef2/` to implementacja KSeF 2.0, która różni się od legacy `/ksef/`:

| Funkcja | `/ksef/` | `/ksef2/` |
|---|---|---|
| Sprawdź status integracji | `GET /ksef/integration.json` | `GET /ksef2/integration.json` |
| Utwórz integrację | `POST /ksef/integration.json` | Brak — onboarding przez kis-app |
| Usuń integrację | `DELETE /ksef/integration.json` | Brak — zarządzanie przez kis-app |
| Wyślij fakturę | `POST /ksef/documents/{uuid}/send.json` | `POST /ksef2/documents/{uuid}/send.json` |
| Wyślij wiele faktur | `POST /ksef/documents/send.json` | `POST /ksef2/documents/send.json` |
| Status wysyłki | `GET /ksef/documents/{uuid}/status.json` | `GET /ksef2/documents/{uuid}/status.json` |
| Pobierz XML | `GET /ksef/documents/{uuid}/download_xml.json` | `GET /ksef2/documents/{uuid}/download_xml.json` |
| Import przychodowych | `GET /ksef/import/incomes.json` | `GET /ksef2/import/incomes.json` |
| Import kosztowych | `GET /ksef/import/costs.json` | `GET /ksef2/import/costs.json` |
| Import pojedynczej | `GET /ksef/import/{ksef_number}.json` | `GET /ksef2/import/{ksef_number}.json` |
| Anuluj eksport | Brak | `POST /ksef2/cancel_export.json` |

**Wymagane scopy dla `/ksef2/integration.json`:** `api:invoices:read` (nie `api:ksef:integration:write`).

Pozostałe endpointy `/ksef2/` działają identycznie jak `/ksef/` — różnica leży w sposobie integracji (onboarding) i wewnętrznym routingu do kis-app.

---

## Wysyłka faktur

### Wyślij pojedynczą fakturę

```bash
POST /api/v3/ksef/documents/{document_uuid}/send.json
```

`document_uuid` to UUID dokumentu przychodowego w inFakt (faktura VAT, korygująca, marża, zaliczkowa lub końcowa). Można go pozyskać odpytując końcówkę danego zasobu z listą dokumentów.

Wysyłka odbywa się asynchronicznie. Status można zweryfikować:
- dedykowaną końcówką: `GET /api/v3/ksef/documents/{document_uuid}/status.json`
- węzłem `ksef_data` na podglądzie faktury
- webhookiem powiadamiającym o finalnym statusie (sukces/błąd)

Użytkownik musi być zintegrowany z KSeF, w innym wypadku zwracany jest kod 422.

**Przykład:**

```bash
curl -H "X-inFakt-ApiKey: KLUCZ" \
  -X POST \
  https://api.infakt.pl/api/v3/ksef/documents/0be870f9-aa52-4d0f-a9ff-994e374ecff0/send.json
```

**Odpowiedź sukces (200):**

```json
{
  "request_uuid": "d34279dc-b1a1-4852-b475-eb36c335992f",
  "invoice_uuid": "0be870f9-aa52-4d0f-a9ff-994e374ecff0",
  "invoice_kind": "vat",
  "ksef_number": null,
  "status": "sent",
  "status_description": "Faktura została wysłana do przetworzenia w KSeF.",
  "timestamps": {
    "request_created_at": "2024-01-15 16:00:00 +0100",
    "request_finished_at": null
  }
}
```

**Odpowiedź błąd — brak integracji (422):**

```json
{
  "error": "Użytkownik nie jest zintegrowany z KSeF."
}
```

**Odpowiedź błąd — nie znaleziono faktury (404):**

```json
{
  "error": "Zasób którego szukasz nie został znaleziony."
}
```

### Wyślij wiele faktur

```bash
POST /api/v3/ksef/documents/send.json
```

**Body:**

```json
{
  "uuids": [
    "0be870f9-aa52-4d0f-a9ff-994e374ecff0",
    "ed46324d-d8bc-4e71-ad8c-265316da97e2",
    "88afc9f7-ae2d-4534-9e74-e813aed9439b"
  ]
}
```

### Wysyłka z powiadomieniem klienta

Przy wysyłce przez endpoint `/invoices/{uuid}/send_to_ksef.json` można jednocześnie wysłać powiadomienie emailem do klienta. Wysłanie emailem zmienia status faktury na „Wysłano".

```json
{
  "inform_via_email": {
    "print_type": "copy",
    "recipient": "klient@example.com",
    "locale": "pl",
    "send_copy": false,
    "content": "Treść wiadomości email"
  }
}
```

| Parametr | Typ | Wymagany | Opis |
|---|---|---|---|
| `inform_via_email` | object | Tak | Zawiera dane powiadomienia |
| `print_type` | string | Tak | `original`, `copy`, `original_duplicate`, `copy_duplicate`, `duplicate`, `regular` |
| `locale` | string | Nie | `pl`, `en`, `pe` |
| `recipient` | string | Nie | Email odbiorcy |
| `send_copy` | boolean | Nie | Czy wysłać kopię do właściciela konta |
| `content` | string | Nie | Treść wiadomości email |

---

## Sprawdzanie statusu

```bash
GET /api/v3/ksef/documents/{document_uuid}/status.json
```

### Możliwe statusy przetwarzania

| Status | Opis |
|---|---|
| `sent` | Faktura wysłana do przetworzenia w KSeF (status początkowy) |
| `success` | Poprawnie przetworzona w KSeF, nadano `ksef_number` |
| `error` | Nie przetworzona w KSeF, szczegóły błędu w `status_description` |

> Zachęcamy do skonfigurowania webhooka, który poinformuje o zmianie statusu przetwarzania na końcowy — ograniczy to ilość zbędnych zapytań do endpointu `status.json`.

### Odpowiedź — w trakcie przetwarzania

```json
{
  "request_uuid": "d34279dc-b1a1-4852-b475-eb36c335992f",
  "invoice_uuid": "0be870f9-aa52-4d0f-a9ff-994e374ecff0",
  "invoice_kind": "vat",
  "ksef_number": null,
  "status": "sent",
  "status_description": "Faktura została wysłana do przetworzenia w KSeF.",
  "timestamps": {
    "request_created_at": "2024-01-15 16:00:00 +0100",
    "request_finished_at": null
  }
}
```

### Odpowiedź — przetworzona pomyślnie

```json
{
  "request_uuid": "d34279dc-b1a1-4852-b475-eb36c335992f",
  "invoice_uuid": "0be870f9-aa52-4d0f-a9ff-994e374ecff0",
  "invoice_kind": "vat",
  "ksef_number": "7343521162-20231004-47A70D8BD670-57",
  "status": "success",
  "status_description": "Faktura została przetworzona.",
  "timestamps": {
    "request_created_at": "2024-01-15 16:00:00 +0100",
    "request_finished_at": "2024-01-15 16:01:30 +0100"
  }
}
```

### Odpowiedź — błąd

```json
{
  "request_uuid": "d34279dc-b1a1-4852-b475-eb36c335992f",
  "invoice_uuid": "0be870f9-aa52-4d0f-a9ff-994e374ecff0",
  "invoice_kind": "vat",
  "ksef_number": null,
  "status": "error",
  "status_description": "Szczegółowy opis błędu walidacji...",
  "timestamps": {
    "request_created_at": "2024-01-15 16:00:00 +0100",
    "request_finished_at": "2024-01-15 16:01:00 +0100"
  }
}
```

---

## Pobieranie XML

### Pobierz XML faktury przez KSeF endpoint

```bash
GET /api/v3/ksef/documents/{document_uuid}/download_xml.json
```

`document_uuid` — UUID dokumentu w inFakt.

### Pobierz XML przez endpoint zasobu

```bash
GET /api/v3/invoices/{invoice_uuid}/download_xml.json
GET /api/v3/corrective_invoices/{uuid}/download_xml.json
GET /api/v3/margin_invoices/{uuid}/download_xml.json
GET /api/v3/advance_invoices/{uuid}/download_xml.json
GET /api/v3/final_invoices/{uuid}/download_xml.json
```

Plik XML jest zgodny ze schematem FA(2) wymaganym przez KSeF.

> Pobranie faktury w szkicu jako XML powoduje zmianę statusu i uwzględnienie jej w księgowości.

**Wymagany scope:** `api:invoices:read`

---

## Import faktur

Import faktur pobiera dokumenty bezpośrednio z KSeF — nie muszą istnieć w inFakt.

### Import faktur przychodowych

```bash
GET /api/v3/ksef/import/incomes.json
```

### Import faktur kosztowych

```bash
GET /api/v3/ksef/import/costs.json
```

### Parametry (wspólne)

| Parametr | Opis |
|---|---|
| `offset` | Przesunięcie (stronicowanie) |
| `limit` | Liczba wyników (max 100) |
| `order` | Sortowanie, np. `invoice_date desc` |
| `q[invoice_date_gteq]` | Faktury z datą wystawienia >= |
| `q[invoice_date_lteq]` | Faktury z datą wystawienia <= |

### Przykłady

```bash
# Ze stronicowaniem
GET /api/v3/ksef/import/incomes.json?offset=0&limit=25

# Sortowanie po dacie malejąco
GET /api/v3/ksef/import/incomes.json?order=invoice_date desc

# Faktury z datą przed 1 czerwca 2024
GET /api/v3/ksef/import/incomes.json?q[invoice_date_lteq]=2024-06-01
```

### Odpowiedź

```json
{
  "metainfo": {
    "count": 2,
    "total_count": 15
  },
  "entities": [
    {
      "client_name": "Magda Krakowska",
      "client_tax_code": "3423016760",
      "created_at": "2024-08-23T07:06:08.407Z",
      "currency": "PLN",
      "gross_price": 12300,
      "invoice_date": "2024-09-29",
      "invoice_kind": "vat",
      "invoice_number": "2/09/2024",
      "ksef_number": "6020124091-20220823-8EA71B8A5257-F6",
      "net_price": 10000,
      "schema_version": "V1",
      "seller_name": "Firma Sp. z o.o.",
      "seller_tax_code": "5268969361",
      "tax_price": 2300
    }
  ]
}
```

| Pole | Typ | Opis |
|---|---|---|
| `client_name` | string | Nazwa nabywcy |
| `client_tax_code` | string | NIP nabywcy |
| `created_at` | datetime | Data pobrania do inFakt |
| `currency` | string | Waluta faktury |
| `gross_price` | integer | Kwota brutto w groszach |
| `invoice_date` | string | Data wystawienia |
| `invoice_kind` | string | Typ faktury |
| `invoice_number` | string | Numer faktury |
| `ksef_number` | string | Numer nadany w KSeF |
| `net_price` | integer | Kwota netto w groszach |
| `schema_version` | string | Wersja schematu XML (np. `V1`, `V2`) |
| `seller_name` | string | Nazwa sprzedawcy |
| `seller_tax_code` | string | NIP sprzedawcy |
| `tax_price` | integer | Kwota VAT w groszach |

### Import pojedynczej faktury

```bash
GET /api/v3/ksef/import/{ksef_number}.json
GET /api/v3/ksef/import/{ksef_number}.json?file_format=pdf
GET /api/v3/ksef/import/{ksef_number}.json?file_format=html
```

| Format | Opis |
|---|---|
| `xml` | Plik XML (domyślnie) |
| `pdf` | Plik PDF |
| `html` | Plik HTML |

Faktura nie musi istnieć w inFakt. `ksef_number` pozyskuje się z listy importu.

---

## Obiekt ksef_data

Każda faktura (VAT, korygująca, marża, zaliczkowa, końcowa) zawiera pole `ksef_data` z informacjami o statusie w KSeF:

```json
{
  "ksef_data": {
    "request_uuid": "d34279dc-b1a1-4852-b475-eb36c335992f",
    "ksef_number": "7343521162-20231004-47A70D8BD670-57",
    "status": "success",
    "status_description": "Faktura została przetworzona.",
    "timestamps": {
      "request_created_at": "2024-01-15 16:00:00 +0100",
      "request_finished_at": "2024-01-15 16:01:30 +0100"
    }
  }
}
```

| Pole | Typ | Opis |
|---|---|---|
| `request_uuid` | string | Unikalny numer zlecenia wysyłki do KSeF w inFakt |
| `ksef_number` | string | Unikalny numer faktury nadany przez KSeF |
| `status` | string | `sent`, `success`, `error` |
| `status_description` | string | Opis statusu przetwarzania |
| `timestamps.request_created_at` | datetime | Data utworzenia zlecenia wysyłki do KSeF |
| `timestamps.request_finished_at` | datetime | Data zakończenia przetwarzania w KSeF |

Jeśli faktura nie była wysyłana do KSeF, `ksef_data` wynosi `null`.

> **Uwaga:** Endpointy wysyłki (`send.json`, `send_to_ksef.json`) oraz statusu (`status.json`) zwracają rozszerzoną odpowiedź zawierającą dodatkowo pola `invoice_kind` i `invoice_uuid`. Te pola nie są częścią obiektu `ksef_data` przechowywanego na fakturze.

---

## Obsługiwane typy faktur

Do KSeF można wysłać następujące typy dokumentów:

| Typ | Endpoint wysyłki | `invoice_kind` |
|---|---|---|
| Faktura VAT | `/invoices/{uuid}/send_to_ksef.json` | `vat` |
| Faktura korygująca | `/corrective_invoices/{uuid}/send_to_ksef.json` | `corrective_invoice` |
| Faktura marża | `/margin_invoices/{uuid}/send_to_ksef.json` | `margin` |
| Faktura zaliczkowa | `/advance_invoices/{uuid}/send_to_ksef.json` | `advance` |
| Faktura końcowa | `/final_invoices/{uuid}/send_to_ksef.json` | `final` |

Alternatywnie, uniwersalny endpoint:
```
POST /api/v3/ksef/documents/{document_uuid}/send.json
```

---

## Wysyłka przez endpoint zasobu

Każdy typ faktury posiada dedykowany endpoint KSeF, który działa identycznie jak uniwersalny `/ksef/documents/`:

```bash
# Faktura VAT
POST /api/v3/invoices/{invoice_uuid}/send_to_ksef.json

# Faktura korygująca
POST /api/v3/corrective_invoices/{corrective_invoice_uuid}/send_to_ksef.json

# Faktura marża
POST /api/v3/margin_invoices/{margin_invoice_uuid}/send_to_ksef.json

# Faktura zaliczkowa
POST /api/v3/advance_invoices/{advance_invoice_uuid}/send_to_ksef.json

# Faktura końcowa
POST /api/v3/final_invoices/{final_invoice_uuid}/send_to_ksef.json
```

Wybór endpointu zależy od preferencji integratora — oba działają identycznie.

---

## Obsługa błędów

### Typowe przyczyny odrzucenia

- Brak aktywnej integracji z KSeF
- Brakujące wymagane dane sprzedawcy/nabywcy (NIP, adres)
- Faktura w statusie szkicu bez wymaganych pól
- Przekroczenie limitów długości pól (nazwa pozycji max 256 znaków, uwagi max 3500 znaków)
- Brak numeru konta bankowego zweryfikowanego w aplikacji

### Strategia ponowień

Przy statusie `error` od strony KSeF:
1. Sprawdź `status_description` — identyfikuje przyczynę
2. Popraw dane na fakturze
3. Wyślij ponownie

Przy błędach `429` / `503`:
- Zastosuj exponential backoff (do 3 prób)

---

## Migracja do KSeF

### Checklist

1. **Aktywuj integrację** — wygeneruj token w aplikacji MF i połącz z inFakt
2. **Uzupełnij dane sprzedawcy** — NIP, pełny adres (ulica, miasto, kod pocztowy, kraj)
3. **Uzupełnij dane nabywców** — NIP dla firm, pełny adres
4. **Sprawdź długości pól:**
   - Nazwa pozycji: max 256 znaków
   - Uwagi/opis faktury: max 3500 znaków
   - Telefon: max 16 znaków
   - Email: max 255 znaków
   - Powód korekty: max 256 znaków
5. **Testuj na Sandbox** — użyj `api.sandbox-infakt.pl` do testów
6. **Obsłuż statusy** — zaimplementuj polling `status.json` lub webhooki
7. **Pobierz XML** — przetestuj pobieranie dokumentów w formacie XML
8. **Skonfiguruj automatyzację** — wysyłka po wystawieniu lub ręczna

### Środowiska

| Środowisko | API Endpoint | KSeF | Token |
|---|---|---|---|
| Sandbox | `api.sandbox-infakt.pl/api/v3` | Demo KSeF (pre-produkcja) | [ksef-test.mf.gov.pl](https://ksef-test.mf.gov.pl/web/login) |
| Produkcja | `api.infakt.pl/api/v3` | Produkcyjny KSeF | [ksef.mf.gov.pl](https://ksef.mf.gov.pl/web/login) |

> Na środowisku Sandbox należy zalogować się na `ksef-test.mf.gov.pl`, wpisać NIP `1111111111` i skorzystać z uwierzytelniania certyfikatem kwalifikowanym.

---

## Pełny przykład workflow

### 1. Sprawdź integrację

```bash
curl -H "X-inFakt-ApiKey: KLUCZ" \
  https://api.infakt.pl/api/v3/ksef/integration.json
```

### 2. Utwórz fakturę

```bash
curl -H "X-inFakt-ApiKey: KLUCZ" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{"invoice":{
    "payment_method": "transfer",
    "bank_account": "70102010130000010200026526",
    "client_company_name": "Klient Sp. z o.o.",
    "client_tax_code": "9452121681",
    "client_country": "PL",
    "client_street": "Uliczna",
    "client_street_number": "1",
    "client_city": "Kraków",
    "client_post_code": "30-001",
    "services": [{
      "name": "Usługa IT",
      "unit_net_price": 1000000,
      "quantity": 1,
      "tax_symbol": "23"
    }]
  }}' \
  https://api.infakt.pl/api/v3/async/invoices.json
```

### 3. Wyślij do KSeF

```bash
curl -H "X-inFakt-ApiKey: KLUCZ" \
  -X POST \
  https://api.infakt.pl/api/v3/ksef/documents/{document_uuid}/send.json
```

### 4. Sprawdź status

```bash
curl -H "X-inFakt-ApiKey: KLUCZ" \
  https://api.infakt.pl/api/v3/ksef/documents/{document_uuid}/status.json
```

### 5. Pobierz XML

```bash
curl -H "X-inFakt-ApiKey: KLUCZ" \
  https://api.infakt.pl/api/v3/ksef/documents/{document_uuid}/download_xml.json \
  -o faktura.xml
```

---

## Przydatne linki

- Dokumentacja API inFakt: https://docs.infakt.pl
- KSeF Portal MF: https://ksef.mf.gov.pl
- KSeF Test MF: https://ksef-test.mf.gov.pl
- Sandbox inFakt: https://konto.sandbox-infakt.pl/rejestracja
- Schemat FA(2): Specyfikacja Ministerstwa Finansów
