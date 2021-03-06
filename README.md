![Release](https://img.shields.io/github/release/bmatcuk/terraform-provider-vagrant.svg?branch=master)
[![Build Status](https://travis-ci.com/bmatcuk/terraform-provider-vagrant.svg?branch=master)](https://travis-ci.com/bmatcuk/terraform-provider-vagrant)
[![Go Report Card](https://goreportcard.com/badge/github.com/bmatcuk/terraform-provider-vagrant)](https://goreportcard.com/report/github.com/bmatcuk/terraform-provider-vagrant)

# terraform-provider-vagrant
A Vagrant provider for terraform.

## Installation
Download [the latest release] for your appropriate OS and architecture and
extract the archive. Then copy the binary to [the terraform plugin directory].

## Usage
```hcl
resource "vagrant_vm" "my_vagrant_vm" {
  vagrantfile_dir = "path/to/dir"
  env = {
    KEY = "value",
  }
}
```

**vagrantfile_dir** is the path to a directory where a Vagrantfile lives. The
Vagrantfile must exist when terraform runs or else it will throw an error. This
option defaults to `.`, ie, the current directory and you may set this value to
absolute or relative paths.

**env** is a map of additional environment variables to pass to the Vagrantfile.
The environment variables set by the calling process are always passed.

### Outputs
* `machine_names.#` - a list of machine names as defined in the Vagrantfile.
* `ssh_config.#` - SSH connection info. Since a Vagrantfile may create multiple
  machines, this is a list with the following variables:

  * `ssh_config.*.type` - always "ssh" for now
  * `ssh_config.*.user` - the user for the connection
  * `ssh_config.*.host` - the address to connect to
  * `ssh_config.*.port` - the port to connect to
  * `ssh_config.*.private_key` - private ssh key for the connection
  * `ssh_config.*.agent` - whether or not to use the agent for authentication
    (always "false" for now).

  If there is only one machine built by the Vagrantfile, the connection info
  will be set in the `resource` block so you can include provisioners without
  any additional configuration. However, if there is more than one machine, the
  connection info will not be set; you'll need to create some `null_resources`
  to do your provisioning.

Note that `machine_names` and `ssh_config` are guaranteed to be in the same
order (ie, `ssh_config[0]` is the corresponding config for the machine named
`machine_names[0]`), but the order is undefined (ie, don't count on
`machine_names[0]` being the first machine defined in the Vagrantfile).

## Forcing an Update
The easiest way to force an update is to set, or change the value of, some
environment variable. This will signal to terraform that the vagrant_vm
resource needs to update.

For example, if you want to force updates when your Vagrantfile changes, try
something like this:

```hcl
resource "vagrant_vm" "my_vagrant_vm" {
  vagrantfile_dir = "path/to/dir"
  env = {
    VAGRANTFILE_HASH = "${md5(file("path/to/dir/Vagrantfile"))}",
  }
}
```

When the file changes, the hash will change, and terraform will ask for an
update.

## Removing Machines
Sadly, due to some limitations in vagrant, it's not possible to automatically
remove a portion of machines from a Vagrantfile. In other words, if your
Vagrantfile defines 5 machines and you remove 2 of them from the Vagrantfile,
they will be left running in your vagrant provider (ie, virtualbox or whatever)
with no way of removing them via vagrant (or terraform).

If you intend of removing some machines, you should manually run `vagrant
destroy MACHINE_NAME` on those machines you wish to remove *before* editing the
Vagrantfile. Then update your Vagrantfile and allow terraform to do the rest.

If you forget, you can manually cleanup these old VMs by launching your vagrant
provider's UI and deleting the machines. Then run `vagrant global-status
--prune` to cleanup vagrant's cache of these machines.

## Debugging
If terrafrom is failing on the vagrant step, you can get additional output by
running terraform with logging output enabled. Try something like:

```bash
env TF_LOG=TRACE terraform apply ...
```

And, of course, you can always run vagrant on your Vagrantfile directly.

[the latest release]: https://github.com/bmatcuk/terraform-provider-vagrant/releases/latest
[the terraform plugin directory]: https://www.terraform.io/docs/configuration/providers.html#third-party-plugins
