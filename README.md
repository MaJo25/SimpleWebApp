# SimpleWebApp
This repository contains the nessecary docker-compose configuration to deploy a simple static webapplication protected with ModSecurity web application firewall and secured docker container runtimes. 

An overview of the security configuration is available [here](#security)

For production setup: [Production setup](#production)

For local development and testing: [Quickstart](#quickstart)

---
## <a id="directory"></a>Directory structure

- [`docker-compose.yml`](docker-compose.yml) - Docker compose file to launch the application.
- [`.env`](.env) - Specifies project configuration, e.g Domain name, location of TLS certificates and keys as well as number of replicas to use for scalability.
- [`src`/](src/) - Source code of the application to be hosted
    - [`index.php`](index.php)

    ---
## <a id="conf"></a>Configuration file structure
In the configuration file there are these settings currently.
- **DOMAIN** - Used in configuring your servername and domain to use when accessing your site.
- **CERTIFICATE_PATH** - Path to your TLS certificate. Needs to be in PEM format.
- **CERTIFICATE_PRIVATE_KEY_PATH** - Path to the private key for the TLS certificate. Needs to be in PEM format.
- **REPLICAS** - Number of replicas to add to the web application server(default: 1)

---
## <a id="production"></a> Production Setup

In order to setup your site for production you need to do the below. 

1. Have a internet routable machine to host this on.
2. Configure the proper DNS records. For instance if I were hosting this machine using the domain: Security-engineer.test I would configure my DNS records as following. 
    
    **DNS records**

| Type  | Hostname                      | Value                                    |
| ----- | ----------------------------- | ---------------------------------------- |
| A     | `security-engineer.test`     | directs to IP address of my hosting server `X.X.X.X`          |
| CNAME | `www.security-engineer.test` | is an alias of `security-engineer.test` |
|

3. Aquire a commercial certificate and install on the machine. Add the path to the certificate and path to the private key of the certificate to the configuration file in the **CERTIFICATE_PATH** and the **CERTIFICATE_PRIVATE_KEY_PATH** variables respectively.

4. Configure the domain in the configuration file at the **DOMAIN** variable.

5. Install [docker](https://docs.docker.com/install/) and [docker-compose](https://docs.docker.com/compose/install/) on the machine.

5. Clone the contents of this repository on the host and run at the root of this folder. 
```bash 
docker-compose up -d 
```

---
## <a id="security"></a> Security

This application is configured to be a 2-tier application serving a static webpage. It is protected by a Web-application firewall that routes the requests to the backend server hosting the application. 
TLS is terminated in the Web-application firewall and the requests are rejected if malicious payloads ment to attack the application are sent to the site.


The Web-application firewall is Modsecurity provided by [OWASP](https://github.com/coreruleset/modsecurity-crs-docker) I'm using the common ruleset available freely by Owasp.


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
3. Utilize some form of container runtime monitoring on the Node. E.g. [Aqua Security](https://www.aquasec.com/)
---

## <a id=quickstart></a>Quick start for local development **ONLY**

0. Install [docker](https://docs.docker.com/install/) and [docker-compose](https://docs.docker.com/compose/install/) on the machine and clone this repository.

1. Edit your /etc/hosts file to include: 
```
127.0.0.1    security-engineer.test 
```

2.Generate self-signed certificates using:
``` bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365
```

3. Save the cert.pem and key.pem files to disk and update the variables **CERTIFICATE_PATH** and **CERTIFICATE_PRIVATE_KEY_PATH** in the .env file.

4. Run the below in the root of this repository.
```bash 
docker-compose up -d 
``` 

5. Open a browser and navigate to security-engineer.test.