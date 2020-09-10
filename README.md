# A Simple NGINX Plus API Gateway

A example base Configuration for an egress NGINX Plus API Gateway

Please refer to the techical blog post **Deploying NGINX Plus as an API Gateway** for detailed examples on implementing 
a complete API Gateway:
*  [Part 1](https://www.nginx.com/blog/deploying-nginx-plus-as-an-api-gateway-part-1/)
*  [Part 2](https://www.nginx.com/blog/deploying-nginx-plus-as-an-api-gateway-part-2-protecting-backend-services/)
*  [Part 3](https://www.nginx.com/blog/deploying-nginx-plus-as-an-api-gateway-part-3-publishing-grpc-services/)

## Example Topology

```
    +----------------+                   +--------------------------+
    |                |                   |                          |
    |  API CLIENT    |    TLS            |        NGINX PLUS        |
    |  APPLICATIONS  | Internal FQDN     |       API Gateway        |
    |                +------------------->                          +--------> api.mastercard.com/openbanking/sandbox...
    |                |     e.g.          | Domain translation,      |          api.mastercard.com/openbanking/...
    |                | api.example.com   | Routing, and more        |          etc...
    |                |                   |                          |
    +----------------+                   +--------------------------+
```

## Challenges / Todo

API clients must connect over hostname api.mastercard.com to properly
authenicate with mastercard API endpoints Todo: Figure out how to connect over
an internal FDQN e.g. `api.example.com` and allow NGINX to provide domain
translation without breaking the authenication requirement 


## File Structure

To achieve this separation, we create a configuration layout that supports a
multi‑purpose NGINX Plus instance and provides a convenient structure for
automating configuration deployment through CI/CD pipelines. The resulting
directory structure under `/etc/nginx` looks like this:

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

## Testing with Postman API Client

### Domain name resolution

**Required:** You need point `api.mastercard.com` to the IP address of the NGINX
API gateway, this is a 

This demo NGINX API gateway listens on the hostname `api.mastercard.com`. Domain
name translation using a internal FQDN has not been configured yet (TODO) and so
for this demo, you will need to configure hostname resolution for
`api.mastercard.com` to your NGINX instance.

1. You will need to add hostname bindings to your client hosts file. For
   example, on Linux/Unix/macOS the host file is `/etc/hosts`

```
#/etc/hosts
# NGINX Plus demo (local docker host)
127.0.0.1 api.mastercard.com
```

2. When using Postman, disable SSL certificate verification, i.e. **Enable SSL
   certificate verification**: `OFF`


### Quick API client setup for testing:

1. [Import](https://learning.postman.com/docs/getting-started/importing-and-exporting-data/) the Mastercard [Open Banking Connect  OpenAPI Specification](https://static.developer.mastercard.com/content/open-banking-connect/swagger/api-accounts-service.yaml)

2. You should now see the a new Postman collections as available as `Open
   Banking - Accounts information service` 

3. [Create a new Environment](https://learning.postman.com/docs/sending-requests/managing-environments/#creating-environments), named what ever you like, e.g. `Mastercard sandbox testing`

4. Set a new variable to store the base URL: 

   ```
   baseUrl = https://api.mastercard.com/openbanking/sandbox/connect/api
   ```


   Set as the **initial value** and **current value**

5. Save Environment
6. Select your new Environment, `Mastercard sandbox testing` in your open workspace
7. Without providing the expected Authorization parameters all  API calls will fail with errors: 

```
{
  "Errors": {
    "Error": [
      {
        "Source": "Gateway",
        "ReasonCode": "INVALID_OAUTH_SBS",
        "Description": "Problem with signature base string. Missing oauth_consumer_key",
        "Recoverable": false,
        "Details": null
      }
    ]
  }
}
```

6. Configure correct Authorization Parameters:

   Authorization tab: 

   * **TYPE**: `OAUTH 1.0` 
   * **Add authorization data to:** `Request Headers` 
   * **Signature Method:** `RSA-SHA256`
   * **Consumer key**:  [Enter your own consumer key] i.e., the `PKCS-12` key provided when creating a new project) 
   * **Private key:** [Insert Private key in  format] i.e. see [Generating the private key and public key for using the SDK](https://developer.mastercard.com/page/generate-the-private-and-public-key-for-use-with-sdk)
   * Leave other values in authorization as default

7. You can now access the mastercard API. A good test is the health check page
   `https://api.mastercard.com/openbanking/sandbox/connect/api/accounts/health`