vmh(1) -- Virtual Machine Handler, Virsh CLI wrapper with extra features
=============================================

## SYNOPSIS

'vmh (virtualmachinehandler)': Advanced actions for libvirt domains
`vmh <ACTION> <DOMAIN|DOMAIN_DESTINATION|DISK_NAME|ENVIRONMENT> [DOMAIN]`

## DESCRIPTION

Virsh CLI wrapper with extra features.

* Mass and parallel deployment of domains
* Automatic deployment of virtual networks
* Create/compress/import/export images that use disk chains
* More verifications and better error handeling than plain virsh
* More automation, e.g. auto shutdown and then force shutdown after a timeout
* Wait for SSH to become available
* Logging system (rsyslog)
* Useful for automating kiosk applications that use VMs

## OPTIONS

* `-h`, `--help` :  
  Display this help screen.

## CONFIG

Config file is in /etc/vmh.conf  
Open the default config file for more information.

## ACTION

Use the `<action>` variable to manipulate domains in different ways.

* `chain <DOMAIN_DESTINATION> <DOMAIN>` : 'shutdown', create new empty disk,
  appends disk to chain and renames the domain to DOMAIN_DESTINATION.
  DOMAIN_DESTINATION is also used to name the disk files. This is helpful to
  see what images belong to what domain.
* `chain-start <DOMAIN_DESTINATION> <DOMAIN>` : Same as 'chain' and 'start'.
* `chain-start-wait <DOMAIN_DESTINATION> <DOMAIN>` : Same as 'chain', 'start'
  and 'wait'.
* `chain-info <DOMAIN>` : Show the disk chain in POOL and IMMUTABLE paths
* `chain-rebase-immutable <DISK_NAME>` : In the IMMUTABLE folder, change a
  disk's metadata to point to a relative path. Use this if you import disk
  chains that are not created with vmh.
* `destroy <DOMAIN>` : Hard shutdown the domain.
* `erase <DOMAIN>` : 'destroy', remove all disk chains and undefine the domain.
  Will also remove the host from the /etc/hosts file.
* `erase-net <DOMAIN>` : Erases all virtual networks defined in a domains
  template XML, not the deployed domain. Use this if there are no more domains
  that require a certain network. Or use it if you made changes to a networks
  XML file, in the `IMMUTABLE/networks` folder. Then reimport the network via
  'import-...' or 'env-deploy' . Remember domains that where active during
  deletion of a network, require a full shutdown and start, to reload the new
  network settings.
* `export-copy <DOMAIN_DESTINATION> <DOMAIN>` : 'shutdown', export the domain,
  XML and an exact disk copy. Will not strip snapshots or old disk data.
  Largest file. Only recommendet if image is already compressed. Most
  compatible.
* `export-merge <DOMAIN_DESTINATION> <DOMAIN>` : 'shutdown', export the domain,
  XML, compress and sparsify the disk copy, can take a long time. Smallest disk
  image size. Can result in a none working image.
* `export-merge-fast <DOMAIN_DESTINATION> <DOMAIN>` : 'shutdown', export the
  domain, XML and compress the disk copy, Faster than 'export-merge'.
  Reasonable disk image size. Sometimes more compatible than 'export-merge'.
* `import-chain <DOMAIN_DESTINATION> <DOMAIN>` : Define a domain from a
  template XML found in the IMMUTABLE folder. Automatically detects and links
  disk chains to libvirts POOL folder. Also adds as a new disk to the chain.
  Requires two files: The disk image with the name 'DOMAIN' and a libvirt
  compatible XML named 'DOMAIN.xml'. Will automatical import and start networks
  from XMLs found in `<PATH_IMMUTABLE>/networks/<NETWORK>.xml` .
* `import-chain-start <DOMAIN_DESTINATION> <DOMAIN>` : Same as 'import-chain'
  and 'start'. This is true for all 'import-...' actions.
* `import-chain-start-wait <DOMAIN_DESTINATION> <DOMAIN>` : Same as '
  import-chain', 'start' and 'wait'.
* `import-clone <DOMAIN_DESTINATION> <DOMAIN>` : Define a domain from a
  template XML found in the IMMUTABLE folder. Automatically detects and copies
  disk chains to libvirts POOL folder. Preferably use 'import-link' or
  'import-chain', to save disk space.
* `import-link <DOMAIN_DESTINATION> <DOMAIN>` : Define a domain from a template
  XML found in the IMMUTABLE folder. Automatically detects and links disk
  chains to IMMUTABLE folder. Please don't start the domain after this. Use '
  chain' to add a new disk to write to. Else libvirt will try to write to the
  immutable disk instead, which is not recommendet. Preferably use '
  import-chain'.
* `purge <DOMAIN>` : Remove the disk image with the same name as the domain in
  the POOL folder. Use after 'erase' to remove broken domain and linked disk
  images.
* `purge-immutable <DOMAIN>` : Remove the disk image with the same name as the
  domain in the IMMUTABLE folder. Also removes the XML file with the same name.
  Useful to clean up exported domains before exporting the same domain again.
* `shutdown <DOMAIN>` : OS shutdown, after 3 minutes do 'destroy'.
* `start <DOMAIN>` : Boot a VM and continue with shell execution.
* `start-wait <DOMAIN>` : Same as 'start' and 'wait'.
* `wait <DOMAIN>` : Wait for an IP and an open SSH port, will block shell
  execution. Timeouts are: get MAC 30 sec., get IP 3 min., get SSH 30 sec. On
  success will automatically add/update the host to the /etc/hosts file.
* `env-deploy <ENVIRONMENT>` : Smart deploy multible domains in parallel or/and
  in series. Iterates through the file: `<PATH_IMMUTABLE>/<ENVIRONMENT>.csv` .
  Will automatical import and start networks from XMLs, found
  in `<PATH_IMMUTABLE>/networks/<NETWORK>.xml` .
  See 'ENVIRONMENT FILE STRUCTURE'. Based on 'import-chain-start' and 'wait'.
* `env-erase <ENVIRONMENT>` : Works in reverse to 'env-deploy'. Based on the
  action 'erase'. Requires the ENVIRONMENT file.
* `env-erase-net <ENVIRONMENT>` : Erase all networks found in an environment
  CSV. Based on the action 'erase-net'. Requires the ENVIRONMENT file.
* `env-erase-full` : Same as 'env-erase' and then 'env-erase-net' combined.
* `env-wait <ENVIRONMENT>` : Based on the action 'wait'. Requires the
  ENVIRONMENT file. Can be used to quickly test/verify an already deployed
  environment.

## ENVIRONMENT FILE STRUCTURE

Each line represents a single domain. Which consists of three strings/settings,
seperatet by a comma.  
`<ORDER>,<DOMAIN_NAME>,<DOMAIN_SOURCE>`

* `ORDER` as integer : Is the group number for all domains to deploy in
  paralell. For example all domains with the order '1', will deploy in paralell
  and testet for SSH, before the next batch, with the order number '2' is
  deployed.
* `DOMAIN_NAME` as string : The libvirt name of the domain to deploy to.
* `DOMAIN_SOURCE` as string : The name of the image to deploy from.

## EXAMPLES

Start a single, already imported, domain with the name 'myvm'

    $ vmh start myvm

Import the disk image 'my-disk-source' and start a new domain, as an image
chain, with the name 'office'. Also boot/start the domain after that.

    $ vmh import-chain-start office my-disk-source

Deploy the following environment file 'my-test1.csv' in two paralell turns. In
practice this means that the 'gateway' domains will be up before deploying the
office domains.

    1,gateway-a,my-pfsense-image  
    1,gateway-b,my-pfsense-image  
    2,ws-office-win,my-windows-image  
    2,ws-office-lnx,my-debian-image

Run:

    $ vmh env-deploy my-test1

Deploy the following environment file 'my-test2.csv' in series.

    1,ws-office-a,my-debian-image  
    2,ws-office-b,my-debian-image  
    3,ws-office-c,my-debian-image  
    4,ws-office-d,my-debian-image

Run:

    $ vmh env-deploy my-test2

## LOGGING

To enable logging

    $ vmh-logging-enable

To disable logging

    $ vmh-logging-disable

Log file is /var/log/vmhandler.log

## COPYRIGHT

See license file

## SEE ALSO

virsh, virt-sparsify, waitforit
