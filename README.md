# Vaultron

![](https://github.com/brianshumate/vaultron/blob/master/share/vaultron.png?raw=true)

## What?

**Vaultron** uses [Terraform](https://www.terraform.io/) to build a tiny cluster of [Consul](https://www.consul.io/)-backed and highly-available [Vault](https://www.vaultproject.io/) servers for development, evaluation, and issue reproduction on [Docker](https://www.docker.com/).

## Why?

It's a reasonably useful Vault & Consul environment deployed on your macOS or Linux computer _in a about 1 minute_.

Some of the more popular uses of Vaultron are:

- Getting acquainted with Vault
- Evaluating specific Vault features
- Issue reproduction and troubleshooting
- Testing
- **NOT PRODUCTION**

## How?

Terraform assembles individual pieces to form Vaultron from the official [Vault Docker image](https://hub.docker.com/_/vault/) and [Consul Docker image](https://hub.docker.com/_/consul/).

### Quickest Start for macOS

Provided you have previously installed Consul, Terraform, Vault and Docker on your Mac, you can use the following example verbatim to fully bootstrap Vaultron and open the the Vault UI in your browser.

You will likely be prompted for your password to add the Vaultron CA certificate to the System keychain. This will prevent TLS errors about an untrusted CA when using the Consul and Vault web UIs:

```
$ git clone https://github.com/brianshumate/vaultron.git \
&& cd vaultron \
&& ./form \
&& . ./ion_darts \
&& ./blazing_sword \
&& sudo security add-trusted-cert -d -r trustAsRoot \
-k /Library/Keychains/System.keychain ./etc/tls/ca-bundle.pem \
&& open https://localhost:8200
```

**NOTE**: This script persists the unseal keys and initial root authentication token to a file in the `vault` folder named like `./vault/vault_1500766014.tmp`. If this behavior makes you feel some type of way, you're welcome at any time to put Vaultron down and pick up another toy project instead.

### Quick Start for Linux or macOS

Vaultron uses the latest Consul and Vault versions by default, make sure that you have first installed the latest binaries for [Consul](https://releases.hashicorp.com/consul/), [Vault](https://releases.hashicorp.com/vault/), and [Terraform](https://releases.hashicorp.com/terraform/) locally for your OS, and that you have have [Docker](https://docs.docker.com/engine/installation/) installed as well.

After doing so, it takes just 3 steps to form your own Vaultron:

1. `$ git clone https://github.com/brianshumate/vaultron.git`
2. `$ cd vaultron`
3. `$ ./form`

When Vaultron is successfully formed, the output looks like this:

```
[=] Form Vaultron! ...
[=] Terraform has been successfully initialized!
[=] Vault Docker image version:      0.10.1
[=] Consul Docker image version:     1.0.7
[=] Terraform plan: 11 to add, 0 to change, 0 to destroy.
[=] Terraform apply complete! resources: 11 added, 0 changed, 0 destroyed.
[^] Vaultron formed!
```

You are now almost ready interact with `vault` and `consul` CLI utilities or the Vault or Consul HTTP APIs and web user interfaces.

Take a moment to verify that all of the Docker containers are indeed live:

```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                                                                                                                                NAMES
f8b9ca0d83f5        2d9787f3b752        "vault server -log-l…"   23 minutes ago      Up 23 minutes       0.0.0.0:8201->8200/tcp                                                                                                                                                               vault1
b9c037a3469a        2d9787f3b752        "vault server -log-l…"   23 minutes ago      Up 23 minutes       0.0.0.0:8202->8200/tcp                                                                                                                                                               vault2
0055663e5523        2d9787f3b752        "vault server -log-l…"   23 minutes ago      Up 23 minutes       0.0.0.0:8200->8200/tcp                                                                                                                                                               vault0
9b385ea654c7        40ef6c98b292        "consul agent -confi…"   23 minutes ago      Up 23 minutes       8300-8302/tcp, 8500/tcp, 8301-8302/udp, 8600/tcp, 8600/udp                                                                                                                           consulc1
4c6360f8c43a        40ef6c98b292        "consul agent -confi…"   23 minutes ago      Up 23 minutes       8300-8302/tcp, 8500/tcp, 8301-8302/udp, 8600/tcp, 8600/udp                                                                                                                           consulc2
c1e4b4979cb8        40ef6c98b292        "consul agent -confi…"   23 minutes ago      Up 23 minutes       8300-8302/tcp, 8500/tcp, 8301-8302/udp, 8600/tcp, 8600/udp                                                                                                                           consulc0
62f94445902e        40ef6c98b292        "consul agent -serve…"   23 minutes ago      Up 23 minutes       8300-8302/tcp, 8500/tcp, 8301-8302/udp, 8600/tcp, 8600/udp                                                                                                                           consuls2
9343b3204353        40ef6c98b292        "consul agent -serve…"   23 minutes ago      Up 23 minutes       8300-8302/tcp, 8500/tcp, 8301-8302/udp, 8600/tcp, 8600/udp                                                                                                                           consuls1
307a02b668e0        40ef6c98b292        "consul agent -serve…"   23 minutes ago      Up 23 minutes       0.0.0.0:8300-8302->8300-8302/tcp, 0.0.0.0:8500->8500/tcp, 0.0.0.0:8555->8555/tcp, 0.0.0.0:8301-8302->8301-8302/udp, 8600/tcp, 8600/udp, 0.0.0.0:8600->53/tcp, 0.0.0.0:8600->53/udp   consuls0
```

Then, export the necessary environment variables:

```
$ export CONSUL_CACERT="$(pwd)/red_lion/tls/ca-bundle.pem"
$ export CONSUL_HTTP_ADDR="127.0.0.1:8500"
$ export CONSUL_HTTP_SSL=true
$ export CONSUL_HTTP_TOKEN="vaultron-forms-and-eats-all-the-tacos-in-town"
$ export VAULT_ADDR="https://127.0.0.1:8200"
$ export VAULT_CA_CERT="$(pwd)/black_lion/tls/ca-bundle.pem"
```

Note the completion message about setting important environment variables before executing the `vault` and `consul` CLI commands. You'll want these environment variables in your shell before trying to use the CLI tools with Vaultron.

You can instead source the `ion_darts` script to do this for you:

```
$ . ./ion_darts
[=] Exporting Vaultron environment variables ...
[^] Exported Vaultron environment variables!
```

Before accessing the Vault or Consul web UIs you should add the Vaultron CA cert to your OS trust store.

It is located at `etc/tls/ca-bundle.pem` from the root of this project.

See the TLS by Default section for more details on handling Vaultron's Intermediate Certificate Authority certificate.

### What's Next?

If you are new to Vault, then using Vaultron is a nice way to get quickly acquainted! Please begin by checking out the official [Vault Getting Started documentation](https://www.vaultproject.io/intro/getting-started/install.html).

You can follow along from the [Your First Secret](https://www.vaultproject.io/intro/getting-started/first-secret.html) page onwards after initializing, unsealing, and authenticating with the root token.

Speaking of which, here are some things you can do after Vaultron is formed:

1. Initialize Vault with `vault init`
2. Unseal Vault with `vault unseal` using 3 of the 5 unseal keys presented when you initialized Vault
3. Authenticate to Vault with the initial root token presented during initialization
4. Use your local `consul` and `vault` binaries in CLI mode to interact with Vault servers
5. Use the Consul web UI at [https://localhost:8500](https://localhost:8500)
6. Use the [Vault HTTP API](https://www.vaultproject.io/api/index.html)
7. When done having fun, disassemble Vaultron and clean up with `./unform`

**NOTE: `./unform` REMOVES ALMOST EVERYTHING including the existing Vault data, logs, and Terraform state — be careful!**

The Terraform provider modules _are not removed_ to save on resources and time involved in re-downloading them.

If you want to tear down the containers, but preserve data, logs, and state, you can use `terraform destroy` for that instead:

```
$ terraform destroy -state=./tfstate/terraform.tfstate
```

If you are already familiar with Vault, but would like to save time by rapidly initializing, unsealing, and enabling a wide range of authentication and secret backends, execute the `./blazing_sword` script to do all of this for you.

If you are familiar with Terraform you can also use Terraform commands instead, but you'll need to manually specify the `CONSUL_HTTP_ADDR` and `VAULT_ADDR` environment variables before you can access either the Consul or Vault instances:

```
$ export CONSUL_HTTP_ADDR="127.0.0.1:8500"
$ export CONSUL_HTTP_SSL=true
$ export VAULT_ADDR="https://127.0.0.1:8200"
$ export CONSUL_HTTP_TOKEN="vaultron-forms-and-eats-all-the-tacos-in-town"
```

## What's in the Box?

Vaultron technical specifications quick reference card:

```
Name:          Vaultron
Type:          Secret Management Unit V (defaults to latest Vault software)
Builder:       Terraform
Blueprints:    vaultron.tf
Modules:       black_lion, red_lion, yellow_lion
Datacenter:    arus
Infra-cell:    Distributed storage cell (defaults to latest Consul software)
Universe:      Docker
Telemetry:     statsd, Graphite, Grafana (optional)
HashiStack:    ★★★
Agility:       ★★★★
Damage:        ★★
Mass:          ★★
Speed:         ★★★★★
```

Here are some slightly more serious notes and questions about what Vaultron is and how it can work for you.

### Basic Architecture Overview

Vaultron has to work around some current networking quirks of Docker for Mac to do its thing and is only currently tested to function on Linux and macOS, but here is basically what you are getting by default:

```
+-----------+-----------------------------------+------------------------+
| __     __ |   Yellow Lion (optional)          |                        |
| \ \   / / |   +----------------------------+  |                        |
|  \ \ / /  |   |     Grafana Dashboard      |  |                        |
|   \ V /   |   +----------------------------+  |                        |
|    \_/    |   |     statsd / Graphite      |  |                        |
|           |   +-^------------^------------^+  |                        |
|           +-----------------------------------+                        |
|                 |            |            |                            |
+------------------------------------------------------------------------+
|                 |            |            |                 Black Lion |
|  +--------------+-   --------+--------   -+---------------             |
|  |               |   |               |   |               |             |
|  |     vault0    |   |     vault1    |   |     vault2    |             |
|  |               |   |               |   |               |             |
|  +-------+-^-----+   +-------+-^-----+   +-------+-^-----+             |
|          | |                 | |                 | |                   |
+------------------------------------------------------------------------+
|          | |                 | |                 | |        Red Lion   |
|  +-------v-+-----+   +-------v-+-----+   +-------v-+-----+             |
|  |               |   |               |   |               |             |
|  | Consul Client |   | Consul Client |   | Consul Client |             |
|  |   (consulc0)  |   |   (consulc1)  |   |   (consulc2)  |             |
|  |               |   |               |   |               |             |
|  +-------+-^-----+   +-------+-^-----+   +-------+-^-----+             |
|          | |                 | |                 | |                   |
|          | |                 | |                 | |                   |
|          | |                 | |                 | |                   |
|  +-------v-+-----+   +-------v-+-----+   +-------v-+-----+             |
|  |               |   |               |   |               |             |
|  | Consul Server +---> Consul Server +---> Consul Server |             |
|  |   (consuls0)  <---+   (consuls1)  <---+   (consuls2)  |             |
|  |               |   |               |   |               |             |
|  +---------------+   +---------------+   +---------------+             |
+------------------------------------------------------------------------+
```

Vaultron consists of 3 Vault server containers, 3 Consul client containers, and 3 Consul server containers.

An optional telemetry gathering and graphing stack (Yellow Lion) can be enabled at runtime via environment variable; see the **Telemetry Notes** section for more details.

Vault servers connect directly to the Consul clients, which in turn connect to the Consul server cluster. In this configuration, Vault is using Consul for both storage and high availability functionality.

### Published Ports

Each Vault instance is available to the local computer, but through Docker's published ports scheme only, so the addresses of the Vault servers are:

- localhost:8200
- localhost:8210
- localhost:8220

### Changing Vault and Consul Versions

Vaultron runs the `:latest` official Vault Docker container image, but if you would prefer to run a different version, you can export the `TF_VAR_vault_version` environment variable to override:

```
$ export TF_VAR_vault_version=0.6.5
$ ./form
$ ./blazing_sword
...
Version: 0.6.5
...
```

Similarly, to run a different version of the Consul container, set the `TF_VAR_consul_version` environment variable like this:

```
$ export TF_VAR_consul_version=0.7.5

$ ./form

$ . ./ion_darts
[=] Exporting Vaultron environment variables ...
[^] Exported Vaultron environment variables!

$ consul members
Node                 Address          Status  Type    Build  Protocol  DC
consulc0  172.17.0.6:8301  alive   client  0.7.5  2         arus
consulc1  172.17.0.7:8301  alive   client  0.7.5  2         arus
consulc2  172.17.0.5:8301  alive   client  0.7.5  2         arus
consuls0  172.17.0.2:8301  alive   server  0.7.5  2         arus
consuls1  172.17.0.3:8301  alive   server  0.7.5  2         arus
consuls2  172.17.0.4:8301  alive   server  0.7.5  2         arus
```

Be sure to always use the same versions of Consul and Vault for both the CLI binaries on your host system and the container image.

Also note that if the version of Consul or Vault you want to use does not have an official Docker image, you'll encounter an error.

### Consul DNS

The 3 Consul servers have DNS exposed to port 53 of their internal container addresses, and the Consul clients and Vault sever containers are configured to use those Consul servers for DNS as well.

Additionally Consul DNS API is also published from the first Consul server at `localhost:8600`, so you can query services and nodes using DNS like so:

```
$ dig -p 8600 @localhost consul.service.consul
...
;; ANSWER SECTION:
consul.service.consul.  0 IN  A 172.17.0.3
consul.service.consul.  0 IN  A 172.17.0.2
consul.service.consul.  0 IN  A 172.17.0.4
```

or

```
$ dig -p 8600 @localhost active.vault.service.consul
;; ANSWER SECTION:
active.vault.service.consul. 0  IN  A 172.17.0.5
```

or

```
$ dig -p 8600 @localhost vault.service.consul SRV
...
;; ANSWER SECTION:
vault.service.consul. 0 IN  SRV 1 1 8200 consulc0.node.arus.consul.
vault.service.consul. 0 IN  SRV 1 1 8200 consulc2.node.arus.consul.
vault.service.consul. 0 IN  SRV 1 1 8200 consulc1.node.arus.consul.
```

or

```
$ dig -p 8600 @localhost consuls0.node.consul
...
;; ANSWER SECTION:
consuls0.node.consul. 0 IN A 172.17.0.2
```

### Security Configuration?

Given the intended use cases for this project, the working solution that results when Vaultron is formed is essentially a blank canvas that emphasizes immediate unhindered usability over security.

#### Consul ACLs by Default

> **Consul ACLs with a **default allow policy** are enabled for Vaultron v1.8.0 (using Vault v0.9.5/Consul v1.0.6) and beyond**

This was chosen to allow for ease of experimentation with ACL policies and the Vault Consul Secrets Engine. It is not the same as a production installation because it makes use of a shared **acl_master_token** for ease of configuration.

The value used for the shared ACL Master Token is:

- `vaultron-forms-and-eats-all-the-tacos-in-town`

#### TLS by Default

Vaultron uses self-signed certificates for full mutual TLS communication between Vault servers and Consul agents. The certificates and keys were generated from Vault PKI Secrets Backends as described in [examples/tls/README.md](https://github.com/brianshumate/vaultron/blob/master/examples/tls/README.md).

With this in mind, you need to ensure that the certificate authority is recognized by Vault and Consul; you can do this in a number of ways:

1. Import the `black_lion/tls/ca-pundle.pem` into your OS trust store
2. For Vault: use the `VAULT_CACERT` environment variable
3. For Vault: Pass `-ca-cert=` option with path to the `ca-pundle.pem` for all `vault` commands
4. For Consul: use the `CONSUL_CACERT` environment variable
5. For Consul: pass the `-ca-file=` option with path to the `ca-pundle.pem` for all `consul` commands

Here are some additional resources related to configuring ACLs and TLS:

- [Consul ACL System guide](https://www.consul.io/docs/guides/acl.html)
- [Consul Encryption documentation](https://www.consul.io/docs/agent/encryption.html)
- [Vault TCP Listener documentation](https://www.vaultproject.io/docs/configuration/listener/tcp.html)

### Where's My Vault Data?

Vault data are stored in Consul's key/value store, which in turn is written into the `consul/oss_server_*/data` directories for each of the three Consul servers.

Here is a tree showing the folder structure for a Consul server:

```
└── consul
    └── consuls0
        ├── config
        │   └── extra_config.hcl
        └── data
            ├── checkpoint-signature
            ├── node-id
            ├── raft
            │   ├── peers.info
            │   ├── raft.db
            │   └── snapshots
            └── serf
                ├── local.snapshot
                └── remote.snapshot
```

### What Are The Default TTL Values?

Vaultron tries to be reasonable in accommodating developer use cases, but also wants to keep cruft to a minimum. To that end, the default TTL value is lowered, and the maximum TTL value is raised to these values:

- `default_lease_ttl`: **50000h** (2083 days)
- `max_lease_ttl`: **50000h** (2083 days)

That's 7 days and 999 days respectively.

### What About Logs?

The Docker containers are named as shown in the [Basic Architecture Overview](#basic-architecture-overview).

You can view operational logs for any container with `docker logs` like so:

```
docker logs vault0
```

The Vault audit logs for any given _active server_ are available as:

- `./vault/vault0/audit_log/audit.log`
- `./vault/vault1/audit_log/audit.log`
- `./vault/vault2/audit_log/audit.log`

### Telemetry Notes

Vaultron includes a comprehensive telemetry gathering and graphing stack provided by the Yellow Lion module. This module is optional and easily enabled. It provides statsd, Graphite, and Grafana from the addition of two official container images.

You can enable Yellow Lion by setting the value of the *TF_VAR_vaultron_telemetry_count* environment variable to **1**, with something like:

```
$ export TF_VAR_vaultron_telemetry_count=1
```

prior to the execution of `./form`.

See the [Visualizing Vault Telemetry](https://github.com/brianshumate/vaultron/blob/master/examples/telemetry/README.md) documentation for more details on setup.

### A note about custom Binaries

Vaultron installs the official open source Vault binaries through the official Docker container images, but if you'd prefer to use recent source builds or some other Vault binary, just drop `vault` into `custom/` and set these environment variables prior to forming Vaultron:

```
export TF_VAR_vault_oss_instance_count=0 \
       TF_VAR_vault_custom_instance_count=3
```

## Basic Troubleshooting Questions

### I can access the Consul UI but it states that there are no services to show

Access the settings (gear icon) in the navigation and ensure that the ACL master token value "vaultron-forms-and-eats-all-the-tacos-in-town" is present in the text field, then click **Close**.

### Vault is Orange/Failing in the Consul Web UI

Vault is expected to appear as failing in the Consul UI if you have not yet unsealed it.

Unsealing Vault should solve that for you!

### Something, Something — HA Problem!

High Availability mode has been shown to work as expected, however because of the current published ports method for exposing the Vault servers, you must be sure to point your client to the correct Vault server with `VAULT_ADDR` once that server becomes the new active server.

Here is simple method to watch HA mode in action using two terminal sessions:

```
Terminal 1                              Terminal 2
+-----------------------------------+   +------------------------------------+
| VAULT_ADDR=https://localhost:8210\|   | docker stop vault0                 |
| watch -n 1 vault status           |   |                                    |
|                                   |   |                                    |
| ...                               |   |                                    |
| HA Enabled             true       |   |                                    |
| HA Cluster             https://...|   |                                    |
| HA Mode                standby    |   |                                    |
| ...                               |   |                                    |
|                                   |   |                                    |
|                                   |   |                                    |
+-----------------------------------+   +------------------------------------+
```

1. In Terminal 1, set `VAULT_ADDR` to one of the two Vault standby containers and use `watch` to keep an eye on the output of `vault status` while noting the values of `Mode:` and `Leader:`
2. In Terminal 2, stop the *active* Vault instance with `docker stop`
3. You should notice that the value of `Leader:` changes instantly and if the second standby Vault is elected the new active, the value of `Mode:` will also reflect that instantly as well

### Vaultron Does Not Form — Halp!

Instead of seeing the glorious interlocks activated, dyna-therms connected, infra-cells up, and mega-thrusters going, Vaultron fails to form and I get:

```
[e] Vaultron cannot form! Check terraform apply output.
```

or this:

```
[e] Vaultron cannot form! Check terraform plan output.
```

This means that Vaultron had problems durring the `terraform plan` or `terraform apply` steps. You can run those commands manually and inspect their output to troubleshoot the issue.

Other red and equally frightening errors could occur, and these are usually accompanied by an explanation from Terraform regarding the nature of the problem.

### Unsupported Versions?

If you try exporting `TF_VAR_consul_version` or `TF_VAR_vault_version` to a specific version, but get this error when you attempt to form Vaultron:

```
[e] Sorry, Vaultron does not support Consul version: 0.6.4
```

or:

```
[e] Sorry, Vaultron does not support Vault version: 0.6.0
```

You are specifying either a non-existent version (maybe a typo?) or you are specifying a version for which no Docker images exists. This second case is not a problem with Vaultron, there are some versions of Consul and Vault which were released as binaries, but not available as Docker images.

## Resources

Here are some links to resources for the technologies used in this project:

1. [Vault Docker Hub repository](https://hub.docker.com/_/vault/)
2. [hashicorp/docker-consul](https://github.com/hashicorp/docker-consul)
3. [Consul Docker Hub repository](https://hub.docker.com/_/consul/)
4. [hashicorp/docker-vault](https://github.com/hashicorp/docker-vault)
5. [Consul ACL System guide](https://www.consul.io/docs/guides/acl.html)
6. [Consul Encryption documentation](https://www.consul.io/docs/agent/encryption.html)
7. [Official Consul Docker Image blog post](https://www.hashicorp.com/blog/official-consul-docker-image/)
8. [Terraform](https://www.terraform.io/)
9. [Consul](https://www.consul.io/)
10. [Vault](https://www.vaultproject.io/)
11. [Vault TCP Listener documentation](https://www.vaultproject.io/docs/configuration/listener/tcp.html)
12. [Docker](https://www.docker.com/)

## Who?

Vaultron was created by [Brian Shumate](https://github.com/brianshumate) and made possible through the generous time of the good people named in [CONTRIBUTORS.md](https://github.com/brianshumate/vaultron/blob/master/CONTRIBUTORS.md)

## Special Thanks

:robot: [Voltron Legendary Defender Theme Song Acapella](https://www.youtube.com/embed/W_yr9KvldZY)
