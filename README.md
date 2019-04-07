## Problem statement

An healthcheck is addded to Dockerfile to check the status of docker via docker inspect using HTTP protocol. LINUX/MAC OS has CURL or WGET command but freshly created docker container does not contain these command.

So to overcome this problem, the idea to add another following Docker layer:

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

WORKDIR /go/src/github.com/Soluto/golang-docker-healthcheck
COPY main.go .
COPY healthcheck ./healthcheck

RUN CGO_ENABLED=0 go build -a -installsuffix cgo -o server
RUN CGO_ENABLED=0 go build -a -installsuffix cgo -o health-check "github.com/mgorav/go-docker-healthcheck/healthcheck"

# Stage 2: Create release image
FROM scratch as releaseImage

COPY --from=buildImage /go/src/github.com/mgorav/go-docker-healthcheck/server ./server
COPY --from=buildImage /go/src/github.com/mgorav/go-docker-healthcheck/health-check ./healthcheck

HEALTHCHECK --interval=1s --timeout=1s --start-period=2s --retries=3 CMD [ "/healthcheck" ]

ENV PORT=8080
EXPOSE $PORT

ENTRYPOINT [ "/server" ]

```

## Conclusion

This project demonstrates health-check for a server implemented in Go, for a docker container built from scratch.
