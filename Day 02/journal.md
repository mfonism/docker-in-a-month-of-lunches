# Commanding Docker

I've started forming some intuition around how Docker commands are structured. Right now, I'm breaking them into what I (for now) call "top-level" commands &mdash; things like `container` and `image`. For now the book has only introduced Docker containers and images, so my write-up will be limited to them.

So, the top-level commands are:

- `docker container`
- `docker image`

And the cool thing is, you can run `--help` at _any_ level to see what's available.

Examples:

```sh
docker --help
docker container --help
docker container run --help
```

And so on. It's like peeling an onion... or unlocking nested spellbooks.

Another pattern I noticed is that there are shortcuts. For example:

- `docker run` is the same as `docker container run`
- `docker build` is the same as `docker image build`

My educated guess is that since these commands are common and unambiguous &mdash; the context is clear &mdash; one can skip the part that spells out the context.

And this makes sense, because, even though listing things is common, we can't do something like `docker ls` or `docker list`. Docker simply wouldn't be able to tell whether we're trying to list containers or images.

## Always ask for `--help`

I could never over-emphasis the importance of asking for help!

The option `--help` is a super valuable tool. It's like a turbo-charged weapon. Each time I use it, I get the feeling Arthur must have had when he first held Excalibur &mdash; not because I’m powerful yet, but because I’m holding something that _could_ make me powerful if I learn how to wield it.
