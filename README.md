#rabbitmq

####Table of Contents

1. [Overview](#overview)
2. [Module Description - What the module does and why it is useful](#module-description)
3. [Setup - The basics of getting started with rabbitmq](#setup)
    * [What rabbitmq affects](#what-rabbitmq-affects)
    * [Setup requirements](#setup-requirements)
    * [Beginning with rabbitmq](#beginning-with-rabbitmq)
4. [Usage - Configuration options and additional functionality](#usage)
5. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
5. [Limitations - OS compatibility, etc.](#limitations)
   * [RedHat module dependencies](#redhat-module-dependecies)
6. [Development - Guide for contributing to the module](#development)

##Overview

This module manages RabbitMQ (www.rabbitmq.com)

##Module Description
The rabbitmq module sets up rabbitmq and has a number of providers to manage
everything from vhosts to exchanges after setup.

This module has been tested against 2.7.1 and is known to not support
all features against earlier versions.

##Setup

###What rabbitmq affects

* rabbitmq repository files.
* rabbitmq package.
* rabbitmq configuration file.
* rabbitmq service.

###Beginning with rabbitmq


```puppet
include '::rabbitmq'
```

##Usage

All options and configuration can be done through interacting with the parameters
on the main rabbitmq class.  These are documented below.

##rabbitmq class

To begin with the rabbitmq class controls the installation of rabbitmq.  In here
you can control many parameters relating to the package and service, such as
disabling puppet support of the service:

```puppet
class { '::rabbitmq':
  service_manage    => false,
  port              => '5672',
  delete_guest_user => true,
}
```

Or such as offline installation from intranet or local mirrors:

```puppet
class { '::rabbitmq':
   key_content      => template('openstack/rabbit.pub.key'),
   package_gpg_key  => '/tmp/rabbit.pub.key',
}
```

And this one will use external package key source for any (apt/rpm) package provider:

```puppet
class { '::rabbitmq':
   package_gpg_key  => 'http://www.some_site.some_domain/some_key.pub.key',
}
```

### Environment Variables
To use RabbitMQ Environment Variables, use the parameters `environment_variables` e.g.:

```puppet
class { 'rabbitmq':
  port              => '5672',
  environment_variables   => {
    'NODENAME'     => 'node01',
    'SERVICENAME'  => 'RabbitMQ'
  }
}
```

### Variables Configurable in rabbitmq.config
To change RabbitMQ Config Variables in rabbitmq.config, use the parameters `config_variables` e.g.:

```puppet
class { 'rabbitmq':
  port              => '5672',
  config_variables   => {
    'hipe_compile'  => true,
    'frame_max'     => 131072,
    'log_levels'    => "[{connection, info}]"
  }
}
```

To change Erlang Kernel Config Variables in rabbitmq.config, use the parameters
`config_kernel_variables` e.g.:

```puppet
class { 'rabbitmq':
  port              => '5672',
  config_kernel_variables  => {
    'inet_dist_listen_min' => 9100,
    'inet_dist_listen_max' => 9105,
  }
}
```

### Clustering
To use RabbitMQ clustering facilities, use the rabbitmq parameters
`config_cluster`, `cluster_nodes`, and `cluster_node_type`, e.g.:

```puppet
class { 'rabbitmq':
  config_cluster           => true,
  cluster_nodes            => ['rabbit1', 'rabbit2'],
  cluster_node_type        => 'ram',
  erlang_cookie            => 'A_SECRET_COOKIE_STRING',
  wipe_db_on_cookie_change => true,
}
```

##Reference

##Classes

* rabbitmq: Main class for installation and service management.
* rabbitmq::config: Main class for rabbitmq configuration/management.
* rabbitmq::install: Handles package installation.
* rabbitmq::params: Different configuration data for different systems.
* rabbitmq::service: Handles the rabbitmq service.
* rabbitmq::repo::apt: Handles apt repo for Debian systems.
* rabbitmq::repo::rhel: Handles rpm repo for Redhat systems.

###Parameters

####`admin_enable`

Boolean, if enabled sets up the management interface/plugin for RabbitMQ.

####`cluster_node_type`

Choose between disk and ram nodes.

####`cluster_nodes`

An array of nodes for clustering.

####`cluster_partition_handling`

Value to set for `cluster_partition_handling` RabbitMQ configuration variable.

####`config`

The file to use as the rabbitmq.config template.

####`config_cluster`

Boolean to enable or disable clustering support.

####`config_kernel_variables`

Hash of Erlang kernel configuration variables to set (see [Variables Configurable in rabbitmq.config](#variables-configurable-in-rabbitmq.config)).

####`config_mirrored_queues`

DEPRECATED

Configuring queue mirroring should be done by setting the according policy for
the queue. You can read more about it
[here](http://www.rabbitmq.com/ha.html#genesis)

####`config_path`

The path to write the RabbitMQ configuration file to.

####`config_stomp`

Boolean to enable or disable stomp.

####`config_variables`

To set config variables in rabbitmq.config

####`default_user`

Username to set for the `default_user` in rabbitmq.config.

####`default_pass`

Password to set for the `default_user` in rabbitmq.config.

####`delete_guest_user`

Boolean to decide if we should delete the default guest user.

####`env_config`

The template file to use for rabbitmq_env.config.

####`env_config_path`

The path to write the rabbitmq_env.config file to.

####`environment_variables`

RabbitMQ Environment Variables in rabbitmq_env.config

####`erlang_cookie`

The erlang cookie to use for clustering - must be the same between all nodes.
This value has no default and must be set explicitly if using clustering.

####`file_limit`

Set rabbitmq file ulimit. Defaults to 16384. Only available on systems with
`$::osfamily == 'Debian'` or `$::osfamily == 'RedHat'`.

####`key_content`

Uses content method for Debian OS family. Should be a template for apt::source
class. Overrides `package_gpg_key` behavior, if enabled. Undefined by default.

####`ldap_auth`

Boolean, set to true to enable LDAP auth.

####`ldap_server`

LDAP server to use for auth.

####`ldap_user_dn_pattern`

User DN pattern for LDAP auth.

####`ldap_other_bind`

How to bind to the LDAP server. Defaults to 'anon'.

####`ldap_config_variables`

Hash of other LDAP config variables.

####`ldap_use_ssl`

Boolean, set to true to use SSL for the LDAP server.

####`ldap_port`

Numeric port for LDAP server.

####`ldap_log`

Boolean, set to true to log LDAP auth.

####`manage_repos`

Boolean, whether or not to manage package repositories.

####`management_port`

The port for the RabbitMQ management interface.

####`node_ip_address`

The value of NODE_IP_ADDRESS in rabbitmq_env.config

####`package_ensure`

Determines the ensure state of the package.  Set to installed by default, but could
be changed to latest.

####`package_gpg_key`

RPM package GPG key to import. Uses source method. Should be a URL for Debian/RedHat
OS family, or a file name for RedHat OS family.
Set to http://www.rabbitmq.com/rabbitmq-signing-key-public.asc by default.
Note, that `key_content`, if specified, would override this parameter for Debian OS family.

####`package_name`

The name of the package to install.

####`package_provider`

What provider to use to install the package.

####`package_source`

Where should the package be installed from?

On Debian- and Arch-based systems using the default package provider,
this parameter is ignored and the package is installed from the
rabbitmq repository, if enabled with manage_repo => true, or from the
system repository otherwise. If you want to use dpkg as the
package_provider, you must specify a local package_source.

####`plugin_dir`

Location of RabbitMQ plugins.

####`port`

The RabbitMQ port.

####`service_ensure`

The state of the service.

####`service_manage`

Determines if the service is managed.

####`service_name`

The name of the service to manage.

####`ssl`

Configures the service for using SSL.

####`ssl_only`

Configures the service to only use SSL.  No cleartext TCP listeners will be created.
Requires that ssl => true and port => UNSET also

####`ssl_cacert`

CA cert path to use for SSL.

####`ssl_cert`

Cert to use for SSL.

####`ssl_key`

Key to use for SSL.

####`ssl_management_port`

SSL management port.

####`ssl_stomp_port`

SSL stomp port.

####`ssl_verify`

rabbitmq.config SSL verify setting.

####`ssl_fail_if_no_peer_cert`

rabbitmq.config `fail_if_no_peer_cert` setting.

####`ssl_versions`

Choose which SSL versions to enable. Example: `['tlsv1.2', 'tlsv1.1']`.

Note that it is recommended to disable `sslv3` and `tlsv1` to prevent against POODLE and BEAST attacks. Please see the [RabbitMQ SSL](https://www.rabbitmq.com/ssl.html) documentation for more information.

####`ssl_ciphers`

Support only a given list of SSL ciphers. Example: `['dhe_rsa,aes_256_cbc,sha','dhe_dss,aes_256_cbc,sha','ecdhe_rsa,aes_256_cbc,sha']`.

Supported ciphers in your install can be listed with:
 rabbitmqctl eval 'ssl:cipher_suites().'
Functionality can be tested with cipherscan or similar tool: https://github.com/jvehent/cipherscan.git

####`stomp_port`

The port to use for Stomp.

####`stomp_ensure`

Boolean to install the stomp plugin.

####`tcp_keepalive`

Boolean to enable TCP connection keepalive for RabbitMQ service.

####`version`

Sets the version to install.

On Debian- and Arch-based operating systems, the version parameter is
ignored and the latest version is installed from the rabbitmq
repository, if enabled with manage_repo => true, or from the system
repository otherwise.

####`wipe_db_on_cookie_change`

Boolean to determine if we should DESTROY AND DELETE the RabbitMQ database.

####`rabbitmq_user`

String: OS dependent, default defined in param.pp. The system user the rabbitmq daemon runs as.

####`rabbitmq_group`

String: OS dependent, default defined in param.pp. The system group the rabbitmq daemon runs as.

####`rabbitmq_home`

String: OS dependent. default defined in param.pp. The home directory of the rabbitmq deamon.

##Native Types

### rabbitmq\_user

query all current users: `$ puppet resource rabbitmq_user`

```
rabbitmq_user { 'dan':
  admin    => true,
  password => 'bar',
}
```
Optional parameter tags will set further rabbitmq tags like monitoring, policymaker, etc.
To set the administrator tag use admin-flag.
```puppet
rabbitmq_user { 'dan':
  admin    => true,
  password => 'bar',
  tags     => ['monitoring', 'tag1'],
}
```


### rabbitmq\_vhost

query all current vhosts: `$ puppet resource rabbitmq_vhost`

```puppet
rabbitmq_vhost { 'myhost':
  ensure => present,
}
```

### rabbitmq\_exchange

```puppet
rabbitmq_exchange { 'myexchange@myhost':
  user     => 'dan',
  password => 'bar',
  type     => 'topic',
  ensure   => present,
  internal => false,
  auto_delete => false,
  durable => true,
  arguments => {
    hash-header => 'message-distribution-hash'
  }
}
```

### rabbitmq\_queue

```puppet
rabbitmq_queue { 'myqueue@myhost':
  user        => 'dan',
  password    => 'bar',
  durable     => true,
  auto_delete => false,
  arguments   => {
    x-message-ttl => 123,
    x-dead-letter-exchange => 'other'
  },
  ensure      => present,
}
```

### rabbitmq\_binding

```puppet
rabbitmq_binding { 'myexchange@myqueue@myhost':
  user             => 'dan',
  password         => 'bar',
  destination_type => 'queue',
  routing_key      => '#',
  arguments        => {},
  ensure           => present,
}
```

### rabbitmq\_user\_permissions

```puppet
rabbitmq_user_permissions { 'dan@myhost':
  configure_permission => '.*',
  read_permission      => '.*',
  write_permission     => '.*',
}
```

### rabbitmq\_policy

```puppet
rabbitmq_policy { 'ha-all@myhost':
  pattern    => '.*',
  priority   => 0,
  applyto    => 'all',
  definition => {
    'ha-mode'      => 'all',
    'ha-sync-mode' => 'automatic',
  },
}
```

### rabbitmq\_plugin

query all currently enabled plugins `$ puppet resource rabbitmq_plugin`

```puppet
rabbitmq_plugin {'rabbitmq_stomp':
  ensure => present,
}
```

### rabbitmq\_erlang\_cookie

This is essentially a private type used by the rabbitmq::config class
to manage the erlang cookie. It replaces the rabbitmq_erlang_cookie fact
from earlier versions of this module. It manages the content of the cookie
usually located at "${rabbitmq_home}/.erlang.cookie", which includes
stopping the rabbitmq service and wiping out the database at
"${rabbitmq_home}/mnesia" if the user agrees to it. We don't recommend using
this type directly.

##Limitations

This module has been built on and tested against Puppet 3.x.

The module has been tested on:

* RedHat Enterprise Linux 5/6
* Debian 6/7
* CentOS 5/6
* Ubuntu 12.04/14.04

Testing on other platforms has been light and cannot be guaranteed.

### Module dependencies

If running CentOS/RHEL, and using the yum provider, ensure the epel repo is present.

To have a suitable erlang version installed on RedHat and Debian systems,
you have to install another puppet module from http://forge.puppetlabs.com/garethr/erlang with:

    puppet module install garethr-erlang

This module handles the packages for erlang.
To use the module, add the following snippet to your site.pp or an appropriate profile class:

For RedHat systems:

    include 'erlang'
    class { 'erlang': epel_enable => true}

For Debian systems:

    include 'erlang'
    package { 'erlang-base':
      ensure => 'latest',
    }

This module also depends on the excellent nanliu/staging module on the Forge:

    puppet module install nanliu-staging

### Downgrade Issues

Be advised that there were configuration file syntax and other changes made between RabbitMQ
versions 2 and 3. In order to downgrade from 3 to 2 (not that this is a terribly good idea)
you will need to manually remove all RabbitMQ configuration files (``/etc/rabbitmq``) and
the mnesia directory (usually ``/var/lib/rabbitmq/mnesia``). The latter action will delete
any and all messages stored to disk.

Failure to do this will result in RabbitMQ failing to start with a cryptic error message about
"init terminating in do_boot", containing "rabbit_upgrade,maybe_upgrade_mnesia".

##Development

Puppet Labs modules on the Puppet Forge are open projects, and community
contributions are essential for keeping them great. We can???t access the
huge number of platforms and myriad of hardware, software, and deployment
configurations that Puppet is intended to serve.

We want to keep it as easy as possible to contribute changes so that our
modules work in your environment. There are a few guidelines that we need
contributors to follow so that we can have a chance of keeping on top of things.

You can read the complete module contribution guide [on the Puppet Labs wiki.](http://projects.puppetlabs.com/projects/module-site/wiki/Module_contributing)

### Authors
* Jeff McCune <jeff@puppetlabs.com>
* Dan Bode <dan@puppetlabs.com>
* RPM/RHEL packages by Vincent Janelle <randomfrequency@gmail.com>
* Puppetlabs Module Team
