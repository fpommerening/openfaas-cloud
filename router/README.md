## Router for wildcard domain-name

This is the single point of entry for all HTTP requests for OpenFaaS Cloud. When deployed it sits in front of the gateway to translate DNS entries to function paths.

### Roadmap

- [x] sub-domain mapping
- [ ] [authz via OAuth 2.0 for protected URL routes via #145](https://github.com/openfaas/openfaas-cloud/issues/145)

### Example of host-to-URL translations:

The convention is that a `username` becomes a sub-domain and the first part of the HTTP path becomes the function the user sees in a URL. At the gateway both parts are combined in the function name: `username-function`

Username: alexellis (Git user)
Function: kubecon-tester

Deployed function: alexellis-kubecon-tester

User-facing proxy address: https://alexellis.domain.io/kubecon-tester

Gateway address: http://gateway:8080/function/alexellis-kubecon-tester

### Usage:

The value `upstream_url` should point to an OpenFaaS API Gateway

```sh
upstream_url=http://127.0.0.1:8080 port=8081 go run main.go
```

Test it:

```sh
curl -H "Host: alexellis.domain.io" localhost:8081/kubecon-tester
```

### Development

```sh
TAG=0.5.1 make build ; make push
```

> Note: on Kubernetes change `gateway:8080` to `gateway.openfaas:8080`.

If you wish to bypass authentication you can run the router auth an auth_url of the `echo` function deployed via the `stack.yml`.

``` sh
TAG=0.5.1

# As a container

docker rm -f of-router

docker run \
 -e upstream_url=http://gateway:8080 \
 -e auth_url=http://echo:8080 \
 -p 8081:8080 \
 --network=func_functions \
 --name of-router \
 -d openfaas/cloud-router:$TAG

# Or as a service

docker service rm of-router

docker service create --network=func_functions \
 --env upstream_url=http://gateway:8080 \
 --env auth_url=http://echo:8080 \
 --publish 8081:8080 \
 --name of-router \
 -d openfaas/cloud-router:$TAG
```
