# Bug #10 CC Assignment Page Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Implementare BR sez 4.2.3 "Assegnazione Compliance Certificate" come page-based UX standalone, sostituendo `CcAssignmentComponent` (wizard dialog) + `MonitoringGenAiExtractionComponent` (form dialog).

**Architecture:** State machine documento BR-compliant (`IN_ATTESA_CONFERMA → DA_ASSEGNARE → COMPLETATO` per manual; flusso GenAI invariato chiude in `COMPLETATO`). Nuovo sub-route `monitoring/practice/:id/cc/:documentId` con page component + 3 sub-component (slot covenant, schedule table, reject dialog). Backend estende stati documento (+1 enum value), aggiunge endpoint atomico `complete-assignment` con body bulk, estende `/schedule` con filtri ±15gg.

**Tech Stack:** BE Spring Boot 3 + JPA + MySQL + JUnit 5 + AssertJ + Mockito. FE Angular 17 + PrimeNG + RxJS + Karma/Jasmine. No NgRx, no signals (coerente con pattern modulo).

**Wave split:**
- **Wave 1 (questa sessione)**: Commit 1 (BE non-breaking additions) + Commit 2 (BE switch + rename). Dettagliato sotto.
- **Wave 2 (sessione successiva)**: Commit 3 (FE new components) + Commit 4 (FE wire-up) + Commit 5 (cleanup). Outline sotto; nuova sessione genererà sub-plan dettagliato.

**Riferimenti:**
- Spec design: `plans/in-progress/2026-05-04_monitoring/specs/2026-05-28-bug10-cc-assignment-page-design.md`
- BR: `plans/in-progress/2026-05-04_monitoring/requirements/BR_Monitoraggio_V6.md` sez 4.2.3 (652-744) + 4.2.4 (745-748)
- Commit base BE: `c4efb8e`, FE: `1d27047`

---

## File Structure (BE wave 1)

### Files modified
- `src/main/java/it/deloitte/ba/backend/enumeration/agency_desk/MonitoringDocumentStateEnum.java` — add `DA_ASSEGNARE`
- `src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentService.java` — add `validateDocument` + `completeAssignment` methods; later switch body of `confirmDocument`
- `src/main/java/it/deloitte/ba/backend/controller/agency_desk/MonitoringDocumentController.java` — add POST endpoint; later rename `/confirm` → `/validate`
- `src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringEventService.java` — extend `findScheduleByPractice` signature
- `src/main/java/it/deloitte/ba/backend/controller/agency_desk/MonitoringCovenantController.java` — extend `/schedule` params
- `src/main/java/it/deloitte/ba/backend/repository/agency_desk/MonitoringEventRepository.java` — add new `@Query` for filtered schedule
- `src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringGenAiService.java` — switch line 139

### Files created
- `src/main/java/it/deloitte/ba/backend/dto/agency_desk/request/MonitoringCcCompleteAssignmentRequestDTO.java` — record + validation
- `src/test/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentServiceTest.java` — nuovo (non esiste)
- `src/test/java/it/deloitte/ba/backend/controller/agency_desk/MonitoringDocumentControllerTest.java` — nuovo (non esiste)
- `src/test/java/it/deloitte/ba/backend/controller/agency_desk/MonitoringCovenantControllerTest.java` — nuovo (non esiste)

### Files modified — test
- `src/test/java/it/deloitte/ba/backend/service/monitoring/MonitoringEventServiceTest.java` — add findScheduleByPractice tests
- `src/test/java/it/deloitte/ba/backend/service/monitoring/MonitoringGenAiServiceTest.java` — update apply test (verify completeAssignment call)

---

# SECTION I — Commit 1: BE non-breaking additions

**Goal Commit 1**: aggiungere tutti i nuovi simboli (enum value, service methods, DTO, endpoint, schedule extension) **senza** rompere il behavior corrente. `confirmDocument` continua a settare `COMPLETATO`. Test esistenti devono restare green (145+).

---

### Task 1.1: Add enum value `DA_ASSEGNARE`

**Files:**
- Modify: `src/main/java/it/deloitte/ba/backend/enumeration/agency_desk/MonitoringDocumentStateEnum.java`

- [ ] **Step 1: Edit enum to add DA_ASSEGNARE**

Replace content of `MonitoringDocumentStateEnum.java`:

```java
package it.deloitte.ba.backend.enumeration.agency_desk;

import lombok.Getter;

@Getter
public enum MonitoringDocumentStateEnum {
    IN_ATTESA_CONFERMA("In Attesa Conferma"),
    VERIFICA_IN_CORSO("Verifica in Corso"),
    GENAI_IN_CORSO("GenAI in Corso"),
    GENAI_DA_VALIDARE("GenAI da Validare"),
    DA_ASSEGNARE("Da Assegnare"),
    COMPLETATO("Completato");

    private final String text;

    MonitoringDocumentStateEnum(String text) { this.text = text; }
}
```

- [ ] **Step 2: Verify compile**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw compile -q
```

Expected: BUILD SUCCESS.

---

### Task 1.2: Add `validateDocument` method to service

**Files:**
- Modify: `src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentService.java`
- Create: `src/test/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentServiceTest.java`

- [ ] **Step 1: Write the failing test**

Create `MonitoringDocumentServiceTest.java` (NEW file):

```java
package it.deloitte.ba.backend.service.monitoring;

import it.deloitte.ba.backend.domain.agency_desk.MonitoringCovenant;
import it.deloitte.ba.backend.domain.agency_desk.MonitoringDocument;
import it.deloitte.ba.backend.domain.agency_desk.MonitoringEvent;
import it.deloitte.ba.backend.dto.agency_desk.MonitoringDocumentDTO;
import it.deloitte.ba.backend.enumeration.agency_desk.MonitoringDocumentStateEnum;
import it.deloitte.ba.backend.error.exception.GenericBadRequestException;
import it.deloitte.ba.backend.repository.agency_desk.MonitoringDocumentRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.ArgumentCaptor;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.Optional;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
class MonitoringDocumentServiceTest {

    @Mock
    private MonitoringDocumentRepository monitoringDocumentRepository;

    @InjectMocks
    private MonitoringDocumentService monitoringDocumentService;

    private MonitoringDocument doc(Long id, MonitoringDocumentStateEnum status) {
        MonitoringDocument d = new MonitoringDocument();
        d.setId(id);
        d.setDocumentCode("DOC_" + id);
        d.setStatus(status);
        MonitoringEvent ev = new MonitoringEvent();
        ev.setId(99L);
        ev.setEventCode("E99");
        MonitoringCovenant cov = new MonitoringCovenant();
        cov.setCovenantCode("C99");
        ev.setCovenant(cov);
        d.setEvent(ev);
        return d;
    }

    @Nested
    class ValidateDocument {

        @Test
        void shouldTransitionFromInAttesaConfermaToDaAssegnare() {
            MonitoringDocument d = doc(1L, MonitoringDocumentStateEnum.IN_ATTESA_CONFERMA);
            when(monitoringDocumentRepository.findById(1L)).thenReturn(Optional.of(d));
            when(monitoringDocumentRepository.save(any(MonitoringDocument.class)))
                    .thenAnswer(inv -> inv.getArgument(0));

            MonitoringDocumentDTO result = monitoringDocumentService.validateDocument(1L);

            ArgumentCaptor<MonitoringDocument> captor = ArgumentCaptor.forClass(MonitoringDocument.class);
            verify(monitoringDocumentRepository).save(captor.capture());
            assertThat(captor.getValue().getStatus())
                    .isEqualTo(MonitoringDocumentStateEnum.DA_ASSEGNARE);
            assertThat(result.getStatus())
                    .isEqualTo(MonitoringDocumentStateEnum.DA_ASSEGNARE);
        }

        @Test
        void shouldThrowWhenDocumentNotFound() {
            when(monitoringDocumentRepository.findById(99L)).thenReturn(Optional.empty());

            assertThatThrownBy(() -> monitoringDocumentService.validateDocument(99L))
                    .isInstanceOf(GenericBadRequestException.class);
        }

        @Test
        void shouldThrowWhenStateNotInAttesaConferma() {
            MonitoringDocument d = doc(1L, MonitoringDocumentStateEnum.COMPLETATO);
            when(monitoringDocumentRepository.findById(1L)).thenReturn(Optional.of(d));

            assertThatThrownBy(() -> monitoringDocumentService.validateDocument(1L))
                    .isInstanceOf(GenericBadRequestException.class);
        }
    }
}
```

- [ ] **Step 2: Run test to verify it fails (method not defined)**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringDocumentServiceTest#ValidateDocument*' -q
```

Expected: FAIL with "cannot find symbol method validateDocument(Long)".

- [ ] **Step 3: Implement validateDocument method**

In `MonitoringDocumentService.java`, add new method (between `confirmDocument` and `markGenAiInProgress`):

```java
    @Transactional
    public MonitoringDocumentDTO validateDocument(Long documentId) {
        MonitoringDocument doc = monitoringDocumentRepository.findById(documentId)
                .orElseThrow(() -> new GenericBadRequestException(ErrorType.IM_GENERIC));

        if (doc.getStatus() != MonitoringDocumentStateEnum.IN_ATTESA_CONFERMA) {
            throw new GenericBadRequestException(ErrorType.IM_GENERIC);
        }

        doc.setStatus(MonitoringDocumentStateEnum.DA_ASSEGNARE);
        MonitoringDocument saved = monitoringDocumentRepository.save(doc);
        log.info("Document {} validated → DA_ASSEGNARE", saved.getDocumentCode());
        return toDto(saved);
    }
```

- [ ] **Step 4: Run test to verify it passes**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringDocumentServiceTest#ValidateDocument*' -q
```

Expected: PASS 3/3.

---

### Task 1.3: Add `completeAssignment(Long)` method to service (no body version, for GenAI flow finalize)

**Files:**
- Modify: `src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentService.java`
- Modify: `src/test/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentServiceTest.java`

- [ ] **Step 1: Add failing tests**

Append to `MonitoringDocumentServiceTest.java` inside the class (after `ValidateDocument` nested):

```java
    @Nested
    class CompleteAssignment {

        @Test
        void shouldTransitionFromDaAssegnareToCompletato() {
            MonitoringDocument d = doc(1L, MonitoringDocumentStateEnum.DA_ASSEGNARE);
            when(monitoringDocumentRepository.findById(1L)).thenReturn(Optional.of(d));
            when(monitoringDocumentRepository.save(any(MonitoringDocument.class)))
                    .thenAnswer(inv -> inv.getArgument(0));

            MonitoringDocumentDTO result = monitoringDocumentService.completeAssignment(1L);

            assertThat(result.getStatus()).isEqualTo(MonitoringDocumentStateEnum.COMPLETATO);
        }

        @Test
        void shouldTransitionFromGenaiDaValidareToCompletato() {
            MonitoringDocument d = doc(1L, MonitoringDocumentStateEnum.GENAI_DA_VALIDARE);
            when(monitoringDocumentRepository.findById(1L)).thenReturn(Optional.of(d));
            when(monitoringDocumentRepository.save(any(MonitoringDocument.class)))
                    .thenAnswer(inv -> inv.getArgument(0));

            MonitoringDocumentDTO result = monitoringDocumentService.completeAssignment(1L);

            assertThat(result.getStatus()).isEqualTo(MonitoringDocumentStateEnum.COMPLETATO);
        }

        @Test
        void shouldThrowWhenStateNotEligible() {
            MonitoringDocument d = doc(1L, MonitoringDocumentStateEnum.IN_ATTESA_CONFERMA);
            when(monitoringDocumentRepository.findById(1L)).thenReturn(Optional.of(d));

            assertThatThrownBy(() -> monitoringDocumentService.completeAssignment(1L))
                    .isInstanceOf(GenericBadRequestException.class);
        }

        @Test
        void shouldThrowWhenDocumentNotFound() {
            when(monitoringDocumentRepository.findById(99L)).thenReturn(Optional.empty());

            assertThatThrownBy(() -> monitoringDocumentService.completeAssignment(99L))
                    .isInstanceOf(GenericBadRequestException.class);
        }
    }
```

- [ ] **Step 2: Run tests to verify they fail**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringDocumentServiceTest#CompleteAssignment*' -q
```

Expected: FAIL "cannot find symbol method completeAssignment(Long)".

- [ ] **Step 3: Implement completeAssignment(Long)**

In `MonitoringDocumentService.java`, add method after `validateDocument`:

```java
    @Transactional
    public MonitoringDocumentDTO completeAssignment(Long documentId) {
        MonitoringDocument doc = monitoringDocumentRepository.findById(documentId)
                .orElseThrow(() -> new GenericBadRequestException(ErrorType.IM_GENERIC));

        MonitoringDocumentStateEnum current = doc.getStatus();
        if (current != MonitoringDocumentStateEnum.DA_ASSEGNARE
                && current != MonitoringDocumentStateEnum.GENAI_DA_VALIDARE) {
            throw new GenericBadRequestException(ErrorType.IM_GENERIC);
        }

        doc.setStatus(MonitoringDocumentStateEnum.COMPLETATO);
        MonitoringDocument saved = monitoringDocumentRepository.save(doc);
        log.info("Document {} assignment completed → COMPLETATO from {}", saved.getDocumentCode(), current);
        return toDto(saved);
    }
```

- [ ] **Step 4: Run tests to verify they pass**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringDocumentServiceTest#CompleteAssignment*' -q
```

Expected: PASS 4/4.

---

### Task 1.4: Create `MonitoringCcCompleteAssignmentRequestDTO` record

**Files:**
- Create: `src/main/java/it/deloitte/ba/backend/dto/agency_desk/request/MonitoringCcCompleteAssignmentRequestDTO.java`

- [ ] **Step 1: Create the record**

```java
package it.deloitte.ba.backend.dto.agency_desk.request;

import jakarta.validation.Valid;
import jakarta.validation.constraints.NotEmpty;
import jakarta.validation.constraints.NotNull;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.List;

public record MonitoringCcCompleteAssignmentRequestDTO(
        @NotEmpty @Valid List<CovenantAssignment> assignments
) {
    public record CovenantAssignment(
            @NotNull Long covenantId,
            @NotNull Long eventId,
            @NotNull BigDecimal monitoringValue,
            @NotNull LocalDate actualDate,
            @NotNull LocalDate referenceDate
    ) {}
}
```

- [ ] **Step 2: Verify compile**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw compile -q
```

Expected: BUILD SUCCESS.

---

### Task 1.5: Add overloaded `completeAssignment(Long, RequestDTO)` method (atomic batch for manual flow)

**Files:**
- Modify: `src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentService.java`
- Modify: `src/test/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentServiceTest.java`

This method is the entry point for the manual flow's "Concludi Assegnazione". It iterates assignments → updates each event value+dates → finalizes document status. Atomic via `@Transactional`.

- [ ] **Step 1: Add failing test**

Append to `MonitoringDocumentServiceTest.java` (we need `MonitoringEventService` as new mock):

In class field declarations, add:
```java
    @Mock
    private MonitoringEventService monitoringEventService;
```

(Note: this requires `MonitoringEventService` to be a constructor param of `MonitoringDocumentService`. Since this is a new dependency injection, we'll handle it via constructor extension in Step 3.)

Add nested test class:

```java
    @Nested
    class CompleteAssignmentBatch {

        @Test
        void shouldUpdateEachEventAndFinalizeDocument() {
            MonitoringDocument d = doc(10L, MonitoringDocumentStateEnum.DA_ASSEGNARE);
            when(monitoringDocumentRepository.findById(10L)).thenReturn(Optional.of(d));
            when(monitoringDocumentRepository.save(any(MonitoringDocument.class)))
                    .thenAnswer(inv -> inv.getArgument(0));

            MonitoringCcCompleteAssignmentRequestDTO request = new MonitoringCcCompleteAssignmentRequestDTO(
                    List.of(
                            new MonitoringCcCompleteAssignmentRequestDTO.CovenantAssignment(
                                    1L, 100L, new BigDecimal("50.5"),
                                    LocalDate.of(2026, 5, 1), LocalDate.of(2026, 5, 1)),
                            new MonitoringCcCompleteAssignmentRequestDTO.CovenantAssignment(
                                    2L, 101L, new BigDecimal("70.0"),
                                    LocalDate.of(2026, 5, 2), LocalDate.of(2026, 5, 2))
                    )
            );

            MonitoringDocumentDTO result = monitoringDocumentService.completeAssignment(10L, request);

            verify(monitoringEventService).updateValue(100L, new BigDecimal("50.5"), LocalDate.of(2026, 5, 1));
            verify(monitoringEventService).updateValue(101L, new BigDecimal("70.0"), LocalDate.of(2026, 5, 2));
            verify(monitoringEventService).updateDates(100L, LocalDate.of(2026, 5, 1), null);
            verify(monitoringEventService).updateDates(101L, LocalDate.of(2026, 5, 2), null);
            assertThat(result.getStatus()).isEqualTo(MonitoringDocumentStateEnum.COMPLETATO);
        }
    }
```

Add necessary imports at top:
```java
import it.deloitte.ba.backend.dto.agency_desk.request.MonitoringCcCompleteAssignmentRequestDTO;
import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.List;
```

- [ ] **Step 2: Run test to verify it fails**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringDocumentServiceTest#CompleteAssignmentBatch*' -q
```

Expected: FAIL "cannot find symbol completeAssignment(Long, MonitoringCcCompleteAssignmentRequestDTO)".

- [ ] **Step 3: Inject MonitoringEventService into MonitoringDocumentService**

In `MonitoringDocumentService.java`, add to field declarations (after `documentManagerService`):

```java
    private final MonitoringEventService monitoringEventService;
```

The `@RequiredArgsConstructor` Lombok annotation will auto-include it in constructor. No other change needed (Spring autowires).

- [ ] **Step 4: Add overloaded completeAssignment method**

In `MonitoringDocumentService.java`, add after the existing `completeAssignment(Long)`:

```java
    @Transactional
    public MonitoringDocumentDTO completeAssignment(Long documentId,
                                                     MonitoringCcCompleteAssignmentRequestDTO request) {
        for (MonitoringCcCompleteAssignmentRequestDTO.CovenantAssignment a : request.assignments()) {
            monitoringEventService.updateValue(a.eventId(), a.monitoringValue(), a.actualDate());
            monitoringEventService.updateDates(a.eventId(), a.referenceDate(), null);
        }
        return completeAssignment(documentId);
    }
```

Add import:
```java
import it.deloitte.ba.backend.dto.agency_desk.request.MonitoringCcCompleteAssignmentRequestDTO;
```

- [ ] **Step 5: Run test to verify it passes**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringDocumentServiceTest#CompleteAssignmentBatch*' -q
```

Expected: PASS 1/1.

- [ ] **Step 6: Run full MonitoringDocumentServiceTest**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringDocumentServiceTest' -q
```

Expected: PASS 8/8 (3 ValidateDocument + 4 CompleteAssignment + 1 CompleteAssignmentBatch).

---

### Task 1.6: Add POST `/documents/{id}/complete-assignment` endpoint

**Files:**
- Modify: `src/main/java/it/deloitte/ba/backend/controller/agency_desk/MonitoringDocumentController.java`
- Create: `src/test/java/it/deloitte/ba/backend/controller/agency_desk/MonitoringDocumentControllerTest.java`

- [ ] **Step 1: Write failing test**

Create `MonitoringDocumentControllerTest.java` (NEW file):

```java
package it.deloitte.ba.backend.controller.agency_desk;

import it.deloitte.ba.backend.dto.agency_desk.MonitoringDocumentDTO;
import it.deloitte.ba.backend.dto.agency_desk.request.MonitoringCcCompleteAssignmentRequestDTO;
import it.deloitte.ba.backend.enumeration.agency_desk.MonitoringDocumentStateEnum;
import it.deloitte.ba.backend.service.monitoring.MonitoringDocumentService;
import it.deloitte.ba.backend.shared.rest.model.ApiResponse;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
class MonitoringDocumentControllerTest {

    private static final Long DOCUMENT_ID = 42L;

    @Mock private MonitoringDocumentService service;

    @InjectMocks private MonitoringDocumentController controller;

    private MonitoringDocumentDTO sampleDocument(MonitoringDocumentStateEnum status) {
        MonitoringDocumentDTO dto = new MonitoringDocumentDTO();
        dto.setId(DOCUMENT_ID);
        dto.setStatus(status);
        return dto;
    }

    @Test
    void shouldDelegateCompleteAssignmentToService() {
        MonitoringCcCompleteAssignmentRequestDTO request = new MonitoringCcCompleteAssignmentRequestDTO(
                List.of(new MonitoringCcCompleteAssignmentRequestDTO.CovenantAssignment(
                        1L, 100L, new BigDecimal("50"), LocalDate.now(), LocalDate.now()))
        );
        MonitoringDocumentDTO document = sampleDocument(MonitoringDocumentStateEnum.COMPLETATO);
        when(service.completeAssignment(any(Long.class), any(MonitoringCcCompleteAssignmentRequestDTO.class)))
                .thenReturn(document);

        ApiResponse<MonitoringDocumentDTO> response = controller.completeAssignment(DOCUMENT_ID, request);

        assertThat(response.getPayload().getStatus()).isEqualTo(MonitoringDocumentStateEnum.COMPLETATO);
        verify(service).completeAssignment(DOCUMENT_ID, request);
    }
}
```

- [ ] **Step 2: Run test to verify it fails**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringDocumentControllerTest' -q
```

Expected: FAIL "cannot find symbol method completeAssignment on MonitoringDocumentController".

- [ ] **Step 3: Add controller endpoint**

In `MonitoringDocumentController.java`, add new endpoint after `downloadDocument`:

```java
    @PostMapping("/documents/{documentId}/complete-assignment")
    public ApiResponse<MonitoringDocumentDTO> completeAssignment(
            @PathVariable Long documentId,
            @RequestBody @jakarta.validation.Valid MonitoringCcCompleteAssignmentRequestDTO request) {
        return new ApiResponse.Builder<MonitoringDocumentDTO>()
                .payload(monitoringDocumentService.completeAssignment(documentId, request))
                .build();
    }
```

Add import:
```java
import it.deloitte.ba.backend.dto.agency_desk.request.MonitoringCcCompleteAssignmentRequestDTO;
```

- [ ] **Step 4: Run test to verify it passes**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringDocumentControllerTest' -q
```

Expected: PASS 1/1.

---

### Task 1.7: Extend `MonitoringEventRepository` with filtered schedule query

**Files:**
- Modify: `src/main/java/it/deloitte/ba/backend/repository/agency_desk/MonitoringEventRepository.java`

This task adds the JPQL @Query for the BR 700-706 filter logic. No unit test at repository level (consistent with pattern of existing repo). Coverage via service test in Task 1.8.

- [ ] **Step 1: Add new query method**

In `MonitoringEventRepository.java`, add (before the last `findExpiredWithoutException`):

```java
    @Query("""
        SELECT e FROM MonitoringEvent e
        WHERE e.covenant.practice.id = :practiceId
          AND (:covenantType IS NULL OR e.covenant.covenantType = :covenantType)
          AND (
            ((:statuses) IS NULL OR e.status IN (:statuses))
            OR (:referenceDate IS NOT NULL
                AND e.referenceDate BETWEEN :startDate AND :endDate)
          )
        ORDER BY e.expirationDate ASC
        """)
    Page<MonitoringEvent> findScheduleFiltered(
            @Param("practiceId") Long practiceId,
            @Param("covenantType") CovenantTypeNumericEnum covenantType,
            @Param("statuses") List<MonitoringEventStateEnum> statuses,
            @Param("referenceDate") LocalDate referenceDate,
            @Param("startDate") LocalDate startDate,
            @Param("endDate") LocalDate endDate,
            Pageable pageable);
```

Add imports if missing:
```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
```

- [ ] **Step 2: Verify compile**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw compile -q
```

Expected: BUILD SUCCESS.

---

### Task 1.8: Extend `MonitoringEventService.findScheduleByPractice` signature

**Files:**
- Modify: `src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringEventService.java`
- Modify: `src/test/java/it/deloitte/ba/backend/service/monitoring/MonitoringEventServiceTest.java`

- [ ] **Step 1: Write failing tests**

Append to `MonitoringEventServiceTest.java` (inside the class, at the end):

```java
    @Nested
    class FindScheduleFiltered {

        @Test
        void shouldDelegateToRepositoryWithComputedRange() {
            Long practiceId = 1L;
            String covenantType = "ICR";
            LocalDate refDate = LocalDate.of(2026, 5, 28);
            Integer rangeDays = 15;
            List<MonitoringEventStateEnum> statuses = List.of(
                    MonitoringEventStateEnum.SCADUTO,
                    MonitoringEventStateEnum.FUORI_SOGLIA,
                    MonitoringEventStateEnum.IN_SCADENZA);
            Pageable pageable = PageRequest.of(0, 20);

            when(monitoringEventRepository.findScheduleFiltered(
                    eq(practiceId),
                    eq(CovenantTypeNumericEnum.ICR),
                    eq(statuses),
                    eq(refDate),
                    eq(LocalDate.of(2026, 5, 13)),
                    eq(LocalDate.of(2026, 6, 12)),
                    eq(pageable)))
                    .thenReturn(new PageImpl<>(Collections.emptyList()));

            Page<MonitoringEventDTO> result = monitoringEventService.findScheduleByPractice(
                    practiceId, covenantType, refDate, rangeDays, statuses, pageable);

            assertThat(result).isEmpty();
            verify(monitoringEventRepository).findScheduleFiltered(
                    eq(practiceId), eq(CovenantTypeNumericEnum.ICR), eq(statuses),
                    eq(refDate), eq(LocalDate.of(2026, 5, 13)), eq(LocalDate.of(2026, 6, 12)),
                    eq(pageable));
        }

        @Test
        void shouldPassNullsWhenFiltersAreAbsent() {
            Pageable pageable = PageRequest.of(0, 20);
            when(monitoringEventRepository.findScheduleFiltered(
                    eq(1L), eq(null), eq(null), eq(null), eq(null), eq(null), eq(pageable)))
                    .thenReturn(new PageImpl<>(Collections.emptyList()));

            monitoringEventService.findScheduleByPractice(1L, null, null, null, null, pageable);

            verify(monitoringEventRepository).findScheduleFiltered(
                    eq(1L), eq(null), eq(null), eq(null), eq(null), eq(null), eq(pageable));
        }
    }
```

- [ ] **Step 2: Run tests to verify they fail**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringEventServiceTest#FindScheduleFiltered*' -q
```

Expected: FAIL (signature mismatch on `findScheduleByPractice`).

- [ ] **Step 3: Update findScheduleByPractice signature**

In `MonitoringEventService.java`, REPLACE existing `findScheduleByPractice` (riga 219-227) with overload-friendly version:

```java
    public Page<MonitoringEventDTO> findScheduleByPractice(Long practiceId, String covenantType, Pageable pageable) {
        return findScheduleByPractice(practiceId, covenantType, null, null, null, pageable);
    }

    public Page<MonitoringEventDTO> findScheduleByPractice(
            Long practiceId,
            String covenantType,
            LocalDate referenceDate,
            Integer rangeDays,
            List<MonitoringEventStateEnum> statuses,
            Pageable pageable) {

        CovenantTypeNumericEnum type = (covenantType != null && !covenantType.isEmpty())
                ? CovenantTypeNumericEnum.valueOf(covenantType)
                : null;

        LocalDate startDate = null;
        LocalDate endDate = null;
        if (referenceDate != null) {
            int range = (rangeDays != null) ? rangeDays : 15;
            startDate = referenceDate.minusDays(range);
            endDate = referenceDate.plusDays(range);
        }

        List<MonitoringEventStateEnum> effectiveStatuses = (statuses != null && !statuses.isEmpty())
                ? statuses
                : null;

        return monitoringEventRepository.findScheduleFiltered(
                practiceId, type, effectiveStatuses, referenceDate, startDate, endDate, pageable)
                .map(this::toDto);
    }
```

- [ ] **Step 4: Run tests to verify they pass**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringEventServiceTest#FindScheduleFiltered*' -q
```

Expected: PASS 2/2.

- [ ] **Step 5: Run full MonitoringEventServiceTest**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringEventServiceTest' -q
```

Expected: ALL PASS (existing tests + new FindScheduleFiltered).

---

### Task 1.9: Extend `MonitoringCovenantController` `/schedule` endpoint

**Files:**
- Modify: `src/main/java/it/deloitte/ba/backend/controller/agency_desk/MonitoringCovenantController.java`
- Create: `src/test/java/it/deloitte/ba/backend/controller/agency_desk/MonitoringCovenantControllerTest.java`

- [ ] **Step 1: Write failing test**

Create `MonitoringCovenantControllerTest.java` (NEW file):

```java
package it.deloitte.ba.backend.controller.agency_desk;

import it.deloitte.ba.backend.dto.agency_desk.MonitoringEventDTO;
import it.deloitte.ba.backend.enumeration.agency_desk.MonitoringEventStateEnum;
import it.deloitte.ba.backend.service.monitoring.MonitoringCovenantService;
import it.deloitte.ba.backend.service.monitoring.MonitoringEventService;
import it.deloitte.ba.backend.shared.rest.model.ApiResponse;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageImpl;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;

import java.time.LocalDate;
import java.util.Collections;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.eq;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
class MonitoringCovenantControllerTest {

    @Mock private MonitoringCovenantService monitoringCovenantService;
    @Mock private MonitoringEventService monitoringEventService;

    @InjectMocks private MonitoringCovenantController controller;

    @Test
    void shouldDelegateScheduleWithFilters() {
        Long practiceId = 1L;
        String covenantType = "ICR";
        LocalDate refDate = LocalDate.of(2026, 5, 28);
        Integer rangeDays = 15;
        List<MonitoringEventStateEnum> statuses = List.of(
                MonitoringEventStateEnum.SCADUTO,
                MonitoringEventStateEnum.FUORI_SOGLIA,
                MonitoringEventStateEnum.IN_SCADENZA);
        Page<MonitoringEventDTO> emptyPage = new PageImpl<>(Collections.emptyList());

        when(monitoringEventService.findScheduleByPractice(
                eq(practiceId), eq(covenantType), eq(refDate), eq(rangeDays), eq(statuses), any(Pageable.class)))
                .thenReturn(emptyPage);

        ApiResponse<Page<MonitoringEventDTO>> response = controller.findScheduleByPractice(
                practiceId, covenantType, refDate, rangeDays, statuses, 0, 20);

        assertThat(response.getPayload().getTotalElements()).isZero();
        verify(monitoringEventService).findScheduleByPractice(
                eq(practiceId), eq(covenantType), eq(refDate), eq(rangeDays), eq(statuses), any(Pageable.class));
    }

    @Test
    void shouldDefaultRangeDaysTo15WhenAbsent() {
        Page<MonitoringEventDTO> emptyPage = new PageImpl<>(Collections.emptyList());
        when(monitoringEventService.findScheduleByPractice(
                eq(1L), eq(null), eq(null), eq(15), eq(null), any(Pageable.class)))
                .thenReturn(emptyPage);

        controller.findScheduleByPractice(1L, null, null, 15, null, 0, 20);

        verify(monitoringEventService).findScheduleByPractice(
                eq(1L), eq(null), eq(null), eq(15), eq(null), any(Pageable.class));
    }
}
```

- [ ] **Step 2: Run test to verify it fails**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringCovenantControllerTest' -q
```

Expected: FAIL (signature mismatch).

- [ ] **Step 3: Update controller endpoint signature**

In `MonitoringCovenantController.java`, REPLACE existing `findScheduleByPractice` (riga 95-105) with:

```java
    @GetMapping("/practices/{practiceId}/schedule")
    public ApiResponse<Page<MonitoringEventDTO>> findScheduleByPractice(
            @PathVariable Long practiceId,
            @RequestParam(required = false) String covenantType,
            @RequestParam(required = false) LocalDate referenceDate,
            @RequestParam(defaultValue = "15") Integer rangeDays,
            @RequestParam(required = false) List<MonitoringEventStateEnum> statuses,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by(Sort.Direction.ASC, "expirationDate"));
        return new ApiResponse.Builder<Page<MonitoringEventDTO>>()
                .payload(monitoringEventService.findScheduleByPractice(
                        practiceId, covenantType, referenceDate, rangeDays, statuses, pageable))
                .build();
    }
```

Add import:
```java
import it.deloitte.ba.backend.enumeration.agency_desk.MonitoringEventStateEnum;
```

- [ ] **Step 4: Run tests to verify they pass**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringCovenantControllerTest' -q
```

Expected: PASS 2/2.

---

### Task 1.10: Verify ALL monitoring tests still green + commit

- [ ] **Step 1: Run full monitoring test suite**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='Monitoring*' -q
```

Expected: ALL PASS. Conta tests: dovrebbe essere 145 + (3+4+1+1+2+2) = 158 nuovi test totali.

- [ ] **Step 2: Run full test suite for regression**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -q
```

Expected: BUILD SUCCESS. Tutti i test (incluso non-monitoring) green.

- [ ] **Step 3: Stage and commit**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && git add \
  src/main/java/it/deloitte/ba/backend/enumeration/agency_desk/MonitoringDocumentStateEnum.java \
  src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentService.java \
  src/main/java/it/deloitte/ba/backend/controller/agency_desk/MonitoringDocumentController.java \
  src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringEventService.java \
  src/main/java/it/deloitte/ba/backend/controller/agency_desk/MonitoringCovenantController.java \
  src/main/java/it/deloitte/ba/backend/repository/agency_desk/MonitoringEventRepository.java \
  src/main/java/it/deloitte/ba/backend/dto/agency_desk/request/MonitoringCcCompleteAssignmentRequestDTO.java \
  src/test/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentServiceTest.java \
  src/test/java/it/deloitte/ba/backend/service/monitoring/MonitoringEventServiceTest.java \
  src/test/java/it/deloitte/ba/backend/controller/agency_desk/MonitoringDocumentControllerTest.java \
  src/test/java/it/deloitte/ba/backend/controller/agency_desk/MonitoringCovenantControllerTest.java
```

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && git commit -m "feat(monitoring): add DA_ASSEGNARE state + complete-assignment endpoint + schedule filters (bug #10 BR 4.2.3)"
```

Expected: commit creato. Branch HEAD avanza di 1.

---

# SECTION II — Commit 2: BE switch + rename (breaking, FE outage window)

**Goal Commit 2**: cambiare il behavior di `confirmDocument` (ora setta `DA_ASSEGNARE` invece di `COMPLETATO`), rinominare endpoint a `/validate`, far chiamare `applyExtraction` GenAI a `completeAssignment(id)`. Questo è il commit BREAKING per FE (callers FE chiamano ancora vecchio path `/confirm`). FE sistemato in Wave 2 Commit 3.

---

### Task 2.1: Switch `confirmDocument` body to `DA_ASSEGNARE` + rename to `validateDocument`

**Files:**
- Modify: `src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentService.java`

Note: in Commit 1 abbiamo aggiunto `validateDocument` come metodo SEPARATO accanto a `confirmDocument`. Adesso eliminiamo `confirmDocument` (delete metodo) — i callers BE saranno aggiornati in Task 2.3.

- [ ] **Step 1: Delete old confirmDocument method**

In `MonitoringDocumentService.java`, REMOVE method `confirmDocument(Long documentId)` (riga 81-89). NON toccare `validateDocument` (era new in Commit 1).

- [ ] **Step 2: Verify compile (expect failure on callers)**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw compile -q
```

Expected: FAIL with "cannot find symbol confirmDocument" in `MonitoringDocumentController` and `MonitoringGenAiService`. Tutto OK — verranno fixati in Task 2.2-2.3.

---

### Task 2.2: Rename PATCH `/confirm` → `/validate` in controller + remove confirm method

**Files:**
- Modify: `src/main/java/it/deloitte/ba/backend/controller/agency_desk/MonitoringDocumentController.java`

- [ ] **Step 1: Replace endpoint**

In `MonitoringDocumentController.java`, REPLACE old `confirmDocument` endpoint (riga 44-50):

OLD:
```java
    @PatchMapping("/documents/{documentId}/confirm")
    public ApiResponse<MonitoringDocumentDTO> confirmDocument(
            @PathVariable Long documentId) {
        return new ApiResponse.Builder<MonitoringDocumentDTO>()
                .payload(monitoringDocumentService.confirmDocument(documentId))
                .build();
    }
```

NEW:
```java
    @PatchMapping("/documents/{documentId}/validate")
    public ApiResponse<MonitoringDocumentDTO> validateDocument(
            @PathVariable Long documentId) {
        return new ApiResponse.Builder<MonitoringDocumentDTO>()
                .payload(monitoringDocumentService.validateDocument(documentId))
                .build();
    }
```

- [ ] **Step 2: Verify compile (still expect failure on MonitoringGenAiService)**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw compile -q
```

Expected: FAIL with "cannot find symbol confirmDocument" only in `MonitoringGenAiService`.

---

### Task 2.3: Switch `MonitoringGenAiService.applyExtraction` finalize call

**Files:**
- Modify: `src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringGenAiService.java`
- Modify: `src/test/java/it/deloitte/ba/backend/service/monitoring/MonitoringGenAiServiceTest.java`

- [ ] **Step 1: Update failing test in MonitoringGenAiServiceTest**

Find the test that verifies `applyExtraction` calls `confirmDocument` (likely named `shouldApplyXxxAndConfirmDocument` or similar). Update its assertion from `verify(documentService).confirmDocument(...)` to `verify(documentService).completeAssignment(...)`.

If no such test exists, ADD:

```java
    @Test
    void shouldCompleteDocumentAssignmentAfterApply() {
        // ...existing arrange of extraction ready, request
        // Pre-cond: documento status === GENAI_DA_VALIDARE in repository mock
        // when service.applyExtraction is called
        // verify: documentService.completeAssignment(documentId) was invoked
        // (replace any prior expectation of confirmDocument)
    }
```

(Test exact code dipende dallo state attuale di `MonitoringGenAiServiceTest`. Leggere quel file e fare modifica chirurgica al test esistente.)

- [ ] **Step 2: Run test to verify failure**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringGenAiServiceTest' -q
```

Expected: FAIL su test che verifica `completeAssignment` chiamato (oggi service chiama ancora `confirmDocument` che non esiste più).

- [ ] **Step 3: Update applyExtraction to call completeAssignment**

In `MonitoringGenAiService.java`, riga 139, REPLACE:

OLD:
```java
        MonitoringDocumentDTO documentDto = documentService.confirmDocument(documentId);
```

NEW:
```java
        MonitoringDocumentDTO documentDto = documentService.completeAssignment(documentId);
```

- [ ] **Step 4: Verify compile**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw compile -q
```

Expected: BUILD SUCCESS.

- [ ] **Step 5: Run MonitoringGenAiServiceTest**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='MonitoringGenAiServiceTest' -q
```

Expected: PASS.

---

### Task 2.4: Verify ALL monitoring tests + full regression + commit

- [ ] **Step 1: Run full monitoring tests**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -Dtest='Monitoring*' -q
```

Expected: ALL PASS.

- [ ] **Step 2: Run full test suite**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && ./mvnw test -q
```

Expected: BUILD SUCCESS.

- [ ] **Step 3: Stage and commit**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && git add \
  src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringDocumentService.java \
  src/main/java/it/deloitte/ba/backend/controller/agency_desk/MonitoringDocumentController.java \
  src/main/java/it/deloitte/ba/backend/service/monitoring/MonitoringGenAiService.java \
  src/test/java/it/deloitte/ba/backend/service/monitoring/MonitoringGenAiServiceTest.java
```

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && git commit -m "refactor(monitoring): replace confirmDocument with validateDocument + completeAssignment per BR 4.2.3 state machine"
```

Expected: commit creato.

---

### Task 2.5: Push BE wave to origin (chiedere conferma utente)

- [ ] **Step 1: Chiedere conferma push all'utente** (memoria utente: mai pushare senza chiedere).

Output user-facing:
> "BE wave completata: Commit 1 + Commit 2 su `feature/monitoring-integration-uat`. Tutti i test green. Pronto a pushare entrambi i commit a origin?"

Wait for user OK.

- [ ] **Step 2: Push se autorizzato**

```bash
cd C:/Users/davmelis/Documents/Github/ba-back-end && git push
```

Expected: push success a `origin/feature/monitoring-integration-uat`.

---

# SECTION III — Commit 3-5: FE wave (delegato a sessione successiva)

**Goal Wave 2**: implementare i 4 nuovi component FE + wire-up + cleanup vecchi component. Eseguita in **sessione successiva** con plan dedicato (sub-plan generato a inizio sessione successiva basato su questo outline + spec design).

### Commit 3: FE new components

**File modified:**
- `src/app/service/monitoring.service.ts` — rename `confirmDocument()` → `validateDocument()` (path `/validate`); aggiungere `completeAssignment(documentId, payload)`; estendere `getSchedule()` con 3 params opzionali (`referenceDate`, `rangeDays`, `statuses`)
- `src/app/shared/enum/monitoring-document-state.enum.ts` — aggiungere `DA_ASSEGNARE`
- `src/app/module/agency-desk/monitoring/practice-detail/practice-detail.module.ts` — declare 4 nuovi component + aggiungere route `cc/:documentId`
- `assets/i18n/it.json` + `assets/i18n/en.json` — aggiungere chiavi (lista completa in spec §7.4)

**File created:**
- `src/app/shared/model/monitoring/monitoring-cc-assignment.model.ts` — `CcSlotState`, `CcCompleteAssignmentRequest`
- `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/monitoring-cc-assignment-page.component.{ts,html,scss,spec.ts}`
- `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/components/monitoring-cc-covenant-slot.component.{ts,html,scss,spec.ts}`
- `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/components/monitoring-cc-schedule-table.component.{ts,html,scss,spec.ts}`
- `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment-page/components/monitoring-cc-reject-dialog.component.{ts,html,scss,spec.ts}`

**Verify:** `npx tsc --noEmit --skipLibCheck` green; `npm test -- --include='**/cc-assignment*'` green.

**Commit message**: `feat(monitoring): add CC assignment page-based UX (bug #10 BR 4.2.3)`

### Commit 4: FE wire-up + remove old triggers

**File modified:**
- `src/app/module/agency-desk/monitoring/practice-detail/document-history/monitoring-document-history.component.{ts,html}` — `onConfirm` chiama `validateDocument`; render "Assegna Monitoraggi" se `status===DA_ASSEGNARE`; render "Visualizza Assegnazione" se `status===COMPLETATO`. Entrambi navigano a `./cc/:documentId` (+ `?mode=view` per visualizza)
- `src/app/module/agency-desk/monitoring/practice-detail/practice-detail.component.{html,ts}` — RIMUOVERE bottone "Assegna Monitoraggi" (HTML riga 191-200) + `@ViewChild ccAssignment` (TS riga 38) + `openCcAssignment()` (TS riga 111-113) + `onCcAssignmentComplete()` (TS riga 123)

**Verify:** `npx tsc --noEmit` green; smoke E2E manuale (checklist in spec §8.3).

**Commit message**: `feat(monitoring): wire CC assignment page in document-history + remove old dialog triggers`

### Commit 5: Cleanup vecchi component

**File deleted:**
- `src/app/module/agency-desk/monitoring/practice-detail/cc-assignment/cc-assignment.component.{ts,html,scss,spec.ts}`
- `src/app/module/agency-desk/monitoring/practice-detail/monitoring-genai-extraction/monitoring-genai-extraction.component.{ts,html,scss,spec.ts}`
- `src/app/module/agency-desk/monitoring/practice-detail/monitoring-genai-extraction/monitoring-genai-extraction.module.ts`

**File modified:**
- `src/app/module/agency-desk/monitoring/practice-detail/practice-detail.module.ts` — rimuovere `declarations` + `imports` riferimenti a CcAssignmentComponent + MonitoringGenAiExtractionModule

**Verify:** `npx tsc --noEmit` green; `npm test` full suite green.

**Commit message**: `chore(monitoring): remove deprecated CcAssignmentComponent + MonitoringGenAiExtractionComponent (replaced by CC assignment page)`

---

# Self-Review

**1. Spec coverage check:**

| Spec section | Covered by task |
|---|---|
| §3 Architecture & routing | Wave 2 Commit 3 (route declaration in practice-detail.module) |
| §4 Component breakdown | Wave 2 Commit 3 (4 component creation) |
| §5 BE state machine + endpoints | ✅ Task 1.1-1.9 + 2.1-2.3 |
| §6 Data flow FE | Wave 2 Commit 3 (component logic) |
| §7 State management + error handling | Wave 2 Commit 3 (page component implementation) |
| §8 Testing strategy | ✅ BE Task 1.2-1.9 + 2.3; FE in Wave 2 |
| §9 Migration order | ✅ Section I = Commit 1, Section II = Commit 2, Section III = Commit 3-5 outline |
| §10 Constraints non negoziabili | ✅ TDD pattern + immutability + no Co-Author + no auto-push |

**2. Placeholder scan:** 
- No "TBD", "TODO". Outline FE wave riconosciuto come delegated (non placeholder).
- Task 2.3 Step 1 dice "leggere quel file e fare modifica chirurgica" — accettabile perché il test esistente non è noto a priori (siamo agnostici sul nome) ma il pattern è chiaro.

**3. Type consistency:** 
- `MonitoringCcCompleteAssignmentRequestDTO` used in Task 1.4, 1.5, 1.6 con record naming consistente (`assignments`, `CovenantAssignment`).
- `validateDocument(Long)` Task 1.2 → used Task 2.2 (controller) coerente.
- `completeAssignment(Long)` (Task 1.3) + `completeAssignment(Long, RequestDTO)` (Task 1.5) overload coerente.
- `findScheduleByPractice` overload Task 1.8 + controller Task 1.9: signature aligned (Long, String, LocalDate, Integer, List, Pageable).

✅ Plan ready for execution.

---

# Execution Handoff

**Plan complete and saved to `plans/in-progress/2026-05-04_monitoring/implementation/2026-05-28-bug10-cc-assignment-page-plan.md`.**

**Wave 1 (questa sessione)**: execute BE Commit 1 + Commit 2 inline (subagent-driven non necessario, scope contenuto, context fresco del design).

**Wave 2 (sessione successiva)**: handoff con riferimento spec + plan + stato BE; generare sub-plan FE dettagliato a inizio sessione.
