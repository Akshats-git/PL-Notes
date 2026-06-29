# Node.js — Complete Guide

> Based on the code in `code+files/nodejs/` — every concept explained with context, not just syntax.

---

## Table of Contents

1. [What Is Node.js?](#1-what-is-nodejs)
2. [How Node Runs Your Code](#2-how-node-runs-your-code)
3. [Modules & `require`](#3-modules--require)
4. [The `fs` Module — Reading & Writing Files](#4-the-fs-module--reading--writing-files)
5. [Buffers, Strings & JSON](#5-buffers-strings--json)
6. [Command-Line Arguments (`process.argv`)](#6-command-line-arguments-processargv)
7. [Project: A To-Do CLI](#7-project-a-to-do-cli)
8. [The `os` Module — System Information](#8-the-os-module--system-information)
9. [Events & the `EventEmitter`](#9-events--the-eventemitter)
10. [Timers — `setInterval` & `setTimeout`](#10-timers--setinterval--settimeout)
11. [Project: An Event Logger](#11-project-an-event-logger)
12. [The `http` Module — Building a Server](#12-the-http-module--building-a-server)
13. [The `path` Module & `__dirname`](#13-the-path-module--__dirname)
14. [Project: A Static File Server](#14-project-a-static-file-server)
15. [Sync vs Async in Node](#15-sync-vs-async-in-node)
16. [Quick Reference — Mental Model](#16-quick-reference--mental-model)

---

## 1. What Is Node.js?

JavaScript was originally a language that only ran *inside a browser*. It could change web pages, respond to clicks, and talk to servers — but it had no way to touch the file system, open a network port, or read system memory. Those things are dangerous, so browsers deliberately lock JavaScript out of them.

**Node.js** changes that. It takes the same JavaScript engine that Chrome uses (a high-performance engine called **V8**) and runs it *outside* the browser, directly on your computer. On top of V8, Node adds a set of built-in libraries that give JavaScript real-world powers: reading and writing files, creating web servers, accessing the operating system, and more.

```
Where JavaScript runs:

  Browser JavaScript                 Node.js
  ──────────────────                 ───────
  V8 engine                          V8 engine  (same engine!)
  + DOM (document, window)           + fs    (file system)
  + fetch, alert, localStorage       + http  (web servers)
                                     + os    (system info)
                                     + path, events, process...

  Can't touch your files             CAN read/write files,
  or open network ports              open ports, run as a backend
```

So Node.js lets you use JavaScript — a language you already know — to write **backend** programs: web servers, command-line tools, file processors, automation scripts. The three small programs in this folder (`todo`, `eventLogger`, `server`) are exactly these kinds of programs.

**Key idea:** Node is not a new language. It's the JavaScript you already know, plus a toolbox of modules for talking to the computer.

---

## 2. How Node Runs Your Code

You run a Node program from the terminal by typing `node` followed by the filename:

```bash
node todo.js
node server.js
node logger.js
```

Node reads the file top to bottom and executes it, just like the browser does. The big differences are:

- There is **no `window` and no `document`** — those only exist in browsers. Trying to use them throws an error.
- There is a global object called **`process`** that represents the running program itself (its arguments, environment, input/output).
- `console.log` prints to your **terminal**, not a browser console.
- The program keeps running as long as there is work pending. A script that just prints something exits immediately. But a script with a `setInterval` (section 10) or a listening server (section 12) stays alive, waiting.

```
The lifecycle of `node todo.js`:

  1. Node starts up the V8 engine
  2. Loads and runs todo.js from top to bottom
  3. If there's pending async work (timers, servers, file ops),
     Node stays alive and the event loop keeps spinning
  4. When there's nothing left to do, Node exits
```

That last point is why `node todo.js add "buy milk"` finishes instantly (it does its job and exits), while `node logger.js` runs forever (its `setInterval` never stops scheduling new work), and `node server.js` runs forever (it's holding a network port open).

---

## 3. Modules & `require`

A **module** is just a file. Node treats every `.js` file as its own private module — variables and functions inside it are *not* visible to other files unless you explicitly export them. This keeps code organised and prevents name clashes.

Node uses the **CommonJS** module system (the original Node module system). You bring code into your file with the `require()` function.

There are two kinds of things you can `require`:

```js
// 1. Built-in (core) modules — shipped with Node, no installation needed.
//    You require them by their bare name.
const fs   = require("fs");      // file system
const os   = require("os");      // operating system info
const http = require("http");    // web server
const path = require("path");    // file path helpers
const EventEmitter = require("events"); // event system

// 2. Your own files — require them by their relative path (note the "./").
const tasks = require("./tasks.json");
```

`require()` runs the target file (if it hasn't run already) and returns whatever that file *exported*. For a core module, it returns the library object. For a `.json` file, Node helpfully parses it and returns the resulting object/array automatically.

```
What require() gives you back:

  require("fs")          →  the file-system library (an object full of functions)
  require("os")          →  the OS-info library
  require("./tasks.json")→  the parsed contents of that JSON file
  require("./myCode.js") →  whatever myCode.js set on module.exports
```

> **Note:** The leading `./` matters. `require("fs")` looks for a *built-in* module named `fs`. `require("./fs")` looks for a *file* called `fs.js` next to the current file. Forgetting the `./` on your own files is a very common beginner error.

Exporting from your own file uses `module.exports`:

```js
// mathHelpers.js
function add(a, b) { return a + b; }
module.exports = { add };       // make `add` available to other files

// otherFile.js
const { add } = require("./mathHelpers.js");
add(2, 3); // 5
```

---

## 4. The `fs` Module — Reading & Writing Files

**Source:** `todo/todo.js`, `eventLogger/logger.js`

`fs` ("file system") is the module that lets your program read from and write to files on disk. This is one of the main superpowers Node adds that browsers don't have. All three projects in this folder use it.

Most `fs` functions come in **two versions**:

- **Synchronous** (`...Sync`) — the program *stops and waits* for the file operation to finish before moving to the next line. Simple to reason about, but it blocks everything else while it works.
- **Asynchronous** (callback-based) — the operation runs in the background, and a callback function runs when it's done. Doesn't block, but the code flow is less linear.

We'll see both. For a small CLI tool, sync is fine. For a server handling many requests at once, async is essential (more in section 15).

### Reading a file — `readFileSync`

From `todo/todo.js`:

```js
const fs = require("fs");
const filePath = "./tasks.json";

const loadTasks = () => {
  try {
    const dataBuffer = fs.readFileSync(filePath); // read the whole file (blocking)
    const dataJSON   = dataBuffer.toString();     // raw bytes → text
    return JSON.parse(dataJSON);                  // text → JS array/object
  } catch (error) {
    return [];   // file doesn't exist yet → start with an empty list
  }
};
```

`readFileSync` reads the entire file and hands it back. Notice it's wrapped in a `try/catch`: if the file doesn't exist (first run of the program), `readFileSync` *throws an error*. Instead of crashing, we catch it and return an empty array — a sensible default that says "no tasks yet."

### Writing a file — `writeFileSync`

```js
const saveTasks = (tasks) => {
  const dataJSON = JSON.stringify(tasks);   // JS array → text
  fs.writeFileSync(filePath, dataJSON);     // overwrite the file with that text
};
```

`writeFileSync` **completely overwrites** the file with whatever you give it. So the save pattern is always: load the whole list → modify it in memory → write the whole list back.

### Appending to a file — `appendFileSync`

From `eventLogger/logger.js`:

```js
fs.appendFileSync(logFile, logMessage); // add to the END, keep existing content
```

Unlike `writeFileSync`, `appendFileSync` **adds to the end** of the file without erasing what's already there. This is exactly what you want for a log file — each new entry stacks under the previous ones.

### Reading a file — async with a callback

From `server/server.js`:

```js
fs.readFile(filePath, (err, content) => {
  if (err) {
    // something went wrong (e.g. file not found)
  } else {
    // `content` holds the file data — use it here
  }
});
```

`fs.readFile` (no `Sync`) does **not** return the data. Instead you pass it a callback that Node calls *later*, once the file has been read. The callback follows Node's universal convention: **the first argument is always an error** (or `null` if all went well), and the data comes after. This "error-first callback" pattern is everywhere in Node.

```
Three fs functions used in this folder:

  readFileSync(path)            → returns file contents (blocks until done)
  writeFileSync(path, data)     → OVERWRITES the file
  appendFileSync(path, data)    → ADDS to the end of the file
  readFile(path, callback)      → async version; data arrives in the callback
```

---

## 5. Buffers, Strings & JSON

**Source:** `todo/todo.js`

When you read a file, Node doesn't hand you a nice string by default. It hands you a **Buffer** — a chunk of raw binary data (a sequence of bytes). That's because a file might be an image, a video, or text in any encoding; Node stays neutral and gives you the raw bytes.

To turn those bytes into readable text, you call `.toString()`:

```js
const dataBuffer = fs.readFileSync(filePath); // a Buffer (raw bytes)
const dataJSON   = dataBuffer.toString();     // now it's a normal string
```

### JSON — the bridge between text files and JS objects

Files store **text**. Your program works with **objects and arrays**. JSON (JavaScript Object Notation) is the format that bridges the two. Two functions do all the work:

- `JSON.parse(text)` — turns a JSON *string* into a real JS value (object/array).
- `JSON.stringify(value)` — turns a JS value back into a JSON *string* for storage.

```js
// Reading:  text on disk  →  usable JS array
JSON.parse('[{"task":"buy milk"}]')
// → [ { task: "buy milk" } ]

// Writing:  JS array  →  text to store
JSON.stringify([ { task: "buy milk" } ])
// → '[{"task":"buy milk"}]'
```

This is exactly the round-trip the to-do app performs. The file `tasks.json` literally contains:

```json
[{"task":"buy milk"},{"task":"record videos"}]
```

```
The full data round-trip in the todo app:

  DISK (tasks.json)              IN MEMORY (your program)
  ─────────────────              ────────────────────────
  raw bytes                                       
     │  readFileSync                              
     ▼                                            
  Buffer ──.toString()──► JSON string ──JSON.parse()──► JS array
                                                            │
                                                       (add/remove a task)
                                                            │
  JSON string ◄──JSON.stringify()─────────────────────────┘
     │
     │  writeFileSync
     ▼
  DISK (tasks.json) updated
```

> **Why `JSON.parse` can crash:** if the file is empty or contains broken JSON, `JSON.parse` throws. That's another reason `loadTasks` wraps everything in `try/catch`.

---

## 6. Command-Line Arguments (`process.argv`)

**Source:** `todo/todo.js`

A command-line tool needs to know *what the user asked it to do*. When you run:

```bash
node todo.js add "buy milk"
```

…the words after `node` are available inside your program through `process.argv` — an **array of strings**. The catch is that the first two entries are always filled in by Node itself:

```
process.argv for:  node todo.js add "buy milk"

  index 0  →  "/path/to/node"      ← the Node executable    (ignore)
  index 1  →  "/path/to/todo.js"   ← your script's path     (ignore)
  index 2  →  "add"                ← FIRST real argument
  index 3  →  "buy milk"           ← SECOND real argument
```

So your actual arguments start at index **2**. That's why the to-do app reads them like this:

```js
const command  = process.argv[2];   // "add", "list", or "remove"
const argument = process.argv[3];   // the task text, or a number
```

`process` is a global object that exists in every Node program (no `require` needed). Besides `argv` it also gives you `process.env` (environment variables), `process.exit()` (quit the program), and more — but `argv` is the one you reach for when building CLIs.

> Everything in `argv` is a **string** — even numbers. That's why later code does `parseInt(argument)` to turn `"3"` into the number `3`.

---

## 7. Project: A To-Do CLI

**Source:** `todo/todo.js`, `tasks.json`

Now we can read the whole to-do program and see every piece working together. It's a command-line task manager that stores its data in `tasks.json`.

```js
const fs = require("fs");
const filePath = "./tasks.json";

// --- read the saved tasks (or [] if none yet) ---
const loadTasks = () => {
  try {
    const dataBuffer = fs.readFileSync(filePath);
    const dataJSON   = dataBuffer.toString();
    return JSON.parse(dataJSON);
  } catch (error) {
    return [];
  }
};

// --- write the tasks back to disk ---
const saveTasks = (tasks) => {
  const dataJSON = JSON.stringify(tasks);
  fs.writeFileSync(filePath, dataJSON);
};

// --- add a new task ---
const addTask = (task) => {
  const tasks = loadTasks();      // 1. read current list
  tasks.push({ task });           // 2. add the new one  ({ task } = { task: task })
  saveTasks(tasks);               // 3. save it back
  console.log("Task addded ", task);
};

// --- print all tasks, numbered ---
const listTasks = () => {
  const tasks = loadTasks();
  tasks.forEach((task, index) => console.log(`${index + 1} - ${task.task}`));
};

// --- figure out what the user wants and do it ---
const command  = process.argv[2];
const argument = process.argv[3];

if (command === "add") {
  addTask(argument);
} else if (command === "list") {
  listTasks();
} else if (command === "remove") {
  removeTask(parseInt(argument));
} else {
  console.log("Command not found !");
}
```

### How to use it

```bash
node todo.js add "buy milk"      # → Task addded  buy milk
node todo.js add "record videos" # → Task addded  record videos
node todo.js list                # → 1 - buy milk
                                 #   2 - record videos
```

After those commands, `tasks.json` holds:

```json
[{"task":"buy milk"},{"task":"record videos"}]
```

### Things worth noticing

- **`{ task }` is shorthand** for `{ task: task }` (ES6 object property shorthand). Each task is stored as a tiny object `{ task: "buy milk" }` rather than a bare string — that makes it easy to add more fields later (like `done: false`).
- **The dispatch at the bottom** is the heart of any CLI: read the command, match it with `if/else`, call the right function. This is a hand-rolled version of what libraries like `yargs` or `commander` do for bigger tools.
- **A bug worth spotting:** the `remove` branch calls `removeTask(...)`, but `removeTask` is *never defined* in the file. Running `node todo.js remove 1` would crash with `ReferenceError: removeTask is not defined`. This is a great exercise: implement it yourself. A correct version:

  ```js
  const removeTask = (index) => {
    const tasks = loadTasks();
    const filtered = tasks.filter((_, i) => i !== index - 1); // index is 1-based
    saveTasks(filtered);
    console.log("Task removed at position", index);
  };
  ```

---

## 8. The `os` Module — System Information

**Source:** `eventLogger/logger.js`

`os` ("operating system") gives your program facts about the machine it's running on — how much memory exists, how many CPUs, what platform, the current user, and so on. The event logger uses it to track memory:

```js
const os = require("os");

os.freemem();    // free memory in bytes (RAM not currently in use)
os.totalmem();   // total memory in bytes (all the RAM in the machine)
```

The logger turns these two numbers into a *percentage of free memory*:

```js
const memoryUsage = (os.freemem() / os.totalmem()) * 100;
// e.g. 0.1641 * 100 = 16.41  →  16.41% of RAM is free
```

`.toFixed(2)` then rounds it to two decimal places for a clean log line (`16.41`). Other handy `os` functions include `os.platform()` ("linux"/"darwin"/"win32"), `os.cpus()` (array of CPU cores), and `os.hostname()`.

---

## 9. Events & the `EventEmitter`

**Source:** `eventLogger/logger.js`

A lot of Node is built around **events**: "when X happens, run Y." You already met this idea in the browser with `addEventListener`. Node has its own, more general version through the built-in **`events`** module and its `EventEmitter` class.

An `EventEmitter` is an object that can do two things:

- **`emit("name", data)`** — announce that something happened ("fire" an event).
- **`on("name", callback)`** — register a listener that runs whenever that event fires.

It's the **publish/subscribe** pattern: one part of your code shouts "this happened!" and any number of other parts can quietly listen and react, without the two needing to know about each other directly.

```js
const EventEmitter = require("events");

// Make our own class that IS an EventEmitter (inherits emit/on).
class Logger extends EventEmitter {
  log(message) {
    this.emit("message", { message }); // fire a "message" event, passing data
  }
}

const logger = new Logger();

// Subscribe: whenever a "message" event fires, run logToFile.
logger.on("message", logToFile);

// Publish: this emits "message", which triggers logToFile.
logger.log("Application started");
```

```
EventEmitter flow:

  logger.log("Application started")
        │  (calls this.emit("message", { message }))
        ▼
  "message" event fires  ───►  every listener registered with
                               logger.on("message", ...) runs
        │
        ▼
  logToFile({ message: "Application started" })  ← writes to the file
```

By `extends EventEmitter`, the `Logger` class inherits all the event machinery (`emit`, `on`, etc.) and adds a friendly `log()` method on top. This is the same inheritance you saw in the JS OOP notes — here it's used to build on Node's built-in capabilities.

**Why bother instead of just calling `logToFile` directly?** Decoupling. The code that *generates* events doesn't need to know what happens to them. Today one listener writes to a file; tomorrow you could add a second listener that also prints to the console or sends an alert — without touching the code that emits the events. Node's own `http` servers, streams, and file watchers all work this way.

---

## 10. Timers — `setInterval` & `setTimeout`

**Source:** `eventLogger/logger.js`

Node has the same timer functions as the browser:

- **`setTimeout(fn, ms)`** — run `fn` **once**, after `ms` milliseconds.
- **`setInterval(fn, ms)`** — run `fn` **repeatedly**, every `ms` milliseconds, forever (until stopped).

The logger uses `setInterval` to record memory usage every 3 seconds:

```js
setInterval(() => {
  const memoryUsage = (os.freemem() / os.totalmem()) * 100;
  logger.log(`Current memory usage: ${memoryUsage.toFixed(2)}`);
}, 3000);   // 3000 ms = 3 seconds
```

Because `setInterval` schedules work *forever*, it keeps the Node process **alive**. This is why `node logger.js` doesn't exit — there's always another tick coming in 3 seconds. (You stop it with `Ctrl+C`, or in code with `clearInterval(id)`.)

This ties back to the **event loop** from the JS notes: the interval callback isn't run by blocking; Node registers the timer, keeps doing other work, and runs the callback each time the 3-second mark passes and the call stack is free.

---

## 11. Project: An Event Logger

**Source:** `eventLogger/logger.js`, `eventLogger/eventlog.txt`

Putting sections 4, 8, 9, and 10 together, here is the complete logger. It combines a custom event emitter, the OS module, a timer, and file appending into a small monitoring tool.

```js
const fs = require("fs");
const os = require("os");
const EventEmitter = require("events");

// 1. A Logger that emits a "message" event whenever you log something.
class Logger extends EventEmitter {
  log(message) {
    this.emit("message", { message });
  }
}

const logger  = new Logger();
const logFile = "./eventlog.txt";

// 2. The listener: format the message with a timestamp and append it to the file.
const logToFile = (event) => {
  const logMessage = `${new Date().toISOString()} - ${event.message} \n`;
  fs.appendFileSync(logFile, logMessage);
};

// 3. Wire the listener to the event.
logger.on("message", logToFile);

// 4. Every 3 seconds, log the current free-memory percentage.
setInterval(() => {
  const memoryUsage = (os.freemem() / os.totalmem()) * 100;
  logger.log(`Current memory usage: ${memoryUsage.toFixed(2)}`);
}, 3000);

// 5. Log two startup messages immediately.
logger.log("Application started");
logger.log("Application event occurred");
```

### What it produces

Each call to `logger.log(...)` fires a `"message"` event → `logToFile` runs → a timestamped line is appended to `eventlog.txt`. After running for a while you get:

```
2024-09-11T13:13:23.458Z - Application started
2024-09-11T13:13:23.459Z - Application event occurred
2024-09-11T13:13:26.459Z - Current memory usage: 16.41
2024-09-11T13:13:29.459Z - Current memory usage: 16.50
2024-09-11T13:13:32.460Z - Current memory usage: 16.75
...
```

The two "Application..." lines appear instantly (they're logged at the very end of the script). Then, every 3 seconds, a fresh memory reading gets appended — and it keeps going until you press `Ctrl+C`.

`new Date().toISOString()` produces the standardised timestamp (`2024-09-11T13:13:23.458Z`) — sortable, timezone-neutral, and perfect for logs.

```
How a single log line comes to exist:

  setInterval fires (every 3s)
        │
        ▼
  logger.log("Current memory usage: 16.41")
        │  emits "message"
        ▼
  logToFile(event)
        │  builds "2024-...Z - Current memory usage: 16.41 \n"
        ▼
  fs.appendFileSync  →  line added to eventlog.txt
```

---

## 12. The `http` Module — Building a Server

**Source:** `server/server.js`

The `http` module lets your program **become a web server** — it can listen on a port, receive HTTP requests from browsers, and send back responses. This is the foundation of every Node web backend (Express and other frameworks are built on top of this).

```js
const http = require("http");
const port = 3000;

const server = http.createServer((req, res) => {
  // This callback runs ONCE FOR EVERY incoming request.
  // req = the request  (what the browser asked for)
  // res = the response (what we send back)
});

server.listen(port, () => {
  console.log(`Server is listening on port ${port}`);
});
```

Two pieces:

1. **`http.createServer(callback)`** — creates the server. The callback you give it runs every time a request comes in, receiving two objects: `req` (the incoming request) and `res` (the response you build and send).
2. **`server.listen(port, callback)`** — starts the server listening on a **port** (here, 3000). The port is like a door number on your machine; the browser reaches this server at `http://localhost:3000`. The callback runs once, when the server has successfully started.

Because the server is holding the port open and waiting for requests, the process **stays alive** — just like the logger's interval kept it alive.

### Reading the request and writing the response

```js
req.url    // the path the browser asked for: "/", "/about.html", "/style.css"...

res.writeHead(200, { "Content-Type": "text/html" }); // status code + headers
res.end(content, "utf-8");                            // send the body and finish
```

- **`req.url`** tells you *what* the user wants. Visiting `localhost:3000/about.html` gives `req.url === "/about.html"`.
- **`res.writeHead(status, headers)`** sets the HTTP status code (`200` = OK, `404` = Not Found) and headers. The crucial header is `Content-Type`, which tells the browser *what kind of data* it's getting so it knows how to handle it.
- **`res.end(data)`** sends the response body and signals "I'm done." Every request must be ended, or the browser hangs waiting forever.

```
HTTP request/response cycle:

  Browser                         Your Node server
  ───────                         ────────────────
  GET /about.html   ──────────►   createServer callback runs
                                  req.url === "/about.html"
                                       │
                                  read the file, set headers
                                       │
  page renders      ◄──────────   res.writeHead(200, ...) + res.end(content)
```

---

## 13. The `path` Module & `__dirname`

**Source:** `server/server.js`

When your program works with files, building file paths by hand with string concatenation (`folder + "/" + file`) is fragile — different operating systems use different separators (`/` on Linux/macOS, `\` on Windows), and you get double-slash or missing-slash bugs. The **`path`** module builds correct paths for you.

```js
const path = require("path");

path.join(__dirname, "index.html");
// safely joins folder + file with the right separator for this OS

path.extname("index.html");   // ".html"  — pulls out the file extension
```

- **`path.join(...parts)`** — glues path pieces together using the correct separator and cleans up any redundant slashes.
- **`path.extname(file)`** — returns the extension (`.html`, `.css`, `.png`). The server uses this to decide the `Content-Type`.

### `__dirname` — "the folder this file lives in"

`__dirname` is a special variable Node fills in automatically: it's the **absolute path of the directory containing the current file**. (There's also `__filename` for the file itself.)

Why it matters: if you use a relative path like `"./index.html"`, it's resolved relative to *wherever you ran `node` from*, not where the script is. That breaks the moment you run the server from a different folder. `path.join(__dirname, "index.html")` always points to the file next to the script, no matter where you launched it from.

```
The difference:

  "./index.html"
     → relative to your current terminal location — fragile

  path.join(__dirname, "index.html")
     → always /full/path/to/server/index.html — rock solid
```

---

## 14. Project: A Static File Server

**Source:** `server/server.js`, `server/index.html`, `server/about.html`, `server/contact.html`

This program is a complete (if minimal) **static file server**: it serves HTML files from its own folder to any browser that asks. Visiting `localhost:3000/` serves `index.html`; `localhost:3000/about.html` serves `about.html`, and so on.

```js
const http = require("http");
const fs   = require("fs");
const path = require("path");

const port = 3000;

const server = http.createServer((req, res) => {
  // 1. Work out which file the browser wants.
  //    "/" is a special case → serve index.html.
  const filePath = path.join(
    __dirname,
    req.url === "/" ? "index.html" : req.url
  );
  console.log(filePath);

  // 2. Pick the right Content-Type based on the file extension.
  const extName = String(path.extname(filePath)).toLowerCase();

  const mimeTypes = {
    ".html": "text/html",
    ".css":  "text/css",
    ".js":   "text/javascript",
    ".png":  "text/png",
  };
  const contentType = mimeTypes[extName] || "application/octet-stream";

  // 3. Read the file and respond.
  fs.readFile(filePath, (err, content) => {
    if (err) {
      if (err.code === "ENOENT") {            // ENOENT = "Error NO ENTry" = file not found
        res.writeHead(404, { "Content-Type": "text/html" });
        res.end("404: File Not Found BRooooo");
      }
    } else {
      res.writeHead(200, { "Content-Type": contentType });
      res.end(content, "utf-8");
    }
  });
});

server.listen(port, () => {
  console.log(`Server is listening on port ${port}`);
});
```

### Walking through one request

Say the browser requests `localhost:3000/about.html`:

1. **Resolve the path.** `req.url` is `"/about.html"`. Since it's not `"/"`, `filePath` becomes `<folder>/about.html` via `path.join(__dirname, "/about.html")`.
2. **Determine the content type.** `path.extname` gives `.html`, which maps to `"text/html"` in the `mimeTypes` lookup. If the extension isn't in the table, it falls back to `"application/octet-stream"` — the generic "raw bytes, just download it" type.
3. **Read and send.** `fs.readFile` reads the file asynchronously. On success (`else` branch), it sends `200 OK` with the right content type and the file contents. On failure, if the error code is `ENOENT` (file doesn't exist), it sends a `404` with a friendly (if cheeky) message.

```
The MIME-type lookup — why it matters:

  file        extname   →  Content-Type        browser does
  ─────────   ───────      ────────────        ─────────────────────
  index.html  .html        text/html           renders as a web page
  style.css   .css         text/css            applies as a stylesheet
  app.js      .js          text/javascript     runs as a script
  (unknown)   .xyz         application/octet-   offers a download
                           stream
```

If you sent an HTML file with the wrong content type, the browser would show the raw HTML source as plain text instead of rendering it. The `Content-Type` header is how the browser knows what it's looking at.

### Why `readFile` (async) here, not `readFileSync`?

A server may handle **many requests at the same time**. If it used `readFileSync`, reading one large file would *freeze the entire server* — no other request could be handled until that read finished. The async `readFile` lets Node start the read, move on to serve other requests, and come back to send the response when the data is ready. For servers, non-blocking I/O is the whole point of Node. (Next section.)

### A small thing to improve

If `fs.readFile` fails with an error that *isn't* `ENOENT` (say, a permissions problem), the current code sends **no response at all** — the inner `if (err.code === "ENOENT")` has no `else`, so the browser would hang. A robust server adds a generic `500` response for other errors:

```js
} else {
  res.writeHead(500);
  res.end("500: Server Error");
}
```

---

## 15. Sync vs Async in Node

This distinction shows up across all three projects, so it's worth pinning down. Node is **single-threaded** (one call stack, like browser JS), so *how* you do slow work — reading files, network calls — decides whether your program stays responsive.

- **Synchronous (`readFileSync`, `writeFileSync`, `appendFileSync`)**: the line *blocks*. Nothing else runs until the operation completes. Simple, predictable, and totally fine for a short-lived CLI tool like the to-do app — there's nobody else waiting.

- **Asynchronous (`readFile` with a callback)**: the operation is handed off to run in the background; your code continues, and a callback fires when it's done. Essential for servers, where blocking would stall every other user.

```
Why a server must be async:

  SYNCHRONOUS server (bad):
    Request A → readFileSync (busy 200ms) ─── everything frozen ───►
    Request B waits...                                  then runs
    Request C waits...

  ASYNCHRONOUS server (good):
    Request A → readFile starts ─┐
    Request B → readFile starts ─┤  all in flight at once
    Request C → readFile starts ─┘
    callbacks fire as each file finishes — nobody blocks the others
```

**Rule of thumb:** use the `Sync` versions for simple scripts and startup code where blocking doesn't hurt; use the async versions inside servers and anywhere that handles concurrent work. The to-do app and logger use `Sync` (they're simple); the server uses async (it serves many requests).

---

## 16. Quick Reference — Mental Model

```
Node.js — how the pieces in this folder connect:

  ┌────────────────────────────────────────────────────────────────────┐
  │  THE RUNTIME                                                        │
  │  V8 engine (runs JS)  +  core modules  +  the `process` global     │
  │  Run a file with:  node file.js                                     │
  └───────────────────────────────┬────────────────────────────────────┘
                                  │  require("...") brings in modules
            ┌─────────────────────┼─────────────────────┐
            ▼                     ▼                     ▼
  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────┐
  │  fs              │  │  os / events /   │  │  http / path         │
  │  read & write    │  │  timers          │  │  build a web server  │
  │  files           │  │  react & repeat  │  │  serve files         │
  └────────┬─────────┘  └────────┬─────────┘  └──────────┬───────────┘
           │                     │                       │
           ▼                     ▼                       ▼
  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────┐
  │  TODO CLI        │  │  EVENT LOGGER    │  │  STATIC FILE SERVER  │
  │  process.argv +  │  │  EventEmitter +  │  │  http.createServer + │
  │  JSON + fs       │  │  os + setInterval│  │  fs + path + MIME    │
  │  (sync, exits)   │  │  + appendFile    │  │  (async, stays up)   │
  │                  │  │  (stays alive)   │  │                      │
  └──────────────────┘  └──────────────────┘  └──────────────────────┘

  Core skills threaded through all three:
    • require() to load modules        • Buffer.toString() & JSON
    • sync vs async file I/O           • the process stays alive only
    • the event loop (timers, servers,   while there's pending work
      and callbacks all rely on it)
```

### Cheat sheet of everything used in this folder

| Thing | Module | What it does |
|---|---|---|
| `require("fs")` | core | read/write files |
| `readFileSync` / `writeFileSync` / `appendFileSync` | fs | blocking file ops |
| `readFile(path, cb)` | fs | non-blocking read (error-first callback) |
| `require("os")` | core | system info |
| `os.freemem()` / `os.totalmem()` | os | RAM in bytes |
| `require("events")` → `EventEmitter` | core | emit/on event system |
| `setInterval(fn, ms)` | global | run `fn` repeatedly |
| `require("http")` | core | create a web server |
| `http.createServer(cb)` / `server.listen(port)` | http | handle requests on a port |
| `req.url` / `res.writeHead` / `res.end` | http | read request, send response |
| `require("path")` | core | safe path building |
| `path.join` / `path.extname` | path | join paths, get extension |
| `__dirname` | global | folder of the current file |
| `process.argv` | global | command-line arguments |
| `JSON.parse` / `JSON.stringify` | global | text ⇄ JS value |

---

*All examples taken directly from the `nodejs/` project files: `todo/todo.js`, `eventLogger/logger.js`, and `server/server.js`.*
