+++
title = ""
menuTitle = "Service Broker Init"
chapter = false
weight = 8
description = ""
draft = false
+++
# Creating Your First Service Broker

## Get Service Broker Skeleton Project

1. Install Golang, setup $GOROOT. Golang has great documentation on how to do this [here](https://golang.org/doc/install).

1. Create a working directory to store our service broker in. We will use `/tmp` for the rest of this workshop.

    ``` 
    mkdir -p /tmp/service-broker
    ```

1. Set the working directory($GOPATH) for the `go` cli.

    ```
    export GOPATH=$(pwd)
    ```

1. Get the skeleton service broker code using `go get`.

    ```
    go get github.com/Pivotal-Field-Engineering/service-broker
    ```

## Run the Service Broker

1. Run the service broker using `go run`.

    ```
    go run src/github.com/Pivotal-Field-Engineering/service-broker/main.go
    ```

**Note:** By default the service broker runs on port `8080` and with basic authentication credentials of `pivotal`/`pivotal`

## View the Service Broker Catalog

Choose one of the following:

- Using `curl` hit the `/v2/catalog` endpoint via `curl -X GET -H 'X-Broker-API-Version:2.14' -u pivotal:pivotal localhost:8080/v2/catalog`

- Using Postman
    - *AUTH_HEADER* - Username: `pivotal`, Password: `pivotal`
    - *HEADER* - `X-Broker-API-Version`: `2.14`
    - *HTTP TYPE* - `GET`
    - *ADDRESS* - `http://localhost:8080/v2/catalog`
    - ![catalog1](postman_catalog1.png)
    - ![catalog2](postman_catalog2.png)


