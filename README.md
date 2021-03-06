# Ingress Service for Docker Swarm

[![Docker Stars](https://img.shields.io/docker/stars/opcycle/swarm-ingress.svg?style=flat-square)](https://hub.docker.com/r/opcycle/swarm-ingress) [![Docker Pulls](https://img.shields.io/docker/pulls/opcycle/swarm-ingress.svg?style=flat-square)](https://hub.docker.com/r/opcycle/swarm-ingress)

Swarm Ingress OpenResty is a ingress service for Docker in Swarm mode that makes deploying microservices easy. It configures itself automatically and dynamically using services labels.

#### Docker Images

- All images based on Fedora Linux
- [GitHub actions builds](https://github.com/opcycle/docker-swarm-ingress/actions) 
- [Docker Hub](https://hub.docker.com/r/opcycle/swarm-ingress)

### Features

- No external load balancer or config files needed making for easy deployments
- Integrated TLS decryption for services which provide a certificate and key
- Automatic service discovery and load balancing handled by Docker
- Scaled and maintained by the Swarm for high resilience and performance
- On the fly SSL registration and renewal

### SSL registration and renewal

This OpenResty plugin automatically and transparently issues SSL certificates from Let's Encrypt as requests are received using [lua-resty-auto-ssl](https://github.com/auto-ssl/lua-resty-auto-ssl) plugin. It works like:

- A SSL request for a SNI hostname is received.
- If the system already has a SSL certificate for that domain, it is immediately returned (with OCSP stapling).
- If the system does not yet have an SSL certificate for this domain, it issues a new SSL certificate from Let's Encrypt. Domain validation is handled for you. After receiving the new certificate (usually within a few seconds), the new certificate is saved, cached, and returned to the client (without dropping the original request).


### Run the Service

The Ingress service acts as a reverse proxy in your cluster. It exposes port 80 and 443
to the public an redirects all requests to the correct service in background.
It is important that the ingress service can reach other services via the Swarm
network (that means they must share a network).

```bash
docker service create --name ingress \
  --network ingress-routing \
  -p 80:80 \
  -p 443:443 \
  --mount type=bind,source=/var/run/docker.sock,destination=/var/run/docker.sock \
  --constraint node.role==manager \
  opcycle/swarm-ingress
```

It is important to mount the docker socket, otherwise the service can't update
its configuration.

The ingress service should be scaled to multiple nodes to prevent short outages
when the node with the ingress servic becomes unresponsive (use `--replicas X` when starting the service).

### Register a Service for Ingress

A service can easily be configured using ingress. You must simply provide a label
`ingress.host` which determines the hostname under wich the service should be
publicly available.

## Configuration Labels

Additionally to the hostname you can also map another port and path of your service.
By default a request would be redirected to `http://service-name:80/`.

| Label   | Required | Default | Description |
| ------- | -------- | ------- | ----------- |
| `ingress.host` | `yes` | `-`      | When configured ingress is enabled. The hostname which should be mapped to the service. Multiple domain supported using `ingress.host0` .. `ingress.hostN` |
| `ingress.port` | `no`  | `80`    | The port which serves the service in the cluster. |
| `ingress.path` | `no`  | `/`     | A optional path which is prefixed when routing requests to the service. |
| `ingress.ssl` | `no` | `-` | Enable SSL provisioning for host | 
| `ingress.ssl_redirect` | `no` | `-` | Enable automatic redirect from HTTP to HTTPS | 
| `ingress.max_body_size` | `no` | `10m` | Max request body size | 
| `ingress.proxy_timeout` | `no` | `600` | Proxy timeout | 


### Run a Service with Enabled Ingress

It is important to run the service which should be used for ingress that it
shares a network. A good way to do so is to create a common network `ingress-routing`
(`docker network create --driver overlay ingress-routing`).

To start a service with ingress simply pass the required labels on creation.

```bash
docker service create --name my-service \
  --network ingress-routing \
  --label ingress.host=my-service.company.tld \
  --label ingress.ssl=yes \
  --label ingress.ssl_redirect=yes \
  nginx
```

It is also possible to later add a service to ingress using `service update`.

```bash
docker service update \
  --label-add ingress.host=my-service.company.tld \
  --label-add ingress.port=8080 \
  my-service
```

# Contributing
We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/opcycle/docker-swarm-ingress/issues), or submit a [pull request](https://github.com/opcycle/docker-swarm-ingress/pulls) with your contribution.

# Issues
If you encountered a problem running this container, you can file an issue. For us to provide better support, be sure to include the following information in your issue:

- Host OS and version
- Docker version
- Output of docker info
- Version of this container
- The command you used to run the container, and any relevant output you saw (masking any sensitive information)
