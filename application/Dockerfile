FROM golang AS builder
ARG SERVICE_NAME
ARG BRANCH_NAME
ARG BUILD_NAME
ARG SEALIGHTS_TOKEN
ENV OS_ARCH=linux-amd64

# Echo build arguments for debugging
RUN echo "SERVICE_NAME: ${SERVICE_NAME}"
RUN echo "BRANCH_NAME: ${BRANCH_NAME}"
RUN echo "BUILD_NAME: ${BUILD_NAME}"
RUN echo "SEALIGHTS_TOKEN: ${SEALIGHTS_TOKEN}"

# Add the source code
#ADD . /go/src

#COPY .git /go/src/.git
 
WORKDIR /go/src
RUN git clone https://github.com/liornabat-sealights/go-calc-demo.git .

# Download dependencies
RUN go mod download

### Sealights instrumentation
# Download and extract Sealights Go agent
RUN wget -U "slgoagent" -q -O slgoagent.tar.gz \
   https://agents.sealights.co/slgoagent/v1.1.80/slgoagent-${OS_ARCH}.tar.gz \
    && tar -xvf slgoagent.tar.gz \
    && rm slgoagent.tar.gz

# Download and extract Sealights CLI
RUN wget -U "slcli" -q -O slcli.tar.gz \
    https://agents.sealights.co/slcli/v1.0.38/slcli-${OS_ARCH}.tar.gz \
    && tar -xvf slcli.tar.gz \
    && rm slcli.tar.gz
	
RUN ./slgoagent -v 2> /dev/null | grep version
RUN ./slcli -v 2> /dev/null | grep version

# Save the Sealights token to a file and print its contents
RUN echo "${SEALIGHTS_TOKEN}" > ./token.txt \
    && cat ./token.txt

# Sealights configuration
RUN ./slcli config init --lang go --token ./token.txt
RUN ./slcli config create-bsid --app ${SERVICE_NAME} --branch ${BRANCH_NAME} --build ${BUILD_NAME}

# Run Sealights scan
RUN ./slcli scan --bsid buildSessionId.txt --path-to-scanner ./slgoagent --workspacepath ./ --disable-on-init true
RUN cat build.json

# Run tests
RUN go test -v ./...

# Build the Go application
RUN go mod vendor

# Build the Go application
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a -mod=vendor -o ./go-calc-demo

# Define the entry point for the container
CMD ["./go-calc-demo"]
