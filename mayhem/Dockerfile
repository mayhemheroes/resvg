# Build Stage
FROM ghcr.io/evanrichter/cargo-fuzz:latest as builder

## Add source code to the build stage.
ADD . /src
WORKDIR /src

RUN echo building instrumented harnesses && \
    bash -c "pushd usvg/fuzz && cargo +nightly -Z sparse-registry fuzz build && popd" && \
    mv usvg/fuzz/target/x86_64-unknown-linux-gnu/release/tree_from_bytes /tree_from_bytes && \
    echo done

RUN echo building non-instrumented harnesses && \
    export RUSTFLAGS="--cfg fuzzing -Clink-dead-code -Cdebug-assertions -C codegen-units=1" && \
    bash -c "pushd usvg/fuzz && cargo +nightly -Z sparse-registry build --release && popd" && \
    mv usvg/fuzz/target/release/tree_from_bytes /tree_from_bytes_no_inst && \
    echo done

# Package Stage
FROM rustlang/rust:nightly

COPY --from=builder /tree_from_bytes /tree_from_bytes_no_inst /
RUN mkdir /testsuite
COPY --from=builder /src/usvg/tests/files/*svg /testsuite/
