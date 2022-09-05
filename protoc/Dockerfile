# https://github.com/GoogleCloudPlatform/cloud-builders-community/blob/master/protoc/Dockerfile
FROM golang AS gen_go_grpc

ARG GEN_GO_VER=1.28.1
ARG GEN_GO_GRPC_VER=1.2.0

RUN GOBIN=/ GO111MODULE=on go install google.golang.org/protobuf/cmd/protoc-gen-go@v${GEN_GO_VER}
RUN GOBIN=/ GO111MODULE=on go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v${GEN_GO_GRPC_VER}


FROM debian:buster-slim

ARG TARGETPLATFORM

ARG PROTOC_URL=https://github.com/protocolbuffers/protobuf/releases/download
ARG PROTOC_VER=21.5

ARG GEN_GRPC_WEB_URL=https://github.com/grpc/grpc-web/releases/download
ARG GEN_GRPC_WEB_VER=1.3.1

RUN case "${TARGETPLATFORM}" in \
      linux/amd64) \
        echo "PROTOC_ARCH=linux-x86_64" > /tmp/env; \
        echo "GEN_GRPC_WEB_ARCH=linux-x86_64" >> /tmp/env \
				;; \
# gen-grpc-web does not have aarch binary
#      linux/arm64) \
#        echo "PROTOC_ARCH=linux-aarch_64" >> /tmp/env \
#				;; \
      *) exit 99 ;; \
    esac

RUN echo "protoc ${PROTOC_VER}-${TARGETPLATFORM}"

RUN apt-get update -y && apt-get upgrade -y && \
    apt-get install wget unzip make -y && \
    apt-get clean -y && \
    rm -rf /var/lib/apt/lists/*

RUN . /tmp/env; wget "${PROTOC_URL}/v${PROTOC_VER}/protoc-${PROTOC_VER}-${PROTOC_ARCH}.zip" -O protoc.zip && \
    unzip "protoc.zip" -d protoc && \
    rm "protoc.zip"

RUN . /tmp/env; wget "${GEN_GRPC_WEB_URL}/${GEN_GRPC_WEB_VER}/protoc-gen-grpc-web-${GEN_GRPC_WEB_VER}-${GEN_GRPC_WEB_ARCH}" \
      -O /protoc/bin/protoc-gen-grpc-web && \
    chmod +x /protoc/bin/protoc-gen-grpc-web

COPY --from=gen_go_grpc /protoc-gen-go /protoc-gen-go-grpc /protoc/bin/

ENV PATH=$PATH:/protoc/bin/
ENTRYPOINT ["protoc"]
CMD ["--help"]
