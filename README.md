## WHY?

A healthcheck is added to Dockerfile to check the status of docker via docker inspect using HTTP protocol. LINUX/MAC OS has CURL or WGET command but freshly created docker container does not contain these command.

## HOW?
So to overcome this problem, the idea to add another following healthcheck Docker layer:

```go
package main
import (
	"fmt"
	"net/http"
	"os"
)

func main() {
	_, err := http.Get(fmt.Sprintf("http://127.0.0.1:%s/health", os.Getenv("PORT")))
	if err != nil {
		os.Exit(1)
	}
}
```

Now build the package as another executable and simply add HEALTHCHECK instruction to Dockerfile

```dockerfile
# Stage 1: Build executable
FROM golang:1.12.1 as buildImage

WORKDIR /go/src/github.com/mgorav/golang-docker-healthcheck
COPY main.go .
COPY healthcheck ./healthcheck

RUN CGO_ENABLED=0 go build -a -installsuffix cgo -o server
RUN CGO_ENABLED=0 go build -a -installsuffix cgo -o health-check "/go-docker-healthcheck/healthcheck"

# Stage 2: Create release image
FROM scratch as releaseImage

COPY --from=buildImage /go-docker-healthcheck/server ./server
COPY --from=buildImage /go-docker-healthcheck/health-check ./healthcheck

HEALTHCHECK --interval=1s --timeout=1s --start-period=2s --retries=3 CMD [ "/healthcheck" ]

ENV PORT=8080
EXPOSE $PORT

ENTRYPOINT [ "/server" ]

```

## Conclusion

This project demonstrates health-check for a server implemented in Go, for a docker container build from scratch. This example demonstrates adding extension layer(s) to docker.


## Top docker utilities

1. ctop: top like interface for container
```bash
brew install ctop
```
2. rocker
```bash
brew install grammarly/tap
brew install grammarly/top/rocker

```

3. docker-slim

```
https://github.com/docker-slim/docker-slim/releases
```

4. docker-gc

```
https://github.com/spotify/docker-gc/blob/master/README.md
```

5. watchtower: Automatically update Docker containers

