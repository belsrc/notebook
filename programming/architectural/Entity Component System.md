---
tags:
  - pattern
  - architectural
gardening: 🌱
reference:
  - https://en.wikipedia.org/wiki/Entity_component_system
  - https://github.com/SanderMertens/ecs-faq
  - https://medium.com/ingeniouslysimple/entities-components-and-systems-89c31464240d
  - https://ajmmertens.medium.com/building-an-ecs-2-archetypes-and-vectorization-fe21690805f9
  - https://www.simplilearn.com/entity-component-system-introductory-guide-article#:~:text=add%20new%20features-,How%20Is%20ECS%20Different%20From%20OOP%3F,plain%20old%20data%20(POD)%20objects
---
**Entity**: An entity represents a general-purpose object. In a game engine context every coarse game object is represented as an entity. For example, if you’re playing Skyrim, all the tangible, visible “things” in the game’s universe are entities. They contain no actual data or behaviors. Only consisting of a unique id.

**Component**: A component characterizes an _entity_ as possessing a particular aspect, and holds the data needed to model that aspect. For example, every game object that can take damage might have a Health component associated with its entity. Implementations typically use structs, classes, or associative arrays.

**System**: A system is a process (function) which acts on all entities with the desired components. For example, a physics system may query for entities having mass, velocity and position components, and iterate over the results doing physics calculations on the set of components for each entity.

The behavior of an entity can be changed at runtime by systems that add, remove or modify components. This eliminates the ambiguity problems of deep and wide inheritance hierarchies often found in Object Oriented Programming techniques that are difficult to understand, maintain, and extend. Common ECS approaches are highly compatible with, and are often combined with, data-oriented design techniques. Data for all instances of a component are contiguously stored together in physical memory, enabling efficient memory access for systems which operate over many entities.

The normal way to transmit data between systems is to store the data in components, and then have each system access the component sequentially. For example, the position of an object can be updated regularly. This position is then used by other systems. If there are a lot of different infrequent events, a lot of flags will be needed in one or more components. Systems will then have to monitor these flags every iteration, which can become inefficient. A solution could be to use the observer pattern. All systems that depend on an event subscribe to it. The action from the event will thus only be executed once, when it happens, and no polling is needed.

The ECS has no trouble with dependency problems commonly found in Object-oriented Programming since components are simple data buckets, they have no dependencies. Each system will typically query the set of components an entity must have for the system to operate on it. For example, a render system might register the model, transform, and drawable components. When it runs, the system will perform its logic on any entity that has all of those components. Other entities are simply skipped, with no need for complex dependency trees. However this can be a place for bugs to hide, since propagating values from one system to another through components may be hard to debug. ECS may be used where uncoupled data needs to be bound to a given lifetime.

The ECS uses composition, rather than inheritance trees. An entity will be typically made up of an ID and a list of components that are attached to it. Any game object can be created by adding the correct components to an entity. This allows the developer to easily add features to an entity, without any dependency issues. For example, a player entity could have a _bullet_ component added to it, and then it would meet the requirements to be manipulated by some _bulletHandler_ system, which could result in that player doing damage to things by running into them.

## Data Flow

We can break down the ECS data flow into the following steps:

- System: The system listens to outside events and publishes updates to the components.
- Component: The components listen to system events, then update their state.
- Entity: The entity gains behavior through the changes in component states.

So, a user presses the “right arrow” key while adventuring in a game world. The player input system detects the key press and updates the motion component. The motion system activates and “sees” that the entity’s motion is to the right, so it applies the Physics force accordingly. Then, the render system takes over and reads the entity’s current position, drawing it according to its new spatial definition.

![](../../images/architecture/ecs-dark.png)