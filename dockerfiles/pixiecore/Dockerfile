FROM golang:alpine3.6 as builder

MAINTAINER David Anderson <dave@natulte.net>

ARG PIXIECORE_SANDBOX=/tmp/sandbox
ARG PIXIECORE_CONTEXT="$PIXIECORE_SANDBOX"/context

COPY . "$PIXIECORE_CONTEXT"

RUN set -uxe;                                                                   \
                                                                                \
    apk upgrade --update-cache;                                                 \
    apk add --virtual .build-deps                                               \
        git                                                                     \
        glide                                                                   \
        musl-dev;                                                               \
                                                                                \
    # Pixiecore assets                                                          \
    NAMESPACE=go.universe.tf;                                                   \
    REPO=netboot;                                                               \
    PKG=cmd/pixiecore;                                                          \
                                                                                \
    export GOPATH="$PIXIECORE_SANDBOX"/go;                                      \
    export GLIDE_HOME="$PIXIECORE_SANDBOX";                                     \
                                                                                \
    NAMESPACE_PATH="$GOPATH/src/$NAMESPACE";                                    \
    REPO_PATH="$NAMESPACE_PATH/$REPO";                                          \
    PKG_PATH="$REPO_PATH/$PKG";                                                 \
                                                                                \
    if [ -d "$PIXIECORE_CONTEXT"/"$PKG" ]; then                                 \
        echo "Building from local dev copy";                                    \
        mkdir -p "$NAMESPACE_PATH";                                             \
        mv -v "$PIXIECORE_CONTEXT" "$REPO_PATH";                                \
    else                                                                        \
        echo "Building from upstream git repo";                                 \
    fi;                                                                         \
                                                                                \
    go get -v -d "$NAMESPACE/$REPO/$PKG";                                       \
    cd "$REPO_PATH";                                                            \
    glide install;                                                              \
    go test $(glide nv);                                                        \
    GOBIN=/ go install -ldflags -s ./"$PKG";

FROM alpine:3.6

RUN apk upgrade --update-cache;                                                 \
    apk add ca-certificates;                                                    \
    rm -rf /var/cache/apk/*;

COPY --from=builder /pixiecore /usr/local/bin

ENTRYPOINT ["/usr/local/bin/pixiecore"]
