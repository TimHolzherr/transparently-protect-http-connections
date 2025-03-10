# Transparently protect an HTTP connection

This demo shows how an existing HTTP connection between two machines can be protected using HTTPS.
We assume both machines use Docker to run their respective applications.
The machine that initiates the connection is called the client, and the other is called the server.
Both the client and the server each have a sidecar container running Caddy. Caddy forwards the connection via HTTPS and ensures the identity of the client using a client certificate.

```
+--------+      +----------------+              +----------------+      +--------+
| Client | ---> | Client-Sidecar | --(HTTPS)--> | Server-Sidecar | ---> | Server |
+--------+      +----------------+              +----------------+      +--------+
```

## Needed adjustments before trying out the demo

- Choose two devices and determine which one will be the server and which one will be the client.
- Note down the IP address of the server and replace the placeholder <ip or domain name of server> in ./client-sidecar/Caddyfile.
- Download the repo on both devices, on the server run `docker compose -f docker-compose-server.yml up` and on the client `docker compose -f docker-compose-client.yml up`
- Simulate a client by requesting `http://localhost:8001` on the client

## Generating the certificates using openssl

### For the server

```bash
cd certs
# Generate the key
openssl genrsa -out server.key 2048
# Create a certificate signing request (CSR)
openssl req -new -key server.key -out server.csr -subj "/CN=FOO"
# Self-sign the server certificate which is valid for 365 days
# subjectAltName is crucial but in this demo it can be set to any value, but it must match the tls_server_name variable of the client
openssl x509 -req -in server.csr -signkey server.key -out server.crt -days 365 -sha256 -extfile <(printf "subjectAltName=DNS:FOO")
```

### For the client

```bash
cd certs
# Generate the key
openssl genrsa -out client.key 2048
# Create a CSR for the client
openssl req -new -key client.key -out client.csr -subj "/CN=BAR"
# Self-sign the client certificate (valid for 365 days)
openssl x509 -req -in client.csr -signkey client.key -out client.crt -days 365 -sha256
```

## Troubleshooting

- To validate a Caddyfile use `docker run --rm -v "$(pwd)/client-sidecar/Caddyfile":/etc/caddy/Caddyfile caddy:latest caddy validate --config /etc/caddy/Caddyfile`
- Enable debug logs in caddy by adding `debug` to the top level block.

```
{
	debug
}
```
