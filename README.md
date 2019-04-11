# go-xray

go-xray is a Go client library for accessing the [JFrog Xray API](https://www.jfrog.com/confluence/display/XRAY/Xray+REST+API).

It is based on the excellent work on the [go-artifactory](https://github.com/atlassian/go-artifactory)
library and attempts to follow a similar API.

## Requirements

- Go version 1.10+
- Go module system is used for dependency management

## Usage

```go
import "github.com/xero-github/go-xray/xray"
```

Construct a new Xray client like so:

```go
client := xray.NewClient(baseUrl, httpClient)
// List all watches
watches x:= client.ListWatches(context.Background())
```

The services of a client divide the API into logical chunks and correspond to
the structure of the [XRay API documentation.](https://www.jfrog.com/confluence/display/RTF/Artifactory+REST+API.) 

NOTE: Using the context package, one can easily pass cancelation signals and
deadlines to various services of the client for handling a request. In case
there is no context available, then `context.Background()` can be used as a starting point.

### Authentication

Most REST API calls need to be authenticated using your Xray user and password or
through an authentication token. A few calls (such as SYSTEM calls) do not require authentication.

When creating a new xray client, pass in an `http.Client` that can handle authentication.

For API methods that require HTTP Basic Authentication, use the `BasicAuth` or `AccessTokenAuth`

```go
package main

import (
	"github.com/xero-github/go-xray/xray"
	"fmt"
	"context"
)

func main() {
	tp := xray.BasicAuth{
		Username: "<YOUR_USERNAME>",
		Password: "<YOUR_PASSWORD>",
	}
	
	client, err := xray.NewClient("https://localhost/", tp.Client())
	if err != nil {
		fmt.Println(err.Error())
	}

	repos, resp, err := client.ListWatches(context.Background())

  // Or use a 2 hour token generated by call get token https://www.jfrog.com/confluence/display/XRAY/Xray+REST+API#XrayRESTAPI-GetToken
  tp := xray.TokenTransport{
    AccessToken: xray.GetAccessToken("https://localhost/", "username", "password")
  }

  client, err := xray.NewClient("https://localhost/", tp.Client())
	if err != nil {
		fmt.Println(err.Error())
	}
}
```

### Creating and Updating Resources

All structs for Xray resources use pointer values for all non-repeated fields.
This allows distinguishing between unset fields and those set to a zero-value.
Helper functions have been provided to easily create these pointers for string,
bool, and int values. For example:

```go
// create a new watch named "vuln-prod"
watch := xray.Watch{
  GeneralData: &xray.GeneralData{
    Name: xray.String("vuln-prod"),
    Description: xray.String("This is a new watch created using API V2"),
    Active: true,
  },
  ProjectResources: &xray.ProjectResources{
    Resources: []*xray.ProjectResource{
      &{
        Type: xray.String("repository"),
        BinMgrId: xray.String("art-prod-eu"),
        Name: xray.String("libs-release-local"),
        Filters: []*xray.ResourceFilter{
          &{
            Type: xray.String("regex"),
            Value: xray.String(".*"),
          },
        },
      },
    },
  },
  AssignedPolicies: []*xray.AssignedPolicy{
    &{
      Name: xray.String("medium_policy"),
      Type: xray.String("security"),
    },
  },
}

	client.CreateWatch(context.Background(), &watch)
```

## Versioning

In general, go-xray follows [semver](https://semver.org/) as closely as we
can for tagging releases of the package. For self-contained libraries, the
application of semantic versioning is relatively straightforward and generally
understood. But because go-xray is a client library for the Xray API 
we've adopted the following versioning policy:

* We increment the **major version** with any incompatible change to
	functionality, including changes to the exported Go API surface
	or behavior of the API.
* We increment the **minor version** with any backwards-compatible changes to
	functionality.
* We increment the **patch version** with any backwards-compatible bug fixes.

Generally methods will be annotated with a since version.