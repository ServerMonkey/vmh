vmh(1) -- Advanced libvirt manager for smart mass and parallel deployment
=============================================

## DESCRIPTION

Advanced libvirt manager for smart mass and parallel deployment. Is a wrapper
for libvirt and virsh.

* Mass and parallel deployment of domains
* Automatic deployment of virtual networks
* Create/compress/import/export images that use disk chains
* More verifications and better error handling than plain virsh
* More automation, e.g. auto shutdown and then force shutdown after a timeout
* Wait for SSH to become available
* Logging system (rsyslog)
* Useful for automating kiosk applications that use VMs

## SYNOPSIS

`vmh <ACTION> <DOMAIN|DESTINATION|DISK_NAME|ENVIRONMENT|CONFIG_TAG> [DOMAIN]|<CONFIG_VARIABLE>` :
(virtualmachinehandler) Advanced actions for libvirt domains

`vmhc <DOMAIN> <DISPLAY_OPTION>` : (vmh connector) Simple wrapper for
'virt-viewer'. Connect to a domain. DISPLAY_OPTION can be 'k' for kiosk and 'f'
for fullscreen.

`vmh-screenshot <DOMAIN> <OUT_FILE>` : Take a screenshot of a domain in ppm
format. If the path OUT_FILE is ommited, the screenshot will be saved to
`$HOME/shot_$DOMAIN.ppm`.

## OPTIONS

* `-h`, `--help` :  
  Display this help screen.

## CONFIG

Config file is in /etc/vmh.conf  
Open the default config file for more information.

To create all required folders and set correct file permissions run:  
`sudo vmh init`  
This can be useful if you want to set up environment files before you have run
vmh for the first time.

## ACTION

Use the `<action>` variable to manipulate domains in different ways.

* `chain <DESTINATION> <DOMAIN>` : 'shutdown', create new empty disk, appends
  disk to chain and renames the domain to DESTINATION. DESTINATION is also used
  to name the disk files. This is helpful to see what images belong to what
  domain.
* `chain-start <DESTINATION> <DOMAIN>` : Same as 'chain' and 'start'.
* `chain-start-wait <DESTINATION> <DOMAIN>` : Same as 'chain', 'start' and '
  wait'.
* `chain-info <DOMAIN>` : Show the disk chain in POOL and IMMUTABLE paths
* `chain-rebase-immutable <DISK_NAME>` : In the IMMUTABLE folder, change a
  disk's metadata to point to a relative path. Use this if you import disk
  chains that are not created with vmh.
* `destroy <DOMAIN>` : Hard shutdown the domain.
* `env-deploy <ENVIRONMENT>` : Smart deploy multiple domains in parallel or/and
  in series. Iterates through the file: `<PATH_IMMUTABLE>/<ENVIRONMENT>.csv` .
  Will automatically import and start networks from XMLs, found
  in `<PATH_IMMUTABLE>/networks/<NETWORK>.xml` . See 'ENVIRONMENT FILE
  STRUCTURE'. Based on 'import-chain-start' and 'wait'.
* `env-erase <ENVIRONMENT>` : Works in reverse to 'env-deploy'. Based on the
  action 'erase'. Requires the ENVIRONMENT file.
* `env-erase-full <ENVIRONMENT>` : Same as 'env-erase' and then 'env-erase-net'
  combined.
* `env-erase-net <ENVIRONMENT>` : Erase all networks found in an environment
  CSV. Based on the action 'erase-net'. Requires the ENVIRONMENT file.
* `env-map <ENVIRONMENT>` : Generate an SVG image of the environment network.
  Files will be stored in the PATH_ENVIRONMENTS folder.
* `env-wait <ENVIRONMENT>` : Based on the action 'wait'. Requires the
  ENVIRONMENT file. Can be used to quickly test/verify an already deployed
  environment.
* `erase <DOMAIN>` : 'destroy', remove all disk chains and undefine the domain.
  Will also remove the host from the /etc/hosts file.
* `erase-net <DOMAIN>` : Erases all virtual networks defined in a domain
  template XML, not the deployed domain. Use this if there are no more domains
  that require a certain network. Or use it if you made changes to a networks
  XML file, in the `IMMUTABLE/networks` folder. Then reimport the network via
  'import-...' or 'env-deploy' . Remember domains that where active during
  deletion of a network, require a full shutdown and start, to reload the new
  network settings.
* `export-copy <DESTINATION> <DOMAIN>` : 'shutdown', export the domain, XML and
  an exact disk copy. Will not strip snapshots or old disk data. Largest file.
  Only recommended if image is already compressed. Most compatible.
* `export-merge <DESTINATION> <DOMAIN>` : 'shutdown', export the domain, XML,
  compress and sparse the disk copy, can take a long time. Smallest disk image
  size. Can result in a none working image.
* `export-merge-fast <DESTINATION> <DOMAIN>` : 'shutdown', export the domain,
  XML and compress the disk copy, Faster than 'export-merge'. Reasonable disk
  image size. Sometimes more compatible than 'export-merge'.
* `get <CONFIG_TAG>` : Get a variable from the CONFIG file. For example 'vmh
  get debug'.
* `import-chain <DESTINATION> <DOMAIN>` : Define a domain from a template XML
  found in the IMMUTABLE folder. Automatically detects and links disk chains to
  libvirt's POOL folder. Also adds as a new disk to the chain. Requires two
  files: The disk image with the name 'DOMAIN' and a libvirt compatible XML
  named 'DOMAIN.xml'. Will automatically import and start networks from XMLs
  found in `<PATH_IMMUTABLE>/networks/<NETWORK>.xml` .
* `import-chain-start <DESTINATION> <DOMAIN>` : Same as 'import-chain' and '
  start'. This is true for all 'import-...' actions.
* `import-chain-start-wait <DESTINATION> <DOMAIN>` : Same as '
  import-chain', 'start' and 'wait'.
* `import-clone <DESTINATION> <DOMAIN>` : Define a domain from a template XML
  found in the IMMUTABLE folder. Automatically detects and copies disk chains
  to libvirt's POOL folder. Preferably use 'import-link' or
  'import-chain', to save disk space.
* `import-link <DESTINATION> <DOMAIN>` : Define a domain from a template XML
  found in the IMMUTABLE folder. Automatically detects and links disk chains to
  IMMUTABLE folder. Please don't start the domain after this. Use '
  chain' to add a new disk to write to. Else libvirt will try to write to the
  immutable disk instead, which is not recommended. Preferably use '
  import-chain'.
* `init` : See CONFIG section.
* `list` : List all available environments.
* `purge <DOMAIN>` : Remove the disk image with the same name as the domain in
  the POOL folder. Use after 'erase' to remove broken domain and linked disk
  images.
* `purge-environments` : Removes everything in the PATH_ENVIRONMENTS folder.
* `purge-immutable <DOMAIN>` : Remove the disk image with the same name as the
  domain in the IMMUTABLE folder. Also removes the XML file with the same name.
  Useful to clean up exported domains before exporting the same domain again.
* `set <CONFIG_TAG> <CONFIG_VARIABLE>` : Set a variable in the CONFIG file. For
  example 'vmh set debug true'.
* `shutdown <DOMAIN>` : OS shutdown, after 3 minutes do 'destroy'.
* `start <DOMAIN>` : Boot a VM and continue with shell execution.
* `start-wait <DOMAIN>` : Same as 'start' and 'wait'.
* `wait <DOMAIN>` : Wait for an IP and an open SSH port, will block shell
  execution. Timeouts are: get MAC 30 sec., get IP 3 min., get SSH 30 sec. On
  success will automatically add/update the host to the /etc/hosts file.

## ENVIRONMENT FILE STRUCTURE

Each line represents a single domain. Which consists of three strings/settings,
separated by a comma.  
`<ORDER>,<DOMAIN_NAME>,<DOMAIN_SOURCE>`

* `ORDER` as integer : Is the group number for all domains to deploy in
  parallel. For example all domains with the order '1', will deploy in parallel
  and tested for SSH, before the next batch, with the order number '2' is
  deployed.
* `DOMAIN_NAME` as string : The libvirt name of the domain to deploy to.
* `DOMAIN_SOURCE` as string : The name of the image to deploy from.

## EXAMPLES

Start a single, already imported, domain with the name 'myvm'

    $ vmh start myvm

Import the disk image 'my-disk-source' and start a new domain, as an image
chain, with the name 'office'. Also boot/start the domain after that.

    $ vmh import-chain-start office my-disk-source

Connect to a domain in fullscreen mode

    $ vmhc myvm f

Take a screenshot of the domain 'myvm' and save it as 'myvm.ppm'

    $ vmh-screenshot myvm myvm.ppm

Deploy the following environment file 'my-test1.csv' in two parallel turns. In
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

Log file is /var/log/vmhandler.log

Enable logging

    $ vmh-logging-enable

Disable logging

    $ vmh-logging-disable

Watch logs

    $ vmh-watch

Clear logs

    $ vmh-clear-logs

## COPYRIGHT

See license file

## SEE ALSO

virsh, virt-sparsify, waitforit
