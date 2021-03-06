# -*- mode: dockerfile -*-

# You can override this `--build-arg BASE_IMAGE=...` to use different
# version of Rust or OpenSSL.
ARG BASE_IMAGE=ekidd/rust-musl-builder:latest
FROM alpine/git:latest AS cloner

RUN git clone https://github.com/apollographql/federation.git /src

# Our first FROM statement declares the build environment.
FROM ${BASE_IMAGE} AS builder

# Add our source code.
COPY --chown=rust:rust --from=cloner /src .

# Build our application.
WORKDIR stargate/
RUN cargo build --release

# Now, we need to build our _real_ Docker container, copying in the app
FROM alpine:latest
# TODO: Unsure if needed
RUN apk --no-cache add ca-certificates
COPY --from=builder \
  /home/rust/src/target/x86_64-unknown-linux-musl/release/stargate \
  /usr/local/bin/

# User can provide an alternate port via "-e 'PORT=<your port>'" when running the container.
# PORT defaults to 8080 otherwise.
CMD /usr/local/bin/stargate --port=${PORT:-8080} --manifest=/graph --tracing-endpoint http://localhost:14268/api/traces --structured-logging
