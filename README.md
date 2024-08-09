Not much to see here. This is just a space to document my OpenShift Local
instance configuration that I run at home.

The OpenShift Local instance is accessible from any device in my Tailnet.

# OpenShift Local Setup

Use a host such as a machine running Fedora to run OpenShift Local:

```bash
# Download VM image using crc binary downloaded from cloud.redhat.com
crc setup

# Configure resources
crc config set cpus 14
crc config set memory 26624
crc config set disk-size 64

# Download the pull secret from cloud.redhat.com
crc start --pull-secret-file crc-pull-secret.txt
```

Once OpenShift Local has started, expose it to your network by following the
guide in [Red Hat's remote server documentation](https://docs.redhat.com/en/documentation/red_hat_openshift_local/2.13/html/getting_started_guide/networking_gsg#setting-up-remote-server_gsg).

_Note: Set `fastestmirror = True in /etc/dnf/dnf.conf` if `dnf` speeds are slow in newer Fedora._

# Tailscale and DNS

Install Tailscale on the host machine that's running OpenShift Local:

```bash
curl -fsSL https://tailscale.com/install.sh | sh

sudo tailscale up
sudo systemctl start tailscaled
```

Add a custom dnsmasq configuration file to Pi-hole that resolves the OpenShift
Local hostnames `tailscale ip` obtained on that machine.

```bash
nano pihole/etc-dnsmasq.d/02-pihole-custom.conf 
```

Add the following to the `02-pihole-custom.conf`

```
address=/apps-crc.testing/100.X.X.X
address=/api.crc.testing/100.X.X.X
```

Start the Pi-hole with this file mounted, i.e adding the `-v` option when
starting the container:

```
-v "/home/pi/etc-dnsmasq.d/:/etc/dnsmasq.d/"
```