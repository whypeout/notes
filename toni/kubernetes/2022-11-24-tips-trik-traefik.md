# Tips Trik Traefik

sources:
- [https://community.traefik.io/t/traefik-v2-with-multiple-non-docker-backend/4135/4](https://community.traefik.io/t/traefik-v2-with-multiple-non-docker-backend/4135/4)


## Traefik Reverse Proxy Non-Docker Backend

```yaml
http:
  routers:
    my-api:
      entryPoints:
        # Expose on :8080 aka dashboard
        - dashboard
      # Handle Route /dashboard or /api
      rule: "PathPrefix(`/dashboard`) || PathPrefix(`/api`)"
      # Expose API
      service: api@internal
      # Basic Auth middleware
      middlewares:
        - dashboard-auth
    
    my-secure-api:
      entryPoints:
        # Expose :8080 via https
        - https
      # Handle Route on subdomain dashboard.domain with /dashboard or /api
      rule: "Host(`dashboard.domain`) && (PathPrefix(`/api`) || PathPrefix(`/dashboard`))"
      service: api@internal
      middlewares:
        - dashboard-auth
      tls:
        # Acme http challenge
        certResolver: myhttpchallenge
        
    # Catch all global Router redirect
    https-redirect:
      entryPoints:
        - https
      # Router Any Host req
      rule: "hostregexp(`{host:.+}`)"
      # Service definition for dummy
      service: dummy
      middlewares:
        - redirect-to-https
    
middlewares:
  dashboard-auth:
    basicAuth:
      users:
        # admin:passw0rd
        - "admin:$2y$05$kqK7GlhnGCnt/fYdrCL2AeZykK2T0cN4sBcCaRs61vMz7yMWaR9M."
  redirect-to-https:
    redirectScheme:
      scheme: https
      permanent: true


services:
  web_external:
    loadBalancer:
      servers:
        - url: http://192.168.1.1:80
        
routers:
  web_external:
    entrypoints:
      - https
    service: web_external

```

## Traefik 

```
# docker-compose.yml
traefik:
  command:
  - --providers.file.watch=true
  - --providers.file.filename=/traefik_dynamic.toml
  # enable dashboard insecure connection
  - --api.insecure=true
  - --api.dashboard=true
  - --api.debug=true
  - --log.level=DEBUG
  - --entrypoints.web.address=:80
  - --entrypoints.websecure.address=:443
  # lets encrypt production default url "https://acme-v02.api.letsencrypt.org/directory"
  - --certificatesresolvers.myresolver.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"
  - --certificatesresolvers.myresolver.acme.email=email@domain.com
  - --certificatesresolvers.myresolver.acme.storage=acme.json
  - --certificatesresolvers.myresolver.acme.tlschallenge=true
  volumes:
  - ./traefik_dynamic.toml:/traefik_dynamic.toml:ro
  - letsencrypt:/letsencrypt/acme.json:wo
  ports:
  - target: 80
    published: 80
    mode: host
  - target: 443
    published: 443
    mode: host
  - target: 8080
    published: 8080
    mode: ingress
    # protocol: tcp
  networks:
    - web
volumes:
  letsencrypt:
  
secrets:
  custom_crt:
    external: true
    name: server.crt
  custom_key:
    external: true
    name: server.key
```


```toml
# nano traefik.toml
[http.routers]
  [http.routers.myrouter]
    rule="Host(`goser.domain`)
    middlewares = ["auth"]
    service = "goserver"
    entryPoints = ["websecure"]
    
    # route TLS (ignore non tls req)
    [http.routers.myrouter.tls]
      options = "myoptions"
      #certResolver = "myresolver"
      #[[http.routers.myrouter.tls.domains]]
      # main = "goser.domain"
      
  [http.routers.api]
    rule="Host(`traefik.dom`) && (PathPrefix(`/api`) || PathPrefix(`/dashboard`))"
    entryPoints = ["websecure"]
    middlewares = ["auth"]
    service = "api@internal"
    
    [http.routers.api.tls]
      certResolver = "myresolver"
      [[http.routers.api.tls.domains]]
        main = "traefik.domain"
        
  # redirect http to https utk dashboard
  [http.routers.api-http]
    entryPoints = ["web"]
    rule = "Host(`traefik.domain`) && (PathPrefix(`/path) || (PathPrefix(`/dashboard`))"
    middlewares = ["auth","redirect-to-https"]
    service = "api@internal"

# middlewares
[http.middlewares]
  [http.middlewares.auth.basicAuth]
    users = ["test:XXX"]
    
  [http.middlewares.redirect-to-https.redirectScheme]
    scheme = "https"
    port = "443"
    permanent = true
    
# services
[http.services]
  [http.services.goserver.loadBalancer]
    [[http.services.goserver.loadBalancer.servers]]
      url = "http://instance-private-ip:8001"
      
# custom tls cert
[tls]
  [[tls.certificates]]
    certFile = "/run/secrets/server.crt"
    keyFile = "/run/secrets/server.key"
    
  [tls.stores]
    [tls.stores.default]
      [tls.stores.default.defaultCertificate]
        certFile = "/run/secrets/server.crt"
        keyFile = "/run/secrets/server.key"
        
  [tls.options]
    [tls.options.myoptions]
      minVersion = "VersionTLS12"
      sniStrict = true
   #[tls.options.mintls13]
   #   minVersion = "VersionTLS13"
```
