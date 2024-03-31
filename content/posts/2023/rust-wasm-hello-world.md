---
title: "Creating a simple wasm application in Rust in a dev container with nginx"
date: 2023-03-05T13:42:34Z
tags: ['rust', 'wasm', 'hello world', 'dev containers']
icon: brand-rust
aliases:
  - /posts/rust-wasm-hello-world/
summary: >-
    A guide to creating a simple wasm application in Rust, and serving it with nginx in a dev container.
---

## Introduction

I've been wanting to explore what options there are in [wasm](https://en.wikipedia.org/wiki/WebAssembly) before, but I hadn't really explored this much beyond a play around with [Blazor](https://dotnet.microsoft.com/en-us/apps/aspnet/web-apps/blazor) a while back, and even for that, it was only a brief introduction. Coupling that with wanting to explore with Rust a little more, I thought what better way to do this than to create a simple wasm application in Rust.

For this, it was a really simple application: Can I write a simple wasm application in Rust which takes two values from JavaScript, adds them together, and then returns the result to a page in HTML?

## Setting up the project

To get started, we'll use dev containers again, as that will mean we don't need to worry about setting up the environment on our local machine, nor do we need to install any Rust tooling on our local machine in case we decide we don't want to use Rust in the future. We can do this by creating a new folder for our project, and then using the "Add Dev Container Configuration Files..." option in VS Code. From there, all we need is the standard Rust template. After that, we can reopen the project in a container, and we'll be ready to go.

## Configuring our dev container

First thing we need to do once we're inside of our dev container is to install the wasm tooling. We can do this with the following command:

```bash
cargo install wasm-pack
```

This will install the wasm-pack tool, which we'll use to build our wasm application. Next, we can initialise our project with the following command:

```bash
cargo init
```

This will create the new Rust project, but we'll need to configure this to be a wasm project. To do this, we'll need to add the following to our Cargo.toml file:

```toml
[package]
name = "rust-wasm-hello-world"
version = "0.1.0"
edition = "2021"
authors = ["<YOUR NAME>"]

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen = "0.2"
```

This is our project set up, along with the wasm-bindgen dependency, which we'll use to bind our Rust code to JavaScript.

## Creating our Rust library

When we ran the `cargo init` command, a `src` folder was created for us, and inside of this, there will be a `main.rs` file. This will be the entry point for our code. To make this a little clearer about its purpose, let's rename it to `lib.rs` as we're going to be using this as a library as opposed to a binary.

We can now add our simple Rust code to our `src/lib.rs` file:

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

There's nothing particularly special here, but let's break this down a little.

```rust
use wasm_bindgen::prelude::*;
```

This line is to import the `wasm_bindgen` create, which is how we expose functions to our JavaScript code.

```rust
#[wasm_bindgen]
```

This attribute flags the following function to be exposed to JavaScript.

```rust
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

Here is our *actual* code, which is a simple function which takes two `i32` values, and returns the result of adding them together. Here, `i32` is a 32-bit integer, which is the same as a `int` in C#, Java, and many other languages.

## Building our wasm application

Now that we have written out Rust code, we can build it inside of our dev container with the following command in the terminal:

```bash
wasm-pack build --target web
```

We can also add this as a task in VScode, which will make it easier to run in the future. To do this, we can add the following to our `.vscode/tasks.json` file:

```json
{
    "label": "wasm-pack build",
    "type": "shell",
    "command": "wasm-pack build --target web",
    "group": {
        "kind": "build",
        "isDefault": true
    }
}
```

This will configure it as our default build task so if we ever run the "Task: Run Build Task" command, it will run this command for us.

## The built application

Running the build command will creat a new directory for us, `pkg`, which will contain the built application. This contains numerous files, but the ones we're most interested in are the following:

- `rust_wasm_hello_world_bg.wasm` - This is our wasm compilted library, which we can load into a browser.
- `rust_wasm_hello_world.js` - This is our built JavaScript file, this contains a lot of wrapper functions for us to use in our JavaScript code.
- `rust_wasm_hello_world.d.ts` - This is our TypeScript definition file, which we can use to get intellisense in our JavaScript code.

## Creating our HTML page

To test out wasm application, we only need a simple HTML page, which we can create in the root directory of our project. We can call this `index.html`, with the following code:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Hello World - Rust</title>
    <link rel="preload" href="./index.js" as="script" />
  </head>
  <body>
    <script type="module" src="./index.js"></script>
  </body>
</html>
```

This will pre-load a JavaScript file, `index.js`, which we'll create next, and then load it at the end of the page. Let's now create our JavaScript file, which we'll call `index.js`, again in the root of our project, and add the following code:

```javascript
// Import our outputted wasm ES6 module
// Which, export default's, an initialization function
import init from "./pkg/rust_wasm_hello_world.js";

const runWasm = async () => {
  // Instantiate our wasm module
  const helloWorld = await init("./pkg/rust_wasm_hello_world_bg.wasm");

  // Call the Add function export from wasm, save the result
  const addResult = helloWorld.add(32, 24);

  // Set the result onto the body
  document.body.textContent = `Hello, world! addResult: ${addResult}`;
};
runWasm();
```

This file will create and run a `runWasm` function. This function loads in our compiled wasm code from the `pkg` folder, and then calls the `add` function from our Rust code, and then sets the result onto the body of the page.

## Setting up our application to run locally

As a lot of web browsers are more security conscious, we can't just open our HTML file in the browser, as it will complain about the JavaScript file being loaded from a different domain with a CORS error[^1]. To get around this, let's extend out dev container set up so that it doesn't only have a service for building code, but also have an nginx server to serve our application. To do this, we'll need to rework our devcontainer setup to use a docker-compose file, which we can do by adding the following to our `.devcontainer/devcontainer.json` file instead of the `image` property:

```json
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace/"
```

[^1]: There are other ways to get around this, for example by changing your security settings, but this will work for us and means that we don't need to disable security.

This will replace the `image` property which was there, and will tell vscode that we don't *just* want to run a service, but also want to run a docker-compose file, which we'll create next.

We can now create our `docker-compose.yml` file in the `.devcontainer` directory, which will contain the following:

```yaml
version: '3'

services:
  # Dev container
  app:
    build:
      context: ..
      dockerfile: .devcontainer/Dockerfile

    volumes:
      - ..:/workspace:cached

    # Overrides default command so things don't shut down after the process ends.
    command: sleep infinity
    # Uncomment to connect as root instead. More info: https://aka.ms/dev-containers-non-root.
    # user: root

  nginx:
    image: nginx:latest
    ports:
      - 8080:80
    volumes:
      - ..:/usr/share/nginx/html:readonly
    depends_on:
      - app
```

This is a relatively simple setup for the docker-compose file, but it contains two services:

- `app` - This is our dev container, which will build our code, and run our tests. This is built by using the `Dockerfile` in the `.devcontainer` directory which we'll create next
- `nginx` - This is our web server, which will serve our application. It will serve the files from the root of our project (i.e. our `index.html` and its associated files), and will be available on port 8080 from our host machine.

We can now create our `Dockerfile` in the `.devcontainer` directory, which will contain the following:

```dockerfile
FROM mcr.microsoft.com/devcontainers/rust:0-1-bullseye
```

This is all we need for now, but we can actually have our dev container automatically install the `wasm-pack` tooling for us by extending our `Dockerfile` to the following:

```dockerfile
FROM mcr.microsoft.com/devcontainers/rust:0-1-bullseye

USER vscode

RUN cargo install wasm-pack
```

This will perform the following commands:

1. Start from the default rust container using the 0-1-bullseye tag
2. Switch to the `vscode` user, so that any following commands are executed as the `vscode` user rather than the `root` user
3. Install the `wasm-pack` tooling using cargo, which we did manually earlier

## Running our application

Now that we have a docker-compose setup configured for our dev container, we should reopen the dev container using the "Remote-Containers: Reopen in Container" command. This will rebuild our dev container, and then start our docker-compose file, which will start our dev container, and our nginx server.

Once this is running, we can rebuild the application by running the "Task: Run Build Task" command we configured earlier, and then view what our nginx server is serving by opening the following URL in our browser: <http://localhost:8080>. This should show us our application, which should say "Hello, world! addResult: 56".

## Conclusion

This setup is actually a little overly complicated for what we wanted to accomplish, but it's also given us a good starting point for other Rust projects, and a nice dev container setup which we can re-use for similar projects in other languages.

## Code

You can find the code for this post on GitHub: <https://github.com/cwrenhold/rust-wasm-hello-world>
