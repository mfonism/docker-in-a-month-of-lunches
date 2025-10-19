This chapter helped me wrap my head around three sets of Dockerfile instructions (and related runtime commands) that I'd often seen but hadn't fully internalised &mdash; especially in terms of how they differ.

I'm talking about:

- `COPY` vs `cp`
- `CMD` vs `ENTRYPOINT`
- `EXPOSE` vs actually exposing ports

Now that I've taken the time to break them down, things make a lot more sense.

## `COPY` vs `cp`

### `COPY`

`COPY` is a Dockerfile instruction used during the image build to bring files **into the image**. You usually specify a source path from your local build context and a destination path inside the image.

```dockerfile
COPY source/on/host destination/in/image
```

In multi-stage builds, the source can come from a previous stage instead of your local machine. That looks like this:

```dockerfile
COPY --from=previous-stage-name /path/in/previous-stage /path/in/current/stage
```

This lets you copy just the final output &mdash; like a binary or compiled app &mdash; from an earlier build stage into a clean, production-ready image; without dragging along all the compilers, source files, and build tools used to make it. The result is a leaner and possibly more secure container.

### `cp`

The `docker cp` command is used **at runtime** to move files between your host and a running container. It's not a Dockerfile instruction; you run this from your terminal.

### Copying from host to running container

```bash
docker cp /path/on/host container_id:/path/in/container
```

### Copying from running container to host

```bash
docker cp container_id:/path/in/container /path/on/host
```

### TL;DR Table

| Feature           | `COPY` (Dockerfile)          | `cp` (Shell command)                  |
| ----------------- | ---------------------------- | ------------------------------------- |
| Purpose           | Add files during image build | Move files between host and container |
| When              | Build-time                   | Runtime                               |
| Source            | Local machine or build stage | Host or running container             |
| Destination       | New image layer              | Host or running container             |
| Cacheable         | Yes                          | No                                    |
| Container needed? | No                           | Yes                                   |

_Note: With `docker cp`, if the source is on the host, the destination must be inside a **running container** &mdash; and vice versa._

### `CMD` vs `ENTRYPOINT`

This one confused me more than I care to admit. The way I saw them used convinced me they were instructions for specifying what executables to run (and how). But, why have _two_ instructions for specifying how to run exectuables?

Looking online, I got that:

- `ENTRYPOINT` defines the **main executable**
- `CMD` defines the **default arguments**

But that didn't help much, either &mdash; well, not until I found an intuitive way to understand them.

After lots of digging, I came up with a mental model that finally clicked for me. I now think of the final command that actually runs as:

```
ENTRYPOINT ++ (runtime args || CMD)
```

Where:

- `++` means string concatenation (i.e., join them together)
- `||` means "use the left if present; otherwise, use the right"

With this model, things started to make sense:

- If you specify runtime args when you do `docker container run ...`, those args replace what's specified in `CMD`
- If you don't specify `ENTRYPOINT`, then the evaluation of `(runtime args || CMD)` is treated as the whole command

#### Table of Examples

| In Dockerfile                                                            | Example `docker run` command | What Actually Runs                                         |
| ------------------------------------------------------------------------ | ---------------------------- | ---------------------------------------------------------- |
| `CMD ["node", "server.js"]`                                              | `docker run myapp`           | `node server.js`                                           |
| `CMD ["node", "server.js"]`                                              | `docker run myapp other.js`  | `other.js` (CMD is completely overridden)                  |
| `ENTRYPOINT ["java", "-jar", "app.jar"]`<br>`CMD ["--server.port=8080"]` | `docker run myapp`           | `java -jar app.jar --server.port=8080`                     |
| `ENTRYPOINT ["java", "-jar", "app.jar"]`<br>`CMD ["--server.port=8080"]` | `docker run myapp --debug`   | `java -jar app.jar --debug` (CMD is completely overridden) |

#### When to Use What

| Mode                   | What Happens                                                                             | When to Use It                                                                         |
| ---------------------- | ---------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **`ENTRYPOINT`** only  | Always runs the specified command, and appends any args from runtime                     | When you want to enforce a fixed command, but allow dynamic input                      |
| **`ENTRYPOINT + CMD`** | Runs `ENTRYPOINT` with default args from `CMD`; runtime args, if present, override `CMD` | When you want a fixed command with sensible defaults that can be optionally overridden |
| **`CMD`** only         | Runs full `CMD` if no args are passed; runtime args **replace** it entirely              | When you want to offer a default command, but allow users to fully override it         |

## `EXPOSE`, and the `-p` Option

Another concept I misunderstood was `EXPOSE`. I used to think (or at least expect) it actually opened ports to the outside world. But I now understand that it's purely **metadata**.

`EXPOSE` just documents which port the app inside the container is expected to listen on. It **does not** publish or bind that port to the host.

To actually make the port available externally, you have to use the `-p` option when running the container:

```bash
docker run -p 8080:80 image-name
```

This creates a mapping **from host to container**, so traffic sent to port `8080` on your machine will be forwarded to port `80` inside the container.

And by the way, there's nothing magical about `8080` or `80`. You can choose any valid port numbers. Docker will always listen for traffic on the **host port** and forward it to the **container port** &mdash; from outside to inside.
