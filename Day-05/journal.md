Reading through this chapter, I found myself asking questions that weren't directly answered in the text &mdash; but the curiosity they sparked pushed me to explore deeper.

That exploration took me into topics like:

- The anatomy of Docker image references
- How wildcards behave with `docker image ls -f reference=...`
- The filesystem isolation between containers
- What `docker container run/start` flags like `-i`, `-t`, `-a`, and `-a` actually do under the hood, especially in combinations

By poking at each behavior and tracing the outcomes, I feel like I've developed a clearer mental model of what's going on when I do some things I now know to do.

## Understanding Docker Image References

When you work with Docker images, every image you pull, tag, or push is identified by something called a **Docker image reference**. If you've ever typed something like `python:latest`, you've already used one &mdash; just without realizing what all its parts mean.

Reading this chapter, I unpacked the structure of Docker references and what happens when parts of them are omitted. To make things clear in this writeup, I'll use a consistent notation, where anything inside square brackets (`[]`) is **optional**.

### The Structure of a Docker Image Reference

A Docker image reference generally looks like this:

```
REPOSITORY[:TAG]
```

- `REPOSITORY` &mdash; The image's path, which can include a registry, a namespace, and a repository name
- `TAG` &mdash; An optional label (like a version number). Defaults to `latest` if omitted.

So something like `python:3.12` is shorthand for a specific repository (`python`) with a version tag (`3.12`).

### Breaking Down the Repository

The `REPOSITORY` part itself is made up of smaller segments, separated by slashes.
Here's the full anatomy:

```
[REGISTRY_HOST[:PORT]/]NAMESPACE/REPO
```

_Remember, in this notation anything inside square brackets (`[]`) is optional._

| Segment                | Meaning                                          | Example(s)                         |
| ---------------------- | ------------------------------------------------ | ---------------------------------- |
| `REGISTRY_HOST[:PORT]` | The hostname (and optional port) of the registry | `docker.io`, `roygbivery.com:5050` |
| `NAMESPACE`            | The user or organization name                    | `mfonism`                          |
| `REPO`                 | The actual image name                            | `the-bees-and-the-bees`            |

### Putting It All Together

A **fully specified reference** contains the registry, namespace, repository name, and an optional tag.

Example:

```
roygbivery.com:5050/mfonism/the-bees-and-the-bees:v2
```

<table border="1" cellpadding="6" cellspacing="0">
  <thead>
    <tr>
      <th>Part</th>
      <th>Segments in Part</th>
      <th>Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3">Repository</td>
      <td>Registry</td>
      <td><code>roygbivery.com:5050</code></td>
    </tr>
    <tr>
      <td>Namespace</td>
      <td><code>mfonism</code></td>
    </tr>
    <tr>
      <td>Repo</td>
      <td><code>the-bees-and-the-bees</code></td>
    </tr>
    <tr>
      <td colspan="2">Tag</td>
      <td><code>v2</code></td>
    </tr>
  </tbody>
</table>

### What Happens When Parts Are Left Out?

Only **some parts** of a Docker reference can be omitted &mdash; but when they are, Docker fills in sensible defaults.

#### 1. Omitting the Tag

If you leave out the tag, Docker assumes that the tag is `latest`.

So:

```
roygbivery.com/mfonism/the-bees-and-the-bees
```

is interpreted as:

```
roygbivery.com/mfonism/the-bees-and-the-bees:latest
```

#### 2. Omitting the Registry

If you don't specify a registry, Docker assumes the default public registry, `docker.io`:

So:

```
mfonism/the-bees-and-the-bees
```

is interpreted as:

```
docker.io/mfonism/the-bees-and-the-bees:latest
```

#### 3. Omitting Both Registry and Namespace

This is where **official images** come in. Docker Hub stores official images under a hidden namespace called `library`.

So, when you write something like `ubuntu`, Docker reads it as:

```
docker.io/library/ubuntu:latest
```

This is why official images like `alpine`, `redis`, and `ubuntu` work without specifying anything else.

_Note that you can specify tag to target a particular version of the official image._

## Filtering Images with `docker image ls -f reference=...`

As a side quest, I looked into how **wildcards** work when filtering image references. It turned out to be trickier than I had expected &mdash; I assumed it worked like regular string matching or regex, but it doesn't.

### How Wildcards Actually Work

When filtering with:

```bash
docker image ls -f reference='<pattern>'
```

The `<pattern>` is matched **segment by segment**. In other words, a `*` matches just one segment of the repository path, not just arbitrary runs of characters.

`*` expands across **one slash-delimited segment**, not multiple.

#### Example

If you want to match all images under a registry called `roygbivery.com`, you need to match its full segment structure.

These **work**:

```bash
docker image ls -f reference='roygbivery.com/*/*'
```

because the reference includes:

- A registry (`roygbivery.com`)
- A namespace
- A repository

That pattern correctly accounts for two more segments after the registry.

But:

```bash
docker image ls -f reference='roygbivery.com/*'
```

won't match anything, because the pattern has too few segments.

#### More From My Case

When I tested on my local registry, these worked:

```bash
docker image ls -f reference='registry.local:5010/gallery/*'
docker image ls -f reference='registry.local:5010/*/*'
docker image ls -f reference='registry.local:*/*/*'
docker image ls -f reference='registry.local*/*/*'
```

They all matched:

- `registry.local:5010/gallery/ui:v1`
- `registry.local:5010/gallery/api:v1`

because the pattern structure aligned with the full segment layout of the image references.

But these didn't:

```bash
docker image ls -f reference='registry.local:5010/*'
docker image ls -f reference='registry.local/*/*'
docker image ls -f reference='registry.local*'
docker image ls -f reference='registry.local*/*'
```

Why?

- `*` doesn't span across multiple segments
- If your pattern has fewer segments than the image reference, it fails silently
- Docker expects segment-wise alignment between the pattern and the actual reference
