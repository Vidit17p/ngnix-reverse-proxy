# Kubernetes Dynamic Reverse Proxy with Nginx

Helm chart designed to deploy Nginx as a reverse proxy within a Kubernetes environment. The chart provides a configurable Nginx instance capable of basic authentication and dynamic upstream routing, along with standard Kubernetes resources like Deployment, Service, and an optional Horizontal Pod Autoscaler, enabling robust and scalable proxy capabilities.

## Nginx Reverse Proxy Configuration

The chart includes a Kubernetes ConfigMap (`templates/configmap.yaml`) that defines the Nginx server block. This configuration sets up a reverse proxy with basic authentication and dynamic upstream resolution, allowing it to proxy requests to services within the cluster based on URL patterns.

This Nginx configuration provides a dynamic reverse proxy to services running inside a Kubernetes cluster. It allows external access to internal services using a unified and flexible URL structure.

### 🔧 Nginx Configuration

```nginx
location ~^/([^/]+)/([^/]+)/([^/]+)(/.*)?$ {
    resolver kube-dns.kube-system.svc.cluster.local:53;
    auth_basic "Restricted";
    set $svc "$1.$2.svc.cluster.local";
    set $port "$3";
    set $path "$4";

    proxy_pass http://$svc:$port$path$is_args$args;
    proxy_set_header Host $host;
    proxy_read_timeout 900;
}
```

## 🔍 URL Pattern

The configuration matches URLs of the form:

```
/<service-name>/<namespace>/<port>/<optional-path>
```

| Segment | Description                      |
| ------- | -------------------------------- |
| `$1`    | Kubernetes service name          |
| `$2`    | Kubernetes namespace             |
| `$3`    | Target port of the service       |
| `$4`    | Optional path (e.g., `/api/xyz`) |

## ⚙️ How It Works

- **DNS Resolution**: Uses the Kubernetes DNS service to resolve internal service names.
- **Authentication**: Enables basic HTTP authentication (credentials configured separately).
- **Reverse Proxy**: Forwards the request to the internal service based on the URL.
- **Timeout**: Sets a 15-minute proxy read timeout for long-running requests.

## 📌 Example

A request to:

```
GET /backend/default/5000/api/search?q=abc
```

Would be proxied to:

```
http://backend.default.svc.cluster.local:5000/api/search?q=abc
```

## 🔒 Security Note

To enable basic auth, make sure you define the following in your Nginx config:

```nginx
auth_basic_user_file /etc/nginx/.htpasswd;
```

Generate the file with:

```bash
htpasswd -c /etc/nginx/.htpasswd <username>
```

## ✅ Use Cases

- Exposing internal Kubernetes services via a single public endpoint.
- Embedding internal tools (e.g., dashboards) behind a basic authentication wall.
- Avoiding hardcoded service paths in the Nginx config.