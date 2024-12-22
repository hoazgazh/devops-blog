---
title: "Install podman on Amazon Linux 2"
seoTitle: "Podman Setup on Amazon Linux 2"
datePublished: Sun Dec 22 2024 12:42:46 GMT+0000 (Coordinated Universal Time)
cuid: cm4zlm9gf003r09l13cu17gio
slug: install-podman-on-amazon-linux-2
tags: podman-containers-docker

---

```bash
sudo amazon-linux-extras disable docker
sudo amazon-linux-extras install -y kernel-ng

sudo yum check-update
sudo yum install -y yum-utils yum-plugin-copr

sudo cat <<EOF > /etc/yum.repos.d/devel\:kubic\:libcontainers\:stable.repo
[devel_kubic_libcontainers_stable]
name=Stable Releases of Upstream github.com/containers packages (CentOS_7)
type=rpm-md
baseurl=https://provo-mirror.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/CentOS_7/
gpgcheck=0
gpgkey=https://provo-mirror.opensuse.org/devel:/kubic:/libcontainers:/stable/CentOS_7/repodata/repomd.xml.key
enabled=1
EOF

sudo yum copr enable -y lsm5/container-selinux

sudo yum check-update
sudo yum install -y podman slirp4netns

test ! -f /etc/containers/seccomp.json && \
  sudo wget https://raw.githubusercontent.com/docker/labs/master/security/seccomp/seccomp-profiles/default.json -O /etc/containers/seccomp.json

sudo grubby --update-kernel=ALL \
   --args="systemd.unified_cgroup_hierarchy=1 namespace.unpriv_enable=1 user_namespace.enable=1"

echo "user.max_user_namespaces=10000" | sudo tee /etc/sysctl.d/98-userns.conf

echo "$(id -un):100000:65536" | sudo tee -a /etc/subuid
echo "$(id -un):100000:65536" | sudo tee -a /etc/subgid

sudo yum install -y git-core autoconf gettext-devel automake libtool libxslt byacc libsemanage-devel

mkdir -pv ~/src && cd ~/src
git clone https://github.com/shadow-maint/shadow shadow-utils
cd shadow-utils
./autogen.sh --prefix=/usr/local
make -j $(nproc)

sudo cp src/newgidmap src/newuidmap /usr/local/bin/

sudo setcap cap_setuid+ep /usr/local/bin/newuidmap
sudo setcap cap_setgid+ep /usr/local/bin/newgidmap

sudo systemctl reboot
podman system migrate
podman version
podman run --rm -it hello-world:latest
```