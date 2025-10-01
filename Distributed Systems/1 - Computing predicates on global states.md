## Introduction
A **distributed system** is a kind of computer system consisting of a set of interconnected processes whose communications between them happen only by message exchange.

A large class of problem in distributed computing (monitoring, detection, load balancing) can be cast as executing some notification or reaction when the state of the system satisfies certain conditions. Thus, the ability to construct a **global state** and evaluate a predicate over such a state constitutes the core of solutions to many problems in distributed computing.

The **global state** of a distributed system is the _union of the states_ of the individual processes, which do not shares memory but communicate solely through exchange of messages. 

>The fundamental problem is to ensure that a global state constructed in this manner is meaningful.

## Asynchronous distributed systems
Distributed system is composed of:
- a collection of **processes** $p_1, p_2, ..., p_n$
- **communication channels** between pair of them for message exchange
Two models:
- **Asynchronous** (realistic for actual systems):
	- no bound on processes speeds and on message delay
	- synchronized to local clock
	- communications only mechanism for synchronization
- **Synchronous**:
	- processes speeds and message delays bounded