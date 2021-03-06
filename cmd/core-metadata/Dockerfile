#
# Copyright (c) 2018
# Dell
# Cavium
#
# SPDX-License-Identifier: Apache-2.0
#

ARG BUILDER_BASE=golang:1.13-alpine
FROM ${BUILDER_BASE} AS builder

WORKDIR /edgex-go

# The main mirrors are giving us timeout issues on builds periodically.
# So we can try these.

RUN sed -e 's/dl-cdn[.]alpinelinux.org/nl.alpinelinux.org/g' -i~ /etc/apk/repositories

RUN apk update && apk add make git

COPY go.mod .

RUN go mod download

COPY . .
RUN make cmd/core-metadata/core-metadata

#Next image - Copy built Go binary into new workspace
FROM scratch

LABEL license='SPDX-License-Identifier: Apache-2.0' \
      copyright='Copyright (c) 2018: Dell, Cavium'

ENV APP_PORT=48081
#expose meta data port
EXPOSE $APP_PORT

WORKDIR /
COPY --from=builder /edgex-go/cmd/core-metadata/Attribution.txt /
COPY --from=builder /edgex-go/cmd/core-metadata/core-metadata /
COPY --from=builder /edgex-go/cmd/core-metadata/res/configuration.toml /res/configuration.toml
ENTRYPOINT ["/core-metadata"]
CMD ["-cp=consul.http://edgex-core-consul:8500", "--registry", "--confdir=/res"]
