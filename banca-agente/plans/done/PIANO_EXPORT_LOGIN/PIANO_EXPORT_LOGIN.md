# Piano di Implementazione: Export Login Utenze (Admin)

## Analisi di Fattibilita'

### Risultato: PIENAMENTE FATTIBILE (nessuna modifica DB necessaria)

La tabella `mp_user_login_trace` contiene gia' tutti i dati necessari. Ogni login viene tracciato da `UserLoginAttemptService.loginSucceeded()` con i seguenti campi:

| Campo | Tipo | Descrizione |
|---|---|---|
| `id` | Long | PK |
| `user` | ManyToOne -> User | L'utente che ha fatto login |
| `loggedAt` | LocalDateTime | Timestamp del login |
| `loggedAs` | String | Ruolo al momento del login (es. "ADMIN", "CONSULTANT") |
| `company` | ManyToOne -> Company | Azienda di contesto |
| `ipAddress` | String | IP dell'utente |
| `userAgent` | String | Browser/device |
| `step` | String | Step di autenticazione ("first", "second", "first_update_password", ecc.) |
| `loggedOut` | boolean | Se l'utente ha fatto logout |

Dall'entita' `User` si accede a `UserRegistry` con `firstName`, `lastName`, `email`, ecc.

**Nessuna migrazione DB necessaria.**
**Nessuna modifica al Document Manager o Email Manager.**

---

## Tre Modalita' di Export Richieste

| # | Tipo | Descrizione |
|---|---|---|
| 1 | **DISTINCT_USERS** | Utenti distinti che hanno fatto login in un range di date |
| 2 | **SINGLE_USER_LOGINS** | Tutti i singoli login di un determinato utente in un range di date |
| 3 | **ALL_LOGINS** | Tutti i singoli login di tutti gli utenti in un range di date |

---

## Dettaglio Implementazione

### 1. BE - Enum tipo export

**File**: `src/main/java/it/deloitte/ba/backend/enumeration/LoginTraceExportTypeEnum.java` (NUOVO)

Valori:
- `DISTINCT_USERS`
- `SINGLE_USER_LOGINS`
- `ALL_LOGINS`

---

### 2. BE - DTO di request

**File**: `src/main/java/it/deloitte/ba/backend/dto/request/LoginTraceExportRequestDTO.java` (NUOVO)

Campi:
- `exportType` (LoginTraceExportTypeEnum, obbligatorio)
- `startDate` (LocalDate, obbligatorio)
- `endDate` (LocalDate, obbligatorio)
- `userEmail` (String, opzionale - solo per SINGLE_USER_LOGINS)

---

### 3. BE - Repository: query per i tre tipi di export

**File**: `src/main/java/it/deloitte/ba/backend/repository/UserLoginTraceRepository.java` (MODIFICA)

Il repository estende gia' `JpaSpecificationExecutor<UserLoginTrace>`, quindi si possono usare JPA Specifications per query dinamiche. Query JPQL da aggiungere:

**DISTINCT_USERS:**
```sql
SELECT DISTINCT ult.user
FROM UserLoginTrace ult
WHERE ult.loggedAt BETWEEN :start AND :end
  AND ult.user.customer.id = :customerId
```

**SINGLE_USER_LOGINS:**
```sql
SELECT ult
FROM UserLoginTrace ult
WHERE ult.user.id = :userId
  AND ult.loggedAt BETWEEN :start AND :end
ORDER BY ult.loggedAt DESC
```

**ALL_LOGINS:**
```sql
SELECT ult
FROM UserLoginTrace ult
WHERE ult.loggedAt BETWEEN :start AND :end
  AND ult.user.customer.id = :customerId
ORDER BY ult.loggedAt DESC
```

Nota: il filtro per `customerId` serve per la multi-tenancy (il sistema supporta piu' customer via Referer header).

---

### 4. BE - Service per la logica di export

**File**: `src/main/java/it/deloitte/ba/backend/service/LoginTraceExportService.java` (NUOVO)

Responsabilita':
- Validazione input (date coerenti, email presente quando tipo = SINGLE_USER_LOGINS)
- Routing al metodo di query corretto in base a `exportType`
- Per `SINGLE_USER_LOGINS`: lookup dell'utente tramite email + customerId
- Invocazione dell'Excel writer e restituzione dei byte[]

Dipendenze iniettate:
- `UserLoginTraceRepository`
- `UserRepository`
- `RefererHeaderService` (per ottenere il customerId corrente)
- `LoginTraceExcelWriterService`

---

### 5. BE - Excel Writer per l'export

Pattern da seguire: `ExcelWriterService<View>` + `GenericWriterSheet` (stessi pattern di `GeneratedReportExcelService`, `PracticeExcelWriterService`)

**File nuovi:**
- `src/main/java/it/deloitte/ba/backend/service/excel/login_trace/LoginTraceExcelWriterService.java`
- `src/main/java/it/deloitte/ba/backend/service/excel/login_trace/LoginTraceSheet.java` (implementa `GenericWriterSheet`)

#### Colonne Excel per tipo di export:

**DISTINCT_USERS:**

| Email | Nome | Cognome | Ruolo | Ultimo Login | N. Login nel Periodo |
|---|---|---|---|---|---|

**SINGLE_USER_LOGINS:**

| Data/Ora Login | Ruolo | Azienda | Indirizzo IP | User Agent | Step | Logout |
|---|---|---|---|---|---|---|

**ALL_LOGINS:**

| Email | Nome | Cognome | Data/Ora Login | Ruolo | Azienda | Indirizzo IP | User Agent | Step | Logout |
|---|---|---|---|---|---|---|---|---|---|

---

### 6. BE - Controller endpoint

**File**: `src/main/java/it/deloitte/ba/backend/controller/AdminUtilController.java` (MODIFICA)

Nuovo endpoint:

```
GET /admin-util/login-trace/export
    ?exportType=DISTINCT_USERS|SINGLE_USER_LOGINS|ALL_LOGINS
    &startDate=2026-01-01
    &endDate=2026-04-24
    &userEmail=user@email.com  (opzionale, solo per SINGLE_USER_LOGINS)
```

- Ritorna `ResponseEntity<byte[]>`
- Content-Type: `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`
- Content-Disposition: `attachment; filename="export_login_<tipo>_<data>.xlsx"`
- Protetto con `@RolesAllowed(ADMIN_UTIL)` (permesso gia' presente e assegnato ai ruoli admin)

---

### 7. FE - Service

**File**: `src/app/service/admin-util.service.ts` (MODIFICA)

Aggiungere metodo:

```typescript
exportLoginTrace(
  exportType: string,
  startDate: string,
  endDate: string,
  userEmail?: string
): Observable<HttpResponse<Blob>>
```

Usa `HttpClient.get()` con `responseType: 'blob'` e `observe: 'response'` (stesso pattern di `GeneratedReportService.downloadReport()` in `generated-report.service.ts`).

---

### 8. FE - Pagina di Export

**File nuovi** (sotto `src/app/module/agency-desk/admin/users/login-export/`):
- `login-export.page.ts`
- `login-export.page.html`
- `login-export.page.scss`
- `login-export.module.ts`
- `login-export.routing.module.ts`

**UI della pagina:**

```
+--------------------------------------------------+
|  Export Login Utenze                              |
|                                                   |
|  Tipo di export:                                  |
|  [v] Utenti distinti nel periodo                  |
|  [ ] Login singolo utente                         |
|  [ ] Tutti i login nel periodo                    |
|                                                   |
|  Range date:                                      |
|  [  Data inizio  ]  -  [  Data fine  ]            |
|                                                   |
|  Email utente: (visibile solo per tipo 2)         |
|  [  email@esempio.com  ]                          |
|                                                   |
|  [ Esporta Excel ]                                |
+--------------------------------------------------+
```

Componenti PrimeNG da usare (gia' presenti nel progetto):
- `p-calendar` con `selectionMode="range"` per il range di date (vedi `input-calendar` component)
- `p-dropdown` o `p-selectButton` per il tipo di export
- `p-inputText` per l'email
- `p-button` per il bottone export

Al click del bottone:
1. Chiama `adminUtilService.exportLoginTrace()`
2. Riceve il Blob
3. Crea un link temporaneo `<a>` con `URL.createObjectURL(blob)` e triggera il download

---

### 9. FE - Routing e Sidebar

**File da modificare:**

#### a) `src/app/shared/enum/app-url.enum.ts`
Aggiungere a `AdminUrlEnum`:
```typescript
loginExport = 'login-export',
```

#### b) `src/app/module/agency-desk/admin/users/users-routing.module.ts`
Aggiungere rotta:
```typescript
{
  path: 'login-export',
  loadChildren: () =>
    import('./login-export/login-export.module').then(m => m.LoginExportModule),
}
```

#### c) `src/app/shared/ui/section-element/admin-layout/admin-layout.component.ts`
Aggiungere voce nella sidebar, nei `children` della sezione "Users" (riga ~69):
```typescript
{
  label: this.translateService.instant('admin.sidebar.loginExport'),
  link: `/${AdminUrlEnum.admin}/${AdminUrlEnum.adminUsers}//${AdminUrlEnum.loginExport}`,
}
```

#### d) File i18n
Aggiungere chiave di traduzione `admin.sidebar.loginExport` con valore "Export Login".

---

## Riepilogo File

| Operazione | File | Repo |
|---|---|---|
| NUOVO | `LoginTraceExportTypeEnum.java` | BE |
| NUOVO | `LoginTraceExportRequestDTO.java` | BE |
| MODIFICA | `UserLoginTraceRepository.java` (aggiunta query) | BE |
| NUOVO | `LoginTraceExportService.java` | BE |
| NUOVO | `LoginTraceExcelWriterService.java` | BE |
| NUOVO | `LoginTraceSheet.java` | BE |
| MODIFICA | `AdminUtilController.java` (nuovo endpoint) | BE |
| MODIFICA | `admin-util.service.ts` (nuovo metodo) | FE |
| NUOVO | `login-export.page.ts` | FE |
| NUOVO | `login-export.page.html` | FE |
| NUOVO | `login-export.page.scss` | FE |
| NUOVO | `login-export.module.ts` | FE |
| NUOVO | `login-export.routing.module.ts` | FE |
| MODIFICA | `app-url.enum.ts` (nuova voce enum) | FE |
| MODIFICA | `users-routing.module.ts` (nuova rotta) | FE |
| MODIFICA | `admin-layout.component.ts` (voce sidebar) | FE |
| MODIFICA | File i18n (chiave traduzione) | FE |

---

## Note Architetturali

1. **Filtro step**: Considerare se filtrare solo `step = 'second'` (login completato dopo OTP) per evitare duplicati tra primo e secondo step. Scelta da validare con i funzionali.

2. **Performance**: `SXSSFWorkbook` (gia' usato in `GenericExcel.generateBody()`) gestisce lo streaming in modalita' SAX, evitando di caricare tutto in memoria per export molto grandi.

3. **Colonna conteggio**: Per DISTINCT_USERS, includere una colonna "N. Login nel Periodo" puo' essere utile per dare contesto immediato.

4. **Multi-tenancy**: Tutte le query devono filtrare per `customerId` del customer corrente (ottenuto via `RefererHeaderService`).

5. **Sicurezza**: L'endpoint e' protetto da `@RolesAllowed(ADMIN_UTIL)`, permesso gia' assegnato ai ruoli ADMIN, ADMIN_FULL, SUPER_ADMIN.

---

## Ordine di Implementazione Consigliato

1. BE: Enum + DTO (nessuna dipendenza)
2. BE: Repository query (dipende da 1)
3. BE: Excel Writer (dipende da 1)
4. BE: Service (dipende da 2 e 3)
5. BE: Controller endpoint (dipende da 4)
6. FE: Service method (dipende da 5)
7. FE: Pagina + routing + sidebar (dipende da 6)
