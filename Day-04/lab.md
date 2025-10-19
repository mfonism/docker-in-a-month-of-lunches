# I Optimise Therefore I Am

## The Lab

The task was to optimise the Dockerfile below such that:

- the resulting image size (which was originally around 450MB) would be significantly reduced and,
- changes to `index.html` wouldn't unnecessarily invalidate the build cache

```Dockerfile
FROM diamol/golang:2e

WORKDIR /web
COPY index.html .
COPY go.mod main.go /web/

RUN go build -o /web/server
RUN chmod +x /web/server

CMD ["/web/server"]
ENV USER=sixeyed
EXPOSE 80
```

## My Solution

I approached this by introducing a multi-stage build, allowing me to separate the build environment from the runtime environment. This reduced the final image size from approximately 450MB to just 35MB.

I also made sure that index.html was copied after the final executable and configuration instructions, so that changes to the static HTML file wouldn't invalidate earlier layers in the Docker cache &mdash; particularly those involving the compiled binary.

```Dockerfile
FROM diamol/golang:2e AS builder

WORKDIR /web

COPY go.mod main.go /web/
RUN go build -o /web/server

FROM diamol/base:2e

COPY --from=builder /web/server /web/server
RUN chmod +x /web/server

EXPOSE 80
ENV USER=sixeyed

CMD ["/web/server"]

COPY index.html .
```
