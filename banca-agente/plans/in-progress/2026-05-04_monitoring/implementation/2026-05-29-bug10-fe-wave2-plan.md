# Bug #10 FE Wave 2 Sub-Plan — CC Assignment Page (BR 4.2.3)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Completare la Wave 2 FE del bug #10 (3 commit) ora che BE wave 1 è pushata (`c83a7c0`). Sostituire `CcAssignmentComponent` (wizard dialog) + `MonitoringGenAiExtractionComponent` (form dialog) con una **page-based UX** standalone sub-route `monitoring/practice/:id/cc/:documentId[?mode=view]`. Implementare 4 nuovi componenti (Page + Slot + ScheduleTable + RejectDialog) in TDD ordine bottom-up. Wire-up document-history con bottoni nav "Assegna Monitoraggi" e "Visualizza Assegnazione". Cleanup vecchi component DOPO smoke verde.

**Stack:** Angular + PrimeNG + RxJS (no NgRx, no signals). Test: Karma + Jasmine + `jasmine.createSpyObj` + `NO_ERRORS_SCHEMA` + `MockCustomTranslatePipe`. Pattern modulo: `Subject<void> destroy$` + `takeUntil(destroy$)`, immutability su slot[] via spread, `StatusDialogService.setStatus` per ogni error branch (no silent swallow), `SpinnerComponent.showLoader/hideLoader` per loading globali.

**Repo:** `C:/Users/davmelis/Documents/Github/ba-web` @ branch `feature/monitoring-integration-uat` @ HEAD `1d27047` (clean, in sync con origin).

**Riferimenti:**
- Spec design: `plans/in-progress/2026-05-04_monitoring/specs/2026-05-28-bug10-cc-assignment-page-design.md`
- Plan parent (Wave 1 BE done): `plans/in-progress/2026-05-04_monitoring/implementation/2026-05-28-bug10-cc-assignment-page-plan.md`
- BR: `requirements/BR_Monitoraggio_V6.md` sez 4.2.3 (652-744) + 4.2.4 (745-748)
- Handoff predecessor: `plans/in-progress/2026-05-04_monitoring/handoff/HANDOFF_2026-05-29_bug10_be_wave_done.md`
- BE commit base: `c83a7c0`

**BR validation snapshot:**
- Plan BR-conforme tranne 1 punto: BR 731 dice che nella page CC mode CHOOSE click "Assegnazione con GenAI" → "uno spinner mentre l'estrazione GenAI avrà luogo". Quindi `startGenAiExtraction` parte da `DA_ASSEGNARE`. Ma BE attuale (`markGenAiInProgress`) accetta solo `IN_ATTESA_CONFERMA`/`VERIFICA_IN_CORSO`. → richiede mini-fix BE in Wave 1.5 (vedi sezione sotto), PRIMA del Commit 3 FE.

---

## File Structure (FE Wave 2)

### Files MODIFIED

| File | Commit | Action |
|---|---|---|
| `src/app/service/monitoring.service.ts` | 3 | rename `confirmDocument` → `validateDocument` (path `/validate`); aggiungere `completeAssignment(id, payload)`; estendere `getSchedule(id, params)` (signature invariata — params già `any`, solo doc behaviour) |
| `src/app/shared/enum/monitoring-document-state.enum.ts` | 3 | aggiungere `DA_ASSEGNARE` |
| `src/app/module/agency-desk/monitoring/practice-detail/practice-detail.module.ts` | 3 | declare 4 nuovi component + route `cc/:documentId` + imports nuovi PrimeNG moduli mancanti |
| `src/assets/i18n/it-IT.json` | 3 | aggiungere blocco `monitoring.cc.*` (vedi §4 i18n) |
| `src/assets/i18n/en-GB.json` | 3 | aggiungere blocco `monitoring.cc.*` (vedi §4 i18n) |
| `src/app/module/agency-desk/monitoring/practice-detail/document-history/document-history.component.{ts,html}` | 4 | `onConfirm` chiama `validateDocument`; render "Assegna Monitoraggi" se `DA_ASSEGNARE`; render "Visualizza Assegnazione" se `COMPLETATO`; mapping `getStatusSeverity`/`getStatusLabel` per nuovo enum value; rimuovere `<app-monitoring-genai-extraction>` host + relativo state (`showGenAiDialog`, `selectedDocumentIdForGenAi`, ecc.); rimuovere `onStartGenAi`/`onModifyGenAi` triggers HTML; mantiene `canStartGenAi`/`canModifyGenAi` solo se ancora usati per altri scopi — altrimenti remove |
| `src/app/module/agency-desk/monitoring/practice-detail/document-history/document-history.component.spec.ts` | 4 | aggiornare i test per nuovo wire-up (validateDocument, mode navigation) |
| `src/app/module/agency-desk/monitoring/practice-detail/practice-detail.component.{html,ts}` | 4 | rimuovere bottone "Assegna Monitoraggi" + `<app-cc-assignment>` host + `@ViewChild ccAssignment` + `openCcAssignment()` + `onCcAssignmentComplete()` + import `CcAssignmentComponent` |

### Files CREATED

| File | Commit |
|---|---|
| `src/app/shared/model/monitoring/monitoring-cc-assignment.model.ts` | 3 |
| `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/monitoring-cc-assignment-page.component.ts` | 3 |
| `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/monitoring-cc-assignment-page.component.html` | 3 |
| `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/monitoring-cc-assignment-page.component.scss` | 3 |
| `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/monitoring-cc-assignment-page.component.spec.ts` | 3 |
| `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/components/monitoring-cc-covenant-slot.component.{ts,html,scss,spec.ts}` | 3 |
| `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/components/monitoring-cc-schedule-table.component.{ts,html,scss,spec.ts}` | 3 |
| `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/components/monitoring-cc-reject-dialog.component.{ts,html,scss,spec.ts}` | 3 |

### Files DELETED (Commit 5)

- `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment/cc-assignment.component.{ts,html,scss,spec.ts}` (4 file)
- `src/app/module/agency-desk/monitoring/practice-detail/monitoring-genai-extraction/monitoring-genai-extraction.component.{ts,html,scss,spec.ts}` (4 file)
- `src/app/module/agency-desk/monitoring/practice-detail/monitoring-genai-extraction/monitoring-genai-extraction.module.ts` (1 file)

---

# SECTION 0 — Wave 1.5 BE fix (precondizione per Wave 2)

**Goal:** consentire a `startGenAiExtraction` di partire anche da stato `DA_ASSEGNARE`, così la page CC mode CHOOSE può triggerare l'estrazione (BR 731 conforme).

**Repo:** `C:/Users/davmelis/Documents/Github/ba-back-end` @ `feature/monitoring-integration-uat` @ HEAD `c83a7c0`.

**Tempo stimato:** 10-15min.

## Task 0.1: Estendere `markGenAiInProgress` per accettare `DA_ASSEGNARE`

**Files:**
- Modify: `src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentService.java`
- Modify: `src/test/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentServiceTest.java`

- [ ] **Step 1**: Edit `markGenAiInProgress` (riga 130-131) per includere `DA_ASSEGNARE`

LOCALIZA:
```java
        if (current != MonitoringDocumentStateEnum.IN_ATTESA_CONFERMA
                && current != MonitoringDocumentStateEnum.VERIFICA_IN_CORSO) {
            throw new GenericBadRequestException(ErrorType.IM_GENERIC);
        }
```

REPLACE:
```java
        if (current != MonitoringDocumentStateEnum.IN_ATTESA_CONFERMA
                && current != MonitoringDocumentStateEnum.VERIFICA_IN_CORSO
                && current != MonitoringDocumentStateEnum.DA_ASSEGNARE) {
            throw new GenericBadRequestException(ErrorType.IM_GENERIC);
        }
```

- [ ] **Step 2**: Aggiungere test in `MonitoringDocumentServiceTest`

Cercare il `@Nested class CompleteAssignment` (o struttura analoga). APPEND nuovo `@Nested class MarkGenAiInProgress` se non esiste, o aggiungere all'esistente:

```java
    @Nested
    class MarkGenAiInProgress {

        @Test
        void shouldAllowFromDaAssegnareForCcAssignmentPageGenAiMode() {
            MonitoringDocument d = doc(1L, MonitoringDocumentStateEnum.DA_ASSEGNARE);
            when(monitoringDocumentRepository.findById(1L)).thenReturn(Optional.of(d));
            when(monitoringDocumentRepository.save(any(MonitoringDocument.class)))
                    .thenAnswer(inv -> inv.getArgument(0));

            MonitoringDocumentDTO result = monitoringDocumentService.markGenAiInProgress(1L);

            assertThat(result.getStatus())
                    .isEqualTo(MonitoringDocumentStateEnum.GENAI_IN_CORSO);
        }
    }
```

> **Nota**: helper `doc(Long, MonitoringDocumentStateEnum)` già definito in Wave 1, ricondurlo.

- [ ] **Step 3**: Run test
```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringDocumentServiceTest' -q
```
Atteso: ALL PASS (incluso il nuovo).

- [ ] **Step 4**: Verify regression
```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='Monitoring*' -q
```
Atteso: ALL PASS.

- [ ] **Step 5**: Commit
```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && git add \
  src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentService.java \
  src/test/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentServiceTest.java
```

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && git commit -m "fix(monitoring): allow GenAI extraction to start from DA_ASSEGNARE state (BR 4.2.3)"
```

- [ ] **Step 6**: Chiedere conferma push BE (1 commit isolato, basso rischio) o aspettare bundle push fine Wave 2.

---

# SECTION I — Commit 3: New components + service + i18n + route

**Goal Commit 3:** aggiungere TUTTO il nuovo codice FE necessario alla page CC (enum, service, model, 4 component, route, i18n) **senza** toccare il wire-up esistente. Dopo questo commit:
- `npx tsc --noEmit --skipLibCheck` green
- `npm test -- --include='**/cc-assignment-page/**'` green
- Vecchi `app-cc-assignment` + `app-monitoring-genai-extraction` ancora presenti e funzionanti (caller chiamano ancora `confirmDocument()` che però ora chiama `/validate` BE — risposta 404 dovrebbe propagare, dialog di errore. **Non avviare ambiente locale** in mezzo a Commit 3 e 4: il flow Conferma è rotto in quella finestra.)

**Tempo stimato:** 2.5-3.5h.

---

## Task 3.0: Pre-flight — verifica baseline FE

- [ ] **Step 1**: Verifica branch e working tree
```bash
git -C C:/Users/davmelis/Documents/Github/ba-web log --oneline -3
git -C C:/Users/davmelis/Documents/Github/ba-web status --short
```
Atteso: HEAD `1d27047`, no file modificati (file untracked tipo `node_modules/` ok).

- [ ] **Step 2**: Verifica tsc baseline
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx tsc --noEmit --skipLibCheck
```
Atteso: 0 errori. Se ci sono errori pre-esistenti, fermarsi e investigare.

- [ ] **Step 3**: Verifica npm test baseline (full suite — opzionale, slow)
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npm test -- --watch=false --browsers=ChromeHeadless --code-coverage=false
```
Atteso: tutti i test esistenti green. Se troppo lento, skip e fidarsi del CI esistente.

---

## Task 3.1: Estendere enum `MonitoringDocumentStateEnum`

**Files:**
- Modify: `src/app/shared/enum/monitoring-document-state.enum.ts`

- [ ] **Step 1**: Edit enum aggiungendo `DA_ASSEGNARE`

REPLACE intero file:
```typescript
export enum MonitoringDocumentStateEnum {
  IN_ATTESA_CONFERMA = 'IN_ATTESA_CONFERMA',
  VERIFICA_IN_CORSO = 'VERIFICA_IN_CORSO',
  GENAI_IN_CORSO = 'GENAI_IN_CORSO',
  GENAI_DA_VALIDARE = 'GENAI_DA_VALIDARE',
  DA_ASSEGNARE = 'DA_ASSEGNARE',
  COMPLETATO = 'COMPLETATO',
}
```

- [ ] **Step 2**: Verify tsc
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx tsc --noEmit --skipLibCheck
```
Atteso: 0 errori.

---

## Task 3.2: Estendere `monitoring.service.ts` (rename + new + extend)

**Files:**
- Modify: `src/app/service/monitoring.service.ts`

- [ ] **Step 1**: Rename `confirmDocument` → `validateDocument` + path

LOCALIZA riga 165-167:
```typescript
  public confirmDocument(documentId: number): Observable<MonitoringDocumentModel> {
    return this.restService.patch(`${this.baseUrl}/documents/${documentId}/confirm`, {});
  }
```

REPLACE con:
```typescript
  public validateDocument(documentId: number): Observable<MonitoringDocumentModel> {
    return this.restService.patch(`${this.baseUrl}/documents/${documentId}/validate`, {});
  }
```

- [ ] **Step 2**: Aggiungere `completeAssignment(documentId, payload)` subito dopo `validateDocument`

INSERT dopo il blocco appena modificato:
```typescript
  /**
   * BR 4.2.3 (riga 716) — chiude l'assegnazione manuale di un Compliance Certificate
   * applicando atomicamente N covenant assignments + finalize documento → COMPLETATO.
   * Lato BE è una transazione `@Transactional`: ogni event riceve updateValue + updateDates,
   * poi il documento transita DA_ASSEGNARE → COMPLETATO.
   */
  public completeAssignment(
    documentId: number,
    payload: CcCompleteAssignmentRequest,
  ): Observable<MonitoringDocumentModel> {
    return this.restService.post(`${this.baseUrl}/documents/${documentId}/complete-assignment`, payload);
  }
```

- [ ] **Step 3**: Import del tipo nuovo in cima al file

ADD import (dopo gli altri model imports, prima di `PageableModel`):
```typescript
import { CcCompleteAssignmentRequest } from '../shared/model/monitoring/monitoring-cc-assignment.model';
```

(Il file `monitoring-cc-assignment.model.ts` verrà creato in Task 3.3.)

- [ ] **Step 4**: Documentare estensione `getSchedule` con i nuovi filtri (signature invariata — `params: any` già supporta extra fields)

Localizza:
```typescript
  public getSchedule(practiceId: number, params: any): Observable<PageableModel<MonitoringEventModel>> {
    return this.restService.get(`${this.baseUrl}/practices/${practiceId}/schedule`, params);
  }
```

REPLACE con (passa `compactParams` per filtrare null/undefined/empty):
```typescript
  /**
   * BR 4.2.3 (righe 700-706) — Scadenziere filtered.
   * Params supportati: covenantType?: string, referenceDate?: string (yyyy-MM-dd),
   * rangeDays?: number (default 15 lato BE), statuses?: string|string[] (joined da BE),
   * page?: number, size?: number.
   * compactParams filtra null/undefined/'' per non sporcare la querystring.
   */
  public getSchedule(practiceId: number, params: Record<string, any> = {}): Observable<PageableModel<MonitoringEventModel>> {
    return this.restService.get(`${this.baseUrl}/practices/${practiceId}/schedule`, this.compactParams(params));
  }
```

- [ ] **Step 5**: Verify tsc (atteso: errore "Cannot find module monitoring-cc-assignment.model" → risolto in Task 3.3)
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx tsc --noEmit --skipLibCheck
```

---

## Task 3.3: Creare model `monitoring-cc-assignment.model.ts`

**Files:**
- Create: `src/app/shared/model/monitoring/monitoring-cc-assignment.model.ts`

- [ ] **Step 1**: Create file con tutti i tipi necessari

```typescript
import { MonitoringEventModel } from './monitoring-event.model';

/**
 * BR 4.2.3 — Stato locale di uno slot covenant nella page CC Assignment.
 * Il modello driverà la state machine del componente `MonitoringCcCovenantSlot`:
 *
 *   EMPTY            → slot vuoto, solo riga + cestino (BR 669)
 *   FILLING          → 3 box editabili (tipologia, valore, data) + Associa/Annulla (BR 671-679)
 *   SCHEDULE_VISIBLE → scadenziere filtered 9 colonne (BR 680-698)
 *   EVENT_SELECTED   → riga assegnata, compare "Conferma Assegnazione" (BR 708)
 *   CONFIRMED        → scadenziere collapsed a 1 riga, "Modifica Assegnazione" (BR 711-712)
 *   GENAI_PREFILLED  → mode GenAI: selettori prefilled, "Modifica Dati" + "Conferma Assegnazione" (BR 733)
 */
export type CcSlotPhase =
  | 'EMPTY'
  | 'FILLING'
  | 'SCHEDULE_VISIBLE'
  | 'EVENT_SELECTED'
  | 'CONFIRMED'
  | 'GENAI_PREFILLED';

export interface CcSlotState {
  /** UUID locale, usato come trackBy in *ngFor. */
  uid: string;
  /** 1-based "Covenant #n" per label utente. */
  index: number;
  phase: CcSlotPhase;
  selectedCovenantId: number | null;
  monitoringValue: number | null;
  /** ISO yyyy-MM-dd. */
  referenceDate: string | null;
  scheduleEvents: MonitoringEventModel[];
  assignedEventId: number | null;
  confirmed: boolean;
}

/**
 * BR 4.2.3 — Payload `POST /monitoring/documents/{id}/complete-assignment`.
 * Allineato a `MonitoringCcCompleteAssignmentRequestDTO` BE (record nested CovenantAssignment).
 */
export interface CcCompleteAssignmentRequest {
  assignments: CcCovenantAssignmentPayload[];
}

export interface CcCovenantAssignmentPayload {
  covenantId: number;
  eventId: number;
  monitoringValue: number;
  /** ISO yyyy-MM-dd — "Data avvenuto monitoraggio" (BR 4.2.3 — per ora coincide con referenceDate). */
  actualDate: string;
  /** ISO yyyy-MM-dd — "Data di Riferimento" (BR 675). */
  referenceDate: string;
}
```

- [ ] **Step 2**: Verify tsc
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx tsc --noEmit --skipLibCheck
```
Atteso: 0 errori (lo "Cannot find module" di Task 3.2 ora è risolto).

---

## Task 3.4: Component `MonitoringCcRejectDialogComponent` (TDD)

Componente più semplice, isolato, no service deps. Buon punto di partenza TDD.

**Files:**
- Create: `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/components/monitoring-cc-reject-dialog.component.ts`
- Create: `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/components/monitoring-cc-reject-dialog.component.html`
- Create: `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/components/monitoring-cc-reject-dialog.component.scss`
- Create: `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/components/monitoring-cc-reject-dialog.component.spec.ts`

### Step 1: Write failing spec

Create `monitoring-cc-reject-dialog.component.spec.ts`:
```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { FormsModule } from '@angular/forms';
import { NO_ERRORS_SCHEMA, Pipe, PipeTransform } from '@angular/core';

import { MonitoringCcRejectDialogComponent } from './monitoring-cc-reject-dialog.component';
import { CustomTranslatePipe } from '../../../../../../shared/pipe/custom-translate.pipe';

@Pipe({ name: 'customTranslate' })
class MockCustomTranslatePipe implements PipeTransform {
  transform(value: string): string { return value; }
}

describe('MonitoringCcRejectDialogComponent', () => {
  let component: MonitoringCcRejectDialogComponent;
  let fixture: ComponentFixture<MonitoringCcRejectDialogComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [MonitoringCcRejectDialogComponent, MockCustomTranslatePipe],
      imports: [FormsModule],
      providers: [
        { provide: CustomTranslatePipe, useValue: { transform: (k: string) => k } },
      ],
      schemas: [NO_ERRORS_SCHEMA],
    }).compileComponents();

    fixture = TestBed.createComponent(MonitoringCcRejectDialogComponent);
    component = fixture.componentInstance;
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should expose four reason options i18n keys', () => {
    expect(component.reasonOptions.length).toBe(4);
    expect(component.reasonOptions.map(o => o.value)).toEqual([
      'INCOMPLETE', 'FORMAT', 'SIGNATURE', 'OTHER',
    ]);
  });

  it('canSubmit() returns false until reason is selected', () => {
    component.selectedReason = null;
    expect(component.canSubmit()).toBeFalse();
    component.selectedReason = 'INCOMPLETE';
    expect(component.canSubmit()).toBeTrue();
  });

  describe('onConfirm', () => {
    it('emits rejected event with concatenated reason+notes when reason set', () => {
      const spy = jasmine.createSpy('rejected');
      component.rejected.subscribe(spy);
      component.selectedReason = 'INCOMPLETE';
      component.notes = 'pagine mancanti';

      component.onConfirm();

      expect(spy).toHaveBeenCalledTimes(1);
      const arg = spy.calls.mostRecent().args[0];
      expect(arg.reason).toBe('INCOMPLETE');
      expect(arg.notes).toBe('pagine mancanti');
    });

    it('emits rejected with empty notes when notes blank', () => {
      const spy = jasmine.createSpy('rejected');
      component.rejected.subscribe(spy);
      component.selectedReason = 'OTHER';
      component.notes = '';

      component.onConfirm();

      const arg = spy.calls.mostRecent().args[0];
      expect(arg.reason).toBe('OTHER');
      expect(arg.notes).toBe('');
    });

    it('does not emit when reason is null (guarded by canSubmit)', () => {
      const spy = jasmine.createSpy('rejected');
      component.rejected.subscribe(spy);
      component.selectedReason = null;
      component.notes = 'whatever';

      component.onConfirm();

      expect(spy).not.toHaveBeenCalled();
    });
  });

  describe('onCancel', () => {
    it('emits cancelled and resets state', () => {
      const spy = jasmine.createSpy('cancelled');
      component.cancelled.subscribe(spy);
      component.selectedReason = 'INCOMPLETE';
      component.notes = 'whatever';

      component.onCancel();

      expect(spy).toHaveBeenCalledTimes(1);
      expect(component.selectedReason).toBeNull();
      expect(component.notes).toBe('');
    });
  });

  describe('reset on hide', () => {
    it('clears selectedReason and notes when dialog hides', () => {
      component.selectedReason = 'FORMAT';
      component.notes = 'something';

      component.onHide();

      expect(component.selectedReason).toBeNull();
      expect(component.notes).toBe('');
    });
  });
});
```

### Step 2: Run test — atteso FAIL (component non esiste)

```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx ng test --watch=false --browsers=ChromeHeadless --code-coverage=false --include='**/monitoring-cc-reject-dialog.component.spec.ts'
```
Atteso: errore compilazione "Cannot find module".

### Step 3: Create `.ts` component

Create `monitoring-cc-reject-dialog.component.ts`:
```typescript
import { Component, EventEmitter, Input, Output } from '@angular/core';

export interface CcRejectPayload {
  reason: string;
  notes: string;
}

interface ReasonOption {
  value: string;
  labelKey: string;
}

/**
 * BR 4.2.3 (riga 729) — Modale "Rifiuto Documento" attivata dal bottone
 * "Ritorna a documentazione incompleta" nella page CC Assignment (mode GenAI).
 * L'utente seleziona un motivo (dropdown obbligatorio) + note libere opzionali.
 * Il payload `{reason, notes}` viene emesso al parent page-component che chiama
 * `monitoringService.rejectDocument(documentId, "{reason}: {notes}")` e gestisce
 * la status dialog + navigate back.
 */
@Component({
  selector: 'app-monitoring-cc-reject-dialog',
  templateUrl: './monitoring-cc-reject-dialog.component.html',
  styleUrls: ['./monitoring-cc-reject-dialog.component.scss'],
})
export class MonitoringCcRejectDialogComponent {
  @Input() public visible = false;
  @Input() public submitting = false;
  @Output() public visibleChange = new EventEmitter<boolean>();
  @Output() public rejected = new EventEmitter<CcRejectPayload>();
  @Output() public cancelled = new EventEmitter<void>();

  public selectedReason: string | null = null;
  public notes = '';

  public readonly reasonOptions: ReasonOption[] = [
    { value: 'INCOMPLETE', labelKey: 'monitoring.cc.reject.reasons.incomplete' },
    { value: 'FORMAT', labelKey: 'monitoring.cc.reject.reasons.format' },
    { value: 'SIGNATURE', labelKey: 'monitoring.cc.reject.reasons.signature' },
    { value: 'OTHER', labelKey: 'monitoring.cc.reject.reasons.other' },
  ];

  public canSubmit(): boolean {
    return this.selectedReason != null;
  }

  public onConfirm(): void {
    if (!this.canSubmit() || !this.selectedReason) {
      return;
    }
    this.rejected.emit({
      reason: this.selectedReason,
      notes: this.notes.trim(),
    });
  }

  public onCancel(): void {
    this.reset();
    this.cancelled.emit();
  }

  public onHide(): void {
    this.reset();
    this.visibleChange.emit(false);
  }

  private reset(): void {
    this.selectedReason = null;
    this.notes = '';
  }
}
```

### Step 4: Create `.html`

Create `monitoring-cc-reject-dialog.component.html`:
```html
<p-dialog
  [(visible)]="visible"
  [header]="'monitoring.cc.reject.modal.title' | customTranslate"
  [modal]="true"
  [style]="{ width: '480px' }"
  [closable]="!submitting"
  (onHide)="onHide()">

  <div class="p-fluid">
    <div class="field">
      <label class="block mb-2">{{ 'monitoring.cc.reject.modal.reasonLabel' | customTranslate }}</label>
      <p-dropdown
        [options]="reasonOptions"
        [(ngModel)]="selectedReason"
        optionValue="value"
        [placeholder]="'monitoring.cc.reject.modal.reasonPlaceholder' | customTranslate"
        appendTo="body"
        [style]="{ width: '100%' }">
        <ng-template let-opt pTemplate="item">
          {{ opt.labelKey | customTranslate }}
        </ng-template>
        <ng-template let-opt pTemplate="selectedItem">
          {{ opt.labelKey | customTranslate }}
        </ng-template>
      </p-dropdown>
    </div>

    <div class="field mt-3">
      <label class="block mb-2">{{ 'monitoring.cc.reject.modal.notes' | customTranslate }}</label>
      <textarea
        pInputTextarea
        [(ngModel)]="notes"
        [rows]="4"
        [placeholder]="'monitoring.cc.reject.modal.notesPlaceholder' | customTranslate">
      </textarea>
    </div>
  </div>

  <ng-template pTemplate="footer">
    <button pButton type="button"
      [label]="'shared.cancel' | customTranslate"
      class="p-button-text"
      [disabled]="submitting"
      (click)="onCancel()">
    </button>
    <button pButton type="button"
      [label]="'monitoring.cc.reject.modal.confirm' | customTranslate"
      class="p-button-danger"
      icon="pi pi-times-circle"
      [loading]="submitting"
      [disabled]="!canSubmit() || submitting"
      (click)="onConfirm()">
    </button>
  </ng-template>
</p-dialog>
```

### Step 5: Create `.scss` (vuoto/minimo)

Create `monitoring-cc-reject-dialog.component.scss`:
```scss
:host {
  display: contents;
}
```

### Step 6: Run test — atteso PASS

```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx ng test --watch=false --browsers=ChromeHeadless --code-coverage=false --include='**/monitoring-cc-reject-dialog.component.spec.ts'
```
Atteso: PASS (~9 test).

---

## Task 3.5: Component `MonitoringCcScheduleTableComponent` (TDD)

Componente "dumb" — riceve eventi via Input, emette assign/unassign via Output. Nessun service.

**Files:**
- Create: `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/components/monitoring-cc-schedule-table.component.{ts,html,scss,spec.ts}`

### Step 1: Write failing spec

Create `monitoring-cc-schedule-table.component.spec.ts`:
```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { NO_ERRORS_SCHEMA, Pipe, PipeTransform } from '@angular/core';

import { MonitoringCcScheduleTableComponent } from './monitoring-cc-schedule-table.component';
import { CustomTranslatePipe } from '../../../../../../shared/pipe/custom-translate.pipe';
import { MonitoringEventModel } from '../../../../../../shared/model/monitoring/monitoring-event.model';
import { MonitoringEventStateEnum } from '../../../../../../shared/enum/monitoring-event-state.enum';

@Pipe({ name: 'customTranslate' })
class MockCustomTranslatePipe implements PipeTransform {
  transform(value: string): string { return value; }
}

function buildEvent(overrides: Partial<MonitoringEventModel> = {}): MonitoringEventModel {
  return {
    id: 1,
    eventCode: 'E1',
    covenantCode: 'C1',
    covenantType: 'ICR',
    referenceDate: '2026-05-28',
    expirationDate: '2026-06-28',
    monitoringValue: null,
    status: MonitoringEventStateEnum.SCADUTO,
    actualDate: '',
    notes: '',
    hasException: false,
    exceptionMotivation: '',
    daysToNextMonitoring: 5,
    valueLimit: 100,
    ...overrides,
  };
}

describe('MonitoringCcScheduleTableComponent', () => {
  let component: MonitoringCcScheduleTableComponent;
  let fixture: ComponentFixture<MonitoringCcScheduleTableComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [MonitoringCcScheduleTableComponent, MockCustomTranslatePipe],
      providers: [
        { provide: CustomTranslatePipe, useValue: { transform: (k: string) => k } },
      ],
      schemas: [NO_ERRORS_SCHEMA],
    }).compileComponents();

    fixture = TestBed.createComponent(MonitoringCcScheduleTableComponent);
    component = fixture.componentInstance;
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  describe('isAssigned', () => {
    it('returns true only for event matching assignedEventId', () => {
      component.assignedEventId = 42;
      expect(component.isAssigned(buildEvent({ id: 42 }))).toBeTrue();
      expect(component.isAssigned(buildEvent({ id: 1 }))).toBeFalse();
    });

    it('returns false when assignedEventId is null', () => {
      component.assignedEventId = null;
      expect(component.isAssigned(buildEvent({ id: 1 }))).toBeFalse();
    });
  });

  describe('onAssignClick', () => {
    it('emits assign with eventId when not readonly', () => {
      const spy = jasmine.createSpy('assign');
      component.assign.subscribe(spy);
      component.readonly = false;
      component.events = [buildEvent({ id: 7 })];

      component.onAssignClick(7);

      expect(spy).toHaveBeenCalledWith(7);
    });

    it('does not emit assign when readonly', () => {
      const spy = jasmine.createSpy('assign');
      component.assign.subscribe(spy);
      component.readonly = true;

      component.onAssignClick(7);

      expect(spy).not.toHaveBeenCalled();
    });
  });

  describe('onUnassignClick', () => {
    it('emits unassign event when not readonly', () => {
      const spy = jasmine.createSpy('unassign');
      component.unassign.subscribe(spy);
      component.readonly = false;

      component.onUnassignClick();

      expect(spy).toHaveBeenCalledTimes(1);
    });

    it('does not emit unassign when readonly', () => {
      const spy = jasmine.createSpy('unassign');
      component.unassign.subscribe(spy);
      component.readonly = true;

      component.onUnassignClick();

      expect(spy).not.toHaveBeenCalled();
    });
  });

  it('exposes empty events array as default', () => {
    expect(component.events).toEqual([]);
  });
});
```

### Step 2: Run test — atteso FAIL

```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx ng test --watch=false --browsers=ChromeHeadless --code-coverage=false --include='**/monitoring-cc-schedule-table.component.spec.ts'
```
Atteso: errore compilazione.

### Step 3: Create `.ts`

Create `monitoring-cc-schedule-table.component.ts`:
```typescript
import { Component, EventEmitter, Input, Output } from '@angular/core';
import { MonitoringEventModel } from '../../../../../../shared/model/monitoring/monitoring-event.model';

/**
 * BR 4.2.3 (righe 680-712) — Scadenziere inline 9 colonne mostrato dentro uno slot
 * covenant durante l'assegnazione CC. Component "dumb": riceve eventi filtrati e
 * `assignedEventId` corrente, emette `assign`/`unassign` al parent slot.
 *
 * In phase CONFIRMED il parent passa solo `[assignedEvent]` come `events` per ottenere
 * la "riga collapsed" prevista dal BR (riga 711). In phase SCHEDULE_VISIBLE passa la
 * lista filtrata completa.
 */
@Component({
  selector: 'app-monitoring-cc-schedule-table',
  templateUrl: './monitoring-cc-schedule-table.component.html',
  styleUrls: ['./monitoring-cc-schedule-table.component.scss'],
})
export class MonitoringCcScheduleTableComponent {
  @Input() public events: MonitoringEventModel[] = [];
  @Input() public assignedEventId: number | null = null;
  @Input() public readonly = false;
  @Output() public assign = new EventEmitter<number>();
  @Output() public unassign = new EventEmitter<void>();

  public isAssigned(event: MonitoringEventModel): boolean {
    return this.assignedEventId != null && event.id === this.assignedEventId;
  }

  public onAssignClick(eventId: number): void {
    if (this.readonly) {
      return;
    }
    this.assign.emit(eventId);
  }

  public onUnassignClick(): void {
    if (this.readonly) {
      return;
    }
    this.unassign.emit();
  }
}
```

### Step 4: Create `.html`

Create `monitoring-cc-schedule-table.component.html`:
```html
<p-table [value]="events" styleClass="p-datatable-sm" responsiveLayout="scroll">
  <ng-template pTemplate="header">
    <tr>
      <th>{{ 'monitoring.covenant.table.eventId' | customTranslate }}</th>
      <th>{{ 'monitoring.covenant.table.referenceDate' | customTranslate }}</th>
      <th>{{ 'monitoring.covenant.table.expirationDate' | customTranslate }}</th>
      <th>{{ 'monitoring.covenant.table.daysToNext' | customTranslate }}</th>
      <th>{{ 'monitoring.covenant.table.actualDate' | customTranslate }}</th>
      <th>{{ 'monitoring.covenant.table.monitoringValue' | customTranslate }}</th>
      <th>{{ 'monitoring.covenant.table.monitoringStatus' | customTranslate }}</th>
      <th>{{ 'monitoring.covenant.table.document' | customTranslate }}</th>
      <th>{{ 'monitoring.covenant.table.action' | customTranslate }}</th>
    </tr>
  </ng-template>

  <ng-template pTemplate="body" let-event>
    <tr [class.row-assigned]="isAssigned(event)">
      <td>{{ event.eventCode || '—' }}</td>
      <td>{{ event.referenceDate || '—' }}</td>
      <td>{{ event.expirationDate || '—' }}</td>
      <td>{{ event.daysToNextMonitoring ?? '—' }}</td>
      <td>{{ event.actualDate || '—' }}</td>
      <td>{{ event.monitoringValue ?? '—' }}</td>
      <td>{{ event.status || '—' }}</td>
      <td>—</td>
      <td>
        <ng-container *ngIf="!readonly">
          <button *ngIf="!isAssigned(event)"
            pButton type="button" icon="pi pi-check"
            class="p-button-text p-button-sm p-button-success"
            [pTooltip]="'monitoring.cc.schedule.action.assegna' | customTranslate"
            (click)="onAssignClick(event.id)">
          </button>
          <button *ngIf="isAssigned(event)"
            pButton type="button" icon="pi pi-times"
            class="p-button-text p-button-sm p-button-danger"
            [pTooltip]="'monitoring.cc.schedule.action.rimuovi' | customTranslate"
            (click)="onUnassignClick()">
          </button>
        </ng-container>
        <span *ngIf="readonly" class="text-color-secondary">—</span>
      </td>
    </tr>
  </ng-template>

  <ng-template pTemplate="emptymessage">
    <tr>
      <td colspan="9" class="text-center text-color-secondary py-3">
        {{ 'monitoring.cc.schedule.empty' | customTranslate }}
      </td>
    </tr>
  </ng-template>
</p-table>
```

### Step 5: Create `.scss`

Create `monitoring-cc-schedule-table.component.scss`:
```scss
:host {
  display: block;
  width: 100%;
}

.row-assigned {
  background-color: var(--green-50, #ecfdf5) !important;
  font-weight: 600;
}
```

### Step 6: Run test — atteso PASS

```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx ng test --watch=false --browsers=ChromeHeadless --code-coverage=false --include='**/monitoring-cc-schedule-table.component.spec.ts'
```
Atteso: PASS (~8 test).

---

## Task 3.6: Component `MonitoringCcCovenantSlotComponent` (TDD)

Componente con state machine `CcSlotPhase`. Riceve `slotState` e `practiceCovenants` da Input, emette `stateChange` (slot aggiornato), `confirm`, `modify`, `remove`, `requestSchedule` al parent. Service deps: nessuna diretta (il parent fa la `getSchedule()`).

**Files:**
- Create: `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/components/monitoring-cc-covenant-slot.component.{ts,html,scss,spec.ts}`

### Step 1: Write failing spec

Create `monitoring-cc-covenant-slot.component.spec.ts`:
```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { FormsModule } from '@angular/forms';
import { NO_ERRORS_SCHEMA, Pipe, PipeTransform } from '@angular/core';

import { MonitoringCcCovenantSlotComponent } from './monitoring-cc-covenant-slot.component';
import { CustomTranslatePipe } from '../../../../../../shared/pipe/custom-translate.pipe';
import { CcSlotState } from '../../../../../../shared/model/monitoring/monitoring-cc-assignment.model';
import { MonitoringCovenantModel } from '../../../../../../shared/model/monitoring/monitoring-covenant.model';

@Pipe({ name: 'customTranslate' })
class MockCustomTranslatePipe implements PipeTransform {
  transform(value: string): string { return value; }
}

function buildCovenant(overrides: Partial<MonitoringCovenantModel> = {}): MonitoringCovenantModel {
  return {
    id: 1,
    covenantCode: 'C1',
    covenantType: 'ICR',
    valueLimitUpper: 0,
    valueLimitLower: 0,
    sign: '<',
    periodicity: 'ANNUALE',
    variable: '',
    notes: '',
    practiceId: 1,
    state: 'OK',
    countEvents: 0,
    countExpired: 0,
    countExpiring: 0,
    countOk: 0,
    ...overrides,
  };
}

function buildSlotState(overrides: Partial<CcSlotState> = {}): CcSlotState {
  return {
    uid: 'uid-1',
    index: 1,
    phase: 'EMPTY',
    selectedCovenantId: null,
    monitoringValue: null,
    referenceDate: null,
    scheduleEvents: [],
    assignedEventId: null,
    confirmed: false,
    ...overrides,
  };
}

describe('MonitoringCcCovenantSlotComponent', () => {
  let component: MonitoringCcCovenantSlotComponent;
  let fixture: ComponentFixture<MonitoringCcCovenantSlotComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [MonitoringCcCovenantSlotComponent, MockCustomTranslatePipe],
      imports: [FormsModule],
      providers: [
        { provide: CustomTranslatePipe, useValue: { transform: (k: string) => k } },
      ],
      schemas: [NO_ERRORS_SCHEMA],
    }).compileComponents();

    fixture = TestBed.createComponent(MonitoringCcCovenantSlotComponent);
    component = fixture.componentInstance;
    component.practiceCovenants = [buildCovenant({ id: 1 }), buildCovenant({ id: 2, covenantCode: 'C2' })];
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  describe('canAssociate', () => {
    it('returns true when all 3 fields are set in FILLING phase', () => {
      component.slotState = buildSlotState({
        phase: 'FILLING', selectedCovenantId: 1, monitoringValue: 50, referenceDate: '2026-05-28',
      });

      expect(component.canAssociate()).toBeTrue();
    });

    it('returns false when one field is missing', () => {
      component.slotState = buildSlotState({
        phase: 'FILLING', selectedCovenantId: 1, monitoringValue: null, referenceDate: '2026-05-28',
      });

      expect(component.canAssociate()).toBeFalse();
    });
  });

  describe('onAssociaCovenant', () => {
    it('emits requestSchedule with selected covenant + reference date', () => {
      const spy = jasmine.createSpy('requestSchedule');
      component.requestSchedule.subscribe(spy);
      component.slotState = buildSlotState({
        phase: 'FILLING', selectedCovenantId: 2, monitoringValue: 50, referenceDate: '2026-05-28',
      });

      component.onAssociaCovenant();

      expect(spy).toHaveBeenCalledTimes(1);
      const arg = spy.calls.mostRecent().args[0];
      expect(arg.covenantId).toBe(2);
      expect(arg.referenceDate).toBe('2026-05-28');
    });

    it('does not emit when canAssociate is false', () => {
      const spy = jasmine.createSpy('requestSchedule');
      component.requestSchedule.subscribe(spy);
      component.slotState = buildSlotState({ phase: 'FILLING', selectedCovenantId: null });

      component.onAssociaCovenant();

      expect(spy).not.toHaveBeenCalled();
    });
  });

  describe('onAnnulla', () => {
    it('emits stateChange resetting slot to FILLING with empty fields (BR 679)', () => {
      const spy = jasmine.createSpy('stateChange');
      component.stateChange.subscribe(spy);
      component.slotState = buildSlotState({
        phase: 'SCHEDULE_VISIBLE', selectedCovenantId: 1, monitoringValue: 50, referenceDate: '2026-05-28',
      });

      component.onAnnulla();

      const arg = spy.calls.mostRecent().args[0];
      expect(arg.phase).toBe('FILLING');
      expect(arg.selectedCovenantId).toBeNull();
      expect(arg.monitoringValue).toBeNull();
      expect(arg.referenceDate).toBeNull();
      expect(arg.scheduleEvents).toEqual([]);
      expect(arg.assignedEventId).toBeNull();
    });
  });

  describe('onAssignEvent', () => {
    it('updates slot to EVENT_SELECTED with assignedEventId', () => {
      const spy = jasmine.createSpy('stateChange');
      component.stateChange.subscribe(spy);
      component.slotState = buildSlotState({ phase: 'SCHEDULE_VISIBLE', scheduleEvents: [], assignedEventId: null });

      component.onAssignEvent(99);

      const arg = spy.calls.mostRecent().args[0];
      expect(arg.phase).toBe('EVENT_SELECTED');
      expect(arg.assignedEventId).toBe(99);
    });
  });

  describe('onUnassignEvent', () => {
    it('reverts slot to SCHEDULE_VISIBLE clearing assignedEventId', () => {
      const spy = jasmine.createSpy('stateChange');
      component.stateChange.subscribe(spy);
      component.slotState = buildSlotState({ phase: 'EVENT_SELECTED', assignedEventId: 99 });

      component.onUnassignEvent();

      const arg = spy.calls.mostRecent().args[0];
      expect(arg.phase).toBe('SCHEDULE_VISIBLE');
      expect(arg.assignedEventId).toBeNull();
    });
  });

  describe('onConfirmAssegnazione', () => {
    it('emits confirm event and updates slot to CONFIRMED', () => {
      const confirmSpy = jasmine.createSpy('confirm');
      const stateSpy = jasmine.createSpy('stateChange');
      component.confirm.subscribe(confirmSpy);
      component.stateChange.subscribe(stateSpy);
      component.slotState = buildSlotState({ phase: 'EVENT_SELECTED', assignedEventId: 99 });

      component.onConfirmAssegnazione();

      expect(confirmSpy).toHaveBeenCalledTimes(1);
      const arg = stateSpy.calls.mostRecent().args[0];
      expect(arg.phase).toBe('CONFIRMED');
      expect(arg.confirmed).toBeTrue();
    });

    it('confirms GENAI_PREFILLED phase to CONFIRMED (BR 737)', () => {
      const stateSpy = jasmine.createSpy('stateChange');
      component.stateChange.subscribe(stateSpy);
      component.slotState = buildSlotState({
        phase: 'GENAI_PREFILLED', selectedCovenantId: 1, monitoringValue: 50,
        referenceDate: '2026-05-28', assignedEventId: 99,
      });

      component.onConfirmAssegnazione();

      const arg = stateSpy.calls.mostRecent().args[0];
      expect(arg.phase).toBe('CONFIRMED');
      expect(arg.confirmed).toBeTrue();
    });
  });

  describe('onModificaAssegnazione', () => {
    it('reverts CONFIRMED back to SCHEDULE_VISIBLE (BR 712)', () => {
      const stateSpy = jasmine.createSpy('stateChange');
      component.stateChange.subscribe(stateSpy);
      component.slotState = buildSlotState({ phase: 'CONFIRMED', confirmed: true, assignedEventId: 99 });

      component.onModificaAssegnazione();

      const arg = stateSpy.calls.mostRecent().args[0];
      expect(arg.phase).toBe('SCHEDULE_VISIBLE');
      expect(arg.confirmed).toBeFalse();
    });
  });

  describe('onModificaDati (GenAI)', () => {
    it('reverts GENAI_PREFILLED slot to SCHEDULE_VISIBLE per BR 735', () => {
      const stateSpy = jasmine.createSpy('stateChange');
      component.stateChange.subscribe(stateSpy);
      component.slotState = buildSlotState({
        phase: 'GENAI_PREFILLED', selectedCovenantId: 1, monitoringValue: 50, referenceDate: '2026-05-28',
      });

      component.onModificaDati();

      const arg = stateSpy.calls.mostRecent().args[0];
      expect(arg.phase).toBe('SCHEDULE_VISIBLE');
    });
  });

  describe('onRemove', () => {
    it('emits remove event when allowed', () => {
      const spy = jasmine.createSpy('remove');
      component.remove.subscribe(spy);
      component.slotState = buildSlotState({ phase: 'EMPTY' });
      component.mode = 'manual';

      component.onRemove();

      expect(spy).toHaveBeenCalledTimes(1);
    });

    it('does not emit remove when readonly (mode=view)', () => {
      const spy = jasmine.createSpy('remove');
      component.remove.subscribe(spy);
      component.slotState = buildSlotState({ phase: 'CONFIRMED', confirmed: true });
      component.mode = 'view';

      component.onRemove();

      expect(spy).not.toHaveBeenCalled();
    });
  });

  describe('selectedCovenantLabel', () => {
    it('returns code + type for selected covenant', () => {
      component.practiceCovenants = [buildCovenant({ id: 5, covenantCode: 'C5', covenantType: 'PFN_MOL' })];
      component.slotState = buildSlotState({ selectedCovenantId: 5 });

      expect(component.selectedCovenantLabel).toBe('C5 - PFN_MOL');
    });

    it('returns empty when no covenant selected', () => {
      component.slotState = buildSlotState({ selectedCovenantId: null });

      expect(component.selectedCovenantLabel).toBe('');
    });
  });
});
```

### Step 2: Run test — atteso FAIL

```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx ng test --watch=false --browsers=ChromeHeadless --code-coverage=false --include='**/monitoring-cc-covenant-slot.component.spec.ts'
```
Atteso: errore compilazione.

### Step 3: Create `.ts`

Create `monitoring-cc-covenant-slot.component.ts`:
```typescript
import { Component, EventEmitter, Input, Output } from '@angular/core';
import {
  CcSlotState,
} from '../../../../../../shared/model/monitoring/monitoring-cc-assignment.model';
import { MonitoringCovenantModel } from '../../../../../../shared/model/monitoring/monitoring-covenant.model';
import { MonitoringEventModel } from '../../../../../../shared/model/monitoring/monitoring-event.model';

export interface CcSlotScheduleRequest {
  covenantId: number;
  referenceDate: string;
}

export type CcSlotMode = 'manual' | 'genai' | 'view';

/**
 * BR 4.2.3 (righe 660-744) — Componente "slot" per UNA assegnazione covenant
 * dentro la page CC Assignment. State machine driven by `slotState.phase`.
 *
 * Il componente NON chiama servizi: emette `requestSchedule(covenantId, referenceDate)`
 * al parent page-component, che fa la `monitoringService.getSchedule()` e ritorna
 * l'updated `slotState` con `scheduleEvents` popolati.
 *
 * Immutability: ogni `stateChange.emit(updated)` produce un NUOVO oggetto
 * `CcSlotState` via spread `{...slot, ...changes}`. Mai mutare in place.
 */
@Component({
  selector: 'app-monitoring-cc-covenant-slot',
  templateUrl: './monitoring-cc-covenant-slot.component.html',
  styleUrls: ['./monitoring-cc-covenant-slot.component.scss'],
})
export class MonitoringCcCovenantSlotComponent {
  @Input() public slotState!: CcSlotState;
  @Input() public practiceCovenants: MonitoringCovenantModel[] = [];
  @Input() public mode: CcSlotMode = 'manual';
  @Input() public scheduleLoading = false;

  @Output() public stateChange = new EventEmitter<CcSlotState>();
  @Output() public confirm = new EventEmitter<void>();
  @Output() public remove = new EventEmitter<void>();
  @Output() public requestSchedule = new EventEmitter<CcSlotScheduleRequest>();

  public get readonly(): boolean {
    return this.mode === 'view';
  }

  public get selectedCovenantLabel(): string {
    if (!this.slotState?.selectedCovenantId) {
      return '';
    }
    const c = this.practiceCovenants.find(co => co.id === this.slotState.selectedCovenantId);
    return c ? `${c.covenantCode} - ${c.covenantType}` : '';
  }

  public canAssociate(): boolean {
    const s = this.slotState;
    return s != null
      && s.selectedCovenantId != null
      && s.monitoringValue != null
      && s.referenceDate != null
      && s.referenceDate.length > 0;
  }

  public onCovenantChange(covenantId: number): void {
    this.emit({ selectedCovenantId: covenantId });
  }

  public onMonitoringValueChange(value: number | null): void {
    this.emit({ monitoringValue: value });
  }

  public onReferenceDateChange(value: string | null): void {
    this.emit({ referenceDate: value });
  }

  public onAssociaCovenant(): void {
    if (!this.canAssociate()) {
      return;
    }
    this.requestSchedule.emit({
      covenantId: this.slotState.selectedCovenantId!,
      referenceDate: this.slotState.referenceDate!,
    });
  }

  /** BR 679 — Annulla pulisce i 3 box ma NON elimina lo slot. */
  public onAnnulla(): void {
    this.emit({
      phase: 'FILLING',
      selectedCovenantId: null,
      monitoringValue: null,
      referenceDate: null,
      scheduleEvents: [],
      assignedEventId: null,
      confirmed: false,
    });
  }

  /** BR 708 — Click "Assegna" su una riga del scadenziere. */
  public onAssignEvent(eventId: number): void {
    this.emit({
      phase: 'EVENT_SELECTED',
      assignedEventId: eventId,
    });
  }

  public onUnassignEvent(): void {
    this.emit({
      phase: 'SCHEDULE_VISIBLE',
      assignedEventId: null,
    });
  }

  /** BR 711 / 737 — "Conferma Assegnazione". */
  public onConfirmAssegnazione(): void {
    this.emit({
      phase: 'CONFIRMED',
      confirmed: true,
    });
    this.confirm.emit();
  }

  /** BR 712 / 739 — "Modifica Assegnazione" (riapre scadenziere). */
  public onModificaAssegnazione(): void {
    this.emit({
      phase: 'SCHEDULE_VISIBLE',
      confirmed: false,
    });
  }

  /** BR 735 — "Modifica Dati" (mode GenAI: passa a SCHEDULE_VISIBLE). */
  public onModificaDati(): void {
    this.emit({
      phase: 'SCHEDULE_VISIBLE',
    });
  }

  public onRemove(): void {
    if (this.readonly) {
      return;
    }
    this.remove.emit();
  }

  public collapsedEvents(): MonitoringEventModel[] {
    if (!this.slotState || this.slotState.assignedEventId == null) {
      return [];
    }
    const assigned = this.slotState.scheduleEvents.find(
      e => e.id === this.slotState.assignedEventId,
    );
    return assigned ? [assigned] : [];
  }

  private emit(changes: Partial<CcSlotState>): void {
    this.stateChange.emit({ ...this.slotState, ...changes });
  }
}
```

### Step 4: Create `.html`

Create `monitoring-cc-covenant-slot.component.html`:
```html
<div class="cc-slot border-1 surface-border border-round p-3 mb-3">
  <!-- Header riga: index + bottone rimuovi (BR 669) -->
  <div class="flex justify-content-between align-items-center mb-3">
    <h5 class="m-0">
      {{ 'monitoring.cc.covenantNum' | customTranslate: { n: slotState.index } }}
    </h5>
    <button *ngIf="!readonly"
      pButton type="button" icon="pi pi-trash"
      class="p-button-text p-button-sm p-button-danger"
      [pTooltip]="'monitoring.cc.slot.remove' | customTranslate"
      (click)="onRemove()">
    </button>
  </div>

  <!-- EMPTY phase: nessun input, solo placeholder -->
  <ng-container *ngIf="slotState.phase === 'EMPTY'">
    <p class="text-color-secondary mb-0">{{ 'monitoring.cc.slot.empty' | customTranslate }}</p>
  </ng-container>

  <!-- FILLING phase: 3 box (BR 671-676) -->
  <ng-container *ngIf="slotState.phase === 'FILLING'">
    <div class="grid">
      <div class="col-12 md:col-4">
        <label class="block mb-2">{{ 'monitoring.cc.slot.typology' | customTranslate }}</label>
        <p-dropdown
          [options]="practiceCovenants"
          [ngModel]="slotState.selectedCovenantId"
          (ngModelChange)="onCovenantChange($event)"
          optionLabel="covenantCode"
          optionValue="id"
          [placeholder]="'monitoring.cc.slot.typologyPlaceholder' | customTranslate"
          appendTo="body"
          [style]="{ width: '100%' }">
          <ng-template let-item pTemplate="item">
            {{ item.covenantCode }} - {{ item.covenantType }}
          </ng-template>
          <ng-template let-item pTemplate="selectedItem">
            {{ item.covenantCode }} - {{ item.covenantType }}
          </ng-template>
        </p-dropdown>
      </div>
      <div class="col-12 md:col-4">
        <label class="block mb-2">{{ 'monitoring.cc.slot.value' | customTranslate }}</label>
        <input pInputText type="number" class="w-full"
          [ngModel]="slotState.monitoringValue"
          (ngModelChange)="onMonitoringValueChange($event)"
          [placeholder]="'monitoring.cc.slot.valuePlaceholder' | customTranslate"/>
      </div>
      <div class="col-12 md:col-4">
        <label class="block mb-2">{{ 'monitoring.cc.slot.referenceDate' | customTranslate }}</label>
        <input pInputText type="date" class="w-full"
          [ngModel]="slotState.referenceDate"
          (ngModelChange)="onReferenceDateChange($event)"/>
      </div>
    </div>

    <div class="flex gap-2 mt-3">
      <button pButton type="button" icon="pi pi-link"
        [label]="'monitoring.cc.slot.associaCovenant' | customTranslate"
        [disabled]="!canAssociate() || scheduleLoading"
        [loading]="scheduleLoading"
        (click)="onAssociaCovenant()">
      </button>
      <button pButton type="button"
        [label]="'monitoring.cc.slot.annulla' | customTranslate"
        class="p-button-text"
        [disabled]="scheduleLoading"
        (click)="onAnnulla()">
      </button>
    </div>
  </ng-container>

  <!-- SCHEDULE_VISIBLE / EVENT_SELECTED phase (BR 680-708) -->
  <ng-container *ngIf="slotState.phase === 'SCHEDULE_VISIBLE' || slotState.phase === 'EVENT_SELECTED'">
    <div class="surface-50 p-3 mb-3 border-round">
      <div class="grid">
        <div class="col-6 md:col-3">
          <span class="text-sm text-color-secondary block">{{ 'monitoring.cc.slot.typology' | customTranslate }}</span>
          <span class="font-semibold">{{ selectedCovenantLabel }}</span>
        </div>
        <div class="col-6 md:col-3">
          <span class="text-sm text-color-secondary block">{{ 'monitoring.cc.slot.value' | customTranslate }}</span>
          <span class="font-semibold">{{ slotState.monitoringValue }}</span>
        </div>
        <div class="col-6 md:col-3">
          <span class="text-sm text-color-secondary block">{{ 'monitoring.cc.slot.referenceDate' | customTranslate }}</span>
          <span class="font-semibold">{{ slotState.referenceDate }}</span>
        </div>
        <div class="col-6 md:col-3 text-right">
          <button pButton type="button"
            [label]="'monitoring.cc.slot.annulla' | customTranslate"
            class="p-button-text p-button-sm"
            (click)="onAnnulla()">
          </button>
        </div>
      </div>
    </div>

    <app-monitoring-cc-schedule-table
      [events]="slotState.scheduleEvents"
      [assignedEventId]="slotState.assignedEventId"
      [readonly]="false"
      (assign)="onAssignEvent($event)"
      (unassign)="onUnassignEvent()">
    </app-monitoring-cc-schedule-table>

    <div *ngIf="slotState.phase === 'EVENT_SELECTED'" class="flex justify-content-end mt-3">
      <button pButton type="button" icon="pi pi-check-circle"
        [label]="'monitoring.cc.slot.confermaAssegnazione' | customTranslate"
        (click)="onConfirmAssegnazione()">
      </button>
    </div>
  </ng-container>

  <!-- CONFIRMED phase: scadenziere collapsed a 1 riga (BR 711) -->
  <ng-container *ngIf="slotState.phase === 'CONFIRMED'">
    <div class="surface-50 p-3 mb-3 border-round">
      <div class="grid">
        <div class="col-6 md:col-4">
          <span class="text-sm text-color-secondary block">{{ 'monitoring.cc.slot.typology' | customTranslate }}</span>
          <span class="font-semibold">{{ selectedCovenantLabel }}</span>
        </div>
        <div class="col-6 md:col-4">
          <span class="text-sm text-color-secondary block">{{ 'monitoring.cc.slot.value' | customTranslate }}</span>
          <span class="font-semibold">{{ slotState.monitoringValue }}</span>
        </div>
        <div class="col-6 md:col-4">
          <span class="text-sm text-color-secondary block">{{ 'monitoring.cc.slot.referenceDate' | customTranslate }}</span>
          <span class="font-semibold">{{ slotState.referenceDate }}</span>
        </div>
      </div>
    </div>

    <app-monitoring-cc-schedule-table
      [events]="collapsedEvents()"
      [assignedEventId]="slotState.assignedEventId"
      [readonly]="true">
    </app-monitoring-cc-schedule-table>

    <div *ngIf="!readonly" class="flex justify-content-end mt-3">
      <button pButton type="button" icon="pi pi-pencil"
        [label]="'monitoring.cc.slot.modificaAssegnazione' | customTranslate"
        class="p-button-outlined"
        (click)="onModificaAssegnazione()">
      </button>
    </div>
  </ng-container>

  <!-- GENAI_PREFILLED phase (BR 733) -->
  <ng-container *ngIf="slotState.phase === 'GENAI_PREFILLED'">
    <div class="surface-50 p-3 mb-3 border-round">
      <div class="grid">
        <div class="col-12 md:col-4">
          <label class="block mb-2 text-sm text-color-secondary">{{ 'monitoring.cc.slot.typology' | customTranslate }}</label>
          <input pInputText class="w-full" [value]="selectedCovenantLabel" [disabled]="true"/>
        </div>
        <div class="col-12 md:col-4">
          <label class="block mb-2 text-sm text-color-secondary">{{ 'monitoring.cc.slot.value' | customTranslate }}</label>
          <input pInputText type="number" class="w-full" [ngModel]="slotState.monitoringValue" [disabled]="true"/>
        </div>
        <div class="col-12 md:col-4">
          <label class="block mb-2 text-sm text-color-secondary">{{ 'monitoring.cc.slot.referenceDate' | customTranslate }}</label>
          <input pInputText type="date" class="w-full" [ngModel]="slotState.referenceDate" [disabled]="true"/>
        </div>
      </div>
    </div>

    <div class="flex gap-2 justify-content-end">
      <button pButton type="button" icon="pi pi-pencil"
        [label]="'monitoring.cc.slot.modificaDati' | customTranslate"
        class="p-button-outlined"
        (click)="onModificaDati()">
      </button>
      <button pButton type="button" icon="pi pi-check-circle"
        [label]="'monitoring.cc.slot.confermaAssegnazione' | customTranslate"
        (click)="onConfirmAssegnazione()">
      </button>
    </div>
  </ng-container>
</div>
```

### Step 5: Create `.scss`

Create `monitoring-cc-covenant-slot.component.scss`:
```scss
:host {
  display: block;
}

.cc-slot {
  background-color: var(--surface-card, #ffffff);
}
```

### Step 6: Run test — atteso PASS

```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx ng test --watch=false --browsers=ChromeHeadless --code-coverage=false --include='**/monitoring-cc-covenant-slot.component.spec.ts'
```
Atteso: PASS (~14 test).

---

## Task 3.7: Component `MonitoringCcAssignmentPageComponent` (TDD orchestrator)

Page component che orchestra tutto. Service deps: `MonitoringService`, `AuthService`, `StatusDialogService`, `CustomTranslatePipe`, `Router`, `ActivatedRoute`.

**Files:**
- Create: `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/monitoring-cc-assignment-page.component.{ts,html,scss,spec.ts}`

### Step 1: Write failing spec

Create `monitoring-cc-assignment-page.component.spec.ts`:
```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { FormsModule } from '@angular/forms';
import { NO_ERRORS_SCHEMA, Pipe, PipeTransform } from '@angular/core';
import { Router, ActivatedRoute, convertToParamMap } from '@angular/router';
import { of, throwError, BehaviorSubject } from 'rxjs';

import { MonitoringCcAssignmentPageComponent } from './monitoring-cc-assignment-page.component';
import { MonitoringService } from '../../../../../service/monitoring.service';
import { StatusDialogService } from '../../../../../service/status-dialog.service';
import { CustomTranslatePipe } from '../../../../../shared/pipe/custom-translate.pipe';
import { MonitoringDocumentModel } from '../../../../../shared/model/monitoring/monitoring-document.model';
import { MonitoringCovenantModel } from '../../../../../shared/model/monitoring/monitoring-covenant.model';
import { MonitoringEventModel } from '../../../../../shared/model/monitoring/monitoring-event.model';
import { MonitoringDocumentStateEnum } from '../../../../../shared/enum/monitoring-document-state.enum';
import { MonitoringEventStateEnum } from '../../../../../shared/enum/monitoring-event-state.enum';

@Pipe({ name: 'customTranslate' })
class MockCustomTranslatePipe implements PipeTransform {
  transform(value: string): string { return value; }
}

function buildDocument(overrides: Partial<MonitoringDocumentModel> = {}): MonitoringDocumentModel {
  return {
    id: 1,
    documentCode: 'DOC-1',
    status: MonitoringDocumentStateEnum.DA_ASSEGNARE,
    uploadDate: '2026-05-01T10:00:00Z',
    fileName: 'cc.pdf',
    fileSize: 1024,
    uploadedBy: 'mario',
    fileReference: 'ref-1',
    rejectReason: '',
    eventId: 10,
    eventCode: 'E10',
    covenantCode: 'C10',
    ...overrides,
  };
}

function buildCovenant(overrides: Partial<MonitoringCovenantModel> = {}): MonitoringCovenantModel {
  return {
    id: 1, covenantCode: 'C1', covenantType: 'ICR',
    valueLimitUpper: 0, valueLimitLower: 0, sign: '<',
    periodicity: 'ANNUALE', variable: '', notes: '',
    practiceId: 1, state: 'OK',
    countEvents: 0, countExpired: 0, countExpiring: 0, countOk: 0,
    ...overrides,
  };
}

function buildEvent(overrides: Partial<MonitoringEventModel> = {}): MonitoringEventModel {
  return {
    id: 1, eventCode: 'E1', covenantCode: 'C1', covenantType: 'ICR',
    referenceDate: '2026-05-28', expirationDate: '2026-06-28',
    monitoringValue: null, status: MonitoringEventStateEnum.SCADUTO,
    actualDate: '', notes: '', hasException: false, exceptionMotivation: '',
    daysToNextMonitoring: 5, valueLimit: 100,
    ...overrides,
  };
}

describe('MonitoringCcAssignmentPageComponent', () => {
  let component: MonitoringCcAssignmentPageComponent;
  let fixture: ComponentFixture<MonitoringCcAssignmentPageComponent>;
  let monitoringService: jasmine.SpyObj<MonitoringService>;
  let statusDialogService: jasmine.SpyObj<StatusDialogService>;
  let router: jasmine.SpyObj<Router>;
  let queryParamMap$: BehaviorSubject<any>;

  beforeEach(async () => {
    monitoringService = jasmine.createSpyObj('MonitoringService', [
      'getDocumentsByPractice',
      'getCovenantsByPractice',
      'getSchedule',
      'completeAssignment',
      'rejectDocument',
      'startGenAiExtraction',
      'getGenAiResult',
    ]);
    monitoringService.getDocumentsByPractice.and.returnValue(of([buildDocument()]));
    monitoringService.getCovenantsByPractice.and.returnValue(of([buildCovenant()]));
    monitoringService.getSchedule.and.returnValue(of({ content: [], totalElements: 0, totalPages: 0, size: 20, number: 0 } as any));
    monitoringService.completeAssignment.and.returnValue(of(buildDocument({ status: MonitoringDocumentStateEnum.COMPLETATO })));
    monitoringService.rejectDocument.and.returnValue(of(buildDocument({ status: MonitoringDocumentStateEnum.IN_ATTESA_CONFERMA })));
    monitoringService.startGenAiExtraction.and.returnValue(of({} as any));
    monitoringService.getGenAiResult.and.returnValue(of({
      id: 1, documentId: 1, status: 'READY_FOR_VALIDATION', requestedAt: '2026-05-01T10:00:00Z',
      completedAt: null, requestedBy: null, documentType: 'COMPLIANCE_CERTIFICATE', extractedFields: {},
    } as any));

    statusDialogService = jasmine.createSpyObj('StatusDialogService', ['setStatus']);
    router = jasmine.createSpyObj('Router', ['navigate']);
    router.navigate.and.resolveTo(true);

    queryParamMap$ = new BehaviorSubject(convertToParamMap({}));

    await TestBed.configureTestingModule({
      declarations: [MonitoringCcAssignmentPageComponent, MockCustomTranslatePipe],
      imports: [FormsModule],
      providers: [
        { provide: MonitoringService, useValue: monitoringService },
        { provide: StatusDialogService, useValue: statusDialogService },
        { provide: Router, useValue: router },
        {
          provide: ActivatedRoute,
          useValue: {
            snapshot: { params: { id: '1', documentId: '1' } },
            queryParamMap: queryParamMap$.asObservable(),
          },
        },
        { provide: CustomTranslatePipe, useValue: { transform: (k: string) => k } },
      ],
      schemas: [NO_ERRORS_SCHEMA],
    }).compileComponents();

    fixture = TestBed.createComponent(MonitoringCcAssignmentPageComponent);
    component = fixture.componentInstance;
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  describe('Bootstrap', () => {
    it('loads document and covenants on ngOnInit', () => {
      component.ngOnInit();

      expect(monitoringService.getDocumentsByPractice).toHaveBeenCalledWith(1);
      expect(monitoringService.getCovenantsByPractice).toHaveBeenCalledWith(1);
      expect(component.documentInfo?.id).toBe(1);
      expect(component.practiceCovenants.length).toBe(1);
    });

    it('sets mode=CHOOSE when document status is DA_ASSEGNARE', () => {
      monitoringService.getDocumentsByPractice.and.returnValue(of([buildDocument({ status: MonitoringDocumentStateEnum.DA_ASSEGNARE })]));

      component.ngOnInit();

      expect(component.mode).toBe('CHOOSE');
    });

    it('sets mode=GENAI when document status is GENAI_DA_VALIDARE (load existing extraction)', () => {
      monitoringService.getDocumentsByPractice.and.returnValue(of([buildDocument({ status: MonitoringDocumentStateEnum.GENAI_DA_VALIDARE })]));

      component.ngOnInit();

      expect(component.mode).toBe('GENAI');
      expect(monitoringService.getGenAiResult).toHaveBeenCalledWith(1, true);
      expect(monitoringService.startGenAiExtraction).not.toHaveBeenCalled();
    });

    it('selectGenAiMode from CHOOSE (DA_ASSEGNARE) triggers startGenAiExtraction then getGenAiResult', () => {
      monitoringService.getDocumentsByPractice.and.returnValue(of([buildDocument({ status: MonitoringDocumentStateEnum.DA_ASSEGNARE })]));
      component.ngOnInit();
      monitoringService.getGenAiResult.calls.reset();
      monitoringService.startGenAiExtraction.calls.reset();

      component.selectGenAiMode();

      expect(monitoringService.startGenAiExtraction).toHaveBeenCalledWith(1);
      expect(monitoringService.getGenAiResult).toHaveBeenCalledWith(1, true);
    });

    it('selectGenAiMode shows error dialog when startGenAiExtraction fails', () => {
      monitoringService.getDocumentsByPractice.and.returnValue(of([buildDocument({ status: MonitoringDocumentStateEnum.DA_ASSEGNARE })]));
      component.ngOnInit();
      monitoringService.startGenAiExtraction.and.returnValue(throwError(() => ({ error: { errorMessage: 'BAD' } })));

      component.selectGenAiMode();

      const last = statusDialogService.setStatus.calls.mostRecent().args[0];
      expect(last.status).toBe('KO');
    });

    it('sets mode=VIEW when queryParam mode=view and status COMPLETATO', () => {
      monitoringService.getDocumentsByPractice.and.returnValue(of([buildDocument({ status: MonitoringDocumentStateEnum.COMPLETATO })]));
      queryParamMap$.next(convertToParamMap({ mode: 'view' }));

      component.ngOnInit();

      expect(component.mode).toBe('VIEW');
    });

    it('navigates back with error when document not found', () => {
      monitoringService.getDocumentsByPractice.and.returnValue(of([]));

      component.ngOnInit();

      expect(statusDialogService.setStatus).toHaveBeenCalled();
      const last = statusDialogService.setStatus.calls.mostRecent().args[0];
      expect(last.status).toBe('KO');
      expect(router.navigate).toHaveBeenCalled();
    });
  });

  describe('Slot management (manual)', () => {
    beforeEach(() => {
      monitoringService.getDocumentsByPractice.and.returnValue(of([buildDocument({ status: MonitoringDocumentStateEnum.DA_ASSEGNARE })]));
      component.ngOnInit();
      component.selectManualMode();
    });

    it('selectManualMode transitions to MANUAL and pushes first FILLING slot', () => {
      expect(component.mode).toBe('MANUAL');
      expect(component.slots.length).toBe(1);
      expect(component.slots[0].phase).toBe('FILLING');
      expect(component.slots[0].index).toBe(1);
    });

    it('addSlot appends a new FILLING slot with incremented index (immutable)', () => {
      const before = component.slots;

      component.addSlot();

      expect(component.slots).not.toBe(before);
      expect(component.slots.length).toBe(2);
      expect(component.slots[1].index).toBe(2);
      expect(component.slots[1].phase).toBe('FILLING');
    });

    it('removeSlot removes the slot at given uid (immutable)', () => {
      component.addSlot();
      const uidToRemove = component.slots[0].uid;
      const before = component.slots;

      component.removeSlot(uidToRemove);

      expect(component.slots).not.toBe(before);
      expect(component.slots.length).toBe(1);
      expect(component.slots[0].uid).not.toBe(uidToRemove);
    });

    it('onSlotChange replaces the slot immutably', () => {
      const targetUid = component.slots[0].uid;
      const updated = { ...component.slots[0], monitoringValue: 99 };

      component.onSlotChange(targetUid, updated);

      expect(component.slots[0].monitoringValue).toBe(99);
    });

    it('onSlotRequestSchedule calls getSchedule with filters and updates slot with events', () => {
      const events = [buildEvent({ id: 100 })];
      monitoringService.getSchedule.and.returnValue(of({ content: events, totalElements: 1, totalPages: 1, size: 20, number: 0 } as any));
      const targetUid = component.slots[0].uid;

      component.onSlotRequestSchedule(targetUid, { covenantId: 1, referenceDate: '2026-05-28' });

      expect(monitoringService.getSchedule).toHaveBeenCalled();
      expect(component.slots[0].phase).toBe('SCHEDULE_VISIBLE');
      expect(component.slots[0].scheduleEvents.length).toBe(1);
    });

    it('onSlotRequestSchedule shows error dialog and resets phase to FILLING on failure', () => {
      monitoringService.getSchedule.and.returnValue(throwError(() => new Error('boom')));
      const targetUid = component.slots[0].uid;

      component.onSlotRequestSchedule(targetUid, { covenantId: 1, referenceDate: '2026-05-28' });

      expect(statusDialogService.setStatus).toHaveBeenCalled();
      const last = statusDialogService.setStatus.calls.mostRecent().args[0];
      expect(last.status).toBe('KO');
      expect(component.slots[0].phase).toBe('FILLING');
    });
  });

  describe('canConcludi', () => {
    beforeEach(() => {
      monitoringService.getDocumentsByPractice.and.returnValue(of([buildDocument({ status: MonitoringDocumentStateEnum.DA_ASSEGNARE })]));
      component.ngOnInit();
      component.selectManualMode();
    });

    it('returns false when at least one slot is unconfirmed', () => {
      expect(component.canConcludi).toBeFalse();
    });

    it('returns true only when slots.length > 0 AND every slot.confirmed=true', () => {
      const updated = { ...component.slots[0], confirmed: true, phase: 'CONFIRMED' as const };
      component.onSlotChange(component.slots[0].uid, updated);

      expect(component.canConcludi).toBeTrue();
    });
  });

  describe('onConcludi', () => {
    beforeEach(() => {
      monitoringService.getDocumentsByPractice.and.returnValue(of([buildDocument({ status: MonitoringDocumentStateEnum.DA_ASSEGNARE })]));
      component.ngOnInit();
      component.selectManualMode();
      const updated = {
        ...component.slots[0],
        confirmed: true,
        phase: 'CONFIRMED' as const,
        selectedCovenantId: 1,
        monitoringValue: 50,
        referenceDate: '2026-05-28',
        assignedEventId: 99,
      };
      component.onSlotChange(component.slots[0].uid, updated);
    });

    it('calls completeAssignment with slot assignments mapped to BE payload', () => {
      component.onConcludi();

      expect(monitoringService.completeAssignment).toHaveBeenCalled();
      const [docId, payload] = monitoringService.completeAssignment.calls.mostRecent().args;
      expect(docId).toBe(1);
      expect(payload.assignments.length).toBe(1);
      expect(payload.assignments[0]).toEqual({
        covenantId: 1, eventId: 99, monitoringValue: 50, actualDate: '2026-05-28', referenceDate: '2026-05-28',
      });
    });

    it('shows success dialog and navigates back on completeAssignment success', () => {
      component.onConcludi();

      const last = statusDialogService.setStatus.calls.mostRecent().args[0];
      expect(last.status).toBe('OK');
      expect(router.navigate).toHaveBeenCalled();
    });

    it('shows error dialog and stays on page on completeAssignment failure', () => {
      monitoringService.completeAssignment.and.returnValue(throwError(() => ({ error: { errorMessage: 'BAD' } })));

      component.onConcludi();

      const last = statusDialogService.setStatus.calls.mostRecent().args[0];
      expect(last.status).toBe('KO');
      expect(component.concludeInFlight).toBeFalse();
    });

    it('does nothing when canConcludi is false', () => {
      const targetUid = component.slots[0].uid;
      const reset = { ...component.slots[0], confirmed: false, phase: 'EVENT_SELECTED' as const };
      component.onSlotChange(targetUid, reset);
      monitoringService.completeAssignment.calls.reset();

      component.onConcludi();

      expect(monitoringService.completeAssignment).not.toHaveBeenCalled();
    });
  });

  describe('reject flow', () => {
    beforeEach(() => {
      monitoringService.getDocumentsByPractice.and.returnValue(of([buildDocument({ status: MonitoringDocumentStateEnum.GENAI_DA_VALIDARE })]));
      component.ngOnInit();
    });

    it('openReject sets rejectModalOpen=true', () => {
      component.openReject();
      expect(component.rejectModalOpen).toBeTrue();
    });

    it('onRejected calls rejectDocument with combined reason+notes and navigates back', () => {
      component.onRejected({ reason: 'INCOMPLETE', notes: 'pagine mancanti' });

      expect(monitoringService.rejectDocument).toHaveBeenCalledWith(1, 'INCOMPLETE: pagine mancanti');
      expect(router.navigate).toHaveBeenCalled();
    });

    it('onRejected omits "  :" when notes empty', () => {
      component.onRejected({ reason: 'OTHER', notes: '' });

      expect(monitoringService.rejectDocument).toHaveBeenCalledWith(1, 'OTHER');
    });

    it('onRejected shows error dialog on failure', () => {
      monitoringService.rejectDocument.and.returnValue(throwError(() => ({ error: { errorMessage: 'BAD' } })));

      component.onRejected({ reason: 'OTHER', notes: '' });

      const last = statusDialogService.setStatus.calls.mostRecent().args[0];
      expect(last.status).toBe('KO');
    });
  });

  describe('navigation back', () => {
    it('onBack calls router.navigate up two levels to practice detail', () => {
      component.onBack();
      expect(router.navigate).toHaveBeenCalled();
    });
  });

  describe('previewUrl', () => {
    it('returns the download URL with documentId path', () => {
      component.documentInfo = buildDocument({ id: 42 });
      expect(component.previewUrl).toContain('/monitoring/documents/42/download');
    });

    it('returns null when documentInfo is absent', () => {
      component.documentInfo = null;
      expect(component.previewUrl).toBeNull();
    });
  });

  describe('ngOnDestroy', () => {
    it('completes destroy$ subject to clean RxJS subscriptions', () => {
      const completeSpy = spyOn<any>((component as any).destroy$, 'complete');

      component.ngOnDestroy();

      expect(completeSpy).toHaveBeenCalled();
    });
  });
});
```

### Step 2: Run test — atteso FAIL

```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx ng test --watch=false --browsers=ChromeHeadless --code-coverage=false --include='**/monitoring-cc-assignment-page.component.spec.ts'
```
Atteso: errore compilazione.

### Step 3: Create `.ts`

Create `monitoring-cc-assignment-page.component.ts`:
```typescript
import { Component, OnDestroy, OnInit } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import { Subject, forkJoin } from 'rxjs';
import { finalize, take, takeUntil } from 'rxjs/operators';

import { MonitoringService } from '../../../../../service/monitoring.service';
import { StatusDialogService } from '../../../../../service/status-dialog.service';
import { CustomTranslatePipe } from '../../../../../shared/pipe/custom-translate.pipe';
import { App } from '../../../../../shared/const/app.const';
import { environment } from '../../../../../../environments/environment';
import { MonitoringDocumentModel } from '../../../../../shared/model/monitoring/monitoring-document.model';
import { MonitoringCovenantModel } from '../../../../../shared/model/monitoring/monitoring-covenant.model';
import { MonitoringEventModel } from '../../../../../shared/model/monitoring/monitoring-event.model';
import { MonitoringDocumentStateEnum } from '../../../../../shared/enum/monitoring-document-state.enum';
import { MonitoringEventStateEnum } from '../../../../../shared/enum/monitoring-event-state.enum';
import { MonitoringGenAiExtractionStatusEnum } from '../../../../../shared/enum/monitoring-genai-extraction-status.enum';
import {
  CcSlotState,
  CcCovenantAssignmentPayload,
} from '../../../../../shared/model/monitoring/monitoring-cc-assignment.model';
import { CcSlotScheduleRequest, CcSlotMode } from './components/monitoring-cc-covenant-slot.component';
import { CcRejectPayload } from './components/monitoring-cc-reject-dialog.component';

type PageMode = 'CHOOSE' | 'MANUAL' | 'GENAI' | 'VIEW';

const SCHEDULE_DEFAULT_STATUSES: MonitoringEventStateEnum[] = [
  MonitoringEventStateEnum.SCADUTO,
  MonitoringEventStateEnum.FUORI_SOGLIA,
  MonitoringEventStateEnum.IN_SCADENZA,
];
const SCHEDULE_DEFAULT_RANGE_DAYS = 15;

/**
 * BR 4.2.3 (righe 652-744) — Page CC Assignment standalone.
 * Sub-route: `monitoring/practice/:id/cc/:documentId[?mode=view]`.
 *
 * Orchestratore di 4 mode:
 *   CHOOSE → utente sceglie Manual/GenAI
 *   MANUAL → N slot user-driven (FILLING → SCHEDULE_VISIBLE → CONFIRMED) → Concludi
 *   GENAI  → load extraction → N slot prefilled (GENAI_PREFILLED) → Modifica/Conferma → Concludi
 *   VIEW   → mode=view + status=COMPLETATO → slot readonly (1 covenant scope cut)
 *
 * Persist strategy: batch su "Concludi Assegnazione" via `completeAssignment(docId, {assignments})`.
 * State management: campi locali + Subject<void> destroy$ + takeUntil. Immutability slot[] via spread.
 * No silent swallow: ogni error branch chiama StatusDialogService.setStatus.
 */
@Component({
  selector: 'app-monitoring-cc-assignment-page',
  templateUrl: './monitoring-cc-assignment-page.component.html',
  styleUrls: ['./monitoring-cc-assignment-page.component.scss'],
})
export class MonitoringCcAssignmentPageComponent implements OnInit, OnDestroy {
  public practiceId!: number;
  public documentId!: number;
  public documentInfo: MonitoringDocumentModel | null = null;
  public practiceCovenants: MonitoringCovenantModel[] = [];

  public mode: PageMode = 'CHOOSE';
  public slots: CcSlotState[] = [];

  public bootstrapLoading = false;
  public scheduleLoadingByUid: Record<string, boolean> = {};
  public concludeInFlight = false;
  public rejectInFlight = false;
  public extractionLoading = false;

  public rejectModalOpen = false;

  private readonly destroy$ = new Subject<void>();

  constructor(
    private route: ActivatedRoute,
    private router: Router,
    private monitoringService: MonitoringService,
    private statusDialogService: StatusDialogService,
    private customTranslatePipe: CustomTranslatePipe,
  ) {}

  public ngOnInit(): void {
    this.practiceId = Number(this.route.snapshot.params['id']);
    this.documentId = Number(this.route.snapshot.params['documentId']);
    this.route.queryParamMap
      .pipe(take(1))
      .subscribe(params => {
        const requestedMode = params.get('mode');
        this.bootstrap(requestedMode === 'view');
      });
  }

  public ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }

  public get canConcludi(): boolean {
    return this.slots.length > 0 && this.slots.every(s => s.confirmed);
  }

  public get showRejectButton(): boolean {
    return this.mode === 'GENAI'
      && this.documentInfo?.status === MonitoringDocumentStateEnum.GENAI_DA_VALIDARE;
  }

  public get previewUrl(): string | null {
    if (!this.documentInfo) {
      return null;
    }
    return `${App.origin}${environment.apiBaseUrl}/monitoring/documents/${this.documentInfo.id}/download`;
  }

  public get currentSlotMode(): CcSlotMode {
    if (this.mode === 'VIEW') return 'view';
    if (this.mode === 'GENAI') return 'genai';
    return 'manual';
  }

  // ────────────────────────────────────────
  // Mode selection
  // ────────────────────────────────────────

  public selectManualMode(): void {
    this.mode = 'MANUAL';
    this.slots = [this.makeSlot(1, 'FILLING')];
  }

  public selectGenAiMode(): void {
    this.mode = 'GENAI';
    // BR 731: lo spinner si vede mentre l'estrazione avviene. Se lo status doc è
    // ancora DA_ASSEGNARE o IN_ATTESA_CONFERMA, dobbiamo avviare l'estrazione qui.
    // Se è già GENAI_DA_VALIDARE/GENAI_IN_CORSO, riprendi l'extraction già in corso.
    const status = this.documentInfo?.status;
    if (status === MonitoringDocumentStateEnum.GENAI_DA_VALIDARE
        || status === MonitoringDocumentStateEnum.GENAI_IN_CORSO) {
      this.loadGenAiExtraction();
    } else {
      this.startAndLoadGenAiExtraction();
    }
  }

  private startAndLoadGenAiExtraction(): void {
    this.extractionLoading = true;
    this.monitoringService.startGenAiExtraction(this.documentId)
      .pipe(takeUntil(this.destroy$))
      .subscribe({
        next: () => this.loadGenAiExtraction(),
        error: err => {
          this.extractionLoading = false;
          this.errorDialog(err, 'monitoring.cc.genai.failed');
        },
      });
  }

  // ────────────────────────────────────────
  // Slot management (immutable)
  // ────────────────────────────────────────

  public addSlot(): void {
    const nextIndex = this.slots.length + 1;
    this.slots = [...this.slots, this.makeSlot(nextIndex, 'FILLING')];
  }

  public removeSlot(uid: string): void {
    this.slots = this.slots
      .filter(s => s.uid !== uid)
      .map((s, i) => ({ ...s, index: i + 1 }));
  }

  public onSlotChange(uid: string, updated: CcSlotState): void {
    this.slots = this.slots.map(s => (s.uid === uid ? updated : s));
  }

  public onSlotRequestSchedule(uid: string, req: CcSlotScheduleRequest): void {
    const covenant = this.practiceCovenants.find(c => c.id === req.covenantId);
    if (!covenant) {
      return;
    }
    this.scheduleLoadingByUid = { ...this.scheduleLoadingByUid, [uid]: true };

    const params = {
      covenantType: covenant.covenantType,
      referenceDate: req.referenceDate,
      rangeDays: SCHEDULE_DEFAULT_RANGE_DAYS,
      statuses: SCHEDULE_DEFAULT_STATUSES.join(','),
      page: 0,
      size: 50,
    };

    this.monitoringService
      .getSchedule(this.practiceId, params)
      .pipe(
        takeUntil(this.destroy$),
        finalize(() => {
          const next = { ...this.scheduleLoadingByUid };
          delete next[uid];
          this.scheduleLoadingByUid = next;
        }),
      )
      .subscribe({
        next: page => {
          const events = page?.content || [];
          this.applySlotPatch(uid, { phase: 'SCHEDULE_VISIBLE', scheduleEvents: events });
        },
        error: err => {
          this.applySlotPatch(uid, { phase: 'FILLING', scheduleEvents: [] });
          this.errorDialog(err, 'monitoring.cc.schedule.loadError');
        },
      });
  }

  // ────────────────────────────────────────
  // Concludi
  // ────────────────────────────────────────

  public onConcludi(): void {
    if (!this.canConcludi || this.concludeInFlight) {
      return;
    }
    const assignments: CcCovenantAssignmentPayload[] = this.slots
      .filter(s => s.confirmed && s.selectedCovenantId != null && s.assignedEventId != null
        && s.monitoringValue != null && s.referenceDate != null)
      .map(s => ({
        covenantId: s.selectedCovenantId!,
        eventId: s.assignedEventId!,
        monitoringValue: s.monitoringValue!,
        actualDate: s.referenceDate!,
        referenceDate: s.referenceDate!,
      }));

    if (assignments.length === 0) {
      return;
    }

    this.concludeInFlight = true;
    this.monitoringService
      .completeAssignment(this.documentId, { assignments })
      .pipe(takeUntil(this.destroy$), finalize(() => this.concludeInFlight = false))
      .subscribe({
        next: () => {
          this.statusDialogService.setStatus({
            status: 'OK',
            title: this.translate('shared.success'),
            message: this.translate('monitoring.cc.complete.success'),
            closable: true,
          });
          this.navigateBack();
        },
        error: err => this.errorDialog(err, 'monitoring.cc.complete.error'),
      });
  }

  // ────────────────────────────────────────
  // Reject (bad flow GenAI)
  // ────────────────────────────────────────

  public openReject(): void {
    this.rejectModalOpen = true;
  }

  public onRejectedCancel(): void {
    this.rejectModalOpen = false;
  }

  public onRejected(payload: CcRejectPayload): void {
    if (this.rejectInFlight) {
      return;
    }
    const reason = payload.notes
      ? `${payload.reason}: ${payload.notes}`
      : payload.reason;

    this.rejectInFlight = true;
    this.monitoringService
      .rejectDocument(this.documentId, reason)
      .pipe(takeUntil(this.destroy$), finalize(() => this.rejectInFlight = false))
      .subscribe({
        next: () => {
          this.rejectModalOpen = false;
          this.statusDialogService.setStatus({
            status: 'OK',
            title: this.translate('shared.success'),
            message: this.translate('monitoring.cc.reject.successIsp'),
            closable: true,
          });
          this.navigateBack();
        },
        error: err => this.errorDialog(err, 'monitoring.cc.reject.error'),
      });
  }

  // ────────────────────────────────────────
  // Navigation
  // ────────────────────────────────────────

  public onBack(): void {
    this.navigateBack();
  }

  private navigateBack(): void {
    this.router.navigate(['../../'], { relativeTo: this.route });
  }

  // ────────────────────────────────────────
  // Internals
  // ────────────────────────────────────────

  private bootstrap(requestedView: boolean): void {
    this.bootstrapLoading = true;

    forkJoin({
      documents: this.monitoringService.getDocumentsByPractice(this.practiceId),
      covenants: this.monitoringService.getCovenantsByPractice(this.practiceId),
    })
      .pipe(takeUntil(this.destroy$), finalize(() => this.bootstrapLoading = false))
      .subscribe({
        next: ({ documents, covenants }) => {
          const doc = (documents || []).find(d => d.id === this.documentId) || null;
          this.documentInfo = doc;
          this.practiceCovenants = covenants || [];
          this.resolveMode(doc, requestedView);
        },
        error: err => this.errorDialog(err, 'monitoring.cc.bootstrap.error', true),
      });
  }

  private resolveMode(doc: MonitoringDocumentModel | null, requestedView: boolean): void {
    if (!doc) {
      this.errorDialog(null, 'monitoring.cc.bootstrap.notFound', true);
      return;
    }

    if (requestedView && doc.status === MonitoringDocumentStateEnum.COMPLETATO) {
      this.mode = 'VIEW';
      this.buildViewSlot(doc);
      return;
    }

    switch (doc.status) {
      case MonitoringDocumentStateEnum.GENAI_DA_VALIDARE:
        this.selectGenAiMode();
        break;
      case MonitoringDocumentStateEnum.DA_ASSEGNARE:
        this.mode = 'CHOOSE';
        break;
      default:
        this.errorDialog(null, 'monitoring.cc.bootstrap.invalidStatus', true);
    }
  }

  private buildViewSlot(doc: MonitoringDocumentModel): void {
    // BR 4.2.4 scope cut — supporto solo legacy 1-covenant assignment.
    // Multi-covenant lookup richiede tabella associativa BE → bug separato.
    if (doc.eventId == null) {
      this.slots = [];
      return;
    }
    this.slots = [{
      uid: this.generateUid(),
      index: 1,
      phase: 'CONFIRMED',
      selectedCovenantId: null,
      monitoringValue: null,
      referenceDate: null,
      scheduleEvents: [],
      assignedEventId: doc.eventId,
      confirmed: true,
    }];
  }

  private loadGenAiExtraction(): void {
    this.extractionLoading = true;
    this.monitoringService
      .getGenAiResult(this.documentId, true)
      .pipe(takeUntil(this.destroy$), finalize(() => this.extractionLoading = false))
      .subscribe({
        next: extraction => {
          if (extraction.status === MonitoringGenAiExtractionStatusEnum.READY_FOR_VALIDATION) {
            this.slots = this.buildSlotsFromExtraction(extraction.extractedFields || {});
          } else if (extraction.status === MonitoringGenAiExtractionStatusEnum.FAILED) {
            this.errorDialog(null, 'monitoring.cc.genai.failed');
          }
        },
        error: err => this.errorDialog(err, 'monitoring.cc.genai.failed'),
      });
  }

  private buildSlotsFromExtraction(fields: { [key: string]: string }): CcSlotState[] {
    // Mock GenAI ritorna single-block (limite documentato in spec §11). Quando arriva
    // estrazione multi-block servirà parsing per indice/blocco. Per ora N=1.
    if (Object.keys(fields).length === 0) {
      return [];
    }
    return [{
      uid: this.generateUid(),
      index: 1,
      phase: 'GENAI_PREFILLED',
      selectedCovenantId: null,
      monitoringValue: this.parseNumber(fields['COVENANT_NUM_VALORE_EFFETTIVO']),
      referenceDate: fields['COVENANT_NUM_DATA_RIF_PRIMA_RILEV'] || null,
      scheduleEvents: [],
      assignedEventId: null,
      confirmed: false,
    }];
  }

  private applySlotPatch(uid: string, patch: Partial<CcSlotState>): void {
    this.slots = this.slots.map(s => (s.uid === uid ? { ...s, ...patch } : s));
  }

  private makeSlot(index: number, phase: CcSlotState['phase']): CcSlotState {
    return {
      uid: this.generateUid(),
      index,
      phase,
      selectedCovenantId: null,
      monitoringValue: null,
      referenceDate: null,
      scheduleEvents: [],
      assignedEventId: null,
      confirmed: false,
    };
  }

  private generateUid(): string {
    return `slot_${Date.now()}_${Math.random().toString(36).slice(2, 8)}`;
  }

  private parseNumber(value: string | undefined): number | null {
    if (value == null || value === '') return null;
    const n = parseFloat(value);
    return isNaN(n) ? null : n;
  }

  private translate(key: string, params?: Object): string {
    return this.customTranslatePipe.transform(key, params);
  }

  private errorDialog(err: any, fallbackKey: string, navigateBackAfter = false): void {
    const message = err?.error?.errorMessage || err?.message || this.translate(fallbackKey);
    this.statusDialogService.setStatus({
      status: 'KO',
      title: this.translate('shared.error'),
      message,
      closable: true,
      onHide: navigateBackAfter ? () => this.navigateBack() : undefined,
    });
    if (navigateBackAfter) {
      this.navigateBack();
    }
  }
}
```

### Step 4: Create `.html`

Create `monitoring-cc-assignment-page.component.html`:
```html
<div class="cc-page p-4">
  <!-- Header -->
  <div class="surface-card p-4 border-round mb-3 border-1 surface-border">
    <div class="flex align-items-center justify-content-between">
      <div>
        <h2 class="m-0">{{ 'monitoring.cc.page.title' | customTranslate }}</h2>
        <p class="text-color-secondary m-0 mt-1">
          {{ 'monitoring.cc.page.documentLabel' | customTranslate }}: <strong>{{ documentInfo?.fileName || '—' }}</strong>
        </p>
      </div>
      <div class="flex gap-2">
        <button pButton type="button" icon="pi pi-arrow-left"
          [label]="'monitoring.cc.back' | customTranslate"
          class="p-button-text"
          (click)="onBack()">
        </button>
        <button *ngIf="showRejectButton"
          pButton type="button" icon="pi pi-flag"
          [label]="'monitoring.cc.reject.button' | customTranslate"
          class="p-button-outlined p-button-danger"
          (click)="openReject()">
        </button>
      </div>
    </div>
  </div>

  <!-- Bootstrap loading -->
  <div *ngIf="bootstrapLoading" class="text-center my-5">
    <p-progressBar mode="indeterminate" [style]="{ height: '6px' }"></p-progressBar>
    <p class="mt-3 text-color-secondary">{{ 'monitoring.cc.bootstrap.loading' | customTranslate }}</p>
  </div>

  <ng-container *ngIf="!bootstrapLoading">
    <!-- Preview + content layout -->
    <div class="grid">
      <!-- Left: preview -->
      <div class="col-12 md:col-5">
        <div class="surface-card p-3 border-round border-1 surface-border" style="min-height: 600px;">
          <h4 class="mt-0">{{ 'monitoring.cc.preview.title' | customTranslate }}</h4>
          <object *ngIf="previewUrl"
            [data]="previewUrl"
            type="application/pdf"
            width="100%"
            height="540px">
            <p class="text-color-secondary">
              {{ 'monitoring.cc.preview.unavailable' | customTranslate }}
              <a *ngIf="documentInfo" [href]="previewUrl" target="_blank">{{ 'monitoring.cc.preview.download' | customTranslate }}</a>
            </p>
          </object>
        </div>
      </div>

      <!-- Right: action area -->
      <div class="col-12 md:col-7">
        <!-- CHOOSE mode -->
        <div *ngIf="mode === 'CHOOSE'" class="surface-card p-4 border-round border-1 surface-border text-center">
          <h4 class="mt-0">{{ 'monitoring.cc.choose.title' | customTranslate }}</h4>
          <p class="text-color-secondary">{{ 'monitoring.cc.choose.subtitle' | customTranslate }}</p>
          <div class="flex justify-content-center gap-3 mt-4">
            <button pButton type="button" icon="pi pi-bolt"
              [label]="'monitoring.cc.mode.genai' | customTranslate"
              (click)="selectGenAiMode()">
            </button>
            <button pButton type="button" icon="pi pi-pencil"
              [label]="'monitoring.cc.mode.manual' | customTranslate"
              class="p-button-outlined"
              (click)="selectManualMode()">
            </button>
          </div>
        </div>

        <!-- GenAI extraction loading -->
        <div *ngIf="mode === 'GENAI' && extractionLoading" class="surface-card p-4 border-round border-1 surface-border text-center">
          <p-progressBar mode="indeterminate" [style]="{ height: '6px' }"></p-progressBar>
          <p class="mt-3 text-color-secondary">{{ 'monitoring.cc.genai.extracting' | customTranslate }}</p>
        </div>

        <!-- Slots area (MANUAL / GENAI / VIEW) -->
        <ng-container *ngIf="(mode === 'MANUAL' || mode === 'GENAI' || mode === 'VIEW') && !extractionLoading">
          <app-monitoring-cc-covenant-slot *ngFor="let slot of slots; trackBy: slotTrackBy"
            [slotState]="slot"
            [practiceCovenants]="practiceCovenants"
            [mode]="currentSlotMode"
            [scheduleLoading]="!!scheduleLoadingByUid[slot.uid]"
            (stateChange)="onSlotChange(slot.uid, $event)"
            (requestSchedule)="onSlotRequestSchedule(slot.uid, $event)"
            (remove)="removeSlot(slot.uid)">
          </app-monitoring-cc-covenant-slot>

          <div *ngIf="mode === 'MANUAL'" class="flex justify-content-between mt-3">
            <button pButton type="button" icon="pi pi-plus"
              [label]="'monitoring.cc.addCovenant' | customTranslate"
              class="p-button-outlined"
              (click)="addSlot()">
            </button>
            <button pButton type="button" icon="pi pi-check-circle"
              [label]="'monitoring.cc.concludi' | customTranslate"
              [disabled]="!canConcludi || concludeInFlight"
              [loading]="concludeInFlight"
              (click)="onConcludi()">
            </button>
          </div>

          <div *ngIf="mode === 'GENAI'" class="flex justify-content-end mt-3">
            <button pButton type="button" icon="pi pi-check-circle"
              [label]="'monitoring.cc.concludi' | customTranslate"
              [disabled]="!canConcludi || concludeInFlight"
              [loading]="concludeInFlight"
              (click)="onConcludi()">
            </button>
          </div>

          <div *ngIf="mode === 'VIEW' && slots.length === 0" class="surface-card p-4 border-round border-1 surface-border text-center">
            <p class="text-color-secondary">{{ 'monitoring.cc.view.empty' | customTranslate }}</p>
          </div>
        </ng-container>
      </div>
    </div>
  </ng-container>

  <!-- Reject Dialog (mode GENAI) -->
  <app-monitoring-cc-reject-dialog
    [(visible)]="rejectModalOpen"
    [submitting]="rejectInFlight"
    (rejected)="onRejected($event)"
    (cancelled)="onRejectedCancel()">
  </app-monitoring-cc-reject-dialog>
</div>
```

> Aggiungere il `trackBy` nel `.ts`:
```typescript
  public slotTrackBy = (_: number, s: CcSlotState) => s.uid;
```

### Step 5: Create `.scss`

Create `monitoring-cc-assignment-page.component.scss`:
```scss
:host {
  display: block;
  background: var(--surface-ground, #f8f9fa);
  min-height: 100vh;
}

.cc-page {
  max-width: 1400px;
  margin: 0 auto;
}
```

### Step 6: Add `slotTrackBy` to `.ts`

In `monitoring-cc-assignment-page.component.ts`, INSERT inside class (e.g. after `get previewUrl`):
```typescript
  public slotTrackBy = (_: number, s: CcSlotState) => s.uid;
```

### Step 7: Run test — atteso PASS

```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx ng test --watch=false --browsers=ChromeHeadless --code-coverage=false --include='**/monitoring-cc-assignment-page.component.spec.ts'
```
Atteso: PASS (~22 test).

---

## Task 3.8: Wire-up `practice-detail.module.ts` (declare + route + imports)

**Files:**
- Modify: `src/app/module/agency-desk/monitoring/practice-detail/practice-detail.module.ts`

- [ ] **Step 1**: Replace intero `practice-detail.module.ts`

REPLACE:
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule, Routes } from '@angular/router';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
import { DropdownModule } from 'primeng/dropdown';
import { DialogModule } from 'primeng/dialog';
import { ProgressBarModule } from 'primeng/progressbar';
import { InputTextareaModule } from 'primeng/inputtextarea';
import { ButtonModule as PrimeButtonModule } from 'primeng/button';
import { ButtonModule } from '../../../../shared/ui/base-element/button/button.module';
import { TooltipModule } from 'primeng/tooltip';
import { TableModule as PrimeTableModule } from 'primeng/table';
import { InputTextModule } from 'primeng/inputtext';
import { InputNumberModule } from 'primeng/inputnumber';
import { CheckboxModule } from 'primeng/checkbox';
import { TagModule } from 'primeng/tag';
import { TranslateModule } from '@ngx-translate/core';
import { MonitoringPracticeDetailComponent } from './practice-detail.component';
import { MonitoringCovenantEditPageComponent } from './covenant-edit-page/covenant-edit-page.component';
import { ScheduleTabComponent } from './schedule-tab/schedule-tab.component';
import { SpreadTabComponent } from './spread-tab/spread-tab.component';
import { CovenantDetailComponent } from './covenant-detail/covenant-detail.component';
import { MonitoringDocumentUploadComponent } from './document-upload/document-upload.component';
import { MonitoringDocumentHistoryComponent } from './document-history/document-history.component';
import { CcAssignmentComponent } from './cc-assignment/cc-assignment.component';
import { MonitoringCcAssignmentPageComponent } from './cc-assignment-page/monitoring-cc-assignment-page.component';
import { MonitoringCcCovenantSlotComponent } from './cc-assignment-page/components/monitoring-cc-covenant-slot.component';
import { MonitoringCcScheduleTableComponent } from './cc-assignment-page/components/monitoring-cc-schedule-table.component';
import { MonitoringCcRejectDialogComponent } from './cc-assignment-page/components/monitoring-cc-reject-dialog.component';
import { MonitoringExceptionDialogModule } from '../exception-dialog/exception-dialog.module';
import { MonitoringGenAiExtractionModule } from './monitoring-genai-extraction/monitoring-genai-extraction.module';
import { MonitoringCovenantAddDialogModule } from './covenant-add-dialog/covenant-add-dialog.module';
import { FileUploadModule } from '../../../../shared/ui/base-element/file-upload/file-upload.module';
import { DirectiveModule } from '../../../../shared/directive/directive.module';
import { TableModule } from '../../../../shared/ui/base-element/table/table.module';
import { SemaphoreModule } from '../../../../shared/ui/base-element/semaphore/semaphore.module';
import { TabModule } from '../../../../shared/ui/base-element/tab/tab.module';
import { AccordionModule } from '../../../../shared/ui/base-element/accordion/accordion.module';
import { PipeModule } from '../../../../shared/pipe/pipe.module';
import { CustomTranslatePipe } from '../../../../shared/pipe/custom-translate.pipe';
import { LocalizedDatePipe } from '../../../../shared/pipe/localized-date.pipe';

const routes: Routes = [
  { path: '', component: MonitoringPracticeDetailComponent },
  { path: 'covenant/:covenantId/edit', component: MonitoringCovenantEditPageComponent },
  { path: 'cc/:documentId', component: MonitoringCcAssignmentPageComponent },
];

@NgModule({
  declarations: [
    MonitoringPracticeDetailComponent,
    MonitoringCovenantEditPageComponent,
    ScheduleTabComponent,
    SpreadTabComponent,
    CovenantDetailComponent,
    MonitoringDocumentUploadComponent,
    MonitoringDocumentHistoryComponent,
    CcAssignmentComponent,
    MonitoringCcAssignmentPageComponent,
    MonitoringCcCovenantSlotComponent,
    MonitoringCcScheduleTableComponent,
    MonitoringCcRejectDialogComponent,
  ],
  imports: [
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    RouterModule.forChild(routes),
    DropdownModule,
    DialogModule,
    ProgressBarModule,
    InputTextareaModule,
    PrimeButtonModule,
    ButtonModule,
    TooltipModule,
    PrimeTableModule,
    InputTextModule,
    InputNumberModule,
    CheckboxModule,
    TagModule,
    TranslateModule,
    FileUploadModule,
    TableModule,
    SemaphoreModule,
    TabModule,
    AccordionModule,
    DirectiveModule,
    PipeModule,
    MonitoringExceptionDialogModule,
    MonitoringGenAiExtractionModule,
    MonitoringCovenantAddDialogModule,
  ],
  providers: [CustomTranslatePipe, LocalizedDatePipe],
})
export class MonitoringPracticeDetailModule {}
```

> **Note**: `MonitoringGenAiExtractionModule` resta ancora declarata (rimossa in Commit 5 post-smoke). `CcAssignmentComponent` resta declarato (rimosso in Commit 5).

- [ ] **Step 2**: Verify tsc
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx tsc --noEmit --skipLibCheck
```
Atteso: 0 errori.

---

## Task 3.9: i18n keys (it-IT + en-GB)

**Files:**
- Modify: `src/assets/i18n/it-IT.json`
- Modify: `src/assets/i18n/en-GB.json`

L'oggetto `monitoring` parte intorno a riga 3687 di `it-IT.json`. Trovare la chiusura `}` del blocco `monitoring` (cerca grep `"monitoring":` per delimitarne il range) e INSERIRE un nuovo blocco `cc: {...}` PRIMA della chiusura.

- [ ] **Step 1**: Edit `it-IT.json` — aggiungere blocco `cc` dentro `monitoring`

```json
    "cc": {
      "page": {
        "title": "Assegnazione Compliance Certificate",
        "documentLabel": "Documento"
      },
      "bootstrap": {
        "loading": "Caricamento documento in corso…",
        "error": "Errore durante il caricamento dei dati",
        "notFound": "Documento non trovato per questa pratica",
        "invalidStatus": "Il documento non è nello stato richiesto"
      },
      "preview": {
        "title": "Anteprima documento",
        "unavailable": "Anteprima non disponibile,",
        "download": "scarica il documento"
      },
      "choose": {
        "title": "Come vuoi procedere?",
        "subtitle": "Puoi avviare l'estrazione automatica via GenAI oppure compilare i dati manualmente"
      },
      "mode": {
        "genai": "Assegnazione con GenAI",
        "manual": "Assegnazione Manuale"
      },
      "back": "Indietro",
      "reject": {
        "button": "Ritorna a documentazione incompleta",
        "modal": {
          "title": "Rifiuto Documento",
          "reasonLabel": "Motivo del rifiuto *",
          "reasonPlaceholder": "Seleziona un motivo",
          "notes": "Note (facoltative)",
          "notesPlaceholder": "Aggiungi eventuali note per la banca",
          "confirm": "Conferma Rifiuto"
        },
        "reasons": {
          "incomplete": "Documento incompleto",
          "format": "Formato non corretto",
          "signature": "Firma mancante",
          "other": "Altro"
        },
        "successIsp": "Rifiuto inviato a ISP con notifica",
        "error": "Errore durante il rifiuto del documento"
      },
      "covenantNum": "Covenant #{{ n }}",
      "addCovenant": "Aggiungi Covenant",
      "concludi": "Concludi Assegnazione",
      "complete": {
        "success": "Assegnazione completata con successo",
        "error": "Errore durante il completamento dell'assegnazione"
      },
      "slot": {
        "empty": "Aggiungi un covenant per iniziare",
        "remove": "Rimuovi covenant",
        "typology": "Seleziona Tipologia Covenant",
        "typologyPlaceholder": "Seleziona una tipologia",
        "value": "Valore Monitoraggio",
        "valuePlaceholder": "Inserisci valore",
        "referenceDate": "Data di Riferimento",
        "associaCovenant": "Associa Covenant",
        "annulla": "Annulla",
        "confermaAssegnazione": "Conferma Assegnazione",
        "modificaAssegnazione": "Modifica Assegnazione",
        "modificaDati": "Modifica Dati"
      },
      "schedule": {
        "empty": "Nessun evento corrisponde ai filtri",
        "loadError": "Errore durante il caricamento dello scadenziere",
        "action": {
          "assegna": "Assegna",
          "rimuovi": "Rimuovi"
        }
      },
      "genai": {
        "extracting": "Estrazione GenAI in corso…",
        "failed": "Estrazione GenAI fallita"
      },
      "view": {
        "empty": "Nessuna assegnazione disponibile per la visualizzazione",
        "openButton": "Visualizza Assegnazione"
      },
      "assign": {
        "openButton": "Assegna Monitoraggi"
      }
    },
```

> **Posizionamento**: inserire dentro `"monitoring": { … }`, immediatamente PRIMA della chiusura `}`. Mantenere virgola finale dopo l'ultimo blocco esistente.

- [ ] **Step 2**: Edit `en-GB.json` — stessa struttura, traduzioni inglesi

```json
    "cc": {
      "page": {
        "title": "Compliance Certificate Assignment",
        "documentLabel": "Document"
      },
      "bootstrap": {
        "loading": "Loading document…",
        "error": "Error loading data",
        "notFound": "Document not found for this practice",
        "invalidStatus": "The document is not in the required state"
      },
      "preview": {
        "title": "Document preview",
        "unavailable": "Preview not available,",
        "download": "download the document"
      },
      "choose": {
        "title": "How would you like to proceed?",
        "subtitle": "You can start the GenAI extraction or fill the data manually"
      },
      "mode": {
        "genai": "GenAI Assignment",
        "manual": "Manual Assignment"
      },
      "back": "Back",
      "reject": {
        "button": "Return to incomplete documentation",
        "modal": {
          "title": "Reject Document",
          "reasonLabel": "Reject reason *",
          "reasonPlaceholder": "Select a reason",
          "notes": "Notes (optional)",
          "notesPlaceholder": "Add any notes for the bank",
          "confirm": "Confirm Reject"
        },
        "reasons": {
          "incomplete": "Incomplete document",
          "format": "Wrong format",
          "signature": "Missing signature",
          "other": "Other"
        },
        "successIsp": "Reject sent to ISP with notification",
        "error": "Error rejecting the document"
      },
      "covenantNum": "Covenant #{{ n }}",
      "addCovenant": "Add Covenant",
      "concludi": "Finalize Assignment",
      "complete": {
        "success": "Assignment completed successfully",
        "error": "Error finalizing the assignment"
      },
      "slot": {
        "empty": "Add a covenant to start",
        "remove": "Remove covenant",
        "typology": "Select Covenant Type",
        "typologyPlaceholder": "Select a type",
        "value": "Monitoring Value",
        "valuePlaceholder": "Enter value",
        "referenceDate": "Reference Date",
        "associaCovenant": "Associate Covenant",
        "annulla": "Cancel",
        "confermaAssegnazione": "Confirm Assignment",
        "modificaAssegnazione": "Edit Assignment",
        "modificaDati": "Edit Data"
      },
      "schedule": {
        "empty": "No events match the filters",
        "loadError": "Error loading the schedule",
        "action": {
          "assegna": "Assign",
          "rimuovi": "Unassign"
        }
      },
      "genai": {
        "extracting": "GenAI extraction in progress…",
        "failed": "GenAI extraction failed"
      },
      "view": {
        "empty": "No assignment available to view",
        "openButton": "View Assignment"
      },
      "assign": {
        "openButton": "Assign Monitorings"
      }
    },
```

- [ ] **Step 3**: Verify JSON parse valido

```bash
cd C:/Users/davmelis/Documents/Github/ba-web && node -e "JSON.parse(require('fs').readFileSync('src/assets/i18n/it-IT.json', 'utf8')); JSON.parse(require('fs').readFileSync('src/assets/i18n/en-GB.json', 'utf8')); console.log('OK')"
```
Atteso: `OK`. Se errore parse → fix virgole/quote.

---

## Task 3.10: Final sanity Commit 3

- [ ] **Step 1**: tsc full
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx tsc --noEmit --skipLibCheck
```
Atteso: 0 errori.

- [ ] **Step 2**: Test suite componenti nuovi
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx ng test --watch=false --browsers=ChromeHeadless --code-coverage=false --include='**/cc-assignment-page/**/*.spec.ts'
```
Atteso: PASS tutti i nuovi spec (~9+8+14+22 = ~53 test).

- [ ] **Step 3**: Test suite intera monitoring (regression)
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx ng test --watch=false --browsers=ChromeHeadless --code-coverage=false --include='**/monitoring/**/*.spec.ts'
```
Atteso: tutti i test esistenti monitoring + nuovi green. **Se** un test esistente fallisce, è probabile che `getSchedule` signature change abbia rotto qualche caller — investigare.

- [ ] **Step 4**: Stage + commit
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && git add \
  src/app/service/monitoring.service.ts \
  src/app/shared/enum/monitoring-document-state.enum.ts \
  src/app/shared/model/monitoring/monitoring-cc-assignment.model.ts \
  src/app/module/agency-desk/monitoring/practice-detail/practice-detail.module.ts \
  src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/ \
  src/assets/i18n/it-IT.json \
  src/assets/i18n/en-GB.json
```

```bash
cd C:/Users/davmelis/Documents/Github/ba-web && git commit -m "feat(monitoring): add CC assignment page-based UX (bug #10 BR 4.2.3)"
```

Atteso: commit creato. **NON pushare** ora — push autorizzato solo alla fine della Wave 2 quando chiedi conferma.

---

# SECTION II — Commit 4: Wire-up document-history + remove old triggers

**Goal Commit 4:** spostare il flow "Conferma" da `confirmDocument` a `validateDocument` (status va a `DA_ASSEGNARE` invece di `COMPLETATO`). Aggiungere bottoni nav "Assegna Monitoraggi" (DA_ASSEGNARE → naviga a page CC) e "Visualizza Assegnazione" (COMPLETATO → naviga a page CC `?mode=view`). Rimuovere bottone "Assegna Monitoraggi" da `practice-detail.component.html` + relativo `@ViewChild` + handlers TS. Rimuovere `<app-monitoring-genai-extraction>` host da `document-history.component.html` + relativo state TS. Smoke E2E manuale.

**Tempo stimato:** 45-60min (escluso smoke).

---

## Task 4.1: Update `document-history.component.ts` per wire-up

**Files:**
- Modify: `src/app/module/agency-desk/monitoring/practice-detail/document-history/document-history.component.ts`

- [ ] **Step 1**: Aggiungere `Router` + `ActivatedRoute` deps

Localizza i imports in cima e ADD:
```typescript
import { ActivatedRoute, Router } from '@angular/router';
```

Modifica costruttore aggiungendo i due nuovi deps:
```typescript
  constructor(
    private monitoringService: MonitoringService,
    private authService: AuthService,
    private customTranslatePipe: CustomTranslatePipe,
    private statusDialogService: StatusDialogService,
    private router: Router,
    private route: ActivatedRoute,
  ) {}
```

- [ ] **Step 2**: Sostituire `onConfirm` per chiamare `validateDocument`

REPLACE metodo `onConfirm(documentId: number)`:
```typescript
  public onConfirm(documentId: number): void {
    this.monitoringService.validateDocument(documentId).subscribe({
      next: () => {
        this.statusDialogService.setStatus({
          status: 'OK',
          title: this.customTranslatePipe.transform('shared.success'),
          message: this.customTranslatePipe.transform('monitoring.cc.complete.success'),
          closable: true,
        });
        this.loadDocuments();
      },
      error: (err) => {
        this.statusDialogService.setStatus({
          status: 'KO',
          title: this.customTranslatePipe.transform('shared.error'),
          message: err?.error?.errorMessage || err?.message || 'Errore validazione documento',
          closable: true,
        });
      },
    });
  }
```

- [ ] **Step 3**: Aggiungere helper `goToCcAssignment` + `canGoToCc`

INSERT inside class (e.g. after `onModifyGenAi`):
```typescript
  public canAssignMonitorings(status: MonitoringDocumentStateEnum): boolean {
    return status === MonitoringDocumentStateEnum.DA_ASSEGNARE
        || status === MonitoringDocumentStateEnum.GENAI_DA_VALIDARE;
  }

  public canViewAssignment(status: MonitoringDocumentStateEnum): boolean {
    return status === MonitoringDocumentStateEnum.COMPLETATO;
  }

  public goToCcAssignment(documentId: number, viewOnly: boolean = false): void {
    const extras = viewOnly ? { queryParams: { mode: 'view' } } : {};
    this.router.navigate(['./cc', documentId], { relativeTo: this.route, ...extras });
  }
```

- [ ] **Step 4**: Update `getStatusSeverity` + `getStatusLabel` per `DA_ASSEGNARE`

REPLACE i map literals:
```typescript
  public getStatusSeverity(status: MonitoringDocumentStateEnum): 'success' | 'info' | 'warning' | 'danger' {
    const map: Record<string, 'success' | 'info' | 'warning' | 'danger'> = {
      [MonitoringDocumentStateEnum.IN_ATTESA_CONFERMA]: 'warning',
      [MonitoringDocumentStateEnum.VERIFICA_IN_CORSO]: 'info',
      [MonitoringDocumentStateEnum.GENAI_IN_CORSO]: 'info',
      [MonitoringDocumentStateEnum.GENAI_DA_VALIDARE]: 'warning',
      [MonitoringDocumentStateEnum.DA_ASSEGNARE]: 'info',
      [MonitoringDocumentStateEnum.COMPLETATO]: 'success',
    };
    return map[status] || 'info';
  }

  public getStatusLabel(status: MonitoringDocumentStateEnum): string {
    const labels: Record<string, string> = {
      [MonitoringDocumentStateEnum.IN_ATTESA_CONFERMA]: 'In Attesa',
      [MonitoringDocumentStateEnum.VERIFICA_IN_CORSO]: 'Verifica',
      [MonitoringDocumentStateEnum.GENAI_IN_CORSO]: 'GenAI',
      [MonitoringDocumentStateEnum.GENAI_DA_VALIDARE]: 'Da Validare',
      [MonitoringDocumentStateEnum.DA_ASSEGNARE]: 'Da Assegnare',
      [MonitoringDocumentStateEnum.COMPLETATO]: 'Completato',
    };
    return labels[status] || status;
  }
```

- [ ] **Step 5**: Rimuovere state GenAI dialog locale (era host di `app-monitoring-genai-extraction`)

REMOVE field declarations:
```typescript
  public showGenAiDialog: boolean = false;
  public selectedDocumentIdForGenAi: number | null = null;
  public selectedCovenantIdForGenAi: number | null = null;
  public selectedEventIdForGenAi: number | null = null;
```

REMOVE methods:
- `onStartGenAi(documentId, eventId, covenantId)` (incluso doc comment)
- `waitForGenAiCompletion(documentId)`
- `handleGenAiError(err, fallback)`
- `onModifyGenAi(documentId, eventId, covenantId)`
- `onGenAiApplied()`

REMOVE imports non più necessari:
- `MonitoringGenAiExtractionStatusEnum`
- `SpinnerComponent` (se non più usato altrove nel file — verify con grep nel TS)

> **Nota:** `canStartGenAi`/`canModifyGenAi` non sono più chiamati da HTML (rimosso). Posso lasciarli (codice morto inerte) o rimuoverli. **Rimuovere** per pulizia.

REMOVE:
```typescript
  public canStartGenAi(status: MonitoringDocumentStateEnum): boolean { … }
  public canModifyGenAi(status: MonitoringDocumentStateEnum): boolean { … }
```

---

## Task 4.2: Update `document-history.component.html` per wire-up

**Files:**
- Modify: `src/app/module/agency-desk/monitoring/practice-detail/document-history/document-history.component.html`

- [ ] **Step 1**: Sostituire i bottoni azione

LOCALIZA il blocco `*ngIf="isConsultant"` con i bottoni Conferma/Reject/GenAi. REPLACE l'intero blocco azione (riga 41-64) con:

```html
            <ng-container *ngIf="isConsultant">
              <!-- IN_ATTESA_CONFERMA: Valida (era "Conferma") + Reject -->
              <button *ngIf="doc.status === MonitoringDocumentStateEnum.IN_ATTESA_CONFERMA && !doc.rejectReason"
                pButton type="button" icon="pi pi-check"
                class="p-button-text p-button-success p-button-sm"
                [pTooltip]="'monitoring.documents.actions.confirm' | customTranslate"
                (click)="onConfirm(doc.id)">
              </button>
              <button *ngIf="doc.status === MonitoringDocumentStateEnum.IN_ATTESA_CONFERMA && !doc.rejectReason"
                pButton type="button" icon="pi pi-times"
                class="p-button-text p-button-danger p-button-sm"
                [pTooltip]="'monitoring.documents.actions.reject' | customTranslate"
                (click)="onReject(doc.id)">
              </button>

              <!-- DA_ASSEGNARE / GENAI_DA_VALIDARE: vai a page CC -->
              <button *ngIf="canAssignMonitorings(doc.status)"
                pButton type="button" icon="pi pi-file-edit"
                class="p-button-text p-button-sm"
                [pTooltip]="'monitoring.cc.assign.openButton' | customTranslate"
                (click)="goToCcAssignment(doc.id, false)">
              </button>

              <!-- COMPLETATO: visualizza assegnazione -->
              <button *ngIf="canViewAssignment(doc.status)"
                pButton type="button" icon="pi pi-eye"
                class="p-button-text p-button-sm"
                [pTooltip]="'monitoring.cc.view.openButton' | customTranslate"
                (click)="goToCcAssignment(doc.id, true)">
              </button>
            </ng-container>
```

- [ ] **Step 2**: Rimuovere host `<app-monitoring-genai-extraction>` 

REMOVE blocco finale (righe 107-113):
```html
<app-monitoring-genai-extraction
  [(visible)]="showGenAiDialog"
  [documentId]="selectedDocumentIdForGenAi"
  [covenantId]="selectedCovenantIdForGenAi"
  [eventId]="selectedEventIdForGenAi"
  (applied)="onGenAiApplied()">
</app-monitoring-genai-extraction>
```

---

## Task 4.3: Update `document-history.component.spec.ts`

**Files:**
- Modify: `src/app/module/agency-desk/monitoring/practice-detail/document-history/document-history.component.spec.ts`

- [ ] **Step 1**: Aggiungere `Router` + `ActivatedRoute` providers

Aggiungere imports:
```typescript
import { Router, ActivatedRoute } from '@angular/router';
```

Modificare `beforeEach`:
- `monitoringService = jasmine.createSpyObj('MonitoringService', [...])` → sostituire `'confirmDocument'` con `'validateDocument'`. Rimuovere `'startGenAiExtraction'`, `'getGenAiResult'`, `'applyGenAiExtraction'`.
- `monitoringService.validateDocument.and.returnValue(of({} as any))`
- Aggiungere providers:
```typescript
const routerSpy = jasmine.createSpyObj('Router', ['navigate']);
routerSpy.navigate.and.resolveTo(true);
const routeStub = { snapshot: { params: {} } };
```
e dentro `providers`:
```typescript
{ provide: Router, useValue: routerSpy },
{ provide: ActivatedRoute, useValue: routeStub },
```

- [ ] **Step 2**: Sostituire test `confirmDocument` con `validateDocument`

REPLACE describe('onConfirm') con:
```typescript
  describe('onConfirm', () => {
    it('calls validateDocument with the documentId', () => {
      component.ngOnInit();

      component.onConfirm(123);

      expect(monitoringService.validateDocument).toHaveBeenCalledWith(123);
    });

    it('reloads documents after successful validate', () => {
      monitoringService.validateDocument.and.returnValue(of({} as any));
      component.ngOnInit();
      monitoringService.getDocumentsByPractice.calls.reset();

      component.onConfirm(7);

      expect(monitoringService.getDocumentsByPractice).toHaveBeenCalledWith(1);
    });

    it('shows error dialog when validateDocument fails', () => {
      monitoringService.validateDocument.and.returnValue(throwError(() => ({ error: { errorMessage: 'BAD' } })));
      component.ngOnInit();

      component.onConfirm(7);

      expect(statusDialogService.setStatus).toHaveBeenCalled();
      const last = statusDialogService.setStatus.calls.mostRecent().args[0];
      expect(last.status).toBe('KO');
    });
  });
```

- [ ] **Step 3**: Aggiungere test per i nuovi helper

INSERT nuovo describe:
```typescript
  describe('Navigation to CC assignment page', () => {
    it('canAssignMonitorings returns true for DA_ASSEGNARE and GENAI_DA_VALIDARE', () => {
      expect(component.canAssignMonitorings(MonitoringDocumentStateEnum.DA_ASSEGNARE)).toBeTrue();
      expect(component.canAssignMonitorings(MonitoringDocumentStateEnum.GENAI_DA_VALIDARE)).toBeTrue();
      expect(component.canAssignMonitorings(MonitoringDocumentStateEnum.COMPLETATO)).toBeFalse();
    });

    it('canViewAssignment returns true only for COMPLETATO', () => {
      expect(component.canViewAssignment(MonitoringDocumentStateEnum.COMPLETATO)).toBeTrue();
      expect(component.canViewAssignment(MonitoringDocumentStateEnum.DA_ASSEGNARE)).toBeFalse();
    });

    it('goToCcAssignment navigates to ./cc/:id when not viewOnly', () => {
      const router = TestBed.inject(Router) as jasmine.SpyObj<Router>;
      const route = TestBed.inject(ActivatedRoute);

      component.goToCcAssignment(99, false);

      expect(router.navigate).toHaveBeenCalledWith(['./cc', 99], { relativeTo: route });
    });

    it('goToCcAssignment navigates with mode=view queryParam when viewOnly', () => {
      const router = TestBed.inject(Router) as jasmine.SpyObj<Router>;
      const route = TestBed.inject(ActivatedRoute);

      component.goToCcAssignment(42, true);

      expect(router.navigate).toHaveBeenCalledWith(['./cc', 42], { relativeTo: route, queryParams: { mode: 'view' } });
    });
  });
```

- [ ] **Step 4**: Rimuovere tutto il blocco describe('GenAI extraction (T-018)') — i metodi sono stati rimossi.

DELETE tutti i `describe('GenAI extraction (T-018)', …)` e relativi `it`.

- [ ] **Step 5**: Aggiornare mapping enum nei test `getStatusSeverity/Label`

Aggiungere:
```typescript
    it('should map DA_ASSEGNARE status to info severity', () => {
      expect(component.getStatusSeverity(MonitoringDocumentStateEnum.DA_ASSEGNARE)).toBe('info');
    });
```

- [ ] **Step 6**: Run test spec
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx ng test --watch=false --browsers=ChromeHeadless --code-coverage=false --include='**/document-history.component.spec.ts'
```
Atteso: PASS.

---

## Task 4.4: Cleanup `practice-detail.component.{ts,html}`

**Files:**
- Modify: `src/app/module/agency-desk/monitoring/practice-detail/practice-detail.component.ts`
- Modify: `src/app/module/agency-desk/monitoring/practice-detail/practice-detail.component.html`

- [ ] **Step 1**: TS — Rimuovere import `CcAssignmentComponent`

REMOVE riga:
```typescript
import { CcAssignmentComponent } from './cc-assignment/cc-assignment.component';
```

- [ ] **Step 2**: TS — Rimuovere `@ViewChild ccAssignment`

REMOVE riga:
```typescript
  @ViewChild('ccAssignment') ccAssignment!: CcAssignmentComponent;
```

- [ ] **Step 3**: TS — Rimuovere handler `openCcAssignment` e `onCcAssignmentComplete`

REMOVE:
```typescript
  public openCcAssignment(): void {
    this.ccAssignment?.open();
  }

  public onCcAssignmentComplete(): void {
    this.loadData();
    this.docHistory?.loadDocuments();
  }
```

- [ ] **Step 4**: HTML — Rimuovere bottone "Assegna Monitoraggi" + `<app-cc-assignment>` host

REMOVE blocco (righe ~191-207):
```html
        <!-- CC Assignment (T-017 — Deloitte only) -->
        <div class="mt-4" *appHasRole="RoleEnum.Consultant">
          <button pButton
            type="button"
            icon="pi pi-file-edit"
            [label]="'monitoring.deloitte.ccAssignment.openButton' | customTranslate"
            class="p-button-outlined"
            (click)="openCcAssignment()">
          </button>
        </div>

        <app-cc-assignment
          #ccAssignment
          [practiceId]="practiceId"
          [covenants]="covenants"
          (onAssignmentComplete)="onCcAssignmentComplete()">
        </app-cc-assignment>
```

- [ ] **Step 5**: Verify tsc + practice-detail spec

```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx tsc --noEmit --skipLibCheck
cd C:/Users/davmelis/Documents/Github/ba-web && npx ng test --watch=false --browsers=ChromeHeadless --code-coverage=false --include='**/practice-detail.component.spec.ts'
```
Atteso: tsc 0 errori; test verde (se il spec faceva riferimento a `CcAssignmentComponent`, aggiornare di conseguenza con `NO_ERRORS_SCHEMA`).

> **Se** `practice-detail.component.spec.ts` testa esplicitamente i metodi rimossi (`openCcAssignment`, `onCcAssignmentComplete`), RIMUOVERE quei test.

---

## Task 4.5: Smoke E2E manuale (BR 4.2.3 spec §8.3)

> **⚠️ Avvio ambiente prima di Commit 4 = bottoni Conferma rotti** (BE chiama `/validate`, FE chiama ancora `/confirm` se in mezzo a Commit 3). Avvia ambiente SOLO dopo aver committato e completato Task 4.4.

- [ ] **Step 1**: Avvia BE
```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw spring-boot:run -DskipTests -Dspring-boot.run.profiles=local
```
Run in background; check health:
```bash
curl -s http://localhost:8091/actuator/health
```

- [ ] **Step 2**: Avvia FE in nuova shell
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npm run start
```

- [ ] **Step 3**: Login utente Deloitte CONSULTANT
- URL: `http://localhost:4200`
- Email: `davide94x@gmail.com`
- TOTP via Authenticator

- [ ] **Step 4**: Test Manual Flow (BR 4.2.3 righe 652-716)
1. Naviga a una pratica monitoraggio
2. Upload documento CC (mailing list / dashboard upload)
3. Click bottone "✓" (Valida) sul documento in `IN_ATTESA_CONFERMA`
4. Verify: status diventa `DA_ASSEGNARE`, status dialog "Assegnazione completata con successo"
5. Click bottone "📝" (Assegna Monitoraggi) sul documento `DA_ASSEGNARE`
6. Verify: naviga a `monitoring/practice/{id}/cc/{documentId}`
7. Verify: preview PDF visibile
8. Click "Assegnazione Manuale" → mode CHOOSE → MANUAL
9. Verify: 1 slot FILLING appare
10. Seleziona tipologia covenant da dropdown
11. Inserisci valore monitoraggio
12. Inserisci data di riferimento
13. Click "Associa Covenant"
14. Verify: scadenziere appare con eventi (filtrati per status target o referenceDate ±15gg)
15. Click "Assegna" su una riga
16. Verify: phase passa a EVENT_SELECTED, riga evidenziata
17. Click "Conferma Assegnazione"
18. Verify: phase passa a CONFIRMED, scadenziere collapsed, action "Modifica Assegnazione"
19. Click "Aggiungi Covenant" → "Aggiungi Covenant #2"
20. Verify: nuovo slot index=2 appare
21. Annulla slot #2 con cestino (slot rimosso, slot #1 resta)
22. Click "Concludi Assegnazione"
23. Verify: success dialog "Assegnazione completata con successo", navigate back a practice detail
24. Verify: documento ora in stato `COMPLETATO`
25. Verify: dashboard donut/contatori M1 ricalcolati

- [ ] **Step 5**: Test GenAI Flow (BR 4.2.3 righe 718-744)
26. Doc in stato `IN_ATTESA_CONFERMA` → click bottone GenAI (mantieni il vecchio trigger? — **No, è stato rimosso**)
> **GAP**: il bottone "Valida con GenAI" (`pi-bolt`) era nel document-history e l'abbiamo rimosso per pulizia. Il GenAI flow è triggerato lato BE quando il sistema applica la batch GenAI, oppure manualmente via shell endpoint `POST /documents/{id}/genai/extract`. Verificare con utente se serve mantenere un trigger UI o se è BE-driven.

**Decisione**: per ora il GenAI flow è testabile solo da DA-ASSEGNARE post-batch GenAI (status = `GENAI_DA_VALIDARE`). Se serve trigger UI manuale, riaggiungere il bottone `pi-bolt` nel document-history puntando a `monitoringService.startGenAiExtraction(id).subscribe(() => goToCcAssignment(id, false))`.

→ Per smoke step 26+: scenario "documento già in GENAI_DA_VALIDARE" — naviga manualmente al doc, click "Assegna Monitoraggi" → mode auto = GENAI, extraction load, slot prefilled, etc.

- [ ] **Step 6**: Test Bad Flow Reject (BR 728)
- Doc in `GENAI_DA_VALIDARE`, in page CC
- Click "Ritorna a documentazione incompleta" (header destra)
- Reject modal apre
- Seleziona motivo "Documento incompleto"
- Aggiungi note "esempio note"
- Click "Conferma Rifiuto"
- Verify: status doc torna a `IN_ATTESA_CONFERMA`, `rejectReason = "INCOMPLETE: esempio note"`, success dialog "Rifiuto inviato a ISP con notifica"

- [ ] **Step 7**: Test View Mode (BR 4.2.4, scope cut)
- Doc in `COMPLETATO`, click "👁" (Visualizza Assegnazione)
- Verify: page apre con `?mode=view`, slot readonly, scadenziere collapsed con 1 riga (limite legacy)

- [ ] **Step 8**: Status report manuale (1 frase per ogni test)
Compila checklist mentale e annota eventuali problemi.

---

## Task 4.6: Commit 4

- [ ] **Step 1**: Stage + commit
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && git add \
  src/app/module/agency-desk/monitoring/practice-detail/document-history/document-history.component.ts \
  src/app/module/agency-desk/monitoring/practice-detail/document-history/document-history.component.html \
  src/app/module/agency-desk/monitoring/practice-detail/document-history/document-history.component.spec.ts \
  src/app/module/agency-desk/monitoring/practice-detail/practice-detail.component.ts \
  src/app/module/agency-desk/monitoring/practice-detail/practice-detail.component.html
```

```bash
cd C:/Users/davmelis/Documents/Github/ba-web && git commit -m "feat(monitoring): wire CC assignment page in document-history + remove old dialog triggers"
```

Atteso: commit creato.

---

# SECTION III — Commit 5: Cleanup vecchi componenti (post-smoke)

**Goal Commit 5:** DELETE `cc-assignment/` + `monitoring-genai-extraction/` folder. Rimuovere declarations + imports nel module. Verify build green. Eseguire SOLO dopo Smoke verde Commit 4 (rollback safety).

**Tempo stimato:** 15-20min.

---

## Task 5.1: Delete folders

- [ ] **Step 1**: DELETE folder `cc-assignment/`
```bash
rm -r C:/Users/davmelis/Documents/Github/ba-web/src/app/module/agency-desk/monitoring/practice-detail/cc-assignment
```

- [ ] **Step 2**: DELETE folder `monitoring-genai-extraction/`
```bash
rm -r C:/Users/davmelis/Documents/Github/ba-web/src/app/module/agency-desk/monitoring/practice-detail/monitoring-genai-extraction
```

---

## Task 5.2: Update `practice-detail.module.ts`

**Files:**
- Modify: `src/app/module/agency-desk/monitoring/practice-detail/practice-detail.module.ts`

- [ ] **Step 1**: Rimuovere import `CcAssignmentComponent` + `MonitoringGenAiExtractionModule`

REMOVE:
```typescript
import { CcAssignmentComponent } from './cc-assignment/cc-assignment.component';
import { MonitoringGenAiExtractionModule } from './monitoring-genai-extraction/monitoring-genai-extraction.module';
```

- [ ] **Step 2**: Rimuovere `CcAssignmentComponent` da `declarations`

LOCALIZA + REMOVE riga `CcAssignmentComponent,` dentro `declarations: [...]`.

- [ ] **Step 3**: Rimuovere `MonitoringGenAiExtractionModule` da `imports`

LOCALIZA + REMOVE riga `MonitoringGenAiExtractionModule,` dentro `imports: [...]`.

---

## Task 5.3: Verify + commit

- [ ] **Step 1**: tsc full
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx tsc --noEmit --skipLibCheck
```
Atteso: 0 errori.

- [ ] **Step 2**: Test full suite (regression)
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npx ng test --watch=false --browsers=ChromeHeadless --code-coverage=false
```
Atteso: ALL PASS.

- [ ] **Step 3**: Verify build production (opzionale ma raccomandato)
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && npm run build
```
Atteso: BUILD SUCCESS.

- [ ] **Step 4**: Stage + commit
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && git add -A \
  src/app/module/agency-desk/monitoring/practice-detail/cc-assignment \
  src/app/module/agency-desk/monitoring/practice-detail/monitoring-genai-extraction \
  src/app/module/agency-desk/monitoring/practice-detail/practice-detail.module.ts
```

```bash
cd C:/Users/davmelis/Documents/Github/ba-web && git commit -m "chore(monitoring): remove deprecated CcAssignmentComponent + MonitoringGenAiExtractionComponent (replaced by CC assignment page)"
```

---

## Task 5.4: Push autorizzato (chiedere conferma)

- [ ] **Step 1**: Chiedere conferma a Davide PRIMA del push.

Output user-facing:
> "FE Wave 2 completata: Commit 3 + Commit 4 + Commit 5 su `feature/monitoring-integration-uat`. Smoke E2E verde [o riportare risultato smoke]. Pronto a pushare i 3 commit FE a origin?"

Wait for OK.

- [ ] **Step 2**: Push
```bash
cd C:/Users/davmelis/Documents/Github/ba-web && git push
```

---

# Self-Review

### 1. Spec coverage check

| Spec section | Covered by task |
|---|---|
| §3 Architecture & routing | ✅ Task 3.8 (route declaration in practice-detail.module) |
| §4 Component breakdown (Page + Slot + ScheduleTable + RejectDialog) | ✅ Task 3.4-3.7 |
| §5 BE endpoints | (BE già done Wave 1) — wired in Task 3.2 (service) |
| §6 Data flow FE (bootstrap, manual, GenAI, reject, view) | ✅ Task 3.7 (page orchestrator) |
| §7 State management + error handling | ✅ Task 3.7 (destroy$, immutability, error dialog ovunque) |
| §7.4 i18n keys | ✅ Task 3.9 |
| §8.1 Unit test FE | ✅ Task 3.4-3.7 (4 spec con ~50 test totali) |
| §8.3 Smoke E2E manuale | ✅ Task 4.5 |
| §9 Migration order (Commit 3 → 4 → 5) | ✅ Section I → II → III, rollback safety preservata |
| §10 Constraints non negoziabili | ✅ RxJS + destroy$ pattern; spread immutability; statusDialogService ogni error branch; no Co-Author; no auto-push |

### 2. Placeholder scan
- Solo nota su trigger UI GenAI (Task 4.5 Step 5): scenario test-only, da chiarire con utente se serve mantenere bottone `pi-bolt`. Non blocking per Commit 4.
- "rollback safety": preserve old code fino a Commit 5 — corretto.

### 3. Type consistency
- `CcSlotState` definito in `monitoring-cc-assignment.model.ts` (Task 3.3) e consumato da Task 3.6, 3.7. Field names coerenti.
- `CcCompleteAssignmentRequest` consumato da Task 3.2 (service) e Task 3.7 (page). Payload mapping in `onConcludi` matches BE record `MonitoringCcCompleteAssignmentRequestDTO`.
- `MonitoringDocumentStateEnum.DA_ASSEGNARE` consumato da Task 3.7, 4.1, 4.3. Aggiunto in Task 3.1.

### 4. Build sequence
- Commit 3: tsc green, test green ✅
- Commit 4: smoke E2E verde ✅
- Commit 5: tsc green, test green, build prod green ✅

### 5. Risks
- **Trigger UI GenAI mancante** (vedi Task 4.5 Step 5): se utente vuole testare GenAI flow da zero, serve riaggiungere bottone start. → Chiarire prima del commit 4 finale.
- **trackBy slot UID**: il `slotTrackBy` evita rebuild *ngFor su immutability spread. Mancanza causerebbe input perdita focus su typing.
- **Schedule statuses query string**: BE accetta `statuses` come `List<MonitoringEventStateEnum>`. FE manda string CSV `SCADUTO,FUORI_SOGLIA,IN_SCADENZA`. Spring decodifica CSV → List automatica. Se 400, switch a multi-param: `?statuses=SCADUTO&statuses=FUORI_SOGLIA&statuses=IN_SCADENZA` (richiede helper `compactParams` modifica per array → multi-param HTTP).
- **Preview PDF auth**: il `<object>` GET `/download` deve riusare la sessione cookie. Verificare CORS/credentials in `App.origin`. Se preview vuota, fallback link c'è.

---

# Execution Handoff

**Plan completo. Saved to** `plans/in-progress/2026-05-04_monitoring/implementation/2026-05-29-bug10-fe-wave2-plan.md`.

**Strategia execution**:
- Eseguire ogni Task TDD in modalità subagent-driven o inline (scope contenuto, ~3 ore totali).
- Confermare a Davide PRIMA del push finale.
- Stop nei punti di rischio: tra Commit 3 e Commit 4 NON avviare l'ambiente locale (flow Conferma rotto).

**Bug closure**: Wave 2 completata → bug #10 chiuso ufficialmente. Update bug list nel handoff finale.
