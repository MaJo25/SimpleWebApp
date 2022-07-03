# SimpleWebApp
This repository contains the nessecary docker-compose configuration to deploy a simple static webapplication protected with ModSecurity web application firewall and secured docker container runtimes. 

An overview of the security configuration is available [here](#security)
## <a id="directory"></a>Directory structure

- [`docker-compose.yml`](docker-compose.yml) - Docker compose file to launch the application.
- [`.env`](.env) - Specifies project configuration, e.g Domain name, location of TLS certificates and keys.
- [`src`/](src/) - Source code of the application to be hosted
    - [`index.php`](index.php)
## <a id="conf"></a>Configuration file structure
In the configuration file there are these settings currently.
- **DOMAIN** - Used in configuring your servername and domain to use when accessing your site.
- **CERTIFICATE_PATH** - Path to your TLS certificate. Needs to be in PEM format.
- **CERTIFICATE_PRIVATE_KEY_PATH** - Path to the private key for the TLS certificate. Needs to be in PEM format.
- **REPLICAS** - Number of replicas to add to the web application server(default: 1)
## <a id="production"></a> Production Setup

In order to setup your site for production you need to do the below. 

1. Aquire a domain - link
2. Setup a DNS-record for that domain - link
3. Generate certificates for that domain and overwrite the configuration in the .env file with the new domain name along with the path to the certificates and keys you generated.

## <a id="security"></a> Security

This application is configured to be a 2-tier application serving a static webpage. It is protected by a Web-application firewall that routes the requests to the backend server hosting the application. 
TLS is terminated in the Web-application firewall and the requests are rejected if malicious payloads ment to attack the application are sent to the site.

The Web-application firewall is Modsecurity provided by [OWASP](https://github.com/coreruleset/modsecurity-crs-docker)

TLS is configured to only support TLS 1.2.

The hardening of the containers running both the application and the Web application firewall consists of the following:

- Remove all capabilites of the pods except *net_bind_service*
- Mount the file system as read-only
- Run the webapplication server as a non-root user. 
- Prevent privilege escalation by setting the *no-new-privileges* security option.
- Apply the default AppArmor policy for the containers.

### Recommended expansion on the security configuration

1. Enable both containers to send syslog to the proper collector by adding the following code to the container service configurations:
    - ```yaml
        logging:
            driver: syslog
            options:
                syslog-address: "Address of syslog collector"

2. Deploy endpoint protection on the servers. Both the WAF container and the Webapplication container.
3. Utilize some form of container rumtime monitoring on the Node. E.g. [Aqua Security](https://www.aquasec.com/)
