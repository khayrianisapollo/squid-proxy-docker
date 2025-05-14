# Squid Proxy Docker

Deploy Squid Proxy in a Docker container for caching and forwarding HTTP, HTTPS, and FTP requests.

## 1. Configuration Overview

This setup uses Docker Compose to run Squid Proxy with a custom `squid.conf` file.

### docker-compose.yml

```yaml
services:
  proxy:
    image: ubuntu/squid
    ports:
      - "3128:3128"             # Exposes Squid on port 3128
    environment:
      - TZ=UTC                  # Set timezone
    volumes:
      - ./squid.conf:/etc/squid/squid.conf
    configs:
      - source: squid
        target: /etc/squid/squid.conf

configs:
  squid:
    file: ./squid.conf
````

* The service uses the official `ubuntu/squid` image.
* It mounts a custom Squid configuration file.
* Port `3128` is the default Squid proxy port and is exposed for external use.

### squid.conf

```conf
http_port 3128
http_access allow all

# Only allow your IP
# acl allowed_ips src [YOUR_PUBLIC_IP]/32
# http_access allow allowed_ips
# http_access deny all
```

* By default, the config allows access from all IPs.
* To restrict access, uncomment the `acl allowed_ips` line and replace `[YOUR_PUBLIC_IP]` with the IP of the client machine.

> ⚠️ For security, it's highly recommended to restrict access by enabling the `acl` rules.

## 2. Deployment Instructions

This proxy server is designed to be hosted on a remote machine (e.g., VPS or cloud server). Any client machine using its IP address can route requests through the proxy.

### Steps to Run:

1. Place `docker-compose.yml` and `squid.conf` in the same directory.
2. On the remote server, run:

```bash
docker compose up -d
```

3. Ensure port `3128` is open in the server firewall (e.g., `ufw allow 3128` or security group settings for cloud).

## 3. Testing the Proxy

To test the proxy, use a different machine (not the host server) and replace `<SERVER_IP>` with the IP of the server running Squid.

### Using Python (requests library)

```python
import requests

proxies = {
    "http": "<PROXY_SERVER_IP>:<PROXY_SERVER_PORT>",
    "https": "<PROXY_SERVER_IP>:<PROXY_SERVER_PORT>",
}

response = requests.get("https://ipinfo.io/ip", proxies=proxies)
print(response.text)
```

### Using curl

```bash
curl -x <PROXY_SERVER_IP>:<PROXY_SERVER_PORT> https://ipinfo.io/ip
```

> You should see the IP of your proxy server in the response, confirming that traffic is being routed through Squid.

---

## Notes

* You can further secure your proxy by adding authentication or enabling SSL in `squid.conf`.
* Consider using a fixed IP for your client or managing access via VPN if exposing the proxy to the public internet.
* Check out this [GitHub repository](https://github.com/FrancescoDiSalesGithub/Google-cloud-shell-hacking), specifically the section titled **"Use the Google Cloud Shell as Proxy"** for guidance on leveraging Google Cloud Shell as a proxy.

