#  ----------------------------------------------------------------------------------
#  Copyright (c) 2023 Intel Corporation
#
#  Licensed under the Apache License, Version 2.0 (the "License");
#  you may not use this file except in compliance with the License.
#  You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.
#
#  SPDX-License-Identifier: Apache-2.0
#  ----------------------------------------------------------------------------------

# Docker image for Golang Core common config bootstrapper service
ARG BUILDER_BASE=golang:1.18-alpine3.16
FROM ${BUILDER_BASE} AS builder

ARG ADD_BUILD_TAGS=""

WORKDIR /edgex-go

RUN apk add --update --no-cache make git

COPY go.mod vendor* ./
RUN [ ! -d "vendor" ] && go mod download all || echo "skipping..."

COPY . .
RUN make -e ADD_BUILD_TAGS=$ADD_BUILD_TAGS cmd/security-proxy-auth/security-proxy-auth

#Next image - Copy built Go binary into new workspace
FROM alpine:3.16

LABEL license='SPDX-License-Identifier: Apache-2.0' \
      copyright='Copyright (c) 2023: Intel Corporation'

RUN apk add --update --no-cache dumb-init
COPY --from=builder /edgex-go/Attribution.txt /
COPY --from=builder /edgex-go/cmd/security-proxy-auth/security-proxy-auth /
COPY --from=builder /edgex-go/cmd/security-proxy-auth/res/configuration.toml /res/configuration.toml

COPY --from=builder /edgex-go/cmd/security-proxy-auth/entrypoint.sh /usr/local/bin/
RUN chmod 755 /usr/local/bin/entrypoint.sh \
    && ln -s /usr/local/bin/entrypoint.sh /

ENTRYPOINT ["entrypoint.sh"]
CMD ["/security-proxy-auth", "-cp=consul.http://edgex-core-consul:8500"]
