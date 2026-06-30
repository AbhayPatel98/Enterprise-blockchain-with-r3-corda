# Corda Architecture

> Understand the internal architecture of R3 Corda and how its components work together to create secure, private, and enterprise-ready distributed applications.

---

# Learning Objectives

After completing this chapter, you should understand:

- Every core component of a Corda application.
- How components communicate.
- How data moves across the network.
- Why Corda's architecture differs from blockchain platforms.
- How enterprise transactions are processed.

---

# 1. Architecture Overview

Think of Corda as a company.

Every employee has a specific responsibility.

```
                 Corda Node
                     │
     ┌───────────────┼───────────────┐
     │               │               │
  Vault         Contracts        Flows
     │               │               │
     └───────────────┼───────────────┘
                     │
              Transactions
                     │
                 Notary Service
```

Every component performs exactly one job.

This separation makes Corda easier to maintain, test, and scale.

---

# Why is Corda Modular?

Traditional applications often mix business logic, storage, networking, and security.

Corda separates these responsibilities into dedicated components.

Instead of one large system doing everything, each component has a clearly defined purpose.

Benefits include:

- Easier maintenance.
- Better security.
- Clear separation of concerns.
- Independent testing.
- Improved scalability.

This modular design is one of Corda's greatest strengths.

---

# 2. High-Level Architecture

A complete enterprise application usually contains several organisations.

Example:

```
           Bank A

              │

              │

      ┌───────▼────────┐
      │   Corda Node   │
      └───────┬────────┘
              │
      ┌───────┼────────────┐
      │       │            │
   Vault   Contracts     Flows
      │       │            │
      └───────┼────────────┘
              │
        Signed Transaction
              │
           Notary
              │
      ┌───────▼────────┐
      │   Bank B Node  │
      └────────────────┘
```

Notice something important.

There is **no blockchain** connecting everyone.

Instead,

nodes communicate directly.

---

# 3. Corda Node

## What is a Node?

A Node represents an organisation.

Examples:

- HSBC
- Barclays
- JP Morgan
- Insurance Company
- Government Agency

Each organisation runs its own node.

Think of it as the company's digital representative.

---

## Responsibilities

A Node:

- stores data
- executes contracts
- runs flows
- signs transactions
- communicates with other nodes
- hosts APIs
- maintains its vault

Everything an organisation does happens through its node.

---

## Real World Example

Imagine three banks.

```
HSBC

Barclays

Lloyds
```

Each bank owns:

```
Own Database

Own Vault

Own Smart Contracts

Own Identity

Own Private Key
```

No one controls another organisation's node.

---

# Internal Structure of a Node

```
+--------------------------------------+
|             Corda Node               |
+--------------------------------------+
| RPC Server                           |
| Flow Hospital                        |
| Vault                                |
| Identity Service                     |
| Messaging Service                    |
| Contract Verification                |
| Scheduler                            |
| Network Map Cache                    |
+--------------------------------------+
```

Every module has a specific purpose.

We'll explore these throughout this chapter.

---

# 4. Network Map

Imagine joining a company.

First question:

"Who works here?"

Corda answers that through the Network Map.

It stores:

- node identities
- addresses
- certificates
- legal names

Think of it as the company's phone book.

```
Network Map

↓

Bank A

↓

Bank B

↓

Insurance Company

↓

Regulator

↓

Notary
```

Without the Network Map, nodes would not know where to send transactions.

---

# Why is it Needed?

Suppose Bank A wants to communicate with Bank B.

Without a network map,

```
Where is Bank B?

Unknown.
```

With a network map,

```
Bank B

IP Address

Identity

Certificate
```

Everything required for secure communication is already available.

---

# Real Enterprise Example

A trade finance network contains:

- Banks
- Shipping companies
- Customs
- Insurance companies

The Network Map allows every participant to discover authorised members securely.

---

# 5. Identity

One of Corda's defining features is **legal identity**.

Bitcoin identity:

```
0x89AF3...
```

Corda identity:

```
O=HSBC,
L=London,
C=GB
```

The identity represents a real legal organisation.

---

## Why?

Financial institutions cannot transact anonymously.

Regulations require:

- Know Your Customer (KYC)
- Anti-Money Laundering (AML)
- Auditing
- Compliance

Every participant must be identifiable.

---

# Certificates

Every node owns certificates.

```
Root CA

↓

Network CA

↓

Node Certificate

↓

Legal Identity
```

Certificates allow nodes to trust each other.

---

# Why Certificates?

Suppose someone creates:

```
Fake Bank
```

Without certificates,

anyone could join.

Certificates prevent unauthorised participants from entering the network.

---

# Architecture Summary So Far

```
Organization

↓

Node

↓

Identity

↓

Certificate

↓

Network Map

↓

Secure Communication
```

Everything starts with trusted identity.

---

# 6. States

## What is a State?

A **State** is the most fundamental concept in Corda.

Everything on the ledger is represented as a State.

Think of a State as a **business fact**.

Examples:

- A loan
- A house ownership record
- A car title
- An insurance policy
- A purchase order
- Cash
- An invoice
- A trade agreement

Unlike a traditional database where records are updated in place, Corda never modifies an existing State.

States are **immutable**.

---

## Why are States Immutable?

Imagine a bank issues a loan.

```
Customer: Abhay

Amount: £250,000

Status : Active
```

Months later, the customer repays £20,000.

Should the original loan record change?

Traditional databases would simply execute:

```
UPDATE Loan
SET Amount = 230000
```

The original value disappears.

In financial systems, this is dangerous.

Auditors often ask:

> "What exactly was the original agreement?"

If data is overwritten, that information is lost.

Corda avoids this completely.

---

## Corda's Approach

Instead of updating,

it creates a **new State**.

```
Loan State v1

↓

Consumed

↓

Loan State v2
```

Nothing is deleted.

Nothing is overwritten.

History is preserved forever.

---

## Real World Example

Suppose Alice buys a house.

Initial State

```
House

Owner : Builder
```

After purchase

```
House

Owner : Alice
```

Did Corda update the owner?

No.

Instead:

```
Old State

Consumed

↓

New State Created
```

This is known as the **UTXO (Unspent Transaction Output) model**.

Every business fact evolves through new immutable states.

---

# Anatomy of a State

Every State usually contains:

```
Loan

Loan ID

Borrower

Lender

Amount

Interest Rate

Participants
```

Example:

```kotlin
data class LoanState(

    val amount: Int,

    val borrower: Party,

    val lender: Party,

    override val participants: List<AbstractParty>

): ContractState
```

Notice there are **no methods**.

A State is simply data.

Business rules belong elsewhere.

---

# Participants

Every State contains participants.

Why?

Because Corda needs to know:

Who should receive this State?

Example

```
Borrower

↓

Bank

↓

Insurance Company
```

Only these participants receive the finalised transaction.

Other organisations never see it.

---

# State Lifecycle

Every State follows the same lifecycle.

```
Created

↓

Active

↓

Consumed

↓

Archived
```

A consumed State can never be used again.

---

## Example

Invoice

```
Created

↓

Pending

↓

Paid

↓

Consumed
```

Instead of modifying

```
Status = Paid
```

Corda creates

```
Invoice State v2
```

---

# Why is Immutability Important?

Benefits include:

- Complete audit history.
- Easier debugging.
- No accidental modification.
- Strong cryptographic integrity.
- Regulatory compliance.
- Better transaction traceability.

This is one of the biggest differences between enterprise ledgers and relational databases.

---

# 7. StateRef

If States never change,

how does Corda know which one to use?

Answer:

StateRef.

---

## What is a StateRef?

A StateRef is the unique address of a State.

Think of it like a house address.

```
House

↓

Street

↓

City

↓

Country
```

Every house has a unique location.

Similarly,

every State has a unique identifier.

---

A StateRef contains

```
Transaction Hash

+

Output Index
```

Example

```
8D21F...

Output 0
```

or

```
Transaction

↓

Hash

↓

Output #1
```

Together, they uniquely identify one output State.

---

## Why Not Use Loan ID?

Imagine two different banks accidentally create

```
Loan ID = 101
```

Which loan is correct?

StateRef solves this because the transaction hash is globally unique.

---

# Real Example

Transaction

```
Outputs

0 -> Loan

1 -> Cash

2 -> Receipt
```

Loan's StateRef

```
Hash

+

0
```

Cash's StateRef

```
Hash

+

1
```

Receipt

```
Hash

+

2
```

Simple.

Every State has its own address.

---

# 8. Commands

States describe

"What exists."

Commands describe

"What is happening."

---

Example

```
Loan

↓

Issue
```

Issue is the Command.

Loan is the State.

---

Think about a banking system.

Possible commands

```
Issue Loan

Approve Loan

Close Loan

Transfer Loan
```

Each command represents an action.

---

## Why Separate Commands?

Without commands,

the Contract would have to guess.

Example

```
Loan Amount Changed
```

Was it

- repayment?

- correction?

- transfer?

- refinance?

Commands remove ambiguity.

---

Example

```kotlin
class Commands {

    class Issue: CommandData

    class Transfer: CommandData

    class Close: CommandData
}
```

---

# Real World Example

Car Ownership

State

```
Car

Owner = Alice
```

Command

```
Transfer
```

Contract now knows

```
Transfer ownership

NOT

Create new vehicle.
```

---

# 9. Contracts

States contain data.

Commands describe intent.

Contracts enforce rules.

Think of a Contract as a referee.

```
Player

↓

Action

↓

Referee

↓

Allowed?

↓

Yes / No
```

---

## Example

Loan Contract

Rules

```
Loan Amount > 0

Borrower Exists

Lender Exists

Both Signatures Required
```

If any rule fails,

the transaction fails.

---

## verify()

Every Contract implements

```kotlin
override fun verify(tx: LedgerTransaction)
```

This is where every validation rule is written.

Example

```kotlin
require(amount > 0)
```

or

```kotlin
require(signers.contains(bank))
```

The transaction is accepted only if **every rule passes**.

---

# Why Separate Contracts from States?

Imagine if rules were stored inside the State.

Every time the data changed,

business logic would also change.

Instead,

Corda separates:

```
Data

↓

State

Rules

↓

Contract
```

This separation makes applications easier to test, maintain, and evolve.

---

# Relationship Between Components

At this point, the architecture looks like this:

```
            State
              │
              │ Contains
              ▼
      Business Information
              │
              │ Referenced by
              ▼
          StateRef
              │
              │ Used in
              ▼
         Transaction
              ▲
              │
      Executes Command
              │
              ▼
          Contract
      verifies rules
```
---

# 10. Transactions

## Introduction

If a **State** represents a business fact, then a **Transaction** represents a business event.

Think about everyday business.

```
Account Exists
```

Nothing happened.

Now imagine

```
Transfer £10,000
```

Something happened.

That "something" is a **Transaction**.

In Corda, every modification to the ledger occurs through a transaction.

No State can appear, disappear, or change without one.

---

# Real World Example

Imagine a bank issues a mortgage.

Before

```
No Loan Exists
```

After

```
Loan Exists
```

How did that happen?

A transaction created it.

Likewise

```
Loan Closed
```

Another transaction consumed it.

Everything is driven by transactions.

---

# Transaction Structure

Every transaction contains several important components.

```
               Transaction
                    │
    ┌───────────────┼───────────────┐
    │               │               │
 Inputs         Outputs         Commands
    │               │               │
    └───────────────┼───────────────┘
                    │
               Attachments
                    │
                Time Window
                    │
                Signatures
```

Each component has a specific responsibility.

---

# Inputs

Inputs represent existing States.

Example

```
Loan State v1
```

The transaction consumes this State.

Think of it as

```
Previous Version
```

---

# Outputs

Outputs represent newly created States.

```
Loan State v2
```

After the transaction finishes,

only the output remains active.

---

Example

```
Input

Loan £100,000

↓

Repayment

↓

Output

Loan £90,000
```

Notice

the old loan was not updated.

It was replaced.

---

# Commands

Commands explain

WHY

the transaction exists.

Example

```
Issue

Transfer

Repay

Close
```

Without commands,

contracts would never know what validation rules to execute.

---

# Signatures

Every required participant signs the transaction.

Example

```
Borrower

✓

Bank

✓

Notary

✓
```

Missing one signature?

Transaction fails.

---

# Transaction Types

Corda internally uses several transaction representations.

```
WireTransaction

↓

LedgerTransaction

↓

SignedTransaction
```

Understanding the difference is essential.

---

## WireTransaction

The raw transaction.

Contains

- inputs
- outputs
- commands
- attachments

No signatures yet.

Think of it as

```
Draft Contract
```

---

## LedgerTransaction

A verified transaction.

The Contract receives a LedgerTransaction inside

```kotlin
verify(tx: LedgerTransaction)
```

This allows validation.

Example

```
Input Amount

↓

Output Amount

↓

Verify
```

---

## SignedTransaction

Once everyone signs,

the transaction becomes

```
SignedTransaction
```

Only SignedTransactions can be finalised.

Think of it as

```
Contract Signed by Everyone
```

---

# Transaction Flow

```
WireTransaction

↓

Contract Verification

↓

LedgerTransaction

↓

Signatures Added

↓

SignedTransaction

↓

Notary

↓

Finality
```

---

# Why Multiple Transaction Types?

Because each stage serves a different purpose.

```
Draft

↓

Verification

↓

Approval

↓

Final Record
```

Very similar to real legal contracts.

---

# 11. Vault

The Vault is one of the most misunderstood concepts.

Many beginners think

Vault

=

Blockchain

Wrong.

---

## What is the Vault?

The Vault is

YOUR NODE'S DATABASE.

It stores

only

States relevant to your node.

Imagine

```
Bank A

Vault
```

Contains

```
Loans

Cash

Invoices

Assets
```

Bank B has a different Vault.

---

# Important

There is

NO

global database.

Each organisation owns its own Vault.

---

# Vault Example

```
Bank A

Vault

Loan A

Loan B

Cash

Invoice
```

Bank B

```
Vault

Loan C

Loan D

Insurance
```

Neither Vault contains everything.

---

# Why?

Privacy.

Banks should never see each other's confidential information.

---

# Vault States

The Vault stores

```
UNCONSUMED

and

CONSUMED
```

States.

---

Example

```
Loan

Created

↓

Repaid

↓

Consumed
```

The original State still exists.

It is simply marked as consumed.

---

# Vault Query

Applications search the Vault.

Example

```
Find all Active Loans

↓

Vault Query

↓

Results
```

Example Kotlin

```kotlin
serviceHub.vaultService.queryBy<LoanState>()
```

---

# Vault Architecture

```
Application

↓

Vault Service

↓

Database

↓

States
```

Developers never access the database directly.

Everything goes through the Vault Service.

---

# Why a Vault Instead of Blockchain?

Traditional blockchain

```
Download Entire Ledger
```

Corda

```
Store Only Relevant States
```

Benefits

- Faster
- Smaller storage
- Better privacy
- Better performance

---

# 12. Flows

Imagine buying a house.

Steps

```
Call Bank

↓

Verify Identity

↓

Approve Loan

↓

Sign Papers

↓

Transfer Ownership
```

Many steps.

Should developers manually implement networking?

No.

Corda introduces

Flows.

---

## What is a Flow?

A Flow is an automated workflow between two or more nodes.

Think of it as

```
Business Process Automation.
```

---

Example

```
Issue Loan
```

Instead of

```
Send Email

↓

Wait

↓

Call Bank

↓

Wait

↓

Verify

↓

Sign
```

Everything happens automatically.

---

# Flow Responsibilities

Flows

- send messages
- receive messages
- request signatures
- call contracts
- communicate with notaries
- record transactions

Everything happens here.

---

# Flow Example

```
Customer

↓

Loan Request

↓

Bank Flow Starts

↓

Creates Transaction

↓

Signs

↓

Requests Customer Signature

↓

Notary

↓

Finality

↓

Complete
```

---

# Flow Lifecycle

```
Start

↓

Build Transaction

↓

Verify

↓

Sign

↓

Collect Signatures

↓

Notary

↓

Record

↓

Finish
```

Almost every CorDapp follows this lifecycle.

---

# Flow Types

Two kinds exist.

---

## Initiating Flow

Starts the communication.

Example

```
Bank

↓

Issue Loan
```

---

## Responder Flow

Runs automatically on the receiving node.

Example

```
Customer

↓

Receive Request

↓

Verify

↓

Sign
```

Together

```
Initiator

↔

Responder
```

Form one business workflow.

---

# Why Flows Matter

Without Flows,

developers would have to build

- sockets
- messaging
- retries
- acknowledgements
- serialization
- communication protocols

Corda provides this infrastructure out of the box.

---

# Component Relationship

At this point, the architecture looks like this:

```
          Flow
            │
            ▼
     Build Transaction
            │
            ▼
        Contract
            │
            ▼
        Verification
            │
            ▼
      SignedTransaction
            │
            ▼
          Vault
```
---

# 13. Sessions

## Introduction

Before two organizations can exchange transactions, they need a secure communication channel.

In Corda, this communication channel is called a **Session**.

Think of a Session as a secure phone call between two organizations.

```
Bank A
   │
   │ Secure Communication
   ▼
Bank B
```

Without Sessions, Nodes cannot exchange transactions, signatures, or business data.

---

# Why Sessions?

Imagine Bank A wants to issue a loan.

Without Sessions:

```
Create Transaction

↓

Email Transaction

↓

Wait

↓

Customer Signs

↓

Email Back

↓

Bank Checks

↓

Repeat
```

Everything becomes manual.

Corda automates this communication.

---

# Flow vs Session

Many beginners confuse these.

A simple analogy:

| Flow | Session |
|------|----------|
| Business Process | Communication Channel |

Example

```
Issue Loan Flow

↓

Open Session

↓

Exchange Messages

↓

Receive Signature

↓

Finalize
```

Flows perform business logic.

Sessions carry the messages.

---

# Session Lifecycle

```
Initiating Flow

↓

initiateFlow()

↓

Secure Session Created

↓

Send Data

↓

Receive Data

↓

Close Session
```

Sessions exist only while a Flow is running.

---

# Secure Communication

Every message exchanged during a Session is

- authenticated
- encrypted
- verified

Example

```
Bank

↓

Encrypted Message

↓

Customer
```

No third party can read the communication.

---

# Example Kotlin

```kotlin
val session = initiateFlow(otherParty)
```

Now both nodes can exchange messages.

---

# Why Sessions Matter

Without Sessions developers would have to implement

- TCP communication
- Authentication
- Encryption
- Serialization
- Retry Logic
- Error Recovery

Corda provides all of this automatically.

---

# 14. Notary

## Introduction

The Notary is the most misunderstood component in Corda.

Many people think

> "The Notary validates transactions."

This is **not** entirely true.

The Notary **does not validate business rules**.

Contracts do that.

The Notary validates **uniqueness**.

---

# The Double Spending Problem

Imagine Alice owns one house.

```
Alice

↓

House
```

Alice attempts

```
Sell House to Bob

AND

Sell Same House to Charlie
```

Which transaction is valid?

Without protection

Both might succeed.

This is called

**Double Spending**.

---

# Role of the Notary

The Notary ensures

```
Input State

Used Once

ONLY ONCE
```

If someone tries to reuse an already consumed State

```
Reject Transaction
```

Simple.

---

# Real Banking Example

Suppose

```
Loan State
```

already exists.

Someone attempts

```
Repay Loan

AND

Transfer Loan

using same Input State.
```

Only one transaction can succeed.

The second one is rejected.

---

# Notary Workflow

```
Build Transaction

↓

Verify Contract

↓

Collect Signatures

↓

Send to Notary

↓

Input Already Used?

      │

 ┌────┴────┐

 No       Yes

 │          │

Sign      Reject

 │

Return
```

Notice

The Notary never checks

```
Loan Amount

Interest Rate

Business Rules
```

Only

```
Has this State already been consumed?
```

---

# Why?

This separation improves scalability.

Contracts

↓

Business Logic

Notary

↓

Uniqueness

Single responsibility.

---

# Notary Types

## Validating Notary

Receives

Entire Transaction.

Can validate everything.

Pros

- More secure

Cons

- Lower privacy

---

## Non-Validating Notary

Receives only

Transaction IDs

Input References

Pros

- Better privacy

Cons

- Relies on Contract Verification by participants

Most enterprise networks use non-validating Notaries.

---

# Crash Fault Tolerant (CFT)

Protects against

```
Hardware Failure

Network Failure

Server Crash
```

Assumes nodes are honest.

Suitable for trusted consortiums.

---

# Byzantine Fault Tolerant (BFT)

Protects against

```
Malicious Nodes

Compromised Servers

Dishonest Participants
```

More secure

Higher resource usage.

---

# Notary Architecture

```
              Notary
                 │
      ┌──────────┼──────────┐
      │          │          │
Transaction  Check Inputs  Sign
      │          │          │
      └──────────┼──────────┘
                 │
           Return Signature
```

---

# 15. Attachments

Sometimes business transactions need additional documents.

Examples

- PDF Agreement
- Insurance Policy
- Invoice
- Purchase Order
- Engineering Drawing

Instead of storing these documents inside States

Corda uses

Attachments.

---

# Example

```
Loan State

↓

PDF Loan Agreement

↓

Attachment
```

The transaction references the document.

The document itself remains unchanged.

---

# Why Attachments?

Imagine

```
100 MB PDF
```

Should every State contain that?

No.

Instead

```
State

↓

Hash

↓

Attachment
```

Only the hash is referenced.

This improves efficiency.

---

# Common Attachment Types

```
PDF

ZIP

JAR

Images

Documents
```

---

# Benefits

- Tamper detection
- Reusable
- Efficient storage
- Cryptographically verified

---

# 16. Time Windows

Some transactions are only valid during a specific period.

Example

```
Cheque

Valid

1 Jan

↓

31 Jan
```

After January

Invalid.

---

# Corda Solution

Transactions can define

Time Windows.

Example

```
Valid From

09:00

↓

Valid Until

17:00
```

The Notary verifies the transaction falls within the allowed window.

---

# Real Example

Government Bond

```
Issued

↓

Expires

↓

Cannot Trade
```

Time Windows enforce these business rules.

---

# 17. Complete End-to-End Transaction Lifecycle

Now let's connect everything we've learned.

Imagine Bank A issues a loan to Alice.

```
Customer Applies
        │
        ▼
Initiating Flow Starts
        │
        ▼
LoanState Created
        │
        ▼
Transaction Built
        │
        ▼
Command Added
        │
        ▼
Contract verify()
        │
        ▼
Bank Signs
        │
        ▼
Session Opened
        │
        ▼
Customer Signs
        │
        ▼
Send to Notary
        │
        ▼
Input Already Used?
        │
   ┌────┴────┐
   │         │
  No        Yes
   │         │
   ▼         ▼
Sign     Reject
   │
   ▼
Finality Flow
   │
   ▼
Vault Updated
   │
   ▼
Transaction Complete
```

This is the complete lifecycle of almost every Corda transaction.

---

# 18. Complete Architecture Diagram

```
                         Corda Network
────────────────────────────────────────────────────────────

        Bank A Node                     Bank B Node

+----------------------+        +----------------------+
|                      |        |                      |
|      RPC Client      |        |      RPC Client      |
|                      |        |                      |
|----------------------|        |----------------------|
|       Flows          |<------>|       Flows          |
|----------------------| Session|----------------------|
|      Contracts       |        |      Contracts       |
|----------------------|        |----------------------|
|       Vault          |        |       Vault          |
|----------------------|        |----------------------|
|     Identity         |        |     Identity         |
|----------------------|        |----------------------|
| Messaging Layer      |        | Messaging Layer      |
+----------┬-----------+        +-----------┬----------+
           │                                │
           └──────────────┬─────────────────┘
                          │
                    Notary Service
                          │
                 Uniqueness Consensus
```

---

# Architecture Summary

The following table summarises the purpose of each core component.

| Component | Responsibility |
|------------|---------------|
| Node | Represents an organisation on the network |
| State | Immutable business data |
| StateRef | Unique reference to a State |
| Command | Describes the intended action |
| Contract | Validates business rules |
| Transaction | Records a business event |
| Vault | Stores States relevant to the node |
| Flow | Automates business processes |
| Session | Secure communication channel |
| Notary | Prevents double spending |
| Attachment | Stores supporting documents |
| Time Window | Enforces time-based validity |

---

# Module Summary

You now understand the complete architecture of R3 Corda. Every CorDapp is built from the components covered in this chapter:

- **Nodes** host the application and represent organisations.
- **States** model immutable business facts.
- **StateRefs** uniquely identify those facts.
- **Commands** express the intended business action.
- **Contracts** validate business rules.
- **Transactions** transform input States into output States.
- **Vaults** store each node's local view of the ledger.
- **Flows** automate distributed business processes.
- **Sessions** provide secure communication between nodes.
- **Notaries** guarantee uniqueness and prevent double spending.
- **Attachments** associate external documents with transactions.
- **Time Windows** enforce temporal validity.

Together, these components create a secure, private, and enterprise-grade distributed ledger platform that is fundamentally different from traditional blockchain systems.

Flows orchestrate the entire process—they coordinate communication, build and verify transactions, collect signatures, and ensure the finalized transaction is recorded in each participant's Vault.
Everything we've covered so far forms the **foundation of every CorDapp**. States represent immutable business facts, StateRefs uniquely identify them, Commands express the intended business action, and Contracts enforce the rules that determine whether that action is valid.
By the end of this chapter, you'll understand how these building blocks interact to form a secure, private, and enterprise-grade distributed ledger application.
