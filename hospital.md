# uml exo 1: hospital management system

Patients post a health problem online, the organizer asks a doctor, the doctor gives a prescription, the organizer sends it to the patient, the patient pays (check, cash or credit card) and the organizer forwards the payment to the doctor. The organizer is also the administrator (database, member ids and passwords).

## task 1: basic architecture (class diagram)

Who does what?
- the **patient** posts a problem, receives a prescription and pays
- the **doctor** reads the problem and writes the prescription
- the **organizer** is the middleman: forwards problems, sends prescriptions, collects and forwards the money, creates the accounts

```mermaid
classDiagram
    class Patient {
        -id : int
        -name : string
        -password : string
        -address : string
        +postProblem(description)
        +payFee(amount, method)
        +receivePrescription(prescription)
    }
    class Doctor {
        -id : int
        -name : string
        -password : string
        -speciality : string
        +readProblem(problem)
        +writePrescription(problem)
        +receivePayment(amount)
    }
    class Organizer {
        -id : int
        -name : string
        -password : string
        -email : string
        +forwardProblem(problem, doctor)
        +sendPrescription(prescription, patient)
        +forwardPayment(payment, doctor)
        +createMember(name) 
    }
    Organizer "1" --> "0..*" Patient : manages
    Organizer "1" --> "1..*" Doctor : manages
    Patient "0..*" -- "1" Organizer : talks to
    Doctor "1..*" -- "1" Organizer : talks to
```

Multiplicities: one organizer manages many patients (0 or more) and at least one doctor. A patient and a doctor never talk directly, they only talk to the organizer.

## task 2: develop the system

New classes: `HealthProblem`, `Prescription`, `Payment`. Payment types (`Check`, `Cash`, `CreditCard`) are **subclasses** of `Payment` (inheritance), because each one has its own details (check number, card number...) but they all share the amount and the date.

```mermaid
classDiagram
    class Patient {
        -id : int
        -name : string
        -password : string
        +postProblem(description) HealthProblem
        +pay(prescription, method) Payment
    }
    class Doctor {
        -id : int
        -name : string
        -password : string
        -speciality : string
        +writePrescription(problem) Prescription
    }
    class Organizer {
        -id : int
        -name : string
        -password : string
        +forwardProblem(problem, doctor)
        +sendPrescription(prescription)
        +forwardPayment(payment, doctor)
    }
    class HealthProblem {
        -description : string
        -date : date
    }
    class Prescription {
        -text : string
        -date : date
        -fee : float
    }
    class Payment {
        <<abstract>>
        -amount : float
        -date : date
        +pay()*
    }
    class Check {
        -checkNumber : string
        +pay()
    }
    class Cash {
        +pay()
    }
    class CreditCard {
        -cardNumber : string
        -expiry : string
        +pay()
    }
    Patient "1" --> "0..*" HealthProblem : posts
    HealthProblem "1" --> "0..1" Prescription : gets
    Doctor "1" --> "0..*" Prescription : writes
    Patient "1" --> "0..*" Payment : makes
    Prescription "1" --> "0..1" Payment : paid by
    Organizer "1" --> "0..*" Patient : manages
    Organizer "1" --> "1..*" Doctor : manages
    Payment <|-- Check
    Payment <|-- Cash
    Payment <|-- CreditCard
```

Sequence diagram: a patient submits a problem, receives a prescription and pays.

```mermaid
sequenceDiagram
    actor P as Patient
    participant O as Organizer
    participant D as Doctor
    P->>O: postProblem(description)
    O->>D: forwardProblem(problem)
    D-->>O: prescription
    O-->>P: sendPrescription(prescription)
    P->>O: pay(amount, method)
    O->>D: forwardPayment(payment)
    D-->>O: confirmation
    O-->>P: receipt
```

## task 3: advanced

Changes:
- a **`User`** superclass holds what the three roles share (`id`, `name`, `password`, `login()`, `logout()`); `Patient`, `Doctor` and `Organizer` inherit from it
- payment uses **polymorphism** (strategy): `Payment` is an interface, the organizer calls `pay()` without knowing which type it is
- `HealthProblem` has a **status** (pending, in-review, resolved) and timestamps
- a patient keeps the list of **past prescriptions**

```mermaid
classDiagram
    class User {
        <<abstract>>
        #id : int
        #name : string
        #password : string
        +login(password) bool
        +logout()
    }
    class Patient {
        -pastPrescriptions : list
        +postProblem(description) HealthProblem
        +pay(prescription, method) Payment
    }
    class Doctor {
        -speciality : string
        +writePrescription(problem) Prescription
    }
    class Organizer {
        +createMember(name, role) User
        +forwardProblem(problem, doctor)
        +sendPrescription(prescription)
        +forwardPayment(payment, doctor)
    }
    class HealthProblem {
        -description : string
        -status : Status
        -createdAt : datetime
        -updatedAt : datetime
    }
    class Status {
        <<enumeration>>
        PENDING
        IN_REVIEW
        RESOLVED
    }
    class Prescription {
        -text : string
        -fee : float
        -issuedAt : datetime
    }
    class Payment {
        <<interface>>
        +pay(amount)*
    }
    class Check
    class Cash
    class CreditCard
    User <|-- Patient
    User <|-- Doctor
    User <|-- Organizer
    HealthProblem --> Status
    Patient "1" --> "0..*" HealthProblem : posts
    HealthProblem "1..*" --> "1..*" Doctor : reviewed by
    Doctor "1" --> "0..*" Prescription : writes
    Patient "1" o-- "0..*" Prescription : keeps history
    Prescription "1" --> "0..*" Payment : paid by
    Payment <|.. Check
    Payment <|.. Cash
    Payment <|.. CreditCard
```

### sequence 1: patient registration (the organizer creates a new member)

```mermaid
sequenceDiagram
    actor N as New patient
    participant O as Organizer
    participant DB as Database
    N->>O: request an account (name, address)
    O->>DB: createMember(name, role)
    DB-->>O: new id
    O->>O: generate password
    O-->>N: send id and password
    N->>O: login(id, password)
    O-->>N: access granted
```

### sequence 2: consultation flow (problem to prescription)

```mermaid
sequenceDiagram
    actor P as Patient
    participant O as Organizer
    participant D as Doctor
    P->>O: postProblem(description)
    O->>O: status = PENDING
    O->>D: forwardProblem(problem)
    O->>O: status = IN_REVIEW
    D->>D: read problem and past prescriptions
    D-->>O: prescription
    O->>O: status = RESOLVED
    O-->>P: sendPrescription(prescription)
```

### sequence 3: payment processing

```mermaid
sequenceDiagram
    actor P as Patient
    participant O as Organizer
    participant Pay as Payment (Check / Cash / CreditCard)
    participant D as Doctor
    P->>O: pay(prescription, method)
    O->>Pay: create payment of the right type
    O->>Pay: pay(amount)
    alt payment accepted
        Pay-->>O: confirmed
        O->>D: forwardPayment(payment)
        D-->>O: received
        O-->>P: receipt
    else payment refused
        Pay-->>O: refused
        O-->>P: error, try another method
    end
```

## questions to consider

1. **A patient consulting several doctors?** `HealthProblem` to `Doctor` becomes many-to-many (`1..*` on both sides). Each doctor writes his own prescription for the same problem.
2. **Reusable prescriptions?** Keep prescriptions in the patient's history (aggregation: a prescription still exists on its own). A repeat prescription is a new `Prescription` that points to the old one (`renewalOf`), so the original is never changed.
3. **Which design pattern for payments?** **Strategy** (each payment method is a strategy with the same `pay()` method) with a **Factory** to create the right one from "check", "cash" or "credit_card".
4. **Data integrity when paying?** Check the data before charging, never mark a prescription as paid until the payment is confirmed, do "charge + record + forward to doctor" as one transaction (all or nothing), and keep each payment with a unique id so it can't be counted twice.
