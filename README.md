[![Travis](https://img.shields.io/travis/Microsoft/tsyringe.svg)](https://travis-ci.org/Microsoft/tsyringe/)

# TSyringe

A lightweight dependency injection container for TypeScript/JavaScript for
constructor injection.

* [Installation](#installation)
* [API](#api)
  * [injectable()](#injectable)
  * [autoInjectable()](#autoinjectable)
  * [inject()](#inject)
* [Full Examples](#full-examples)
* [Contributing](#contributing)

## Installation

```sh
npm install --save tsyringe
```

Modify your `tsconfig.json` to include the following settings
```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

## API
### injectable()
Class decorator factory that allows the class' dependencies to be injected at
runtime.

#### Usage
```TypeScript
import {decorators} from "tsyringe";
const {injectable} = decorators;

@injectable()
class Foo {
  constructor(private database: Database) {}
}

// some other file
import {container} from "tsyringe";
import {Foo} from "./foo";

const instance = container.resolve(Foo);
```

### autoInjectable()
Class decorator factory that replaces the decorated class' constructor with
a parameterless constructor that has dependencies auto-resolved.

**Note** Resolution is performed using the global container

#### Usage
```TypeScript
import {decorators} from "tsyringe";
const {autoInjectable} = decorators;

@autoInjectable()
class Foo {
  constructor(private database?: Database) {}
}

// some other file
import {Foo} from "./foo";

const instance = new Foo();
```

Notice how in order to allow the use of the empty constructor `new Foo()`, we
need to make the parameters optional, e.g. `database?: Database`

### inject()
Parameter decorator factory that allows for interface and other non-class
information to be stored in the constructor's metadata

#### Usage
```TypeScript
import {decorators} from "tsyringe";
const {injectable, inject} = decorators;

interface Database {
  // ...
}

@injectable()
class Foo {
  constructor(@inject("Database") private database?: Database) {}
}
```

## Full examples
### Example without interfaces
Since classes have type information at runtime, we can resolve them without any
extra information.

```TypeScript
// Foo.ts
export class Foo {}
```
```TypeScript
// Bar.ts
import {Foo} from "./Foo";
import {decorators} from "tsyringe";
const {injectable} = decorators;

@injectable()
export class Bar {
  constructor(public myFoo: Foo) {}
}
```
```TypeScript
// main.ts
import {container} from "tsyringe";
import {Bar} from "./Bar";

const myBar = container.resolve(Bar);
// myBar.myFoo => An instance of Foo
```

### Example with interfaces
Interfaces don't have type information at runtime, so we need to decorate them
with `@inject(...)` so the container knows how to resolve them.

```TypeScript
// SuperService.ts
export interface SuperService {
  // ...
}
```
```TypeScript
// TestService.ts
import {SuperService} from "./SuperService";
export class TestService implements SuperService {
  //...
}
```
```TypeScript
// Client.ts
import {decorators} from "tsyringe";
const {injectable, inject} = decorators;

@injectable()
export class Client {
  constructor(@inject("SuperService") private service: SuperService) {}
}
```
```TypeScript
// main.ts
import {Client} from "./Client";
import {TestService} from "./TestService";
import {container} from "tsyringe";

container.register({
  token: "SuperService",
  useClass: TestService
});

const client = container.resolve(Client);
// client's dependencies will have been resolved
```

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit [https://cla.microsoft.com](https://cla.microsoft.com).

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
