# iDRAC 6 dockerized

![Web interface](https://i.imgur.com/Au9DPmg.png)
*Web interface*

![Guacamole](https://i.imgur.com/8IWAATS.png)
*Directly connected to VNC via Guacamole*

## About

Allows access to the iDRAC 6 console without installing Java or messing with Java Web Start. Java is only run inside of the container and access is provided via web interface or directly with VNC.

Container is based on [baseimage-gui](https://github.com/jlesage/docker-baseimage-gui) by [jlesage](https://github.com/jlesage)

## Usage

See the docker-compose [here](https://github.com/DomiStyle/docker-idrac6/blob/master/docker-compose.yml) or use this command:

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

## Issues & limitations

* User preferences can't be saved
* VNC starts with default 1024x768 resolution instead of fullscreen
  * Use "View" -> "Full Screen" to work around this issue
* Keyboard layout can't be changed
* Only one iDRAC server can be accessed with a single instance
  * Run multiple containers to work around this issue (e.g. srv1.idrac.example.org, srv2.idrac.example.org)

## Traefik Setup

Traefik is a pretty wicked reverse proxy solution.  A good tutorial can be found at [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-use-traefik-as-a-reverse-proxy-for-docker-containers-on-debian-9).  Note: htpasswd is included with macOS.