FROM node:26-alpine@sha256:8996bb5821d62a8fd9bb27c1cb07462f3fdce6e47d640c6a881ead0b064ce216 AS frontend

WORKDIR /src/pkg/web
COPY pkg/web/package.json pkg/web/package-lock.json pkg/web/.npmrc ./
RUN npm ci

COPY pkg/web/ ./
ENV PUBLIC_BACKEND_ENDPOINT="" PUBLIC_BACKEND_WS_ENDPOINT=""
RUN npm run build

FROM golang:1.27-alpine@sha256:fa3d596898d9f126af7c01d14ee7597bab3f39cccf392ef36b69fe77cd681f18 AS backend

WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download

COPY cmd/ ./cmd/
COPY pkg/ ./pkg/
COPY --from=frontend /src/pkg/web/build ./pkg/web/build
RUN CGO_ENABLED=0 go build -trimpath -o /out/quickpizza ./cmd

FROM gcr.io/distroless/static-debian13@sha256:448810357cc8e67b7208988abdba0f5ec3d3aebee0a5b13eb624979792cffc0f

COPY --from=backend /out/quickpizza /usr/local/bin/quickpizza
USER 65532:65532
EXPOSE 3333
ENTRYPOINT ["/usr/local/bin/quickpizza"]
