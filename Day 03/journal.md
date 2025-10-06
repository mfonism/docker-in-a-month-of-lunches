My biggest takeaway today is that there's a difference between **buildtime** and **runtime**.

Instructions like `FROM`, `RUN`, and `COPY` happen during **buildtime**, when the image is being built; but `CMD` is a **runtime** instruction, and sets the **default command** the container runs when it starts.

I used to think everything in a Dockerfile just ran top-to-bottom like a script, but this chapter helped me realise that `CMD` lives in a different phase entirely. That little shift in understanding made a big difference.

A lot of this started clicking for me because of questions I had about the optimised Dockerfile in the chapter. I ended up doing some light research alongside the reading, and it has helped me appreciate the design of Dockerfiles a bit more.

I get the feeling the book will dig into these distinctions more later, but for now, this insight feels like a solid step forward.

---

## CMD, and my buildtime-runtime confusion

In this chapter we worked with a Dockerfile that looked like this:

```Dockerfile
FROM diamol/node:2e

ENV TARGET="blog.sixeyed.com"
ENV METHOD="HEAD"
ENV INTERVAL="3000"

WORKDIR /web-ping
COPY app.js .

CMD ["node", "/web-ping/app.js"]
```

Then we were shown an optimized version:

```Dockerfile
FROM diamol/node:2e

CMD ["node", "/web-ping/app.js"]

ENV TARGET="blog.sixeyed.com" \
    METHOD="HEAD" \
    INTERVAL="3000"

WORKDIR /web-ping
COPY app.js .
```

At first, I was confused about how this could work. It looked like the `CMD` was referencing a file (`/web-ping/app.js`) before the context had even been set up with `WORKDIR` and `COPY`.

But after digging into it, I learned that the key is understanding the difference between **Docker's build time** and **runtime**:

- **Build time** is when Docker reads the Dockerfile and builds the image, one layer at a time.
- **Runtime** is when a container is created from that image and actually starts executing the default command (`CMD`).

So `CMD` doesn't run during the build &mdash; it just gets stored in the image metadata. As long as the file exists in the final image by the time the container starts, everything works as expected.

That's the subtlety.

## Work Right With WORKDIR

- `WORKDIR` sets the **working directory** for all Dockerfile instructions that follow &mdash; including `COPY`, `RUN`, `CMD`, and even subsequent `WORKDIR` commands.

- If the directory you specify **doesn't exist**, Docker will **automatically create it** during the build.

- `WORKDIR` is **persistent** across layers, and **relative paths** work as expected.

  ```Dockerfile
  WORKDIR /app
  WORKDIR ./src     # now in /app/src
  WORKDIR ..        # back to /app
  ```

- All following commands run in that directory:
  - `COPY`
  - `RUN`
  - `CMD`
  - **Even another `WORKDIR`**

### Want to go back to root?

To reset back to the root directory (`/`), just use:

```Dockerfile
WORKDIR /
```

This clears any relative path stack and sets the working directory back to the container's root.

### About `cd`

If you're wondering: _"Can I just use `cd`?"_, well, you sort of can!

You can run `cd` as part of a shell command inside a `RUN` instruction:

```Dockerfile
RUN cd /app && ls
```

But this is **not persistent**. Each `RUN` instruction runs in a fresh shell. The effect of `cd` (or any other state change) disappears after that one instruction/line.

That is why `WORKDIR` exists &mdash; it is the proper Dockerfile command for setting a directory **across layers**.
