---
tags:
  - pattern
  - architectural
gardening: 🌱
date: 2024-12-23
reference:
  - https://en.wikipedia.org/wiki/Entity_component_system
  - https://github.com/SanderMertens/ecs-faq
  - https://medium.com/ingeniouslysimple/entities-components-and-systems-89c31464240d
  - https://ajmmertens.medium.com/building-an-ecs-2-archetypes-and-vectorization-fe21690805f9
  - https://www.simplilearn.com/entity-component-system-introductory-guide-article#:~:text=add%20new%20features-,How%20Is%20ECS%20Different%20From%20OOP%3F,plain%20old%20data%20(POD)%20objects
---
## Overview

Entity Component System (ECS) is a software architectural pattern utilized primarily in video game development and other performance-critical applications. ECS effectively separates concerns by decomposing entities into data components and systems that operate on those components.

![](../../images/architecture/ecs-dark.png)

## What is it?

The behavior of an entity can be modified at runtime by systems that add, remove, or change components. This approach resolves the ambiguity issues associated with deep and wide inheritance hierarchies commonly found in Object-Oriented Programming, which can be challenging to understand, maintain, and extend. Many Entity Component System (ECS) approaches are highly compatible with data-oriented design techniques. In this setup, the data for all instances of a component is stored contiguously in physical memory, allowing for efficient memory access for systems that operate on multiple entities.

The Entity Component System (ECS) does not encounter the dependency issues commonly found in Object-Oriented Programming because components are straightforward data containers that do not have dependencies. Each system typically queries the set of components that an entity must possess for the system to function. For instance, a rendering system might require the model, transform, and drawable components. When the system executes, it applies its logic to any entity that has all of these necessary components, while simply skipping over other entities without the need for complex dependency trees. 

However, this simplicity can also lead to hidden bugs, as propagating values between systems through components can be difficult to debug. ECS can be effectively used in scenarios where uncoupled data needs to be associated with a particular lifetime.

### Pros

- **Decoupling/SoC:** The separation of logic from data simplifies modifications and extensions. Systems handle logic, while components store data.
- **Performance:** Systems can function efficiently by using arrays of components, which enables better utilization of CPU cache locality and parallelism.
- **Flexibility/Scalability:** Introducing new behaviors is straightforward—simply add new systems and components.
- **Reusability:** Components and systems can frequently be reused across different entities and projects.

### Cons

- **Learning Curve:** Effectively understanding and implementing ECS can take time.
- **Overhead:** ECS can introduce unnecessary complexity for smaller projects.
- **Debugging:** Tracking issues across entities, components, and systems may be challenging.

## Entity

Entities act as containers for components and serve as unique identifiers, often represented as integers or UUIDs, that signify "objects" or "things" in the world. Entities do not contain any data or logic; they merely function as references.

```text
Entity 1: Player
Entity 2: Enemy
Entity 3: Buff
```

```ts
// Object-based example

type EcsObject = { name: string };
type Entity = EcsObject & { id: number };

const entityFactory = (id: number): Entity => ({ name: 'Entity', id });
```

```ts
// Class-based example

class Entity {
  id: number;

  constructor(id: number) {
    this.id = id;
  }
}
```

## Component

Components are data containers associated with entities that define their properties or characteristics. Each component typically represents one aspect of behavior or state, such as position, velocity, or health. Components contain no logic—only data.

```text
Position: { x: 10, y: 20 }
Velocity: { dx: 5, dy: 0 }
Health: { current: 100, max: 100 }
```

A single entity might have several components associated with it.

```text
Entity 1: [Position, Velocity, Health]
Entity 2: [Position, Health]
```

```ts
// Object-based example

type Position = EcsObject & { x: number; y: number; };
const positionFactory = (x: number, y: number): Position => ({
  name: 'Position', x, y
});

type Velocity = EcsObject & { dx: number; dy: number; }
const velocityFactory = (dx: number, dy: number): Velocity => ({
  name: 'Velocity', dx, dy
});

type Health = EcsObject & { current: number; max: number; }
const healthFactory = (current: number, max: number): Health => ({
  name: 'Health', current, max
});

type EntityComponents = {
  Position?: Position;
  Velocity?: Velocity;
  Health?: Health;
};
type Components = Position | Velocity | Health;
```

```ts
// Class-based example

class Component {
  name: string;
  
  constructor() {
    this.name = this.constructor.name;
  }
}

class Position extends Component {
  x: number;
  y: number;

  constructor(x: number, y: number) {
    super();
    this.x = x;
    this.y = y;
  }
}

class Velocity extends Component {
  dx: number;
  dy: number;

  constructor(dx: number, dy: number) {
    super();
    this.dx = dx;
    this.dy = dy;
  }
}

class Health extends Component {
  current: number;
  max: number;

  constructor(current: number, max: number) {
    super();
    this.current = current;
    this.max = max;
  }
}

type EntityComponents = {
  Position?: Position;
  Velocity?: Velocity;
  Health?: Health;
};
```

## System

Systems are where the logic resides. They process entities that have specific combinations of components. Systems implement game rules, update state, and perform actions. A system queries entities with the required components and processes them.

```text
// Psuedo Movement System
foreach entity with (Position, Velocity):
  position.x += velocity.dx
  position.y += velocity.dy

// Psuedo Health System
foreach entity with (Health):
  if health.current <= 0:
    destroy(entity)
```

```ts
// Object-based example

type QueryFn = (...componentTypes: (keyof EntityComponents)[]) => number[];
type GetComponentFn = <T>(
  entity: Entity,
  componentType: keyof EntityComponents
) => T | null;
type System = (query: QueryFn, getComponent: GetComponentFn) => void;

const updateMovement: System = (
  query: QueryFn,
  getComponent: GetComponentFn
) => {
  const entities = query('Position', 'Velocity');

  for (const entityId of entities) {
    const position = getComponent<Position>(
      { id: entityId, name: 'Entity' },
      'Position'
    );
    const velocity = getComponent<Velocity>(
      { id: entityId, name: 'Entity' },
      'Velocity'
    );

    if (position && velocity) {
      position.x += velocity.dx;
      position.y += velocity.dy;
      
      console.log(
        `Entity ${entityId} moved to (${position.x}, ${position.y})`
      );
    }
  }
};

const updateHealth: System = (
  query: QueryFn,
  getComponent: GetComponentFn
) => {
  const entities = query('Health');

  for (const entityId of entities) {
    const health = getComponent<Health>(
      { id: entityId, name: 'Entity' },
      'Health'
    );

    if (health) {
      if (health.current <= 0) {
        console.log(`Entity ${entityId} is dead!`);
      }
    }
  }
};
```

```ts
// Class-based example

type QueryFn = (...componentTypes: (keyof EntityComponents)[]) => number[];
type GetComponentFn = <T>(
  entity: Entity,
  componentType: keyof EntityComponents
) => T | null;

class System {
  name: string;

  constructor() {
    this.name = this.constructor.name;
  }

  update(query: QueryFn, getComponent: GetComponentFn) {
    throw new Error('update method must be implemented by subclass');
  }
}

class MovementSystem extends System {
  constructor() {
    super();
  }

  update(query: QueryFn, getComponent: GetComponentFn) {
    const entities = query('Position', 'Velocity');

    for (const entityId of entities) {
      const position = getComponent<Position>(
	    new Entity(entityId),
	    'Position'
	  );
      const velocity = getComponent<Velocity>(
        new Entity(entityId),
        'Velocity'
      );

      if (position && velocity) {
        position.x += velocity.dx;
        position.y += velocity.dy;
        
        console.log(
          `Entity ${entityId} moved to (${position.x}, ${position.y})`
        );
      }
    }
  }
}

class HealthSystem extends System {
  constructor() {
    super();
  }

  update(query: QueryFn, getComponent: GetComponentFn) {
    const entities = query('Health');

    for (const entityId of entities) {
      const health = getComponent<Health>(new Entity(entityId), 'Health');

      if (health) {
        if (health.current <= 0) {
          console.log(`Entity ${entityId} is dead!`);
        }
      }
    }
  }
}
```

## World

The world (sometimes called Engine) ties everything together. It manages entities, components, and systems.

```ts
// Object-based example

type World = {
  entities: Map<number, Entity>;
  components: Map<string, Map<number, Components>>;
  systems: System[];
  currentEntityId: number;
  createEntity: () => Entity;
  deleteEntity: (entity: Entity) => void;
  addComponent: <T extends keyof EntityComponents>(
    entity: Entity, 
    componentType: T,
    componentInstance: EntityComponents[T]
  ) => void;
  getComponent: <T>(
    entity: Entity,
    componentType: keyof EntityComponents
  ) => T | null;
  removeComponent: (
    entity: Entity,
    componentType: keyof EntityComponents
  ) => void;
  query: (...componentTypes: (keyof EntityComponents)[]) => number[];
  addSystem: (sysem: System) => void;
  update: () => void;
};

const createWorld = (): World => ({
  entities: new Map<number, Entity>(),
  components: new Map<string, Map<number, Components>>(),
  systems: [],
  currentEntityId: 0,

  createEntity() {
    const entity = entityFactory(this.currentEntityId++);
    this.entities.set(entity.id, entity);

    return entity;
  },

  deleteEntity(entity) {
    this.entities.delete(entity.id);

    for (const [_, entityMap] of this.components.entries()) {
      entityMap.delete(entity.id);
    }
  },

  addComponent(entity, componentType, componentInstance) {
    if (!this.components.has(componentType)) {
      this.components.set(componentType, new Map());
    }
    
    this.components
      .get(componentType)
      ?.set(entity.id, componentInstance as Components);
  },

  getComponent<T>(entity: Entity, componentType: keyof EntityComponents) {
    return (
      this.components.get(componentType)?.get(entity.id) || null
    ) as T | null;
  },

  removeComponent(entity, componentType) {
    this.components.get(componentType)?.delete(entity.id);
  },

  query(...componentTypes: (keyof EntityComponents)[]) {
    const sets = componentTypes.map((type) => {
      return new Set(this.components.get(type)?.keys() || []);
    });

    return [
      ...sets.reduce((a, b) => new Set([...a].filter((x) => b.has(x))))
    ];
  },

  addSystem(system) {
    this.systems.push(system);
  },

  update() {
    for (const system of this.systems) {
      system(this.query.bind(this), this.getComponent.bind(this));
    }
  },
});
```

```ts
// Class-based example

class World {
  entities: Map<number, Entity>;
  components: Map<string, Map<number, Component>>;
  systems: System[];
  currentEntityId: number;

  constructor() {
    this.entities = new Map();
    this.components = new Map();
    this.systems = [];
    this.currentEntityId = 0;
  }

  createEntity() {
    const entity = new Entity(this.currentEntityId++);
    this.entities.set(entity.id, entity);

    return entity;
  }

  deleteEntity(entity: Entity) {
    this.entities.delete(entity.id);

    for (const [_, entityMap] of this.components.entries()) {
      entityMap.delete(entity.id);
    }
  }

  addComponent(entity: Entity, component: Component) {
    const componentType = component.constructor.name;

    if (!this.components.has(componentType)) {
      this.components.set(componentType, new Map());
    }
    
    this.components.get(componentType)?.set(entity.id, component);
  }

  getComponent<T>(entity: Entity, componentType: keyof EntityComponents) {
    return (
      this.components.get(componentType)?.get(entity.id) || null
    ) as T | null;
  }

  removeComponent(entity: Entity, componentType: keyof EntityComponents) {
    this.components.get(componentType)?.delete(entity.id);
  }

  query(...componentTypes: (keyof EntityComponents)[]) {
    const sets = componentTypes.map((type) => {
      return new Set(this.components.get(type)?.keys() || []);
    });

    return [
      ...sets.reduce((a, b) => new Set([...a].filter((x) => b.has(x))))
    ];
  }

  addSystem(system: System) {
    this.systems.push(system);
  }

  update() {
    for (const system of this.systems) {
      system.update(this.query.bind(this), this.getComponent.bind(this));
    }
  }
}
```

### Example Usage

```ts
// Object-based example

// Create the world
const world = createWorld();

// Create entities
const player = world.createEntity();
const enemy = world.createEntity();

// Add components to entities
world.addComponent(player, 'Position', positionFactory(0, 0));
world.addComponent(player, 'Velocity', velocityFactory(1, 1));
world.addComponent(player, 'Health', healthFactory(100, 100));

world.addComponent(enemy, 'Position', positionFactory(10, 10));
world.addComponent(enemy, 'Health', healthFactory(0, 50)); // Dead enemy

// Add systems
world.addSystem(updateMovement);
world.addSystem(updateHealth);

// Run the game loop
let elapsed = 0;

function gameLoop(deltaTime: number) {
  console.log('Updating world...');
  world.update();
  elapsed += deltaTime;

  // Stop after 5 seconds
  if (elapsed > 5000) {
    clearInterval(interval);
  }
}

// Run at ~60 FPS
const interval = setInterval(() => gameLoop(1000 / 60), 1000 / 60);
```

```ts
// Class-based exmaple

// Create the world
const world = new World();

// Create entities
const player = world.createEntity();
const enemy = world.createEntity();

// Add components to entities
world.addComponent(player, new Position(0, 0));
world.addComponent(player, new Velocity(1, 1));
world.addComponent(player, new Health(100, 100));

world.addComponent(enemy, new Position(10, 10));
world.addComponent(enemy, new Health(0, 50)); // Dead enemy

// Add systems
world.addSystem(new MovementSystem());
world.addSystem(new HealthSystem());

// Run the game loop
let elapsed = 0;

function gameLoop(deltaTime: number) {
  console.log('Updating world...');
  world.update();
  elapsed += deltaTime;

  // Stop after 5 seconds
  if (elapsed > 5000) {
    clearInterval(interval);
  }
}

// Run at ~60 FPS
const interval = setInterval(() => gameLoop(1000 / 60), 1000 / 60);
```

With the output of both being:

```text
Updating world...
Entity 0 moved to (1, 1)
Entity 1 is dead!

Updating world...
Entity 0 moved to (2, 2)
Entity 1 is dead!

Updating world...
Entity 0 moved to (3, 3)
Entity 1 is dead!

//...for 5 seconds
```




----

**Optimizations**
_Sparse Component Arrays_
_Archetype Queries_
_Pooling Entities and Components_
_Threading with Workers_
_Data-Oriented Design_
_Event-Driven Systems_

**Full Example with Optimizations**