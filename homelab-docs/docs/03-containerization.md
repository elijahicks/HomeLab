# Containerization: Docker & Portainer

## Docker Engine

Installed via Docker's official install script:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

**Issue encountered:** the install script's OS auto-detection registered
the Debian `trixie` APT repository instead of the correct Ubuntu `noble`
repository for this Ubuntu-based distribution, producing unresolvable
dependency conflicts on install. Resolved by manually removing the
incorrect repo file and re-registering the correct one:

```bash
sudo rm /etc/apt/sources.list.d/docker.list
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu noble stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

Non-root Docker access:

```bash
sudo usermod -aG docker $USER
```

(requires a new login session to take effect — group membership is
evaluated at session start, not applied retroactively).

## Portainer

Deployed as a container to provide visual management on top of the
Docker CLI:

```bash
docker run -d -p 9000:9000 --name portainer \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  --restart unless-stopped \
  portainer/portainer-ce
```

Used for day-to-day container inspection and log viewing. One
deployment attempted directly through Portainer's UI produced no
container and no visible error; redeploying the identical configuration
via the Docker CLI succeeded and surfaced the actual underlying error
(a host port conflict, documented in
`07-challenges-and-resolutions.md`). This is noted here as a practical
lesson: the CLI remains the more reliable source of truth for
diagnosing failed deployments.
