# docker-vpn-pptp

Run a PPTP VPN server in Docker and manage user credentials through a local `chap-secrets` file. Clients connect with a PPTP VPN client and the credentials you define.

**Security note:** PPTP is an outdated VPN protocol with known security weaknesses. Use it only where legacy compatibility is required, not for sensitive traffic.

## Features

- Run a PPTP server inside a Docker container.
- Authenticate users with `/etc/ppp/chap-secrets`.
- Add or modify users through the mounted credentials file without restarting the container.

## Installation

Install Docker on a Linux host. From the project directory containing the Dockerfile, build the image:

```sh
docker build -t docker-vpn-pptp .
```

Create a credentials file before starting the container:

```sh
mkdir -p config
cat > config/chap-secrets <<'EOF'
# client    server      secret                  acceptable local IP addresses
username    *           replace-this-password   *
EOF
chmod 600 config/chap-secrets
```

Replace the example username and password with your own credentials. Keep this file out of version control.

## Usage

Start the VPN server with your credentials file mounted at the path PPTP expects:

```sh
docker run -d --name docker-vpn-pptp --privileged \
  -p 1723:1723 \
  -v "$PWD/config/chap-secrets:/etc/ppp/chap-secrets" \
  docker-vpn-pptp
```

**Host access:** This container requires `--privileged`, which grants broad access to the host. Run it only on a host where that access is acceptable.

Allow TCP port 1723 and GRE traffic through the host firewall and any upstream router. GRE is IP protocol 47, not TCP or UDP port 47; publishing port 1723 alone does not forward GRE.

Configure a PPTP client with the host's reachable address and the credentials from `config/chap-secrets`. Edit that file to add or modify users. A container restart is not required for those credential changes.

### Connection timeouts and GRE

If connections time out, check whether GRE reaches the container. The donor project reported GRE forwarding issues with Docker 1.7.x and used host networking as a workaround.

To try host networking, remove the existing container and start it again:

```sh
docker rm -f docker-vpn-pptp
docker run -d --name docker-vpn-pptp --privileged --net=host \
  -v "$PWD/config/chap-secrets:/etc/ppp/chap-secrets" \
  docker-vpn-pptp
```

Host networking shares the host's network namespace, so no port publishing is needed. Review Docker's [host networking documentation](https://docs.docker.com/engine/network/drivers/host/) before using this mode.

## License

MIT. This project is based on [mobtitude/docker-vpn-pptp](https://github.com/mobtitude/docker-vpn-pptp), also licensed under MIT. Retain the original copyright and license notices when redistributing the code.

