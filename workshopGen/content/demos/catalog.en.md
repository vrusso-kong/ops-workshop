+++
title = ""
menuTitle = "Service Broker Catalog"
chapter = false
weight = 9
description = ""
draft = false
+++
# Adding a Service Catalog
We are creating a new plan for Medium hypothetical service

## Extending the Catalog with Medium Plan

1. Change into the `broker` source directory 
    
    ```
    cd src/github.com/Pivotal-Field-Engineering/service-broker/broker
    ```

1. Open `broker.go` into your text IDE. All of the service broker logic lives here. 

1. Locate the function called `Services` in `broker.go`. This function corresponds with the `/v2/catalog` endpoint.

1. Locate the Plan `smallPlan`. We see the smallPlan being created as a golang Struct. Let us now create a mediumPlan struct to model a Medium service type.

1. Place the following code below the smallPlan. This will be our new mediumPlan!

    ```
    mediumPlan := brokerapi.ServicePlan{
		Description: "medium-plan",
		Free:        &truePointer,
		Name:        "medium",
		Bindable:    &truePointer,
		ID:          "medium-id",
	}
    ```

1. At this point we have 2 plans. You can continue to add as many as desired.

1. Great! Now we have new plans, but our service broker has no awareness of these plans. To plug the plans into the service broker we add our new plans to the existing `ServicePlan` struct.

    ```
	s := brokerapi.Service{
		ID:            "Pivotal",
		Name:          "Pivotal Service Plans",
		Description:   "Service Plan which gives us Pivotal",
		Bindable:      true,
		PlanUpdatable: true,
		Tags:          []string{},
		Plans:         []brokerapi.ServicePlan{smallPlan, mediumPlan},
	}
    ```

1. Notice the line `Plans: []brokerapi.ServicePlan{smallPlan,mediumPlan},` which has registered the small and medium plans. Save the `broker.go` file and get ready to run.

## Run the Service Broker

1. Run the service broker using `go run`.

    ```
    go run src/github.com/Pivotal-Field-Engineering/service-broker/main.go
    ```

## View the Service Broker Catalog

Choose one of the following:

- Using `curl` hit the `/v2/catalog` endpoint via `curl -X GET -H 'X-Broker-API-Version:2.14' -u pivotal:pivotal localhost:8080/v2/catalog`

- Using Postman
    - *AUTH_HEADER* - Username: `pivotal`, Password: `pivotal`
    - *HEADER* - `X-Broker-API-Version`: `2.14`
    - *HTTP TYPE* - `GET`
    - *ADDRESS* - `http://localhost:8080/v2/catalog`

- The response should represent an Open Service Broker Catalog similar to the following:

    ```
    {
        "services": [
            {
                "id": "Pivotal",
                "name": "Pivotal Service Plans",
                "description": "Service Plan which gives us Pivotal",
                "bindable": true,
                "plan_updateable": true,
                "plans": [
                    {
                        "id": "small-id",
                        "name": "small",
                        "description": "small-plan",
                        "free": true,
                        "bindable": true
                    },
                    {
                        "id": "medium-id",
                        "name": "medium",
                        "description": "medium-plan",
                        "free": true,
                        "bindable": true
                    }
                ]
            }
        ]
    }
    ```