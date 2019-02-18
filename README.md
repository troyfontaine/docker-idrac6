# iDRAC 6 dockerized

![Web interface](https://i.imgur.com/Au9DPmg.png)
*Web interface*

![Guacamole](https://i.imgur.com/8IWAATS.png)
*Directly connected to VNC via Guacamole*

## About

Allows access to the iDRAC 6 console without installing Java or messing with Java Web Start. Java is only run inside of the container and access is provided via web interface or directly with VNC.

Container is based on [baseimage-gui](https://github.com/jlesage/docker-baseimage-gui) by [jlesage](https://github.com/jlesage)

## Usage

See the docker-compose [here](https://github.com/troyfontaine/docker-idrac6/blob/master/docker-compose.yml.example) or use this command:

    docker run -d -p 5800:5800 -p 5900:5900 -e IDRAC_HOST=idrac1.example.org -e IDRAC_USER=root -e IDRAC_PASSWORD=1234 domistyle/idrac6

The web interface will be available on port 5800 while the VNC server can be accessed on 5900. Startup might take a few seconds while the Java libraries are downloaded. You can add a volume on /app if you would like to cache them.

## Configuration

All listed configuration variables are required.

| Variable       | Description                                  | Required |
|----------------|----------------------------------------------|----------|
|`IDRAC_HOST`| Host for your iDRAC instance. Make sure your instance is reachable with https://<IDRAC_HOST>. See IDRAC_PORT for using custom ports. HTTPS is always used. | Yes |
|`IDRAC_USER`| Username for your iDRAC instance. | Yes |
|`IDRAC_PASSWORD`| Password for your iDRAC instance. | Yes |
|`IDRAC_PORT`| The optional port for the web interface. (443 by default) | No |

**For advanced configuration options please take a look [here](https://github.com/jlesage/docker-baseimage-gui#environment-variables).**

## Volumes

| Path       | Description                                  | Required |
|------------|----------------------------------------------|----------|
|`/app`| Libraries downloaded from your iDRAC instance will be stored here. Add a volume to cache those files for a faster container startup. | No |
|`/vmedia`| Can be used to allow virtual media to be mounted. | No |
|`/screenshots`| Screenshots taken from the virtual console can be stored here. | No |

Make sure the container user has read & write permission to these folders on the host. [More info here](https://github.com/jlesage/docker-baseimage-gui#usergroup-ids).

The `docker-compose.yml.example` included mounts the local directory media at `/vmedia`, so if you're needing to mount an ISO image, you can place it there to get up and running quickly.

## Issues & limitations

* User preferences can't be saved
* VNC starts with default 1024x768 resolution instead of fullscreen
  * Use "View" -> "Full Screen" to work around this issue
* Keyboard layout can't be changed
* Only one iDRAC server can be accessed with a single instance
  * Run multiple containers to work around this issue (e.g. srv1.idrac.example.org, srv2.idrac.example.org)

## Traefik Setup

Traefik is a pretty wicked reverse proxy solution.  A good tutorial can be found at [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-use-traefik-as-a-reverse-proxy-for-docker-containers-on-debian-9).  Note: htpasswd is included with macOS, so you can use that to generate the credentials to use with Traefik's dashboard or the iDRAC interfaces.

### Example Files

There are several example files included in this repo that you need to copy to create the files to use.  You must modify these to fit your environment.

| Example file | Proper Name | Purpose |
| --- | --- | --- |
| acme.json.example | acme.json | Leave this empty-this is for caching the acme configuration. |
| docker-compose.yml.example | docker-compose.yml | This is the docker-compose file pre-configured for use with three separate iDRAC hosts. |
| idrac.env.example | idrac1.env | Example file for passing the idrac credentials and hostname to a container. |
| traefik.env.example | traefik.env | Example environment configuration file for storing the credentials for your DNS provider to use for requesting Let's Encrypt certificates.  The example includes the required environment files for Cloudflare. |
| traefik.toml.example | traefik.toml | Example traefik configuration file. |

### Configuring

You will need to modify the `docker-compose` example to match the number of iDRAC hosts you have-by default it is configured for three hosts.  Simply copy one of the existing idrac configurations to add more or remove them as needed.

You will need to have an appropriate `idrac#.env` file for each occurrence you have in your `docker-compose.yml`.

Customizing the `traefik.toml` requires creating new user entries with the htpasswd utility.  The credentials used in the example file literally have the password `password`.  Please, do not reuse them.

In the `[acme]` section, you need to update the email address used, modify the `main` entry under `[[acme.domains]]` section to match your internal domain (this requests a wildcard certificate, you can change this to use a specific host name and Subject Alternative Name-see the Traefik docs for more details).

Lastly, configure the `[docker]` section to reflect your internal domain.