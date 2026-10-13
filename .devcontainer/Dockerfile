# Debian Bookworm + Go. Anubis (iplist2rule) requires Go 1.26.3 or newer.
FROM golang:1.26.9-bookworm

# Official image already sets GOPATH=/go and puts /go/bin on PATH.
# GOTOOLCHAIN=local keeps the build on this Go instead of downloading another.
RUN go install github.com/TecharoHQ/anubis/utils/cmd/iplist2rule@latest

# install robots2policy
RUN go install github.com/TecharoHQ/anubis/cmd/robots2policy@latest