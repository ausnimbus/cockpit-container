# Cockpit Web Service Container

Atomic contains the Cockpit bridge process, but not the Web Service. This means you can add an Atomic host to another Cockpit dashboard, but not connect to it directly.

If you want to connect directly to your Atomic Host with your web browser, use this privileged container.

Run it like so:

    # atomic run ausnimbus/cockpitws

And then use your web browser to log into port 9090 on your host IP address as usual.

Important: This expects that Atomic (the host operating system) has the cockpit-bridge executable and cockpit-shell package.

## Starting on boot

Create the file `/etc/systemd/system/cockpitws.service`

```
[Unit]
Description=Cockpit Web Interface
Requires=docker.service
After=docker.service

[Service]
Restart=on-failure
RestartSec=10
ExecStart=/usr/bin/docker run --rm --privileged --pid host -v /:/host --name %p ausnimbus/cockpitws /container/atomic-run --local-ssh
ExecStop=-/usr/bin/docker stop -t 2 %p

[Install]
WantedBy=multi-user.target
```

```
systemctl daemon-reload
systemctl enable cockpitws.service
systemctl start cockpitws.service
```

Alternatively, it can be added into cloud-init

```
write_files:
-   encoding: b64
    content: W1VuaXRdCkRlc2NyaXB0aW9uPUNvY2twaXQgV2ViIEludGVyZmFjZQpSZXF1aXJlcz1kb2NrZXIuc2VydmljZQpBZnRlcj1kb2NrZXIuc2VydmljZQoKW1NlcnZpY2VdClJlc3RhcnQ9b24tZmFpbHVyZQpSZXN0YXJ0U2VjPTEwCkV4ZWNTdGFydD0vdXNyL2Jpbi9kb2NrZXIgcnVuIC0tcm0gLS1wcml2aWxlZ2VkIC0tcGlkIGhvc3QgLXYgLzovaG9zdCAtLW5hbWUgJXAgYXVzbmltYnVzL2NvY2twaXR3cyAvY29udGFpbmVyL2F0b21pYy1ydW4gLS1sb2NhbC1zc2gKRXhlY1N0b3A9LS91c3IvYmluL2RvY2tlciBzdG9wIC10IDIgJXAKCltJbnN0YWxsXQpXYW50ZWRCeT1tdWx0aS11c2VyLnRhcmdldAo=
    owner: root:root
    path: /etc/systemd/system/cockpitws.service
    permissions: '0644'

runcmd:
 - systemctl daemon-reload
 - systemctl enable cockpitws.service
 - systemctl start --no-block cockpitws.service
 ```

## Kubernetes

We also want to manage Kubernetes so we need to add the `cockpit-kubernetes` package, on atomic host
do this by using `rpm-ostree pkg-add cockpit-kubernetes`.

It was also possible to hack the image and include `cockpit-kubernetes` within the Docker container. You
need to remove `/usr/bin/nsenter --net=/container/target-namespace/ns/net --uts=/container/target-namespace/ns/uts -- `
from `atomic-run` and run an `openssh-server` with the same pam hack `sed -e '/pam_selinux/d' -e '/pam_sepermit/d' /etc/pam.d/sshd`
However this doesn't work too well..

## More Info

 * [Cockpit Project](https://cockpit-project.org)
 * [Cockpit Development](https://github.com/cockpit-project/cockpit)
