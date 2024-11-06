# traefik-cloudflared
Configuration files for traefik v3 and cloudflared docker containers

These are bare-minimum configuration files to set up a tunnel/SSL cert/reverse proxy system using traefik v3 and Cloudflare's cloudflared tunnel endpoint software in docker containers.  I run these in a Debian 12 VM on a Proxmox cluster.  The setup doesn't seem to require a lot of system resources, though I haven't pumped heavy traffic through it.  2 i5 CPU cores and 4GB RAM seems more than enough.

This setup allows me to access any internal web-based application from the public internet by adding a config file to traefik's dynamic configuration.  Once the tunnel is configured, no additional DNS records need to be created for new services.  Since this depends on Cloudflare's tunnel, your domain must have its DNS nameservers on Cloudflare for it to work.  (The domain can be registered anywhere; it just needs to have DNS hosted on Cloudflare.)

These files use docker-compose.yml configuration files plus application-specific config files.  Each container's configuration is in its own directory under opt/ and is started separately.  It would be easy to combine them into a single docker-compose file but I wanted to keep everything isolated.  The traefik config files use YAML syntax and separate the setup functions (in docker-compose.yml), static configuration (in traefik.yml), and dynamic configuration (in separate files under dynamic_config/) for each application.

A few things that most of the tutorials I've found don't mention, or gloss over:

*  The docker default network doesn't provide DNS-like capability, so it's best to use a separate network for the cloudflared<-->traefik  traffic.  Otherwise you have to identify the services by IP address, and those could change as the docker environment changes.  I cared the "cloudflared-traefik" network via a docker command, and it must exist before running the containers.

*  Most traefik configurations enable the "docker" service that allows it to automatically recognize and proxy for any services on the local docker network without manually configuring a router for that service.  At the moment the services I'm interested in run outside docker, so I haven't yet played with that.  If you have services that are *not* running in docker, you need to set up a "file" service that reads "dynamic configuration" files that define routers for each external service.  You can use one big file or, as I am doing, you can have separate configuration files for each router in a "dynamic config" directory.  In either case, traefik will automatically notice changes to the files and update itself without restarting.

*  The Cloudflare tunnel is a "wildcard" with a public hostname of "*" and it must point to a service of type "HTTPS" pointing to the private LAN address of the traefik instance.

   *VERY IMPORTANT* -- in the Cloudflare configuration, click on "Additional Application Settings"/"TLS" and make sure that "No TLS Verify" is enabled.  By default it's off, and this is what caused me to lose hair for hours.

*  Once the tunnel is created, navigate to the Cloudflare DNS for the domain, add a CNAME record named "*" pointing to <tunnel_id>.cfargotunnel.com, and ensure that record has "proxy" turned on.  You *do not* need to create additional CNAME records for each service.  Any URL that traefik cannot process will be treated as a "not found".

My next step, which I'm just beginning to research, is to add an authentication layer so that I can control access to these services.
