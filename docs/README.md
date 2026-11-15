# COBOL Account Management

This directory documents the COBOL account-management example in `src/cobol/`.
The program provides a menu for viewing an account balance, crediting an
account, and debiting an account. Although this documentation refers to
student accounts, the current implementation does not store student names,
IDs, or separate balances for multiple students; it manages one balance.

## COBOL files

### `src/cobol/main.cob` — `MainProgram`

Entry point and interactive menu. It repeatedly accepts a choice and calls
`Operations` to:

- View the balance (`TOTAL`)
- Credit the account (`CREDIT`)
- Debit the account (`DEBIT`)
- Exit the program

Choices outside 1–4 display an invalid-choice message and return to the menu.

### `src/cobol/operations.cob` — `Operations`

Implements the account actions and user interaction:

- `TOTAL` reads and displays the current balance.
- `CREDIT` accepts an amount, reads the balance, adds the amount, saves the
  result, and displays it.
- `DEBIT` accepts an amount and checks whether the balance covers it. If so,
  it subtracts and saves the amount; otherwise, it displays an insufficient
  funds message and leaves the balance unchanged.

### `src/cobol/data.cob` — `DataProgram`

Stores the account balance in working storage and exposes it to `Operations`
through `READ` and `WRITE` operations. The stored balance starts at `1000.00`
and exists only for the lifetime of the running program; it is not persisted
to a file or database.

## Student-account business rules currently implemented

- The account starts with a balance of **1000.00**.
- A credit increases the balance by the amount entered.
- A debit is allowed only when the current balance is at least the debit
  amount. Otherwise, the balance is not changed.
- There is no separate student-account record or per-student balance.
- The code does not add validation for negative or zero transaction amounts,
  nor does it enforce account-specific limits.

## Application data flow

```mermaid
sequenceDiagram
    actor Student
    participant Main as MainProgram
    participant Ops as Operations
    participant Data as DataProgram

    loop Until the student exits
        Main->>Student: Display menu and request choice
        Student->>Main: Enter choice
        alt View balance
            Main->>Ops: TOTAL
            Ops->>Data: READ
            Data-->>Ops: Current balance
            Ops-->>Student: Display balance
        else Credit account
            Main->>Ops: CREDIT
            Ops->>Student: Request credit amount
            Student-->>Ops: Enter amount
            Ops->>Data: READ
            Data-->>Ops: Current balance
            Ops->>Ops: Add amount to balance
            Ops->>Data: WRITE updated balance
            Ops-->>Student: Display credited amount and balance
        else Debit account
            Main->>Ops: DEBIT
            Ops->>Student: Request debit amount
            Student-->>Ops: Enter amount
            Ops->>Data: READ
            Data-->>Ops: Current balance
            alt Balance covers debit
                Ops->>Ops: Subtract amount from balance
                Ops->>Data: WRITE updated balance
                Ops-->>Student: Display debited amount and balance
            else Insufficient funds
                Ops-->>Student: Display insufficient funds
            end
        else Exit
            Student-->>Main: Choose exit
            Main-->>Student: Display goodbye message
        else Invalid choice
            Main-->>Student: Display invalid-choice message
        end
    end
```
