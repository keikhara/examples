# How to use WordPress on DC/OS

[WordPress](https://wordpress.com/) is a popular blogging platform.

- Estimated time for completion: 10 minutes
- Target audience: Anyone interested using a blog.
- Scope: Learn how to use WordPress on DC/OS.

This is a modified version of: [How to use WordPress on DC/OS](https://github.com/dcos/examples/tree/master/1.8/wordpress).

**Table of Contents**:

- [Prerequisites](#prerequisites)
- [Install MySQL](#install-mysql)
- [Install WordPress](#install-wordpress)
- [Access WordPress](#access-wordpress)
- [Use WordPress in production](#use-wordpress-in-production)

## Prerequisites

- A running DC/OS 1.8 cluster with at least 1 [public agent](https://dcos.io/docs/1.8/overview/concepts/#public) node with 2 CPUs and 1 GB of RAM available.
- [DC/OS CLI](https://dcos.io/docs/1.8/usage/cli/install/) installed.

## Install MySQL

Next, we install and set up MySQL via the DC/OS CLI. For this, create a file called `mysql-config.json` and set the following properties (**replace `mesosphere` in the `name` field with your own name!**):

```json
{
  "service": {
    "name": "mysql-mesosphere"
  },
  "database": {
    "name": "wordpress",
    "username": "wordpress",
    "password": "password",
    "root_password": "wordpress"
  },
  "networking": {
    "port": 3306,
    "host_mode": true
  }
}
```

Then, install MySQL:

```bash
$ dcos package install mysql --options=mysql-config.json
This DC/OS Service is currently EXPERIMENTAL. There may be bugs, incomplete features, incorrect documentation, or other discrepancies.

Advanced Installation options notes

storage / *persistence*: create local persistent volumes for internal storage files to survive across restarts or failures.

storage / persistence / *external*: create external persistent volumes. This allows to use an external storage system such as Amazon EBS, OpenStack Cinder, EMC Isilon, EMC ScaleIO, EMC XtremIO, EMC VMAX and Google Compute Engine persistent storage. *NOTE*: To use external volumes with DC/OS, you MUST enable them during CLI or Advanced installation.

storage / *host_volume*:  if persistence is not selected, this package can use a local volume in the host for storage, like a local directory or an NFS mount. The parameter *host_volume* controls the path in the host in which these volumes will be created, which MUST be the same on all nodes of the cluster.

NOTE: If you didn't select persistence in the storage section, or provided a valid value for *host_volume* on installation,
YOUR DATA WILL NOT BE SAVED IN ANY WAY.

networking / *port*: This DC/OS service can be accessed from any other application through a NAMED VIP in the format *`service_name.marathon.l4lb.thisdcos.directory:port`*. Check status of the VIP in the *Network* tab of the DC/OS Dashboard (Enterprise DC/OS only).

networking / *external_access*: create an entry in Marathon-LB for accessing the service from outside of the cluster

networking / *external_access_port*: port to be used in Marathon-LB for accessing the service.
Continue installing? [yes/no] yes
Installing Marathon app for package [mysql] version [5.7.12-0.3]
Service installed.

Default login: `admin`/`password`.
```

## Install WordPress

To make WordPress accessible to the world, we need to use the fully qualified domain name of your DC/OS public agent. To install from the CLI, first create a `wordpress-config.json` as below and replace the `virtual-host` property.

We have created 6 DNS names per `dcosXX` cluster. Please use your position away from the nearest wall to select a hostname for to use for the `virtual-host` property. For example, for `dcos01`:

| User | Wordpress DC/OS Hostname |
| --- | --- | --- |
| 1 | https://wp01.public.dcos01.qcon.mesosphere.com |
| 2 | https://wp02.public.dcos01.qcon.mesosphere.com |
| 3 | https://wp03.public.dcos01.qcon.mesosphere.com |
| 4 | https://wp04.public.dcos01.qcon.mesosphere.com |
| 5 | https://wp05.public.dcos01.qcon.mesosphere.com |
| 6 | https://wp06.public.dcos01.qcon.mesosphere.com |

 i.e. `$PUBLIC_AGENT`, with your own (for example `ec2-52-51-80-141.eu-west-1.compute.amazonaws.com` - be sure to remove the `https://`):

```json
{
  "database": {
    "host": "mysql-mesosphere.marathon.mesos:3306",
    "name": "wordpress",
    "user": "wordpress",
    "password": "password"
  },
  "networking": {
    "virtual-host": "$PUBLIC_AGENT"
  }
}
```

Now use above options file to install WordPress:

```bash
$ dcos package install wordpress --options=wordpress-config.json
Installing Marathon app for package [wordpress] version [1.0.0-4.5.3-apache]
WordPress has been installed.
```

Last but not least, to check if all required services (Marathon-LB, MySQL and WordPress itself) are running, use the DC/OS UI where in the `Services` tab you should see the following:

![Services](img/services.png)

## Access WordPress

Once you've installed the necessary services as outlined above, navigate to the domain name of your DC/OS public agent in your browser. You should now see the WordPress welcome wizard:

![WordPress welcome wizard](img/wordpress-welcome.png)
