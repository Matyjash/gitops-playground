# Argo CD Ingress

Argo CD is exposed through the Traefik ingress controller included with K3s.

Traffic flow:

Browser → HTTPS → Traefik → HTTPS → Argo CD

Argo CD uses an internal TLS certificate which is problematic if the Pod IPs change. After a cluster restart (which happens in local development environments), Traefik failed TLS verification when connecting to the Argo CD backend.

A `ServersTransport` with `insecureSkipVerify: true` is used to disable backend certificate verification. This should not be used as the default approach in production.
