# Deno 2.x Technical Knowledge Base

This document provides a comprehensive technical overview of Deno 2.x, designed for an AI. It synthesizes official migration guides and developer documentation to serve as a single source of truth, focusing on breaking changes, new features, and best practices.

## 1. Deno 1.x to 2.x Migration and Core Changes

This section details the critical changes introduced in Deno 2.x, focusing on migration from Deno 1.x. The primary goal of Deno 2.x is to enhance Deno's usability at scale, emphasizing seamless interoperability with the Node.js and npm ecosystem.

### 1.1. High-Level Strategic Changes

*   **Node.js & npm Backwards Compatibility:** Deno 2 is designed to be backwards compatible with Node.js and npm. It understands `package.json`, the `node_modules` directory, and npm workspaces. This allows for incremental adoption of Deno's toolchain (`deno fmt`, `deno lint`, `deno task`) within existing Node.js projects.
*   **Dependency Installation Behavior:** In projects with a `package.json`, npm packages are no longer auto-installed by default. The recommended practice is to run `deno install` explicitly. To restore the Deno 1.x auto-install behavior, configure `deno.json`:
    ```json
    {
      "nodeModulesDir": "auto"
    }
    ```
*   **Long-Term Support (LTS):** Starting with Deno v2.1.0 (Nov 2024), an LTS channel will be available. LTS versions are supported for 6 months with bug fixes and critical performance updates.
*   **Improved Dependency Management:** The introduction of `deno add` and `deno remove` commands streamlines dependency management for both JSR and npm packages within `deno.json`.
*   **Monorepo & Private Registry Support:** Deno 2 fully supports monorepos (workspaces) and private npm registries via `.npmrc` files, catering to large-scale team development.
*   **Framework Support:** Enhanced Node.js compatibility enables support for major frameworks like Next.js, SvelteKit, Remix, Nuxt, and more, often requiring minimal changes (e.g., replacing `npm run dev` with `deno task dev`).

### 1.2. Configuration Changes (`deno.json`)

#### `nodeModulesDir`
The boolean value for this option is deprecated. It now accepts a string enum to define behavior. The default behavior has also changed.

*   **Old Signature:** `"nodeModulesDir": false | true`
*   **New Signature:** `"nodeModulesDir": "none" | "auto" | "manual"`

*   **Behavior Change (No `package.json`):**
    *   **Deno 1.x Default:** `false` (equivalent to new `"none"`)
    *   **Deno 2.x Default:** `"none"` (No change in behavior)

*   **Behavior Change (With `package.json`):**
    *   **Deno 1.x Default:** `true` (equivalent to new `"auto"`)
    *   **Deno 2.x Default:** `"manual"`
    *   **Action Required:** To retain the Deno 1.x auto-installing behavior in a project with `package.json`, you **must** explicitly set `"nodeModulesDir": "auto"` in your `deno.json`. The new default, `"manual"`, requires the user to manage the `node_modules` directory with commands like `deno install`.

### 1.3. CLI Changes

#### `deno bundle`
*   **Status:** **Removed**.
*   **Replacement:** Use a dedicated bundler like **esbuild** with **esbuild-deno-loader**.
*   **Example:**
    ```typescript
    import * as esbuild from "npm:esbuild";
    import { denoPlugins } from "jsr:@luca/esbuild-deno-loader";

    await esbuild.build({
      plugins: [...denoPlugins()],
      entryPoints: ["https://deno.land/std@0.185.0/bytes/mod.ts"],
      outfile: "./dist/bytes.esm.js",
      bundle: true,
      format: "esm",
    });

    esbuild.stop();
    ```

#### `deno cache`
*   **Status:** **Merged** into `deno install`.
*   **Migration:**
    ```diff
    - deno cache main.ts
    + deno install --entrypoint main.ts
    ```

#### `deno vendor`
*   **Status:** **Replaced** by a configuration option in `deno.json`.
*   **Migration:**
    ```json
    {
      "vendor": true
    }
    ```

#### `--allow-none`
*   **Status:** **Replaced**.
*   **Migration:**
    ```diff
    - deno test --allow-none
    + deno test --permit-no-files
    ```

#### `--jobs`
*   **Status:** **Replaced** by an environment variable.
*   **Migration:**
    ```diff
    - deno test --jobs=4 --parallel
    + DENO_JOBS=4 deno test --parallel
    ```

#### `--ts` / `-T`
*   **Status:** **Replaced**.
*   **Migration:**
    ```diff
    - deno run --ts script.ts
    + deno run --ext=ts script.ts
    ```

#### `--trace-ops`
*   **Status:** **Replaced**.
*   **Migration:**
    ```diff
    - deno test --trace-ops
    + deno test --trace-leaks
    ```

#### `--unstable`
*   **Status:** **Replaced** by granular unstable flags (`--unstable-*`) or configuration options.
*   **Migration (CLI):**
    ```diff
    - deno run --unstable kv.ts
    + deno run --unstable-kv kv.ts
    ```
*   **Migration (`deno.json`):**
    ```json
    {
      "unstable": ["kv"]
    }
    ```

#### Import Assertions
*   **Status:** **Deprecated** following the TC39 proposal downgrade.
*   **Replacement:** Use **Import Attributes**.
*   **Migration:**
    ```diff
    - import data from "./data.json" assert { type: "json" };
    + import data from "./data.json" with { type: "json" };
    ```

### 1.4. API Changes

This section documents the transition from legacy `Deno.*` APIs, often tied to resource IDs (RIDs), to modern, object-oriented, and standard library-based APIs.

#### Deno.Buffer
*   **Status:** **Removed**.
*   **Replacement:** `Buffer` from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import { Buffer } from "jsr:@std/io/buffer";
    - const buffer = new Deno.Buffer();
    + const buffer = new Buffer();
    ```

#### Deno.Closer
*   **Status:** **Removed**.
*   **Replacement:** `Closer` type from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import type { Closer } from "jsr:@std/io/types";
    - function foo(closer: Deno.Closer) {}
    + function foo(closer: Closer) {}
    ```

#### Deno.close()
*   **Status:** **Removed**.
*   **Replacement:** The `.close()` method on the resource object itself.
*   **Migration:**
    ```diff
    const conn = await Deno.connect({ port: 80 });
    // ...
    - Deno.close(conn.rid);
    + conn.close();

    const file = await Deno.open("/foo/bar.txt");
    // ...
    - Deno.close(file.rid);
    + file.close();
    ```

#### Deno.Conn.prototype.rid
*   **Status:** **Removed**.
*   **Replacement:** Use instance methods directly on the `Deno.Conn` object.
*   **Migration:**
    ```diff
    const conn = await Deno.connect({ port: 80 });
    const buffer = new Uint8Array(1_024);
    - await Deno.read(conn.rid, buffer);
    + await conn.read(buffer);

    - await Deno.shutdown(conn.rid);
    + await conn.closeWrite();

    - Deno.close(conn.rid);
    + conn.close();
    ```

#### Deno.ConnectTlsOptions: `certChain`, `certFile`, `privateKey`
*   **Status:** **Replaced** by unified options.
*   **Replacement:** Use `cert` and `key` options, passing the file contents directly.
*   **Migration:**
    ```diff
    using conn = await Deno.connectTls({
      hostname: "192.0.2.1",
      port: 80,
      caCerts: [caCert],
    - certChain: Deno.readTextFileSync("./server.crt"),
    + cert: Deno.readTextFileSync("./server.crt"),
    - certFile: "./server.crt", // also replaced by cert
    + cert: Deno.readTextFileSync("./server.crt"),
    - keyFile: "./server.key", // also replaced by key
    + key: Deno.readTextFileSync("./server.key"),
    });
    ```

#### Deno.copy()
*   **Status:** **Removed**.
*   **Replacement:** `copy()` from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import { copy } from "jsr:@std/io/copy";
    using file = await Deno.open("/foo/bar.txt");
    - await Deno.copy(file, Deno.stdout);
    + await copy(file, Deno.stdout);
    ```

#### Deno.customInspect
*   **Status:** **Removed**.
*   **Replacement:** `Symbol.for("Deno.customInspect")`.
*   **Migration:**
    ```diff
    class Foo {
    - [Deno.customInspect]() {}
    + [Symbol.for("Deno.customInspect")]() {}
    }
    ```

#### Deno.fdatasync() / Deno.fdatasyncSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.syncData()` / `syncDataSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt", { write: true });
    // ...
    - await Deno.fdatasync(file.rid);
    + await file.syncData();
    ```

#### Deno.File
*   **Status:** **Renamed**.
*   **Replacement:** `Deno.FsFile`.
*   **Migration:**
    ```diff
    - function foo(file: Deno.File) {}
    + function foo(file: Deno.FsFile) {}
    ```

#### Deno.flock() / Deno.flockSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.lock()` / `lockSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt");
    - await Deno.flock(file.rid);
    + await file.lock();
    ```

#### Deno.FsFile.prototype.rid
*   **Status:** **Removed**.
*   **Replacement:** Use instance methods directly on the `Deno.FsFile` object.
*   **Migration:**
    ```diff
    const file = await Deno.open("/foo/bar.txt");
    const buffer = new Uint8Array(1_024);
    - await Deno.read(file.rid, buffer);
    + await file.read(buffer);

    - Deno.close(file.rid);
    + file.close();
    ```

#### Deno.fstat() / Deno.fstatSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.stat()` / `statSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt");
    - const fileInfo = await Deno.fstat(file.rid);
    + const fileInfo = await file.stat();
    ```

#### Deno.FsWatcher.prototype.rid
*   **Status:** **Removed**.
*   **Replacement:** Use instance methods directly on the `Deno.FsWatcher` object.
*   **Migration:**
    ```diff
    using watcher = Deno.watchFs("/dir");
    // ...
    - Deno.close(watcher.rid);
    + watcher.close();
    ```

#### Deno.fsync() / Deno.fsyncSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.sync()` / `syncSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt", { write: true });
    // ...
    - await Deno.fsync(file.rid);
    + await file.sync();
    ```

#### Deno.ftruncate() / Deno.ftruncateSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.truncate()` / `truncateSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt");
    - await Deno.ftruncate(file.rid, 7);
    + await file.truncate(7);
    ```

#### Deno.funlock() / Deno.funlockSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.unlock()` / `unlockSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt");
    // ...
    - await Deno.funlock(file.rid);
    + await file.unlock();
    ```

#### Deno.futime() / Deno.futimeSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.utime()` / `utimeSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt");
    - await Deno.futime(file.rid, 1556495550, new Date());
    + await file.utime(1556495550, new Date());
    ```

#### Deno.isatty()
*   **Status:** **Removed**.
*   **Replacement:** `.isTerminal()` method on standard streams or `Deno.FsFile` instances.
*   **Migration:**
    ```diff
    - Deno.isatty(Deno.stdin.rid);
    + Deno.stdin.isTerminal();

    using file = await Deno.open("/dev/tty");
    - Deno.isatty(file.rid);
    + file.isTerminal();
    ```

#### Deno.iter() / Deno.iterSync()
*   **Status:** **Removed**.
*   **Replacement:** `iterateReader()` / `iterateReaderSync()` from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import { iterateReader } from "jsr:@std/io/iterate-reader";
    - for await (const chunk of Deno.iter(Deno.stdin)) {
    + for await (const chunk of iterateReader(Deno.stdin)) {
      // ...
    }
    ```

#### Deno.Listener.prototype.rid
*   **Status:** **Removed**.
*   **Replacement:** Use instance methods directly on the `Deno.Listener` object.
*   **Migration:**
    ```diff
    const listener = Deno.listen({ port: 80 });
    // ...
    - Deno.close(listener.rid);
    + listener.close();
    ```

#### Deno.ListenTlsOptions: `certChain`, `certFile`, `keyFile`
*   **Status:** **Replaced** by unified options.
*   **Replacement:** Use `cert` and `key` options, passing the file contents directly.
*   **Migration:**
    ```diff
    using listener = Deno.listenTls({
      port: 443,
    - certFile: "./server.crt",
    + cert: Deno.readTextFileSync("./server.crt"),
    - keyFile: "./server.key",
    + key: Deno.readTextFileSync("./server.key"),
    });
    ```

#### Deno.metrics()
*   **Status:** **Removed**.
*   **Replacement:** None.

#### Deno.readAll() / Deno.readAllSync()
*   **Status:** **Removed**.
*   **Replacement:** `readAll()` / `readAllSync()` from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import { readAll } from "jsr:@std/io/read-all";
    - const data = await Deno.readAll(Deno.stdin);
    + const data = await readAll(Deno.stdin);
    ```

#### Deno.Reader / Deno.ReaderSync
*   **Status:** **Removed**.
*   **Replacement:** `Reader` / `ReaderSync` types from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import type { Reader } from "jsr:@std/io/types";
    - function foo(reader: Deno.Reader) {}
    + function foo(reader: Reader) {}
    ```

#### Deno.read() / Deno.readSync()
*   **Status:** **Removed**.
*   **Replacement:** The `.read()` / `.readSync()` method on the resource object itself.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt");
    const buffer = new Uint8Array(1_024);
    - await Deno.read(file.rid, buffer);
    + await file.read(buffer);
    ```

#### Deno.resources()
*   **Status:** **Removed**.
*   **Replacement:** None. Resource IDs (RIDs) are being phased out.

#### Deno.run()
*   **Status:** **Soft-removed**. Types are removed, but the implementation remains for backward compatibility.
*   **Replacement:** `new Deno.Command()`.
*   **Migration:**
    ```diff
    - const process = Deno.run({ cmd: [ "echo", "hello" ], stdout: "piped" });
    - const [{ success }, stdout] = await Promise.all([
    -   process.status(),
    -   process.output(),
    - ]);
    - process.close();
    + const command = new Deno.Command("echo", { args: ["hello"] });
    + const { success, stdout } = await command.output();
    ```
*   **Note for legacy code:** Use `@ts-ignore` to suppress TypeScript errors.
    ```typescript
    // @ts-ignore `Deno.run()` is soft-removed as of Deno 2.
    const process = Deno.run({ cmd: ["echo", "hello"] });
    ```

#### Deno.Seeker / Deno.SeekerSync
*   **Status:** **Removed**.
*   **Replacement:** `Seeker` / `SeekerSync` types from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import type { Seeker } from "jsr:@std/io/types";
    - function foo(seeker: Deno.Seeker) {}
    + function foo(seeker: Seeker) {}
    ```

#### Deno.seek() / Deno.seekSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.seek()` / `seekSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt");
    - await Deno.seek(file.rid, 6, Deno.SeekMode.Start);
    + await file.seek(6, Deno.SeekMode.Start);
    ```

#### Deno.serveHttp()
*   **Status:** **Soft-removed**.
*   **Replacement:** `Deno.serve()`.
*   **Migration:**
    ```diff
    - const conn = Deno.listen({ port: 80 });
    - const httpConn = Deno.serveHttp(await conn.accept());
    - const e = await httpConn.nextRequest();
    - if (e) { e.respondWith(new Response("Hello")); }
    + Deno.serve({ port: 80 }, () => new Response("Hello"));
    ```

#### Deno.Server
*   **Status:** **Renamed**.
*   **Replacement:** `Deno.HttpServer`.
*   **Migration:**
    ```diff
    - function foo(server: Deno.Server) {}
    + function foo(server: Deno.HttpServer) {}
    ```

#### Deno.shutdown()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.Conn.prototype.closeWrite()`.
*   **Migration:**
    ```diff
    using conn = await Deno.connect({ port: 80 });
    - await Deno.shutdown(conn.rid);
    + await conn.closeWrite();
    ```

#### Deno.stdin/stdout/stderr.prototype.rid
*   **Status:** **Soft-removed**.
*   **Replacement:** Use instance methods directly on the standard stream objects.
*   **Migration:**
    ```diff
    - if (Deno.isatty(Deno.stdout.rid)) {}
    + if (Deno.stdout.isTerminal()) {}

    const data = new TextEncoder().encode("Hello");
    - await Deno.write(Deno.stdout.rid, data);
    + await Deno.stdout.write(data);
    ```

#### `rid` properties on `Deno.TcpConn`, `Deno.TlsConn`, `Deno.TlsListener`, `Deno.UnixConn`
*   **Status:** All **Removed**.
*   **Replacement:** Use instance methods (`.read()`, `.write()`, `.close()`, `.closeWrite()`, etc.) directly on the connection/listener objects. The pattern is consistent across all resource types.

#### Deno.writeAll() / Deno.writeAllSync()
*   **Status:** **Removed**.
*   **Replacement:** `writeAll()` / `writeAllSync()` from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import { writeAll } from "jsr:@std/io/write-all";
    const data = new TextEncoder().encode("Hello");
    - await Deno.writeAll(Deno.stdout, data);
    + await writeAll(Deno.stdout, data);
    ```

#### Deno.Writer / Deno.WriterSync
*   **Status:** **Removed**.
*   **Replacement:** `Writer` / `WriterSync` types from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import type { Writer } from "jsr:@std/io/types";
    - function foo(writer: Deno.Writer) {}
    + function foo(writer: Writer) {}
    ```

#### Deno.write() / Deno.writeSync()
*   **Status:** **Removed**.
*   **Replacement:** The `.write()` / `.writeSync()` method on the resource object itself.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt", { write: true });
    const buffer = new TextEncoder().encode("My message");
    - await Deno.write(file.rid, buffer);
    + await file.write(buffer);
    ```

#### new Deno.FsFile()
*   **Status:** **Removed**.
*   **Replacement:** Use `Deno.open()` or `Deno.openSync()`.
*   **Migration:**
    ```diff
    - const file = new Deno.FsFile(3); // 3 is an example file descriptor
    + using file = await Deno.open("/foo/bar.txt");
    ```

#### window
*   **Status:** **Deprecated** in favor of the cross-platform standard.
*   **Replacement:** `globalThis`.
*   **Migration:**
    ```diff
    - const hashBuffer = await window.crypto.subtle.digest("SHA-256", messageBuffer);
    + const hashBuffer = await globalThis.crypto.subtle.digest("SHA-256", messageBuffer);
    ```Of course. Here is the first part of the complete markdown documentation for training an AI on Deno 2.x. This section focuses on the most critical information: the breaking changes and migration path from version 1.x to 2.x, directly addressing your core problem.

The information is structured for a technical audience, ensuring all details from the provided migration guide are preserved and categorized logically.

***

# Deno 2.x Technical Knowledge Base

This document provides a comprehensive technical overview of Deno 2.x, designed for an AI. It synthesizes official migration guides and developer documentation to serve as a single source of truth, focusing on breaking changes, new features, and best practices.

## 1. Deno 1.x to 2.x Migration and Core Changes

This section details the critical changes introduced in Deno 2.x, focusing on migration from Deno 1.x. The primary goal of Deno 2.x is to enhance Deno's usability at scale, emphasizing seamless interoperability with the Node.js and npm ecosystem.

### 1.1. High-Level Strategic Changes

*   **Node.js & npm Backwards Compatibility:** Deno 2 is designed to be backwards compatible with Node.js and npm. It understands `package.json`, the `node_modules` directory, and npm workspaces. This allows for incremental adoption of Deno's toolchain (`deno fmt`, `deno lint`, `deno task`) within existing Node.js projects.
*   **Dependency Installation Behavior:** In projects with a `package.json`, npm packages are no longer auto-installed by default. The recommended practice is to run `deno install` explicitly. To restore the Deno 1.x auto-install behavior, configure `deno.json`:
    ```json
    {
      "nodeModulesDir": "auto"
    }
    ```
*   **Long-Term Support (LTS):** Starting with Deno v2.1.0 (Nov 2024), an LTS channel will be available. LTS versions are supported for 6 months with bug fixes and critical performance updates.
*   **Improved Dependency Management:** The introduction of `deno add` and `deno remove` commands streamlines dependency management for both JSR and npm packages within `deno.json`.
*   **Monorepo & Private Registry Support:** Deno 2 fully supports monorepos (workspaces) and private npm registries via `.npmrc` files, catering to large-scale team development.
*   **Framework Support:** Enhanced Node.js compatibility enables support for major frameworks like Next.js, SvelteKit, Remix, Nuxt, and more, often requiring minimal changes (e.g., replacing `npm run dev` with `deno task dev`).

### 1.2. Configuration Changes (`deno.json`)

#### `nodeModulesDir`
The boolean value for this option is deprecated. It now accepts a string enum to define behavior. The default behavior has also changed.

*   **Old Signature:** `"nodeModulesDir": false | true`
*   **New Signature:** `"nodeModulesDir": "none" | "auto" | "manual"`

*   **Behavior Change (No `package.json`):**
    *   **Deno 1.x Default:** `false` (equivalent to new `"none"`)
    *   **Deno 2.x Default:** `"none"` (No change in behavior)

*   **Behavior Change (With `package.json`):**
    *   **Deno 1.x Default:** `true` (equivalent to new `"auto"`)
    *   **Deno 2.x Default:** `"manual"`
    *   **Action Required:** To retain the Deno 1.x auto-installing behavior in a project with `package.json`, you **must** explicitly set `"nodeModulesDir": "auto"` in your `deno.json`. The new default, `"manual"`, requires the user to manage the `node_modules` directory with commands like `deno install`.

### 1.3. CLI Changes

#### `deno bundle`
*   **Status:** **Removed**.
*   **Replacement:** Use a dedicated bundler like **esbuild** with **esbuild-deno-loader**.
*   **Example:**
    ```typescript
    import * as esbuild from "npm:esbuild";
    import { denoPlugins } from "jsr:@luca/esbuild-deno-loader";

    await esbuild.build({
      plugins: [...denoPlugins()],
      entryPoints: ["https://deno.land/std@0.185.0/bytes/mod.ts"],
      outfile: "./dist/bytes.esm.js",
      bundle: true,
      format: "esm",
    });

    esbuild.stop();
    ```

#### `deno cache`
*   **Status:** **Merged** into `deno install`.
*   **Migration:**
    ```diff
    - deno cache main.ts
    + deno install --entrypoint main.ts
    ```

#### `deno vendor`
*   **Status:** **Replaced** by a configuration option in `deno.json`.
*   **Migration:**
    ```json
    {
      "vendor": true
    }
    ```

#### `--allow-none`
*   **Status:** **Replaced**.
*   **Migration:**
    ```diff
    - deno test --allow-none
    + deno test --permit-no-files
    ```

#### `--jobs`
*   **Status:** **Replaced** by an environment variable.
*   **Migration:**
    ```diff
    - deno test --jobs=4 --parallel
    + DENO_JOBS=4 deno test --parallel
    ```

#### `--ts` / `-T`
*   **Status:** **Replaced**.
*   **Migration:**
    ```diff
    - deno run --ts script.ts
    + deno run --ext=ts script.ts
    ```

#### `--trace-ops`
*   **Status:** **Replaced**.
*   **Migration:**
    ```diff
    - deno test --trace-ops
    + deno test --trace-leaks
    ```

#### `--unstable`
*   **Status:** **Replaced** by granular unstable flags (`--unstable-*`) or configuration options.
*   **Migration (CLI):**
    ```diff
    - deno run --unstable kv.ts
    + deno run --unstable-kv kv.ts
    ```
*   **Migration (`deno.json`):**
    ```json
    {
      "unstable": ["kv"]
    }
    ```

#### Import Assertions
*   **Status:** **Deprecated** following the TC39 proposal downgrade.
*   **Replacement:** Use **Import Attributes**.
*   **Migration:**
    ```diff
    - import data from "./data.json" assert { type: "json" };
    + import data from "./data.json" with { type: "json" };
    ```

### 1.4. API Changes

This section documents the transition from legacy `Deno.*` APIs, often tied to resource IDs (RIDs), to modern, object-oriented, and standard library-based APIs.

#### Deno.Buffer
*   **Status:** **Removed**.
*   **Replacement:** `Buffer` from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import { Buffer } from "jsr:@std/io/buffer";
    - const buffer = new Deno.Buffer();
    + const buffer = new Buffer();
    ```

#### Deno.Closer
*   **Status:** **Removed**.
*   **Replacement:** `Closer` type from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import type { Closer } from "jsr:@std/io/types";
    - function foo(closer: Deno.Closer) {}
    + function foo(closer: Closer) {}
    ```

#### Deno.close()
*   **Status:** **Removed**.
*   **Replacement:** The `.close()` method on the resource object itself.
*   **Migration:**
    ```diff
    const conn = await Deno.connect({ port: 80 });
    // ...
    - Deno.close(conn.rid);
    + conn.close();

    const file = await Deno.open("/foo/bar.txt");
    // ...
    - Deno.close(file.rid);
    + file.close();
    ```

#### Deno.Conn.prototype.rid
*   **Status:** **Removed**.
*   **Replacement:** Use instance methods directly on the `Deno.Conn` object.
*   **Migration:**
    ```diff
    const conn = await Deno.connect({ port: 80 });
    const buffer = new Uint8Array(1_024);
    - await Deno.read(conn.rid, buffer);
    + await conn.read(buffer);

    - await Deno.shutdown(conn.rid);
    + await conn.closeWrite();

    - Deno.close(conn.rid);
    + conn.close();
    ```

#### Deno.ConnectTlsOptions: `certChain`, `certFile`, `privateKey`
*   **Status:** **Replaced** by unified options.
*   **Replacement:** Use `cert` and `key` options, passing the file contents directly.
*   **Migration:**
    ```diff
    using conn = await Deno.connectTls({
      hostname: "192.0.2.1",
      port: 80,
      caCerts: [caCert],
    - certChain: Deno.readTextFileSync("./server.crt"),
    + cert: Deno.readTextFileSync("./server.crt"),
    - certFile: "./server.crt", // also replaced by cert
    + cert: Deno.readTextFileSync("./server.crt"),
    - keyFile: "./server.key", // also replaced by key
    + key: Deno.readTextFileSync("./server.key"),
    });
    ```

#### Deno.copy()
*   **Status:** **Removed**.
*   **Replacement:** `copy()` from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import { copy } from "jsr:@std/io/copy";
    using file = await Deno.open("/foo/bar.txt");
    - await Deno.copy(file, Deno.stdout);
    + await copy(file, Deno.stdout);
    ```

#### Deno.customInspect
*   **Status:** **Removed**.
*   **Replacement:** `Symbol.for("Deno.customInspect")`.
*   **Migration:**
    ```diff
    class Foo {
    - [Deno.customInspect]() {}
    + [Symbol.for("Deno.customInspect")]() {}
    }
    ```

#### Deno.fdatasync() / Deno.fdatasyncSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.syncData()` / `syncDataSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt", { write: true });
    // ...
    - await Deno.fdatasync(file.rid);
    + await file.syncData();
    ```

#### Deno.File
*   **Status:** **Renamed**.
*   **Replacement:** `Deno.FsFile`.
*   **Migration:**
    ```diff
    - function foo(file: Deno.File) {}
    + function foo(file: Deno.FsFile) {}
    ```

#### Deno.flock() / Deno.flockSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.lock()` / `lockSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt");
    - await Deno.flock(file.rid);
    + await file.lock();
    ```

#### Deno.FsFile.prototype.rid
*   **Status:** **Removed**.
*   **Replacement:** Use instance methods directly on the `Deno.FsFile` object.
*   **Migration:**
    ```diff
    const file = await Deno.open("/foo/bar.txt");
    const buffer = new Uint8Array(1_024);
    - await Deno.read(file.rid, buffer);
    + await file.read(buffer);

    - Deno.close(file.rid);
    + file.close();
    ```

#### Deno.fstat() / Deno.fstatSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.stat()` / `statSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt");
    - const fileInfo = await Deno.fstat(file.rid);
    + const fileInfo = await file.stat();
    ```

#### Deno.FsWatcher.prototype.rid
*   **Status:** **Removed**.
*   **Replacement:** Use instance methods directly on the `Deno.FsWatcher` object.
*   **Migration:**
    ```diff
    using watcher = Deno.watchFs("/dir");
    // ...
    - Deno.close(watcher.rid);
    + watcher.close();
    ```

#### Deno.fsync() / Deno.fsyncSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.sync()` / `syncSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt", { write: true });
    // ...
    - await Deno.fsync(file.rid);
    + await file.sync();
    ```

#### Deno.ftruncate() / Deno.ftruncateSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.truncate()` / `truncateSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt");
    - await Deno.ftruncate(file.rid, 7);
    + await file.truncate(7);
    ```

#### Deno.funlock() / Deno.funlockSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.unlock()` / `unlockSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt");
    // ...
    - await Deno.funlock(file.rid);
    + await file.unlock();
    ```

#### Deno.futime() / Deno.futimeSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.utime()` / `utimeSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt");
    - await Deno.futime(file.rid, 1556495550, new Date());
    + await file.utime(1556495550, new Date());
    ```

#### Deno.isatty()
*   **Status:** **Removed**.
*   **Replacement:** `.isTerminal()` method on standard streams or `Deno.FsFile` instances.
*   **Migration:**
    ```diff
    - Deno.isatty(Deno.stdin.rid);
    + Deno.stdin.isTerminal();

    using file = await Deno.open("/dev/tty");
    - Deno.isatty(file.rid);
    + file.isTerminal();
    ```

#### Deno.iter() / Deno.iterSync()
*   **Status:** **Removed**.
*   **Replacement:** `iterateReader()` / `iterateReaderSync()` from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import { iterateReader } from "jsr:@std/io/iterate-reader";
    - for await (const chunk of Deno.iter(Deno.stdin)) {
    + for await (const chunk of iterateReader(Deno.stdin)) {
      // ...
    }
    ```

#### Deno.Listener.prototype.rid
*   **Status:** **Removed**.
*   **Replacement:** Use instance methods directly on the `Deno.Listener` object.
*   **Migration:**
    ```diff
    const listener = Deno.listen({ port: 80 });
    // ...
    - Deno.close(listener.rid);
    + listener.close();
    ```

#### Deno.ListenTlsOptions: `certChain`, `certFile`, `keyFile`
*   **Status:** **Replaced** by unified options.
*   **Replacement:** Use `cert` and `key` options, passing the file contents directly.
*   **Migration:**
    ```diff
    using listener = Deno.listenTls({
      port: 443,
    - certFile: "./server.crt",
    + cert: Deno.readTextFileSync("./server.crt"),
    - keyFile: "./server.key",
    + key: Deno.readTextFileSync("./server.key"),
    });
    ```

#### Deno.metrics()
*   **Status:** **Removed**.
*   **Replacement:** None.

#### Deno.readAll() / Deno.readAllSync()
*   **Status:** **Removed**.
*   **Replacement:** `readAll()` / `readAllSync()` from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import { readAll } from "jsr:@std/io/read-all";
    - const data = await Deno.readAll(Deno.stdin);
    + const data = await readAll(Deno.stdin);
    ```

#### Deno.Reader / Deno.ReaderSync
*   **Status:** **Removed**.
*   **Replacement:** `Reader` / `ReaderSync` types from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import type { Reader } from "jsr:@std/io/types";
    - function foo(reader: Deno.Reader) {}
    + function foo(reader: Reader) {}
    ```

#### Deno.read() / Deno.readSync()
*   **Status:** **Removed**.
*   **Replacement:** The `.read()` / `.readSync()` method on the resource object itself.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt");
    const buffer = new Uint8Array(1_024);
    - await Deno.read(file.rid, buffer);
    + await file.read(buffer);
    ```

#### Deno.resources()
*   **Status:** **Removed**.
*   **Replacement:** None. Resource IDs (RIDs) are being phased out.

#### Deno.run()
*   **Status:** **Soft-removed**. Types are removed, but the implementation remains for backward compatibility.
*   **Replacement:** `new Deno.Command()`.
*   **Migration:**
    ```diff
    - const process = Deno.run({ cmd: [ "echo", "hello" ], stdout: "piped" });
    - const [{ success }, stdout] = await Promise.all([
    -   process.status(),
    -   process.output(),
    - ]);
    - process.close();
    + const command = new Deno.Command("echo", { args: ["hello"] });
    + const { success, stdout } = await command.output();
    ```
*   **Note for legacy code:** Use `@ts-ignore` to suppress TypeScript errors.
    ```typescript
    // @ts-ignore `Deno.run()` is soft-removed as of Deno 2.
    const process = Deno.run({ cmd: ["echo", "hello"] });
    ```

#### Deno.Seeker / Deno.SeekerSync
*   **Status:** **Removed**.
*   **Replacement:** `Seeker` / `SeekerSync` types from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import type { Seeker } from "jsr:@std/io/types";
    - function foo(seeker: Deno.Seeker) {}
    + function foo(seeker: Seeker) {}
    ```

#### Deno.seek() / Deno.seekSync()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.FsFile.prototype.seek()` / `seekSync()`.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt");
    - await Deno.seek(file.rid, 6, Deno.SeekMode.Start);
    + await file.seek(6, Deno.SeekMode.Start);
    ```

#### Deno.serveHttp()
*   **Status:** **Soft-removed**.
*   **Replacement:** `Deno.serve()`.
*   **Migration:**
    ```diff
    - const conn = Deno.listen({ port: 80 });
    - const httpConn = Deno.serveHttp(await conn.accept());
    - const e = await httpConn.nextRequest();
    - if (e) { e.respondWith(new Response("Hello")); }
    + Deno.serve({ port: 80 }, () => new Response("Hello"));
    ```

#### Deno.Server
*   **Status:** **Renamed**.
*   **Replacement:** `Deno.HttpServer`.
*   **Migration:**
    ```diff
    - function foo(server: Deno.Server) {}
    + function foo(server: Deno.HttpServer) {}
    ```

#### Deno.shutdown()
*   **Status:** **Removed**.
*   **Replacement:** `Deno.Conn.prototype.closeWrite()`.
*   **Migration:**
    ```diff
    using conn = await Deno.connect({ port: 80 });
    - await Deno.shutdown(conn.rid);
    + await conn.closeWrite();
    ```

#### Deno.stdin/stdout/stderr.prototype.rid
*   **Status:** **Soft-removed**.
*   **Replacement:** Use instance methods directly on the standard stream objects.
*   **Migration:**
    ```diff
    - if (Deno.isatty(Deno.stdout.rid)) {}
    + if (Deno.stdout.isTerminal()) {}

    const data = new TextEncoder().encode("Hello");
    - await Deno.write(Deno.stdout.rid, data);
    + await Deno.stdout.write(data);
    ```

#### `rid` properties on `Deno.TcpConn`, `Deno.TlsConn`, `Deno.TlsListener`, `Deno.UnixConn`
*   **Status:** All **Removed**.
*   **Replacement:** Use instance methods (`.read()`, `.write()`, `.close()`, `.closeWrite()`, etc.) directly on the connection/listener objects. The pattern is consistent across all resource types.

#### Deno.writeAll() / Deno.writeAllSync()
*   **Status:** **Removed**.
*   **Replacement:** `writeAll()` / `writeAllSync()` from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import { writeAll } from "jsr:@std/io/write-all";
    const data = new TextEncoder().encode("Hello");
    - await Deno.writeAll(Deno.stdout, data);
    + await writeAll(Deno.stdout, data);
    ```

#### Deno.Writer / Deno.WriterSync
*   **Status:** **Removed**.
*   **Replacement:** `Writer` / `WriterSync` types from the Standard Library (`@std/io`).
*   **Migration:**
    ```diff
    + import type { Writer } from "jsr:@std/io/types";
    - function foo(writer: Deno.Writer) {}
    + function foo(writer: Writer) {}
    ```

#### Deno.write() / Deno.writeSync()
*   **Status:** **Removed**.
*   **Replacement:** The `.write()` / `.writeSync()` method on the resource object itself.
*   **Migration:**
    ```diff
    using file = await Deno.open("/foo/bar.txt", { write: true });
    const buffer = new TextEncoder().encode("My message");
    - await Deno.write(file.rid, buffer);
    + await file.write(buffer);
    ```

#### new Deno.FsFile()
*   **Status:** **Removed**.
*   **Replacement:** Use `Deno.open()` or `Deno.openSync()`.
*   **Migration:**
    ```diff
    - const file = new Deno.FsFile(3); // 3 is an example file descriptor
    + using file = await Deno.open("/foo/bar.txt");
    ```

#### window
*   **Status:** **Deprecated** in favor of the cross-platform standard.
*   **Replacement:** `globalThis`.
*   **Migration:**
    ```diff
    - const hashBuffer = await window.crypto.subtle.digest("SHA-256", messageBuffer);
    + const hashBuffer = await globalThis.crypto.subtle.digest("SHA-256", messageBuffer);
    ```
	
	Of course. Here is the next part of the technical documentation, building upon the migration guide. This section synthesizes the supplementary information you provided, covering Deno's core philosophy, its built-in toolchain, and advanced concepts in a structured, English-language format suitable for AI training.

***

## 2. Core Concepts and Philosophy

This section outlines the fundamental principles that guide Deno's design, distinguishing it from other runtimes like Node.js.

### 2.1. Core Design Philosophy

*   **Secure by Default:** Deno executes code in a secure sandbox. By default, scripts have no access to the file system, network, environment variables, or sub-processes. Access must be granted explicitly via command-line flags (e.g., `--allow-net`) or runtime permission prompts. This opt-in security model mitigates risks from malicious dependencies.
*   **Native TypeScript and TSX Support:** Deno supports TypeScript out of the box without requiring any external tools (like `ts-node` or Babel) or configuration files (`tsconfig.json`). Deno's runtime automatically compiles and caches TypeScript code on the fly.
*   **ES Modules (ESM) and Web Standards:** Deno exclusively uses the standard ES Module system. Dependencies are imported via URLs (local or remote), which aligns with browser behavior. Deno also implements numerous standard Web APIs like `fetch`, `crypto`, `WebSockets`, and `Web Workers`, enabling code-sharing between server and client environments.
*   **Complete Toolchain:** Deno is distributed as a single executable that includes a comprehensive suite of development tools, eliminating the need for many third-party dependencies. This includes a linter, code formatter, test runner, dependency inspector, and task runner.

### 2.2. Deno vs. Node.js: A Technical Comparison

| Feature                    | Deno                                                                                              | Node.js                                                                          |
| -------------------------- | ------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| **Security**               | Sandbox environment; explicit permissions required for I/O (`--allow-*` flags).                    | Full access to system resources by default.                                      |
| **TypeScript**             | Native, zero-configuration support.                                                               | Requires external tools (`typescript`, `ts-node`) and configuration (`tsconfig.json`). |
| **Module System**          | Standard ES Modules (ESM) with URL-based imports (`http:`, `jsr:`, `npm:`).                         | Primarily CommonJS (`require()`) with `node_modules`. ESM is supported but complex. |
| **Dependency Management**  | No `package.json` required. Uses import maps (`deno.json`) and lock files (`deno.lock`).           | Relies on `package.json` and a package manager (npm, yarn, pnpm) to manage `node_modules`. |
| **Standard Library**       | Ships with a comprehensive, audited standard library.                                             | Minimal standard library; relies heavily on the npm ecosystem for common tasks.  |
| **API Design**             | Modern, Promise-based, and aligned with Web APIs (`fetch`, `Deno.serve`).                         | Mix of callback-based, Promise-based, and synchronous APIs.                      |
| **Tooling**                | Integrated toolchain (`fmt`, `lint`, `test`, `compile`, `task`).                                  | Relies on a diverse ecosystem of third-party tools (ESLint, Prettier, Jest, etc.). |
| **Distribution**           | Single executable with no external dependencies.                                                  | Requires installation of the Node.js runtime and npm.                            |
| **Node.js Compatibility**  | High compatibility through `npm:` specifiers, `node:` built-ins, and `node_modules` directory support. | Native environment.                                                              |

## 3. The Deno Toolchain In-Depth

Deno's integrated toolchain provides a consistent and efficient development experience.

### 3.1. Code Formatter (`deno fmt`)

An opinionated, auto-configuring code formatter for JavaScript, TypeScript, JSON, and Markdown, powered by `dprint`.

*   **Usage:**
    *   `deno fmt`: Formats all supported files in the current directory and subdirectories.
    *   `deno fmt my_file.ts src/`: Formats specific files or directories.
    *   `deno fmt --check`: Checks for formatting issues without applying changes. Exits with a non-zero code if files are not formatted (ideal for CI).

*   **Configuration (`deno.json`):**
    ```json
    {
      "fmt": {
        "options": {
          "useTabs": true,
          "lineWidth": 80,
          "indentWidth": 4,
          "singleQuote": false
        },
        "exclude": ["./dist/"]
      }
    }
    ```
*   **Ignoring Code:**
    *   `// deno-fmt-ignore-file` at the top of a file.
    *   `// deno-fmt-ignore` on the line preceding a statement.

### 3.2. Linter (`deno lint`)

A fast, built-in linter for identifying potential errors and enforcing coding conventions in JavaScript and TypeScript.

*   **Usage:**
    *   `deno lint`: Lints all supported files in the project.
    *   `deno lint --fix`: Automatically fixes linting errors where possible.
    *   `deno lint --json`: Outputs diagnostics in JSON format for machine processing.

*   **Configuration (`deno.json`):**
    ```json
    {
      "lint": {
        "rules": {
          "tags": ["recommended"],
          "exclude": ["no-explicit-any"]
        },
        "exclude": ["./generated/"]
      }
    }
    ```
*   **Ignoring Rules:**
    *   `// deno-lint-ignore-file <rule1> <rule2>` at the top of a file.
    *   `// deno-lint-ignore <rule>` on the line preceding a statement.

### 3.3. Test Runner (`deno test`)

A robust, built-in test runner with support for permissions, code coverage, and filtering.

*   **Writing Tests:** Tests are defined using `Deno.test()`. Deno automatically discovers files ending in `_test.ts`, `.test.ts`, etc.
    ```typescript
    import { assertEquals } from "jsr:@std/assert";

    Deno.test("Simple Addition Test", () => {
      assertEquals(2 + 2, 4);
    });

    Deno.test({
      name: "Async Test",
      async fn() {
        const result = await Promise.resolve("ok");
        assertEquals(result, "ok");
      },
      permissions: { net: true } // Tests can have their own permissions
    });
    ```
*   **Usage:**
    *   `deno test`: Runs all test files in the project.
    *   `deno test --allow-net`: Grants necessary permissions for tests.
    *   `deno test --filter "API"`: Runs only tests whose names contain "API".
    *   `deno test --coverage=./cov_profile`: Generates a code coverage profile.
    *   `deno coverage ./cov_profile`: Displays the coverage report in the terminal.
    *   `deno test --watch`: Re-runs tests on file changes.

### 3.4. Task Runner (`deno task`)

A cross-platform script runner, analogous to npm scripts, defined in `deno.json`.

*   **Defining Tasks (`deno.json`):**
    ```json
    {
      "tasks": {
        "start": "deno run --allow-net --watch main.ts",
        "test": "deno test --allow-all",
        "dev": "deno task build && deno task start",
        "build": "echo 'Building...'"
      }
    }
    ```
*   **Usage:**
    *   `deno task start`: Executes the `start` task.
    *   `deno task`: Lists all available tasks.
    *   **Cross-platform shell commands:** `deno task` includes built-in commands like `cp`, `mv`, `rm`, `mkdir`, `echo`, `sleep`, etc., that work consistently across Windows, macOS, and Linux.

### 3.5. Standalone Executable Compiler (`deno compile`)

Bundles a Deno script and its dependencies into a single, self-contained executable.

*   **How it Works:** It creates an `eszip` bundle (a serialized ES module graph) and injects it into a stripped-down version of the Deno runtime.
*   **Usage:**
    *   `deno compile --allow-net --output my-app main.ts`
*   **Key Flags:**
    *   `--allow-*`: Embeds permissions into the compiled binary. These are the *only* permissions the final executable will have.
    *   `--output <filename>`: Specifies the output executable name.
    *   `--target <platform>`: Cross-compiles for different architectures (e.g., `x86_64-pc-windows-msvc`, `aarch64-apple-darwin`).
    *   `--include <paths>`: Includes additional assets (like images or `.env` files) in the binary, accessible via `Deno.readFile`.

### 3.6. Documentation Generator (`deno doc`)

Generates documentation from JSDoc comments in your source code.

*   **Usage:**
    *   `deno doc main.ts`: Displays documentation for exported symbols in the terminal.
    *   `deno doc --json main.ts`: Outputs documentation as a JSON tree for tooling.
    *   `deno doc --html main.ts`: Serves a local HTML preview of the documentation, styled like `jsr.io`.
    *   `deno doc --lint main.ts`: Checks if all exported symbols are documented.

## 4. Module and Dependency Management

Deno modernizes dependency management by moving away from a central package manager and the `node_modules` directory.

### 4.1. Module Specifiers

Deno resolves modules using different specifiers:

*   **URL Imports:** Import directly from a URL. Versioning is handled by including the version in the URL.
    ```typescript
    import { Application } from "https://deno.land/x/oak@v17.1.3/mod.ts";
    ```
*   **JSR (JavaScript Registry) Specifiers:** The recommended way to import versioned, TypeScript-first modules.
    ```typescript
    import { Hono } from "jsr:@hono/hono@^4.0.0";
    import { assertEquals } from "jsr:@std/assert@^1.0.0";
    ```
*   **npm Specifiers:** Import packages directly from the npm registry.
    ```typescript
    import express from "npm:express@^4.17.1";
    ```

### 4.2. Project Configuration (`deno.json`)

The `deno.json` file is the heart of a Deno project, used for configuration and dependency management.

*   **Import Maps (`imports`):** Create aliases for long module URLs to simplify imports and centralize version management.
    ```json
    {
      "imports": {
        "std/": "jsr:@std/",
        "oak": "jsr:@oak/oak@^17.1.3",
        "express": "npm:express@4.18.2",
        "@/": "./src/"
      }
    }
    ```
    Usage in code:
    ```typescript
    import { serve } from "std/http/server.ts";
    import { Application } from "oak";
    import { myUtil } from "@/utils.ts";
    ```

*   **Managing Dependencies with `deno add/remove`:**
    *   `deno add jsr:@hono/hono`: Adds the latest version of Hono to the `imports` map and updates the lock file.
    *   `deno add npm:chalk`: Adds an npm package.
    *   `deno remove oak`: Removes the "oak" dependency from the `imports` map.

### 4.3. Lock File (`deno.lock`)

The `deno.lock` file ensures reproducible builds by storing the cryptographic hashes of all remote dependencies.

*   **Function:** When a script is run, Deno verifies that the content of cached modules matches the hashes in `deno.lock`. This prevents unexpected changes from upstream dependencies (supply chain attacks).
*   **Lifecycle:**
    *   It is automatically created and updated when a `deno.json` file is present and commands like `deno run`, `deno test`, or `deno add` are executed.
    *   You should commit `deno.lock` to your version control system.
*   **CI/CD Usage:** Use the `--frozen` flag in CI environments (`deno test --frozen`). This command will fail if dependencies need to be updated or if the lock file is out of sync, ensuring build integrity.