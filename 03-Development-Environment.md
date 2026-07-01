# Development Environment

> Learn how to set up a complete R3 Corda development environment, understand the purpose of each tool, configure your workstation, and prepare for building enterprise CorDapps.

---

# Learning Objectives

After completing this chapter, you will:

- Understand every tool required for Corda development.
- Configure a professional development environment.
- Learn how the Corda toolchain works together.
- Understand the structure of a CorDapp project.
- Run and debug your first Corda application.

---

# 1. Introduction

Enterprise applications are fundamentally different from small scripts or standalone programs.

A Corda application consists of multiple layers:

```
Developer
      │
      ▼
 IntelliJ IDEA
      │
      ▼
 Kotlin / Java
      │
      ▼
 Gradle
      │
      ▼
 CorDapp
      │
      ▼
 Corda Node
      │
      ▼
 Enterprise Network
```

Each component plays a specific role.

Before writing any code, it's important to understand why these tools are required.

---

# 2. Development Environment Overview

The complete Corda development stack looks like this:

```
+------------------------------------------------------+
|                 Developer Workstation                |
+------------------------------------------------------+
|                                                      |
|  IntelliJ IDEA                                       |
|        │                                             |
|        ▼                                             |
|  Kotlin / Java                                       |
|        │                                             |
|        ▼                                             |
|  Gradle Build System                                 |
|        │                                             |
|        ▼                                             |
|  CorDapp Project                                     |
|        │                                             |
|        ▼                                             |
|  Corda Node                                          |
|        │                                             |
|        ▼                                             |
|  Local Corda Network                                 |
|                                                      |
+------------------------------------------------------+
```

Every CorDapp you build follows this workflow.

---

# 3. System Requirements

For a smooth development experience, the following minimum specifications are recommended:

| Component | Recommended |
|-----------|-------------|
| CPU | 4+ Cores |
| RAM | 16 GB |
| Storage | SSD with 20 GB free |
| Operating System | Windows 11, macOS, or Linux |
| Internet | Required for dependencies |

While Corda can run on lower specifications, additional memory improves IntelliJ performance and allows multiple nodes to run simultaneously.

---

# 4. Required Software

The standard Corda development environment includes:

| Software | Purpose |
|----------|---------|
| Java JDK | Runs the JVM and Corda platform |
| Kotlin | Primary language for CorDapps |
| Gradle | Dependency management and build automation |
| IntelliJ IDEA | Recommended IDE |
| Git | Version control |
| Corda Runtime | Executes nodes and CorDapps |

Each tool is discussed in detail below.

---

# 5. Java Development Kit (JDK)

## Why Java?

Corda is built on the Java Virtual Machine (JVM).

This means every Corda node executes inside a JVM.

```
CorDapp

↓

JVM

↓

Operating System
```

Without the JDK, Corda cannot run.

---

## Why Not Python or JavaScript?

Enterprise systems often require:

- Stability
- Long-term support
- Mature libraries
- Strong tooling
- High performance

The JVM ecosystem provides these characteristics.

---

## Java vs JVM

Many beginners think Java and JVM are the same.

They are not.

```
Java

↓

Compiled

↓

Bytecode

↓

JVM

↓

Operating System
```

The JVM executes bytecode.

Java is only one language that targets it.

---

# 6. Kotlin

## Why Kotlin?

Although Corda supports Java, Kotlin is the preferred language.

Reasons include:

- Concise syntax
- Null safety
- Excellent interoperability with Java
- Modern language features
- Official recommendation by the Corda team

---

## Kotlin in Corda

Typical Corda components written in Kotlin include:

- States
- Contracts
- Flows
- Services
- Tests

---

# Example

```kotlin
data class LoanState(
    val amount: Int,
    val borrower: Party
): ContractState
```

Notice how concise the code is compared to Java.

---

# 7. Gradle

## What is Gradle?

Gradle is the build automation system used by Corda.

It performs tasks such as:

- Compiling code
- Downloading dependencies
- Running tests
- Packaging CorDapps
- Creating JAR files

Think of Gradle as the project manager for your codebase.

---

# Gradle Workflow

```
Source Code

↓

Compile

↓

Resolve Dependencies

↓

Run Tests

↓

Package

↓

Deploy
```

Every time you build your project, Gradle coordinates these steps.

---

# Why Gradle?

Without a build tool, developers would manually:

- Download libraries
- Configure classpaths
- Compile source files
- Package artifacts

Gradle automates these tasks, ensuring consistent builds across environments.

---

# 8. IntelliJ IDEA

## Why IntelliJ?

JetBrains IntelliJ IDEA is the recommended IDE for Corda development because it provides:

- Kotlin support
- Gradle integration
- Intelligent code completion
- Integrated debugger
- Refactoring tools
- Built-in terminal

---

# Typical Development Workflow

```
Write Code

↓

Compile

↓

Run Tests

↓

Debug

↓

Commit to Git
```

All of these steps can be performed within IntelliJ.

---

# 9. Git

Version control is essential for enterprise development.

Git allows developers to:

- Track changes
- Collaborate
- Manage branches
- Review history
- Revert mistakes

A typical workflow:

```
Clone Repository

↓

Create Feature Branch

↓

Develop

↓

Commit

↓

Push

↓

Open Pull Request
```

---

# 10. Corda Runtime

The Corda Runtime is responsible for executing:

- Nodes
- Flows
- Contracts
- Transactions

Think of it as the engine that powers your CorDapp.

```
CorDapp

↓

Corda Runtime

↓

Node

↓

Network
```

---

# 11. Project Structure

A standard CorDapp project is organised into modules.

```
my-cordapp/

├── contracts/
├── workflows/
├── clients/
├── build.gradle
├── settings.gradle
└── gradle.properties
```

### contracts/

Contains:

- States
- Contracts
- Commands

### workflows/

Contains:

- Flows
- Business processes
- Sessions

### clients/

Contains:

- RPC clients
- REST integrations
- Command-line tools

---

# 12. Creating Your First CorDapp

High-level steps:

1. Create a Gradle project.
2. Add Corda dependencies.
3. Create the `contracts` module.
4. Create the `workflows` module.
5. Define a simple State.
6. Implement a Contract.
7. Write a Flow.
8. Build the project.
9. Run local nodes.

In the next module, we'll implement each of these steps.

---

# 13. Running a CorDapp

The execution pipeline looks like this:

```
Write Code

↓

Gradle Build

↓

Create JAR

↓

Start Nodes

↓

Deploy CorDapp

↓

Run Flows

↓

Observe Transactions
```

Understanding this lifecycle will help you troubleshoot issues as your applications grow.

---

# 14. Debugging

Common debugging tools include:

- IntelliJ debugger
- Breakpoints
- Log output
- Gradle console
- Node shell
- Flow Hospital (for failed flows)

Debugging distributed applications often involves examining logs from multiple nodes.

---

# 15. Common Errors

| Error | Cause | Solution |
|--------|-------|----------|
| JDK version mismatch | Unsupported Java version | Install the supported JDK |
| Gradle sync failed | Dependency issues | Refresh Gradle project |
| Node won't start | Configuration error | Check `node.conf` and logs |
| Missing dependencies | Build configuration | Verify `build.gradle` |
| Port already in use | Another process is listening | Change ports or stop the conflicting process |

---

# 16. Best Practices

- Use the officially supported JDK version for your Corda release.
- Prefer Kotlin for new CorDapps.
- Keep Gradle wrapper files under version control.
- Separate Contracts and Workflows into dedicated modules.
- Use Git from the beginning of the project.
- Commit small, meaningful changes frequently.
- Test changes on a local multi-node network before deployment.

---

# 17. Summary

A professional Corda development environment consists of several integrated tools:

- **Java** provides the runtime platform.
- **Kotlin** is the preferred programming language.
- **Gradle** automates builds and dependency management.
- **IntelliJ IDEA** offers a powerful development experience.
- **Git** manages source code history.
- **Corda Runtime** executes nodes and CorDapps.

Understanding how these tools interact is just as important as knowing how to install them.

---
# What's Next?

In **Module 04 – Build Your First CorDapp**, you'll create a complete CorDapp from scratch. You'll define States, write Contracts, implement Flows, configure Gradle, start local nodes, and execute your first end-to-end transaction on a Corda network.
