# Acme.sh (snap)

Please note that this version of acme.sh has some limitations, as it's built to run in a strict confiment, on systems such as Ubuntu Core.

You will therefor not have all the options available as given by the `--help` flag.

See [acme-sh-snap-integration](https://git.giaever.org/joachimmg/acme-sh-snap-integration) to integrate your snap with this version of acme.sh.

[![Get it from the Snap Store](https://snapcraft.io/static/images/badges/en/snap-store-black.svg)](https://snapcraft.io/acme-sh)
[![Donate with PayPal](https://giaever.online/paypal-donate-button.png)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=69NA8SXXFBDBN&source=https://git.giaever.org/joachimmg/acme-sh)

## Usage

Make sure you've installed the Snap version of Acme.sh. See intallation instructions below.

You'll have the following commands available:

* `acme-sh`: the acme.sh binary
* `acme-sh.dns-manual`: same as running `acme-sh --yes-I-know-dns-manual-mode-enough-go-ahead-please` Please read [Force to use dns manual mode](https://github.com/acmesh-official/acme.sh/wiki/dns-manual-mode) at the official repository of acme.sh.
* `acme-sh.connect`: connect an snap-app to acme-sh to be able to use your certificate. The connecting snap needs an integration.
* `acme-sh.pub-key`: print the public key of the daemon.

### acme-sh
See `acme-sh --help` and examples at [Acme.sh at github](https://github.com/acmesh-official/acme.sh#2-just-issue-a-cert).

Example:
```
acme-sh --issue -d test.tld
```

### acme-sh.dns-manual
See examples at [Acme.sh at github](https://github.com/acmesh-official/acme.sh/wiki/dns-manual-mode). Please be aware that the it can take some time for your TXT entry to be visible, so you should wait 10 minutes from your run `--issue` and added the TXT-entry, until you run `--renew`.

Example:

```
acme-sh.dns-manual --issue -d test.tld
```

Log into your registrar and add the returned TXT-entry to the DNS-entries for your domain. Wait about 10 minutes before you'll run the following command.

```
acme-sh-dns-manual --renew -d test.tld
```

### acme-sh.connect

Connect a snap-app with acme-sh, so acme-sh can expose a selected certificate to the snap-app. Make sure you first issued a certificate.

Example:

```bash
# install acme-sh
$ snap install home-assistant-snap 

# Connect the certs plug
$ snap connect acme-sh:certs home-assistant-snap:certs

# connect with acme-sh
$ acme-sh.connect
```

And continue with the available options:

```
Choose certificate:
1: test.tld
2: 2nd-test.tld
Select a cetificate to expose [1 - 2]: 1

Choose connection for test.kgv14.dev:
1: snap-test (uuid: f83dd5aa-e9e6-11ea-bea6-2b7aa269892a)
2: home-assistant-configurator (uuid: e128c67c-f373-11ea-ba27-6f03a9350960)
3: home-assistant-snap (uuid: 97bb4d40-e6f1-11ea-8e52-5bdd87860a98)
Choose connection [1 - 3]: 3

= You choose connection home-assistant-snap_97bb4d40-e6f1-11ea-8e52-5bdd87860a98
Released test.tld for connection home-assistant-snap_97bb4d40-e6f1-11ea-8e52-5bdd87860a98
```

You'll now have access to the certificate in Home Assistant's own enviroment under the path `/var/snap/home-assistant-snap/current/certs` (encrypted) and `/var/snap/home-assistant-snap/current/.ssl` (unencrypted). See [acme-sh-snap-integration](https://git.giaever.org/joachimmg/acme-sh-snap-integration) for how this is done.

You can now add the certificates to the `configuration.yaml`-file of Home Assistant.

### acme-sh.pub-key
Acme-sh will try to auto-renew every certificate, whenever its due. However for this to work, you'll have to add the public key to your `authorized_keys`, as the running daemon that will renew the certificates is running as root and the certificate-files (configurations, certificates etc) is owned by your user.

The daemon will therefor renew the certificates on behalf of you, using SSH. See [renew-daemon source](https://git.giaever.org/joachimmg/acme-sh/src/master/src/renew-daemon) for how this is done.

Add the certificate to your user by running something similar to
```
acme-sh.pub-key >> $HOME/.ssh/authorized_keys
```

**or** simply just copy the output of `acme-sh.pub-key` and edit `authorized_keys` manually with your favourite text editor.

## Build and installation
### Install from The Snap Store (Recommended)

Make sure you have Snapd installed on your system. See [Installing snapd](https://snapcraft.io/docs/installing-snapd) for a list of distributions with and without snap pre-installed, including installation instructions for those that have not.

```bash
$ snap install acme-sh
```

### Build this snap from source

We recommend that your download a pre-built version of this snap from [The Snap Store](https://snapcraft.io/acme-sh), or at least make sure that you checkout the latest tag, as the master tag might contain faulty code during development.

1. **Clone this repo and checkout the latest tag**

```bash
$ git clone https://git.giaever.org/joachimmg/acme-sh.git

# Go into directory
$ cd ./acme-sh

# Checkout tag
$ git checkout <tag>
```
_**NOTE**: You can find the latest tag with `git ls-remote --tags origin`_

2. **Build and install**

Make sure you have snapd (see [Installing snapd](https://snapcraft.io/docs/installing-snapd)) and latest version of Snapcraft. Install Snapcraft with

```bash
$ sudo snap install snapcraft --classic
```

Or update with

```bash
$ sudo snap refresh snapcraft
```

2.2 **With multipass**

From the «acme-sh»-directory, run

```bash
$ snapcraft
```

Multipass will be installed and a virtual machine will boot up and build your snap. The final snap will be located in the same directory.

2.3 **With LXD** (*required* for Raspberry Pie)

Snapcraft will try to install multiplass and build the snap for you, but on *Raspberry Pi* it will fail. You will have to use an LXD container.

Install LXD and create a container

```bash
$ snap install lxd
$ snap lxd init
```

Make sure your user is a member of lxd-group

```bash
$ sudo adduser $USER lxd
```

Launch an Ubuntu 20.04 container instance

```bash
$ lxc launch ubuntu:20.04 acme-sh
```

Make sure you're in the «acme-sh»-directory and go into the shell of your newly created container

```bash
$ lxc exec -- acme-sh /bin/bash
```

and run

```bash
$ SNAPCRAFT_BUILD_ENVIRONMENT=host snapcraft
```

when the build is complete, you'll have to exit the shell and pull the snap-file from the container. See `lxc file pull --help`.

3. **Install new built snap**

```
$ sudo snap install ./acme-sh_<source-tag>.snap --dangerous
```
