---
lang: en
title: 'Creating components'
keywords: LoopBack 4.0, LoopBack 4
tags:
sidebar: lb4_sidebar
permalink: /doc/en/lb4/Creating-components.html
summary:
---


As explained in [Using Components](Using-components.html), a typical LoopBack component is an npm package exporting a Component class.

```js
import MyController from './controllers/my-controller';
import MyValueProvider from './providers/my-value-provider';

export class MyComponent {
  constructor() {
    this.controllers = [MyController];
    this.providers = {
      'my.value': MyValueProvider
    };
  }
}
```

When a component is mounted to an application, a new instance of the component class is created and then:
 - Each Controller class is registered via `app.controller()`,
 - Each Provider is bound to its key in `providers` object.

The example `MyComponent` above will add `MyController` to application's API and create a new binding `my.value` that will be resolved using `MyValueProvider`.

## Providers

Providers enable components to export values that can be used by the target application or other components. The `Provider`  class provides a `value()` function called by [Context](Context.html) when another entity requests a value to be injected.

```js
import {Provider} from '@loopback/context';

export class MyValueProvider {
  value() {
    return 'Hello world';
  }
}
```

### Specifying binding key

Notice that the provider class itself does not specify any binding key, the key is assigned by the component class.

```js
import MyValueProvider from './providers/my-value-provider';

export class MyComponent {
  constructor() {
    this.providers = {
      'my-component.my-value': MyValueProvider
    };
  }
}
```

### Accessing values from Providers

Applications can use `@inject` decorators to access the value of an exported Provider.
If you’re not familiar with decorators in TypeScript, see [Key Concepts: Decorators](Decorators.html)

```js
const app = new Application({
  components: [MyComponent]
});

class MyController {
  constructor(@inject('my-component.my-value') greeting) {
    // LoopBack sets greeting to 'Hello World' when creating a controller instance
    this.greeting = greeting;
  }

  @get('/greet')
  greet() {
    return this.greeting;
  }
}
```

{% include note.html title="A note on binding names" content="To avoid name conflicts, add a unique prefix to your binding key (for example, `my-component.` in the example above). See [Reserved binding keys](Reserved-binding-keys.html) for the list of keys reserved for the framework use.
" %}

### Asynchronous providers

Provider's `value()` method can be asynchronous too:

```js
const request = require('request-promise-native');
const weatherUrl =
  'http://samples.openweathermap.org/data/2.5/weather?appid=b1b15e88fa797225412429c1c50c122a1'

export class CurrentTemperatureProvider {
  async value() {
    const data = await(request(`${weatherUrl}&q=Prague,CZ`, {json:true});
    return data.main.temp;
  }
}
```

In this case, LoopBack will wait until the promise returned by `value()` is resolved, and use the resolved value for dependency injection.

### Working with HTTP request/response

In some cases, the Provider may depend on other parts of LoopBack; for example the current `request` object. The Provider's constructor should list such dependencies annotated with `@inject` keyword, so that LoopBack runtime can resolve them automatically.

```js
const uuid = require('uuid/v4');

class CorrelationIdProvider {
  constructor(@inject('http.request') request) {
    this.request = request;
  }

  value() {
    return this.request.headers['X-Correlation-Id'] || uuid();
  }
}
```

## Modifying request handling logic

A frequent use case for components is to modify the way requests are handled. For example, the authentication component needs to verify user credentials before the actual handler can be invoked; or a logger component needs to record start time and write a log entry when the request has been handled.

The idiomatic solution has two parts:

 1. The component should define and bind a new [Sequence action](Sequence.html#actions), for example `authentication.actions.authenticate`:

    ```js
    class AuthenticationComponent {
      constructor() {
        this.providers = {
          'authentication.actions.authenticate': AuthenticateActionProvider
        };
      }
    }
    ```

 2. The application should use a custom `Sequence` class which calls this new sequence action in an appropriate place.

    ```js
    class AppSequence implements SequenceHandler {
      constructor(
        @inject('sequence.actions.findRoute') findRoute,
        @inject('sequence.actions.invokeMethod') invoke,
        @inject('sequence.actions.send') send: Send,
        @inject('sequence.actions.reject') reject: Reject,
        // Inject the new action here:
        @inject('authentication.actions.authenticate') authenticate
      ) {
        this.findRoute = findRoute;
        this.invoke = invoke;
        this.send = send;
        this.reject = reject;
        this.authenticate = authenticate;
      }

      async handle(req: ParsedRequest, res: ServerResponse) {
        try {
          const route = this.findRoute(req);

          // Invoke the new action:
          const user = await this.authenticate(req);

          const args = await parseOperationArgs(req, route);
          const result = await this.invoke(route, args);
          this.send(res, result);
        } catch (err) {
          this.reject(res, req, err);
        }
      }
    }
    ```

### Accessing Elements contributed by other Sequence Actions

When writing a custom sequence action, you need to access Elements contributed by other actions run in the sequence. For example, `authenticate()` action needs information about the invoked route to decide whether and how to authenticate the request.

Because all Actions are resolved before the Sequence `handle` function is run, Elements contributed by Actions are not available for injection yet. To solve this problem, use `@inject.getter` decorator to obtain a getter function instead of the actual value. This allows you to defer resolution of your dependency only until the sequence action contributing this value has already finished.

```js
export class AuthenticationProvider {
  constructor(
    @inject.getter(BindingKeys.Authentication.STRATEGY)
    readonly getStrategy
  ) {
    this.getStrategy = getStrategy;
  }

  value() {
    return async (request: ParsedRequest) => {
      const strategy = await getStrategy();
      // ...
    };
  }
}
```


### Contributing Elements from Sequence Actions

Use `@inject.setter` decorator to obtain a setter function that can be used to contribute new Elements to the request context.

```js
export class AuthenticationProvider {
  constructor(
    @inject.getter(BindingKeys.Authentication.STRATEGY)
    readonly getStrategy,
    @inject.setter(BindingKeys.Authentication.CURRENT_USER)
    readonly setCurrentUser,
  ) {
    this.getStrategy = getStrategy;
    this.setCurrentUser = setCurrentUser;
  }

  value() {
    return async (request: ParsedRequest) => {
      const strategy = await getStrategy();
      const user = // ... authenticate
      setCurrentUser(user);
      return user;
    };
  }
}
```

## Extends Application with Mixin

When binding a component to an app, you may want to extend the app with the component's
properties and methods by using mixins.

For an overview of mixins, see [Mixin](Mixin.html).

An example of how a mixin leverages a component is `RepositoryMixin`.
Suppose an app has multiple components with repositories bound to each of them. 
You can use function `RepositoryMixin()` to mount those repositories to application level context.

The following snippet is an abbreviated function
[`RepositoryMixin`](https://github.com/strongloop/loopback-next/blob/master/packages/repository/src/repository-mixin.ts):

{% include code-caption.html content="mixins/src/repository-mixin.ts" %}
```js
export function RepositoryMixin<T extends Class<any>>(superClass: T) {
  return class extends superClass {
    constructor(...args: any[]) {
      super(...args);
      ... ...
      // detect components attached to the app
      if (this.options.components) {
        for (const component of this.options.components) {
          this.mountComponentRepository(component);
        }
      }
    }
  }
  mountComponentRepository(component: Class<any>) {
    const componentKey = `components.${component.name}`;
    const compInstance = this.getSync(componentKey);

    // register a component's repositories in the app
    if (compInstance.repositories) {
      for (const repo of compInstance.repositories) {
        this.repository(repo);
      }
    }
  }
}
```

Then you can extend the app with repositories in a component:

{% include code-caption.html content="index.ts" %}

```js
import {RepositoryMixin} from 'mixins/src/repository-mixin';
import {Application} from '@loopback/core';
import {FooComponent} from 'components/src/Foo';

class AppWithRepoMixin extends RepositoryMixin(Application) {};
let app = new AppWithRepoMixin({components: [FooComponent]});

// `app.find` returns all repositories in FooComponent
app.find('repositories.*');
```

## Configuring components

More often than not, the component may want to offer different value providers depending on the configuration. For example, a component providing an email API may offer different transports (stub, SMTP, and so on).

Components should use constructor-level [Dependency Injection](Context.html#dependency-injection) to receive the configuration from the application.

```js
class EmailComponent {
  constructor(@inject('config#components.email') config) {
    this.config = config;
    this.providers = {
      'sendEmail': config.transport == 'stub' ?
        StubTransportProvider :
        SmtpTransportProvider,
    };
  }
}
```

## Creating your own servers 

LoopBack 4 has the concept of a Server, which you can use to create your own
implementations of REST, SOAP, gRPC, MQTT and more. For an overview, see 
[Server](Server.html).

### A Basic Example
The concept of servers in LoopBack doesn't actually require listening on ports;
you can use this pattern to create workers that process data, write logs, or
whatever you'd like.

Let's create an example server! Please note that this is not a complete example:

```ts
import {Server} from '@loopback/core';
import {processReports} from '../util';
export class ReportProcessingServer implements Server {
  @inject('database') db: DefaultCrudRepository<Report, int>;
  interval: NodeJS.Timer;
  async start() {
    // Run the report processor every 2 minutes.
    let window = Date.now();
    interval = setInterval(async () => {
      const results = await db.find({
        createdDate: {
          $gt: window,
        },
      });
      if (results && results.length > 0) {
        results.forEach(async (rep) => {
          await db.updateById(rep.id, await processReport(rep));
        }); 
      }
      window = Date.now();
    }, 1000 * 120);
    return Promise.resolve();
  }

  async stop() {
    clearInterval(interval);
    return Promise.resolve();
  }
}
```
In this example, the `ReportProcessingServer` will load new reports every two
minutes from the database (using the provided repository), process those entries,
and then update the database.

### Using your server
To use your server, simply bind it to your application's context.

```ts
import {ReportProcessingServer} from '../servers';

export class MyApplication extends Application {
  super();
  // Will automatically start when app.start is called
  this.server(ReportProcessingServer); 
}
```

### A more advanced example
Typically, you'll want server instances that listen for traffic on one or more
ports (this is why they're called "servers", after all). This leads into a key
concept to leverage for creating your custom servers.

### Controllers and routing
LoopBack 4 developers are strongly encouraged to use controllers for their
modules, and this naturally leads to the concept of routing.

No matter what protocol you intend to use for your custom server, you'll need
to use some algorithm to determine _which_ controller and function to send
request data to, and that means you need a router.

For example, consider a "toy protocol" similar to the JSON RPC
specification (but nowhere near as complete or robust).

The toy protocol will require a JSON payload with three properties: `controller`,
`method`, and `input`.

An example request would look something like this:
```json
{
  "controller": "GreetController",
  "method": "basicHello",
  "input": {
    "name": "world",
  }
}
```

### Defining the controller
Here is the `GreetController` and the `Person` type:

{% include code-caption.html content="/src/controllers/greet.controller.ts" %}

```ts
export class GreetController {
  basicHello(input: Person) {
    return `Hello, ${input && input.name || 'World'}!`;
  }

  hobbyHello(input: Person) {
    return `${this.basicHello(input)} I heard you like ${input && input.hobby ||
      'underwater basket weaving'}.`;
  }
}
```

{% include code-caption.html content="/src/models/person.model.ts" %}
```ts
// Note that this can also be a class!
export type Person = {
  name: string;
  hobby: string;
};
```

### Defining our router and server
Now that there is a controller to test with, the following code creates a `Router` and `Server`.
This example uses [express](https://expressjs.com/) and the
`@types/express` package.

{% include code-caption.html content="/src/server/rpc-router.ts" %}
```ts
import {RPCServer} from './rpc-server';
import * as express from 'express';
import * as parser from 'body-parser';

export class RPCRouter {
  routing: express.Router;
  constructor(private server: RPCServer) {
    this.routing = express.Router();
    const jsonParser = parser.json();
    this.routing.post('*', jsonParser, async (request, response) => {
      console.log(JSON.stringify(request.body));
      // Aliasing these values for clarity and brevity.
      const ctrl = request.body.controller;
      const method = request.body.method;
      const input = request.body.input;
      try {
        const controller = await this.server.get(`controllers.${ctrl}`);
        if (!controller) {
          response.statusCode = 400;
          response.send(`No controller was found with name "${ctrl}".`);
          return;
        }
        if (!controller[method]) {
          response.statusCode = 400;
          response.send(
            `No method was found on controller "${ctrl}" with name "${method}".`
          );
        }
        response.send(await controller[method](input));
      } catch (err) {
        // You'll probably want more robust error handling in a real server!
        console.error(err);
        response.statusCode = 500;
        response.send(err);
      }
    });
    // Use our router!
    this.server.expressServer.use(this.routing);
  }
}
```

In addition to the server, the example will include a small type definition for
convenience.

{% include code-caption.html content="/src/server/rpc-server.ts" %}
```ts
import {inject, Context} from '@loopback/context';
import {Server, Application, CoreBindings} from '@loopback/core';
import {RPCRouter} from './rpc-router';
import * as express from 'express';
import * as http from 'http';

export class RPCServer extends Context implements Server {
  _server: http.Server;
  expressServer: express.Application;
  router: RPCRouter;
  constructor(
    @inject(CoreBindings.APPLICATION_INSTANCE) public app?: Application,
    @inject('rpcServer.config') public config?: RPCServerConfig
  ) {
    super(app);
    this.config = config || {};
  }

  async start(): Promise<void> {
    this.expressServer = express();
    this.router = new RPCRouter(this);
    // Currently, there is a limitation with core node libraries
    // that prevents the easy promisification of some functions, like
    // http.Server's listen method.
    this._server = this.expressServer.listen(
      (this.config && this.config.port) || 3000
    );
    return Promise.resolve();
  }
  async stop(): Promise<void> {
    this._server.close();
    return Promise.resolve();
  }
}

export type RPCServerConfig = {
  port?: number;
  [key: string]: any;
};
```

### Putting it all together
Now here is the Application subclass and `index.ts` entry point to
make it go!

{% include code-caption.html content="/src/application.ts" %}
```ts
import {Application, ApplicationConfig} from '@loopback/core';
import {RPCServer} from './servers/rpc-server';
import {GreetController} from './controllers';

export class MyApplication extends Application {
  options: ApplicationConfig;
  constructor(options?: ApplicationConfig) {
    // Allow options to replace the defined components array, if desired.
    super(options);
    this.controller(GreetController);
    this.server(RPCServer);
    this.options = options || {};
    this.options.port = this.options.port || 3000;
    this.bind('rpcServer.config').to(this.options);
  }

  async start() {
    await super.start();
    console.log(`Server is running on port ${this.options.port}`);
  }
}
```

{% include code-caption.html content="index.ts" %}
```ts
import {MyApplication} from './src/application';

const app = new MyApplication();

app.start().catch(err => {
  console.error(`Unable to start application: ${err}`);
});
async function shutdown() {
  await app.stop();
  process.exit(0);
}
// Event handlers for when we are killed; these aren't *required*.
process.on('SIGTERM', shutdown);
process.on('SIGINT', shutdown);
```

### Trying it out

Now, try it out: start the server and run a few REST requests. Feel free to use
whatever REST client you'd prefer (this example will use `curl`).
```sh
# Basic Greeting Calls
$ curl -X POST -d '{ "controller": "GreetController", "method": "basicHello" }' -H "Content-Type: application/json" http://localhost:3000/
Hello, World!
$ curl -X POST -d '{ "controller": "GreetController", "method": "basicHello", "input": { "name": "Nadine" } }' -H "Content-Type: application/json" http://localhost:3000/
Hello, Nadine!
# Advanced Greeting Calls
$ curl -X POST -d '{ "controller": "GreetController", "method": "hobbyHello", "input": { "name": "Nadine" } }' -H "Content-Type: application/json" http://localhost:3000/
Hello, Nadine! I heard you like underwater basket weaving.
$ curl -X POST -d '{ "controller": "GreetController", "method": "hobbyHello", "input": { "name": "Nadine", "hobby": "extreme mountain biking" } }' -H "Content-Type: application/json" http://localhost:3000/
Hello, Nadine! I heard you like extreme mountain biking.
```

While a typical protocol server would be a lot more involved in the
implementation of both its router and server, the general concept remains
the same, and you can use these tools to make whatever server you'd like.

### Other considerations
Some additional concepts to add to your server could include:
- Pre-processing of requests (changing content types, checking the request body,
etc)
- Post-processing of responses (removing sensitive/useless information)
- Caching
- Logging
- Automatic creation of default endpoints
- and more...

LoopBack 4's modularity allows for custom servers of all kinds, while still
providing key utilities like context and injection to make your work easier.
