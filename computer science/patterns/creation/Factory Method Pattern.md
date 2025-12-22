---
tags:
  - design-pattern
  - comp-sci
gardening: 🌿
date: 2025-12-21
reference:
  - https://softwaredesignpatterns.azurewebsites.net/eBooks/Design%20Patterns%20Elements%20of%20Reusable%20Object-Oriented%20Software.pdf
  - https://refactoring.guru/design-patterns/factory-method
---
## What & Why

You need to create objects without specifying their exact classes, allowing subclasses to decide which class to instantiate.

- Decouple object creation from object usage
- Support the Open/Closed Principle (open for extension, closed for modification)
- Enable framework code to work with user-defined classes
- Defer instantiation decisions to subclasses

## Structure Diagram

```
        ┌────────────────────────────┐
        │      Creator (Abstract)    │
        ├────────────────────────────┤
        │ + factoryMethod(): Product │
        │ + someOperation(): void    │
        └────────────────────────────┘
                       △
                       │ inherits
                       │
         ┌─────────────┴─────────────┐
         │                           │
┌────────┴───────────┐  ┌────────────┴─────────┐
│ ConcreteCreatorA   │  │ ConcreteCreatorB     │
├────────────────────┤  ├──────────────────────┤
│ + factoryMethod(): │  │ + factoryMethod():   │
│   ProductA         │  │   ProductB           │
└────────────────────┘  └──────────────────────┘
         │                         │
         │ creates                 │ creates
         ↓                         ↓
┌────────────────────┐  ┌──────────────────────┐
│    ProductA        │  │    ProductB          │
└────────────────────┘  └──────────────────────┘
         △                        △
         └────────────┬───────────┘
                      │
              ┌───────┴────────┐
              │    Product     │ (Interface)
              └────────────────┘
```

### Traditional Implementation

```typescript
// Product interface - what all created objects must implement
interface Document {
  open(): void;
  save(): void;
  close(): void;
  getType(): string;
}

// Concrete Products - actual implementations
class PDFDocument implements Document {
  private readonly filename: string;

  constructor(filename: string) {
    this.filename = filename;
  }

  public open(): void {
    console.log(`Opening PDF: ${this.filename}`);
  }

  public save(): void {
    console.log(`Saving PDF: ${this.filename}`);
  }

  public close(): void {
    console.log(`Closing PDF: ${this.filename}`);
  }

  public getType(): string {
    return 'PDF';
  }
}

class WordDocument implements Document {
  private readonly filename: string;

  constructor(filename: string) {
    this.filename = filename;
  }

  public open(): void {
    console.log(`Opening Word document: ${this.filename}`);
  }

  public save(): void {
    console.log(`Saving Word document: ${this.filename}`);
  }

  public close(): void {
    console.log(`Closing Word document: ${this.filename}`);
  }

  public getType(): string {
    return 'Word';
  }
}

// Creator (abstract base class)
// Defines the factory method and uses it in its operations
abstract class Application {
  // Factory Method - subclasses override to create specific products
  // This is the core of the pattern
  protected abstract createDocument(filename: string): Document;

  // Business logic that uses the factory method
  // This method relies on the factory method but doesn't know
  // which concrete Document class will be created
  public newDocument(filename: string): Document {
    const doc = this.createDocument(filename);
    doc.open();
    console.log(`Created new ${doc.getType()} document`);

    return doc;
  }

  public openExisting(filename: string): Document {
    const doc = this.createDocument(filename);
    doc.open();
    console.log(`Opened existing ${doc.getType()} document`);

    return doc;
  }
}

// Concrete Creators - implement the factory method
class PDFApplication extends Application {
  protected createDocument(filename: string): Document {
    return new PDFDocument(filename);
  }
}

class WordApplication extends Application {
  protected createDocument(filename: string): Document {
    return new WordDocument(filename);
  }
}

// Usage
const pdfApp = new PDFApplication();
const doc1 = pdfApp.newDocument('report.pdf');

doc1.save();
doc1.close();

const wordApp = new WordApplication();
const doc2 = wordApp.openExisting('letter.docx');

doc2.save();
doc2.close();
```

**Key Characteristics**:
- Abstract creator with abstract factory method
- Concrete creators override factory method
- Creator code uses products through interface
- Decouples creation from usage

## Modern Alternative

We achieve the same decoupling through:
1. **Higher-order functions** - Functions that return factory functions
2. **Function composition** - Combining creation with behavior
3. **Closures** - Capturing creation logic

```typescript
// Product type (data structure)
type Document = {
  readonly filename: string;
  readonly type: string;
};

// Operations on documents (separate from data)
type DocumentOperations = {
  open: (doc: Document) => void;
  save: (doc: Document) => void;
  close: (doc: Document) => void;
};

// Factory function type
type DocumentFactory = (filename: string) => Document;

// Concrete factories - simple functions
const createPDFDocument: DocumentFactory = (filename: string): Document => ({
  filename,
  type: 'PDF'
});

const createWordDocument: DocumentFactory = (filename: string): Document => ({
  filename,
  type: 'Word'
});

// Operations implementation using pattern matching on type
const documentOps: DocumentOperations = {
  open: (doc: Document): void => {
    console.log(`Opening ${doc.type}: ${doc.filename}`);
  },
  
  save: (doc: Document): void => {
    console.log(`Saving ${doc.type}: ${doc.filename}`);
  },
  
  close: (doc: Document): void => {
    console.log(`Closing ${doc.type}: ${doc.filename}`);
  }
};

// Higher-order function that creates an application
// Takes a factory and returns operations that use it
// This is the equivalent of the Application base class
type Application = {
  newDocument: (filename: string) => Document;
  openExisting: (filename: string) => Document;
};

const createApplication = (factory: DocumentFactory): Application => ({
  newDocument: (filename: string): Document => {
    const doc = factory(filename);
    documentOps.open(doc);
    console.log(`Created new ${doc.type} document`);

    return doc;
  },
  
  openExisting: (filename: string): Document => {
    const doc = factory(filename);
    documentOps.open(doc);
    console.log(`Opened existing ${doc.type} document`);

    return doc;
  }
});

// Usage - create specialized applications by passing different factories
const pdfApp = createApplication(createPDFDocument);
const doc1 = pdfApp.newDocument('report.pdf');

documentOps.save(doc1);
documentOps.close(doc1);

const wordApp = createApplication(createWordDocument);
const doc2 = wordApp.openExisting('letter.docx');

documentOps.save(doc2);
documentOps.close(doc2);
```

### Discriminated Unions Alternative

```typescript
type PDFDoc = {
  readonly kind: 'pdf';
  readonly filename: string;
  readonly pdfVersion: string;
};

type WordDoc = {
  readonly kind: 'word';
  readonly filename: string;
  readonly wordVersion: string;
};

// Discriminated union
type Document = PDFDoc | WordDoc;

// Factory functions with specific types
const createPDF = (filename: string): PDFDoc => ({
  kind: 'pdf',
  filename,
  pdfVersion: '1.7'
});

const createWord = (filename: string): WordDoc => ({
  kind: 'word',
  filename,
  wordVersion: '2019'
});

// Pattern matching for operations
// TypeScript ensures all cases are handled
const openDocument = (doc: Document): void => {
  switch (doc.kind) {
    case 'pdf':
      console.log(`Opening PDF ${doc.filename} (version ${doc.pdfVersion})`);
      break;
    case 'word':
      console.log(`Opening Word ${doc.filename} (version ${doc.wordVersion})`);
      break;
  }
};

const saveDocument = (doc: Document): void => {
  switch (doc.kind) {
    case 'pdf':
      console.log(`Saving PDF ${doc.filename}`);
      break;
    case 'word':
      console.log(`Saving Word ${doc.filename}`);
      break;
  }
};

// Generic application factory
const createApp = <T extends Document>(factory: (filename: string) => T) => ({
  newDocument: (filename: string): T => {
    const doc = factory(filename);
    openDocument(doc);

    return doc;
  }
});

// Usage
const pdfApp = createApp(createPDF);
const doc = pdfApp.newDocument('report.pdf');

saveDocument(doc);
```

### Comparison: Traditional vs Modern

| Aspect           | Inheritance        | HOF/Closures         |
| ---------------- | ------------------ | -------------------- |
| Extensibility    | New subclass       | New factory function |
| Type Safety      | Compile-time check | Strong with unions   |
| Coupling         | Inheritance chain  | Loose (composition)  |
| Testability      | Need mock subclass | Pass test factory    |
| Code Reuse       | Inheritance        | Composition/HOF      |
| Complexity       | Class hierarchy    | Function signatures  |
| State Management | Instance variables | Closures/parameters  |
| Runtime Behavior | vtable dispatch    | Direct function call |

### Stackblitz Link

[Factory Method Pattern](https://stackblitz.com/edit/vitejs-vite-ipuduldk?file=src%2Fmain.ts)