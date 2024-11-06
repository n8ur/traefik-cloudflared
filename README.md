# traefik-cloudflared
Configuration files for traefik v3 and cloudflared docker containers

These are bare-minimum configuration files to set up a tunnel/SSL cert/reverse proxy system using traefik v3 and Cloudflare's cloudflared tunnel endpoint software in docker containers.  I run these in a Debian 12 VM on a Proxmox cluster.

This setup allows me to access any internal web-based application from the public internet by adding a config file to traefik's dynamic configuration.  Once the tunnel is configured, no additional DNS records need to be created for new services.

These files use docker-compose.yml configuration files plus application-specific config files.  Each container's configuration is in its own directory under opt/ and is started separately.  It would be easy to combine them into a single docker-compose file but I wanted to keep everything isolated.

The traefik config files use YAML syntax and separate the setup functions (in docker-compose.yml), static configuration (in traefik.yml), and dynamic configuration (in separate files under dynamic_config/) for each application.

Three things that most of the tutorials I've found don't mention, or gloss over:

*  The docker default network doesn't provide DNS-like capability, so it's best to use a network for the cloudflared<-->traefik  traffic.  This "cloudflared-traefik" network is created outside these containers and must exist before running them.

*  The Cloudflare tunnel is a "wildcard" with a public hostname of "*" point to a service of type "HTTPS" pointing to the private LAN address of the traefik instance.  *VERY IMPORTANT* -- in the Cloudflare configuration, click on "Additional Application Settings"/"TL" and make sure that "No TLS Verify" is enabled.  By default it's off, and this is what caused me to lose hair for hours.

*  Once the tunnel is created, navigate to the Cloudflare DNS for the domain, add a CNAME record named "*" pointing to <tunnel_id>.cfargotunnel.com, and ensure that record has "proxy" turned on.  You *do not* need to create additional CNAME records for each service.  Any URL that traefik cannot process will be treated as a "not found".

traefik can automatically detect and add proxies for docker services.  I have not experimented with that, as for the moment the services I'm running live outside this docker enrivonment.

My next step, which I'm just beginning to research, is to add an authentication layer so that I can control access to these services.
