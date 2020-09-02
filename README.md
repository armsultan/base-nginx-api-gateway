# Simple NGINX Plus API Gateway

Please refer to the techical blog post **Deploying NGINX Plus as an API Gateway** for detailed examples on implementing a complete API Gateway:
*  ([Part 1](https://www.nginx.com/blog/deploying-nginx-plus-as-an-api-gateway-part-1/)
*  [Part 2](https://www.nginx.com/blog/deploying-nginx-plus-as-an-api-gateway-part-2-protecting-backend-services/))

## File Structure

To achieve this separation, we create a configuration layout that supports a multi‑purpose NGINX Plus instance and provides a convenient structure for automating configuration deployment through CI/CD pipelines. The resulting directory structure under `/etc/nginx` looks like this:

```
etc/
├── nginx/
│    ├── api_conf.d/ ……………………………………………………………… Subdirectory for per-API configuration
│    │   └── mastercard_api_simple.conf …………… Definition and policy for sandbox and production API
│    ├── conf.d/…………………………………………………………………………… Subdirectory for other HTTP configurations (NGINX Dashboard, and other)
│    │   ├── health_checks.conf ………………………………… Example active health check probes
│    │   └── status_api.conf ………………………………………… NGINX Plus live activity monitoring API on port 8080
│    ├── api_backends.conf ……………………………………………… The backend services (upstreams)
│    ├── api_gateway.conf ………………………………………………… Top-level configuration for the API gateway server
│    ├── api_json_errors.conf ……………………………………… HTTP error responses in JSON format
│    └── nginx.conf ………………………………………………………………… Main NGINX configuration file
└── ssl/
    ├── api.example.com.crt …………………………………………… api.example.com self signed cert for HTTPS testing
    ├── api.example.com.key …………………………………………… api.example.com generated ket for HTTPS testing
    └── nginx/ ……………………………………………………………………………… Place your NGINX Plus repo.crt and repo.key here

```