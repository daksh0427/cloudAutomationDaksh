# FestPass CI — Jenkins Red → Green Assignment

**Student alias (Git author):** DakshMohanArora
**Class code:** 001
**Challenge code:** 00532
**Job name:** festpass-ci
**Agent:** student-laptop (permanent, label `student-laptop`, built-in executors = 0)
**Pipeline definition:** Pipeline script from SCM — `git://127.0.0.1:9418/festpass-java`, branch `*/main`, `Jenkinsfile`
**Trigger:** `pollSCM('* * * * *')` committed in the Jenkinsfile

## Build history

| # | Cause | Commit | Result | Tests | Artifact |
|---|---|---|---|---|---|
| 1 | Bootstrap (manual) | `25cbf28` | FAILURE | 28, 3 failures | evidence CSV only |
| 2 | SCM poll (personalization) | `ff0bb89` | FAILURE | 28, 3 failures | evidence CSV only |
| 3 | SCM poll (fix) | `854c551` | SUCCESS | 28, 0 failures | `festpass.jar` + evidence |

Product stages (Build JAR, Smoke test JAR, Save artifact) were skipped on Builds 1 and 2 due to the failing Test stage, confirming JAR gating works. Submission evidence is always archived.

## Diagnosis

The three failing tests all involved **cancellation**:
- `persistCancellation`
- `cliWorkflowPersistsAcrossCommands`
- a third assertion exercising the same path

`BookingService.cancel(eventId, studentId)` validated that the student existed in the event's booking set, but then returned without ever removing them. The booking count therefore never decreased after a cancellation — breaking downstream rules such as "a seat frees up after cancel" and "a canceled student can rebook".

## Fix (one line, src/main/java/BookingService.java)

```java
public void cancel(String eventId, String studentId) {
    validateId(studentId);
    Registration event = requireEvent(eventId);
    if (!event.students.contains(studentId)) {
        throw new IllegalArgumentException("No booking for student: " + studentId);
    }
    event.students.remove(studentId);   // <-- added
}
```

No test, assertion, pipeline stage, or build helper was modified. Only the one application line was added.

## Files in this folder

- `build-1-RED-evidence.csv` — Build 1 submission evidence (FAILURE, 3 fails)
- `build-3-GREEN-evidence.csv` — Build 3 submission evidence (SUCCESS, 28 pass)
- `festpass.jar` — Final successful JAR archived from Build 3
- `build-info.txt` — Build 3 metadata
- `smoke-test.txt` — Build 3 real-JAR smoke-test output
