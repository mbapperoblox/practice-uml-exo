# uml exo 2: abcd records

ABCD Records is a mail-order company that sells CDs and tapes at discount prices to record club members.

Rules:
1. only members can place orders
2. non-members receive a membership application form
3. two kinds of members: **Royal** (discounts + priority) and **Regular**
4. Royal members can order items that are not available
5. payment by cash, check or bank draft
6. the order processing clerk verifies and processes the orders
7. the collection department clerk handles the payments
8. all orders are saved in a daily records file

## level 1

### task 1.1: use case diagram

Actors: Member, Order Processing Clerk. Use cases: Place Order, Verify Membership, Process Order, Make Payment.

```mermaid
flowchart LR
    M[Member]
    C[Order Processing Clerk]
    subgraph system [ABCD Records system]
        U1([Place Order])
        U2([Verify Membership])
        U3([Process Order])
        U4([Make Payment])
    end
    M --- U1
    M --- U4
    C --- U2
    C --- U3
```

### task 1.2: class diagram

```mermaid
classDiagram
    class Member {
        -memberId : int
        -name : string
        -address : string
        -memberType : string
        +placeOrder() Order
        +makePayment(order)
    }
    class Order {
        -orderNumber : int
        -date : date
        -total : float
        -status : string
        +addItem(item, quantity)
        +calculateTotal() float
    }
    class OrderProcessingClerk {
        -clerkId : int
        -name : string
        +verifyMembership(member) bool
        +processOrder(order)
    }
    Member "1" --> "0..*" Order : places
    OrderProcessingClerk "1" --> "0..*" Order : processes
```

## level 2

### task 2.1: complete use case diagram

New actors: Collection Department Clerk, Non-Member. New use cases: Verify Item Availability, Apply Discount, Print Invoice, Print Shipping List, Order Unavailable Items, Send Membership Application.

- **include** = "this step is always done" (mandatory). Place Order always includes Verify Membership and Verify Item Availability; Process Order always includes Print Invoice and Print Shipping List.
- **extend** = "this step happens only sometimes" (optional). Apply Discount extends Place Order (only for Royal members); Order Unavailable Items extends Verify Item Availability (only when an item is missing, and only for Royal members).

```mermaid
flowchart LR
    NM[Non-Member]
    M[Member]
    C[Order Processing Clerk]
    P[Collection Department Clerk]
    subgraph system [ABCD Records system]
        UO([Place Order])
        UV([Verify Membership])
        UA([Verify Item Availability])
        UD([Apply Discount])
        UU([Order Unavailable Items])
        UP([Process Order])
        UI([Print Invoice])
        US([Print Shipping List])
        UM([Make Payment])
        UN([Send Membership Application])
    end
    NM --- UN
    M --- UO
    M --- UM
    C --- UP
    P --- UM
    UO -.->|include| UV
    UO -.->|include| UA
    UP -.->|include| UI
    UP -.->|include| US
    UD -.->|extend| UO
    UU -.->|extend| UA
```

When a non-member tries to order, Verify Membership fails and the system sends a membership application (Send Membership Application).

### task 2.2: complete class diagram

Inheritance is used for the 3 families: **Member** (Royal / Regular), **Item** (CD / Tape) and **Payment** (Cash / Check / BankDraft).

```mermaid
classDiagram
    class Member {
        <<abstract>>
        -memberId : int
        -name : string
        -address : string
        +placeOrder() Order
        +getDiscount()* float
        +canOrderUnavailable()* bool
    }
    class RoyalMember {
        +getDiscount() float
        +canOrderUnavailable() bool
    }
    class RegularMember {
        +getDiscount() float
        +canOrderUnavailable() bool
    }
    class Order {
        -orderNumber : int
        -date : date
        -total : float
        +calculateTotal() float
        +saveToDailyRecords()
    }
    class OrderLine {
        -quantity : int
        -unitPrice : float
        +subtotal() float
    }
    class Item {
        <<abstract>>
        -itemId : int
        -title : string
        -price : float
        -stock : int
        +isAvailable() bool
    }
    class CD
    class Tape
    class Invoice {
        -invoiceNumber : int
        -date : date
        -amountDue : float
        +print()
    }
    class ShippingList {
        -listNumber : int
        +print()
    }
    class Payment {
        <<abstract>>
        -amount : float
        -date : date
        +pay()*
    }
    class Cash
    class Check {
        -checkNumber : string
    }
    class BankDraft {
        -draftNumber : string
    }
    class MembershipApplication {
        -applicantName : string
        -address : string
        -date : date
    }
    class OrderProcessingClerk {
        +verifyMembership(member) bool
        +processOrder(order)
    }
    class CollectionClerk {
        +collectPayment(payment)
    }
    Member <|-- RoyalMember
    Member <|-- RegularMember
    Item <|-- CD
    Item <|-- Tape
    Payment <|-- Cash
    Payment <|-- Check
    Payment <|-- BankDraft
    Member "1" --> "0..*" Order : places
    Order "1" *-- "1..*" OrderLine : contains
    OrderLine "0..*" --> "1" Item : refers to
    Order "1" --> "1" Invoice : produces
    Order "1" --> "1" ShippingList : produces
    Order "1" --> "0..*" Payment : paid by
    OrderProcessingClerk "1" --> "0..*" Order : processes
    CollectionClerk "1" --> "0..*" Payment : handles
    MembershipApplication "0..*" ..> Member : may become
```

Multiplicities: an order has at least one line (`1..*`) and the lines belong only to that order (composition, filled diamond: if the order is deleted, its lines go too). An item can appear in many order lines, and a line refers to exactly one item (aggregation-like link).

Scenario: a Royal member orders 2 CDs (one is unavailable) and pays by check.

```mermaid
classDiagram
    class john {
        :RoyalMember
    }
    class order1 {
        :Order
    }
    class line1 {
        :OrderLine
        quantity = 1
    }
    class line2 {
        :OrderLine
        quantity = 1
    }
    class cdA {
        :CD
        stock = 5
    }
    class cdB {
        :CD
        stock = 0
    }
    class pay1 {
        :Check
    }
    john --> order1
    order1 *-- line1
    order1 *-- line2
    line1 --> cdA
    line2 --> cdB
    order1 --> pay1
```

Because john is a Royal member, he is allowed to order `cdB` even if stock = 0; a Regular member would be refused for that line.

## level 3: sequence diagrams

### complete order processing flow

```mermaid
sequenceDiagram
    actor M as Member
    participant S as System
    participant C as Order Processing Clerk
    participant R as Daily Records
    M->>S: placeOrder(items)
    S->>C: verifyMembership(member)
    C-->>S: member is valid
    loop for each order line
        S->>S: isAvailable(item)
    end
    S->>S: applyDiscount(member)
    S->>C: processOrder(order)
    C->>S: printInvoice()
    C->>S: printShippingList()
    S->>R: saveToDailyRecords(order)
    S-->>M: invoice
```

### membership verification (success and failure)

```mermaid
sequenceDiagram
    actor X as Customer
    participant S as System
    participant C as Order Processing Clerk
    X->>S: placeOrder(items)
    S->>C: verifyMembership(customer)
    alt customer is a member
        C-->>S: valid (Royal or Regular)
        S-->>X: continue the order
    else not a member
        C-->>S: invalid
        S-->>X: send membership application form
        S-->>X: order refused
    end
```

### payment processing (all types)

```mermaid
sequenceDiagram
    actor M as Member
    participant S as System
    participant K as Collection Dept Clerk
    participant P as Payment
    M->>S: choose payment type
    alt cash
        S->>P: new Cash(amount)
    else check
        S->>P: new Check(amount, checkNumber)
    else bank draft
        S->>P: new BankDraft(amount, draftNumber)
    end
    S->>K: collectPayment(payment)
    K->>P: pay()
    alt accepted
        P-->>K: ok
        K-->>S: payment received
        S-->>M: receipt
    else refused
        P-->>K: refused
        K-->>S: payment failed
        S-->>M: ask for another payment
    end
```

### item reordering for royal members

```mermaid
sequenceDiagram
    actor R as Royal Member
    participant S as System
    participant I as Item
    participant C as Order Processing Clerk
    R->>S: placeOrder(item)
    S->>I: isAvailable()
    I-->>S: false (stock = 0)
    S->>S: member.canOrderUnavailable()
    alt Royal member
        S->>S: mark line as back-ordered
        S->>C: processOrder(order) with priority
        S-->>R: item ordered, will ship when back in stock
    else Regular member
        S-->>R: item unavailable
    end
```
