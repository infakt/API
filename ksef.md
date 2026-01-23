# KSeF 2.0 (e-Faktury) — Integracja z inFakt API v3

## Wstęp

Krajowy System e-Faktur (KSeF) to obowiązkowy system elektronicznego fakturowania w Polsce uruchomiony 1 lutego 2026. inFakt API v3 zapewnia pełną integrację z KSeF 2.0, umożliwiając:

- Wysyłanie faktur do KSeF
- Sprawdzanie statusu przetwarzania
- Pobieranie faktur w formacie XML/PDF/HTML
- Import faktur przychodowych i kosztowych z KSeF
- Zarządzanie integracją (token autoryzacyjny)

**Wymagany scope:** `api:ksef:integration:write`

---

## Spis treści

1. [Integracja — zarządzanie połączeniem](#integracja)
2. [Wysyłka faktur do KSeF](#wysyłka-faktur)
3. [Sprawdzanie statusu](#sprawdzanie-statusu)
4. [Pobieranie XML faktury](#pobieranie-xml)
5. [Import faktur z KSeF](#import-faktur)
6. [Obiekt ksef_data](#obiekt-ksef_data)
7. [Obsługiwane typy faktur](#obsługiwane-typy-faktur)
8. [Wysyłka przez endpoint zasobu](#wysyłka-przez-endpoint-zasobu)
9. [Obsługa błędów](#obsługa-błędów)
10. [Migracja do KSeF 2.0](#migracja-do-ksef-20)

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

**Token** generuje się w aplikacji Ministerstwa Finansów (ksef.mf.gov.pl) i musi odpowiadać NIP-owi użytkownika w inFakt.

**Odpowiedź sukces (201):**

```json
{
  "active": true,
  "incomes_last_fetched_at": null,
  "costs_last_fetched_at": null
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

---

## Wysyłka faktur

### Wyślij pojedynczą fakturę

```bash
POST /api/v3/ksef/documents/{document_uuid}/send.json
```

`document_uuid` to UUID dokumentu przychodowego w inFakt (faktura VAT, korygująca, marża, zaliczkowa lub końcowa).

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

**Odpowiedź błąd — nie można wysłać (200):**

```json
{
  "request_uuid": null,
  "invoice_uuid": "88afc9f7-ae2d-4534-9e74-e813aed9439b",
  "invoice_kind": "vat",
  "ksef_number": null,
  "status": "rejected",
  "status_description": "Faktura nie może zostać wysłana do KSeF.",
  "timestamps": {
    "request_created_at": "2024-01-15 16:05:00 +0100",
    "request_finished_at": null
  }
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

Przy wysyłce przez endpoint `/invoices/{uuid}/send_to_ksef.json` można dodać informacje o powiadomieniu klienta (body JSON z parametrami powiadomienia).

---

## Sprawdzanie statusu

```bash
GET /api/v3/ksef/documents/{document_uuid}/status.json
```

### Możliwe statusy

| Status | Opis |
|---|---|
| `sent` | Faktura wysłana do przetworzenia (status początkowy) |
| `success` | Poprawnie przetworzona, nadano `ksef_number` |
| `error` | Nie przetworzona, szczegóły w `status_description` |
| `rejected` | Faktura nie może zostać wysłana |

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
| `q[invoice_date_lteq]` | Faktury z datą wystawienia <= |
| `q[invoice_date_gteq]` | Faktury z datą wystawienia >= |

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
    "invoice_kind": "vat",
    "invoice_uuid": "0be870f9-aa52-4d0f-a9ff-994e374ecff0",
    "timestamps": {
      "request_created_at": "2024-01-15 16:00:00 +0100",
      "request_finished_at": "2024-01-15 16:01:30 +0100"
    }
  }
}
```

| Pole | Typ | Opis |
|---|---|---|
| `request_uuid` | string | Unikalny numer zlecenia wysyłki w inFakt |
| `ksef_number` | string | Numer faktury nadany przez KSeF |
| `status` | string | `sent`, `success`, `error` |
| `status_description` | string | Opis statusu przetwarzania |
| `invoice_kind` | string | Typ faktury |
| `invoice_uuid` | string | UUID faktury w inFakt |
| `timestamps.request_created_at` | datetime | Data utworzenia zlecenia |
| `timestamps.request_finished_at` | datetime | Data zakończenia przetwarzania |

Jeśli faktura nie była wysyłana do KSeF, `ksef_data` wynosi `null`.

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

## Migracja do KSeF 2.0

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

| Środowisko | API Endpoint | KSeF |
|---|---|---|
| Sandbox | `api.sandbox-infakt.pl/api/v3` | Demo KSeF (pre-produkcja) |
| Produkcja | `api.infakt.pl/api/v3` | Produkcyjny KSeF |

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
- KSeF Demo MF: https://ksef-demo.mf.gov.pl
- Sandbox inFakt: https://konto.sandbox-infakt.pl/rejestracja
- Schemat FA(2): Specyfikacja Ministerstwa Finansów
