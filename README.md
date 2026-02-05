## Piko Development Proxy ( Ngrok + Token - Shenanigans )

> Tim's Piko Fork ( [original repo](https://github.com/andydunstall/piko) )

Simply setup a development proxy for your team - [based on the awesome oss piko](https://github.com/andydunstall/piko) - with just a helm chart!

```bash
cd operations/helm
cp values-example.yaml values.yaml
```

Then generate your admin secret: `openssl rand -base64 32` and set `auth.hmac.secret`.
Then modify the other helm values to your needs:

- `tls.certManager.issuerName`: Add your k8s certmanager name here
- `ingress.proxy.host`: Your main domain (e.g., `piko.example.com`)
- `ingress.proxy.additionalHosts`: Add specific subdomains you need certificates for

```bash
helm updade --install piko ./piko -f values.yaml -n piko
```

### Adding Users

> This is miss-using Piko 'services' and assuming they belong to specific users
> This works cause the same token can be used to access multiple services.

A user always needs an `apiToken`, generate it via:

```bash
export HMAC_SECRET='your-secret-from-earlier-here'
./generate-token.sh user1
```

This generates 2 tokens:

```bash
=== Upstream Token (for services connecting to Piko) ===
UPSTREAM_TOKEN=...

=== Proxy Token (for clients accessing services) ===
PROXY_TOKEN=...
```

### Starting A Proxy

Download the Piko binary, then:

```bash
piko agent http api 3000 \
  --connect.url https://upstream.<user>.example.com \
  --connect.token "$UPSTREAM_TOKEN"
```

Now you can access your development proxy if you provide the correc secruity header:

```bash
curl -H "Authorization: Bearer $PROXY_TOKEN" \
     https://my-service.<user>.example.com/
```

> NOTE: The requirement of the `PROXY_TOKEN` can be removed by setting `auth.proxy.enabled` to `false`.
> **BUT: this potentially opens your development proxies up to attacks, devlopment servers are rarely secure!**