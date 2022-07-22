### Architecture Diagram
<a href="cqrs_architecture_diagram.svg?sanitize=true" rel="cqrs diagram">![Diagram](cqrs_architecture_diagram.svg)</a>

##### `CQRS` - Command-Query Responsibility Segregation.

*  Commands (write requests) and queries (read requests) have segregated responsibilites. 
* Write requests and read requests are handled by different objects.
* Data storage can be further split up, having separate read and write stores.
* Separate read/write stores are often discussed in relation with CQRS, this is not CQRS itself. CQRS is just the first split of commands and queries.

##### `DTO` - Data transfer Object.

* An object that aggregates several data transfer calls into one.

##### `DDD` - Domain-Driven Design.

* An approach to development that leverages the Domain Model in application implementation.

##### `Domain` - The field for which a system is built.

* An application can span several different `Domain`s.
* _Ex:_ Shipping logistics, insurance sales, restaurant inventory, analysis of aerodynamics.

##### `Model` - A contextual implementation of a key component that makes up a `Domain`.

* _Ex:_ `LongHaulVehicle` for a shipping company. `LongHaulVehicle` is not a real truck, a model is only meant to capture properties relevant to the immediate context.
* `Domain Model` - a model relevant to the implementation of a `Domain`.

##### `Entity` - Object characterized by having an identity that is independent of their attribute values.

* Two Entities can have equivalent attributes yet still be distinct.

##### `Value Object` - an object with no separate identity solely characterized by their attribute values.

* It is common to make value types immutable. _Ex:_ A string in many languages is immutable and one must derive a new string value to change a present one.

##### `Aggregate` - A group of entities and value objects that belong together at all times.

* Treated as a single unit for the purpose of persistence
* Every transaction is scoped to a single `Aggregate`. The lifetimes of the components of an `Aggregate` are bounded by the lifetime of the entire `Aggregate`.
* An `Aggregate` leverages `Domain` side logic to handle `Command`s and `Event`s
* Has a state model within it that allows it to validate commands in order to uphold the intended business logic of the `Aggregate`.
* `Aggregate Root` - The parent `Entity` to all Entities and `Value Object`s within the relevant `Aggregate`
	* External objects may only reference the`Aggregate Root`.

##### `Event` - a representation of an outcome that took place in the `Domain`.

* Always named with a [past-participle](https://en.wikipedia.org/wiki/Participle#Forms) verb. _Ex:_ `OrderConfirmed`
* Oftentimes named in relation to an `Aggregate` or `Entity`.
* Represents something in the past, thus it can be considered a statement of fact and can be leveraged to make decisions in other parts of the system.

##### `Event Handler` - Recipient of an `Event`.

* Each `Event` type can have one or more `Event` handlers.
* The read model can be rebuilt from scratch any time, by running `Event Handler`s on all the `Event`s from the Event store.
* Triggers the action related to a successful `Event`.
* _Ex:_ The Event `OrderPlaced` triggers a send action of a confirmation email to the customer. The `Event Handler` takes care of the actual logic involved in sending the email.

##### `Event Sourcing` - The concept of restoring the current state of an object from all past relevant `Event`s.

* Events are stored in a transaction log aka an `Event Store`.
* Allows a system to be put into a prior state.
* In a queue of `Event`s, new `Event`s are added to the end of the queue; `Event`s are never removed or changed.
* Correcting erroneous `Event`s are done through new `Event`s.

##### `Event Bus` - Messaging system that supplies `Event`s to `Event Handler`s

##### `Event Store` - Repository of past `Event`s.

1. `Command Handler` replays previous events contained in the `Event Store` to _create_ an `Aggregate` of a `Entity` in its current state.
1. The `Aggregate` uses `Domain` side logic to emit new events `Event`s to the `Event Bus` and the `Event Store`.

##### `Projection` - a set of `Event` handlers that collectively build and mantain a `Read Model`

##### `Read Side` - Listens to `Event`s published from the write side. 

* Events are projected down as changes to a local model, and allows queries to be made on that model.

##### `Command` - Requested changes to the `Domain`.

* Always named with an [imperative mood](https://en.wikipedia.org/wiki/Imperative_mood) verb. _Ex:_ `ConfirmOrder`
* A command is not a statement of fact, only a request and thus may be reused.
* Commands are immutable because their expected usage is to be sent directly to the `Domain` model for processing.

##### `Command Handler` - Route `Command`s to the relevant `Aggregate` actions.

* Initializes `Aggregate` from previous events in the `Event Store`
* Calls specific action of the `Aggregate`
* Saves emitted `Event`s into the `Event Store`
* A result is either a successful application of the command or an exception.
* Common sequence of a command handler:
    1. Validate the command on its own merits.
    2. Validate the command within the `Aggregate`'s context.
    3. If validation is successful, _0..n_ `Event`s are emitted.
    4. Attempt to persist the new `Event`s. If there's a concurrency conflict during this step, either give up, or retry things.
* A Command Handler should be tied to only one `Aggregate`.

##### `Bounded Context` - A subset of the appication

* Contains its own `Domain`, code, persistence and `Ubiquitous Language`.
* Should have minimal interdependence from the rest of the sytem.

##### `Ubiquitous Language` - The terminology used by those involved in a `Domain` and its implementation.


### Sources:
[Microsoft CQRS Archive](https://github.com/MicrosoftArchive/cqrs-journey/wiki/Glossary)

[Edument CQRS getting started](http://www.cqrs.nu/Faq/)
