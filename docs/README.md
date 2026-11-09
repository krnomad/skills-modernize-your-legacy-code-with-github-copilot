# COBOL Account Management

This directory documents the COBOL account management example in `src/cobol/`.
Despite the student-account context of the exercise, the current program models
one generic account; it does not store student identities or multiple accounts.

## Source Files

### `main.cob`

Defines the interactive entry point, `MainProgram`. It displays a menu, reads
the user's selection, and dispatches account actions to `Operations`:

- `1`: view the current balance (`TOTAL`)
- `2`: credit the account (`CREDIT`)
- `3`: debit the account (`DEBIT`)
- `4`: exit the program

Any other selection displays an invalid-choice message.
Operation names use the six-character `PIC X(6)` field, so `TOTAL` and `DEBIT`
are space-padded when passed to `Operations`.

### `operations.cob`

Defines `Operations`, which handles the requested account action. It reads the
current balance before displaying or changing it, prompts for an amount for
credits and debits, and writes the updated balance after a successful change.
For debits, it displays an insufficient-funds message instead of applying a
withdrawal when the balance is less than the requested amount.

### `data.cob`

Defines `DataProgram`, the balance storage interface. `READ` copies the stored
balance to the caller, while `WRITE` replaces it with the caller-provided
balance. The stored balance starts at `1000.00` and is held in the program's
working storage; there is no file or database persistence in this code.

## Account Rules and Limits

- The example maintains a single account balance, initialized to `1000.00`.
- A credit adds the entered amount to the balance.
- A debit is applied only when the current balance is at least the entered
  amount; otherwise, the balance is left unchanged and an insufficient-funds
  message is shown.
- Balances and transaction amounts use `PIC 9(6)V99`, allowing six integer
  digits and two implied decimal places.
- The code does not validate that entered amounts are positive, identify or
  authenticate a student, distinguish among student accounts, or persist data
  outside the running program. No additional student-specific rules are
  implemented.

## Application Sequence

```mermaid
sequenceDiagram
    actor User
    participant Main as MainProgram
    participant Ops as Operations
    participant Data as DataProgram

    User->>Main: Select menu option
    Main->>Main: ACCEPT USER-CHOICE

    alt View balance (option 1)
        Main->>Ops: CALL Operations("TOTAL ")
        Ops->>Data: CALL DataProgram("READ", FINAL-BALANCE)
        Data-->>Ops: Return stored balance
        Ops-->>User: Display current balance
    else Credit account (option 2)
        Main->>Ops: CALL Operations("CREDIT")
        Ops-->>User: Prompt for credit amount
        User->>Ops: Enter amount
        Ops->>Data: READ current balance
        Data-->>Ops: Return stored balance
        Ops->>Ops: Add amount to balance
        Ops->>Data: WRITE updated balance
        Ops-->>User: Display new balance
    else Debit account (option 3)
        Main->>Ops: CALL Operations("DEBIT ")
        Ops-->>User: Prompt for debit amount
        User->>Ops: Enter amount
        Ops->>Data: READ current balance
        Data-->>Ops: Return stored balance
        alt Balance is sufficient
            Ops->>Ops: Subtract amount from balance
            Ops->>Data: WRITE updated balance
            Ops-->>User: Display new balance
        else Insufficient funds
            Ops-->>User: Display insufficient-funds message
        end
    else Exit (option 4)
        Main-->>User: Display goodbye message
    else Invalid option
        Main-->>User: Display invalid-choice message
    end
```
