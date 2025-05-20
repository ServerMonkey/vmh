vmh(1) -- Advanced libvirt manager for smart mass and parallel deployment
=============================================

## DESCRIPTION

Advanced libvirt manager for smart mass and parallel deployment. Is a wrapper
for libvirt and virsh.

* Mass and parallel deployment of domains
* Automatic deployment of virtual networks
* Instant deployment to disk chains from templates (Golden images)
* More verifications and better error handling than plain virsh
* More automation, e.g. auto shutdown and then force shutdown after a timeout
* Wait domains MAC, IP and SSH to become available
* Auto expand domains disk partitions
* Logging system (rsyslog)
* Versioning of libvirt templates
* CSV Inventorymaker SSOT
* Auto linter for Inventorymaker SSOT CSV files
* Auto linter for templates
* Screenshots of domains
* Screenshot montage of domains
* Generate SVG network maps of sites
* Create/compress/import/export images that use disk chains
* Useful for automating kiosk applications that use VMs

## SYNOPSIS

`vmh <ACTION> <CONFIG_TAG|DOMAIN|DESTINATION|NEW_TEMPLATE|SITE> [CONFIG_VARIABLE]|[DOMAIN]|[TEMPLATE]`:
(virtualmachinehandler) Advanced actions for libvirt domains

`virt-viwer-easy <DOMAIN> <DISPLAY_OPTION>` : Simple 'virt-viewer' wrapper.
Auto connect to a domain. DISPLAY_OPTION can be 'k' for
kiosk and 'f' for fullscreen. Always uses 'reconnect' and 'wait'. Useful for
kiosk systems. Will not block the shell.

`virt-screenshot <DOMAIN> <OUT_FILE>` : Take a screenshot of a domain in ppm
format. If the path OUT_FILE is ommited, the screenshot will be saved to
`$HOME/shot_$DOMAIN.ppm`.

vmh will always install domains into the 'default' libvirt pool. This document
will refer to this folder as 'POOL'.

The TEMPLATES folder contains all libvirt templates which are also known as
'Golden images'. Files in this folder are not supposed to be modified.
This document will refer to this folder as PATH_TEMPLATES.

There are two types of templates: domains and networks:

Domain templates are made up of a qcow2 disk image and a libvirt XML file.
They have the following filename format: `libvirt-img-<DOMAIN>_<VERSION>.xml`

Network templates are made up of a single libvirt XML network file and have the
following filename format: `libvirt-net-<NETWORK>_<VERSION>.xml`

## OPTIONS

* `-h`, `--help` :  
  Display this help screen.

## CONFIG

Config file is in /etc/vmh.conf  
Open the default config file for more information.

To create all required folders and set correct file permissions run:  
`sudo vmh init`  
This can be useful if you want to set up site files before you have run
vmh for the first time.

## ACTION

Use the `<action>` variable to manipulate domains in different ways.

### Action: Basic application

* `get <CONFIG_TAG>` : Get a variable from the CONFIG file. For example 'vmh
  get debug'.
* `set <CONFIG_TAG> <CONFIG_VARIABLE>` : Set a variable in the CONFIG file. For
  example 'vmh set debug true'.
* `init` : See CONFIG section.

### Action: Logging

* `log` : Show last 1000 lines of the log file.
* `log-watch` : Follow the log file.
* `log-purge` : Clear the log file.
* `log-enable` : Enable logging.
* `log-disable` : Disable logging.

### Action: Cleanup and maintenance

* `purge <DOMAIN>` : Use carefully! Removes the disk image with the same name
  as the domain in the POOL folder. This makes sure broken domain and linked
  disk images are completely removed. This also removes the 'shared' folder and
  everything in it. Useful to run after running 'erase'.
* `purge-sites` : Use carefully! Removes everything in the PATH_SITES folder.
  Use this if you want to start fresh with sites. This will also remove all
  screenshots and network maps.
* `purge-template <DOMAIN>` : Use carefully! Remove the disk image with the
  same name as the domain in PATH_TEMPLATES. Also removes the XML file with the
  same name. Useful to clean up exported domains before exporting the same
  domain again.
* `unlink` : Use carefully! Will unlink all symbolic links in the POOL folder.
  This will break all symlinks of already installed domains. Only use this if
  you want to clean up the POOL folder. Useful to start a fresh installation of
  domains and to avoid disk chain errors based on old symlinks and non-existing
  files.

### Action: Template management

* `list-templates` : List all available templates.
* `template-add <FILE>` : Copy a qcow2 or xml template to PATH_TEMPLATES.
  Use this in your program to add a new template to the system. Verifies with
  MD5.
* `template-del <FILE>` : Remove a qcow2 or xml template from PATH_TEMPLATES.
  Use this in your program to remove a template from the system. Only
  the basename of the FILE path will be used. So you can pass the same path to
  'template-del' as you used in 'template-add'. This only works if the source
  file still exists.

### Action: Disk chains

* `chain <DESTINATION> <DOMAIN>` : 'shutdown', create new empty disk, appends
  disk to chain and renames the domain to DESTINATION. DESTINATION is also used
  to name the disk files. This is helpful to see what images belong to what
  domain.
* `chain-start <DESTINATION> <DOMAIN>` : Same as 'chain' and 'start'.
* `chain-start-wait <DESTINATION> <DOMAIN>` : Same as 'chain', 'start' and '
  wait'.
* `chain-info <DOMAIN>` : Show the disk chain in POOL and PATH_TEMPLATES
* `chain-rebase-templates <DISK_NAME>` : In PATH_TEMPLATES, change a
  disk's metadata to point to a relative path. Use this if you want to import
  disk chains that are not created with vmh.

### Action: Single Domain

* `destroy <DOMAIN>` : Hard shutdown the domain.
* `erase <DOMAIN>` : 'destroy', remove all disk chains and undefine the domain.
  Will also remove the domain from the /etc/hosts file.
* `erase-net <TEMPLATE>` : Erases all virtual networks defined in a template
  XML, not the deployed domain. Use this if there are no more domains
  that require a certain network. Or use it if you made changes to a networks
  XML template. Then reimport the network via 'import-...' or 'site-deploy' .
  Remember domains that where active during deletion of a network, require a
  full shutdown and reboot, to reload the new network settings.
* `from-site-erase-net <DOMAIN> <SITE>` : Erase all networks of a single
  domain from a site. This erases networks that are defined in the template and
  site file. Use this if you want to erase all networks; after erasing a single
  domain in a site file. But preferably use 'site-erase-full' or
  'site-erase-net' instead. Based on the action 'erase-net'.
* `shutdown <DOMAIN>` : OS shutdown, after 3 minutes do 'destroy'.
* `start <DOMAIN>` : Boot a domain and continue with shell execution.
* `start-wait <DOMAIN>` : Same as 'start' and 'wait'.
* `wait <DOMAIN>` : Wait for an IP and an open SSH port, will block shell
  execution. Timeouts are: get MAC 30 sec., get IP 3 min., get SSH 30 sec. On
  success will automatically add/update the domain to the /etc/hosts file.

### ACTION: Import single domain

* `from-site-import-start <DOMAIN> <SITE>` : Import single domain from a site
  CSV and start it. This is internally used by the action 'site-deploy' to
  deploy sites in parallel. Only use this action to manually debug individual
  domains. Not recommended to be used in your scripts. Use 'site-deploy' for
  normal operations instead. Based on 'import', 'start' and 'wait'.
* `import <DOMAIN> <TEMPLATE>` : Define a domain from a template XML
  found in the TEMPLATE folder. Automatically detects and links disk chains to
  libvirt's POOL folder. Also adds as a new disk to the chain. Requires two
  files a libvirt compatible XML and a QCOW2 disk image. Will automatically
  import and start networks from XMLs found in PATH_TEMPLATES.
* `import-start <DOMAIN> <TEMPLATE>` : Same as 'import' and 'start'.
* `import-start-wait <DOMAIN> <TEMPLATE>` : Same as 'import', '
  start' and 'wait'.

### ACTION: Export single domain

* `export-copy <NEW_TEMPLATE> <DOMAIN>` : 'shutdown', export the domain, XML
  and an exact disk copy. Will not strip snapshots or old disk data. Largest
  file. Only recommended if image is already compressed. Most compatible.
* `export-merge <NEW_TEMPLATE> <DOMAIN>` : 'shutdown', export the domain, XML,
  compress and sparse the disk copy, can take a long time. Smallest disk image
  size. Can result in a none working image.
* `export-merge-fast <NEW_TEMPLATE> <DOMAIN>` : 'shutdown', export the domain,
  XML and compress the disk copy, Faster than 'export-merge'. Reasonable disk
  image size. Sometimes more compatible than 'export-merge'.

### ACTION: Site management

* `list-sites` : List all available sites.
* `site-check <SITE>` : Linter for the `<PATH_TEMPLATES>/<SITE>.csv` file.
* `site-deploy <SITE>` : Smart deploy multiple domains in parallel
  and/or in series. Iterates through the file: `<PATH_TEMPLATES>/<SITE>.csv` .
  Will automatically import/start networks and expand disks. Timeout is 15
  minutes. See 'SITE FILE STRUCTURE'. Based on 'from-site-import-start' and '
  wait'.
* `site-erase <SITE>` : Works in reverse to 'site-deploy'. Based on the
  action 'erase'. Requires the SITE file.
* `site-erase-full <SITE>` : Same as 'site-erase' and then 'site-erase-net'
  combined.
* `site-erase-net <SITE>` : Erase all networks, for each domain, found in a
  site CSV and all templates. Based on the action 'from-site-erase-net'.
* `site-list <SITE>` : List content of a site.
* `site-list-pretty <SITE>` : List content of a site. Pretty print.
* `site-list-csv <SITE>` : List content of a site in CSV format.
* `site-list-json <SITE>` : List content of a site in compact JSON format.
* `site-list-json-pretty <SITE>` : List content of a site in pretty JSON
  format.
* `site-map <SITE>` : Generate an SVG image of the sites network.
  Files will be stored in the PATH_SITES folder.
* `site-screenshot <SITE>` : Create screenshots for all Domains and generate a
  montage. Files will be stored in the PATH_SITES folder. Based on
  'virt-screenshot'.
* `site-wait <SITE>` : Based on the action 'wait'. Requires the
  SITE file. Can also be used to quickly test/verify an already deployed
  site.

### ACTION: All sites

* `all-map` : Generate an SVG and PNG image of all the sites networks. Files
  will be stored in PATH_SITES/_all/. Requires 'site-map' to be run first.
* `all-screenshot` : Montage screenshots of all sites into one. File will be
  stored in PATH_SITES/_all/. Requires 'site-screenshot' to be run first.

## VERSIONING

The CLI <DOMAIN> parameter, and certain Inventorymaker variables will
automatically search for the latest or a specific versions of a template.  
For example if you have a file named 'my-template_0.3.xml' it is enough to pass
the argument 'my-template' to the CLI in order to use the latest version of the
template.

Example:

Let's say you have a Golden Image template named 'my-template_0.3.xml' and '
my-template_1.xml'.

* `my-template` : Will use the latest version, 'my-template_1.xml'.
* `my-template_0.3` : Will use the specific version '0.3' of 'my-template'.

No need to specify the file extension.

The same goes for the following Inventorymaker SSOT CSV variables:

- network templates (variables 'NET' and 'MAC')
- CD/DVD images (variable 'CD')
- Golden Images (variable 'OS')

The CD/DVD search path is '/var/opt/disk_images'. Put all your CD/DVD iso
images in this folder. Or better, use symbolic links to the actual files.

See the man page of sort-by-version(1) to understand how the latest version is
determined.

## PLACEHOLDER VARIABLES IN XML

When importing and exporting a domain, the following variables will be replaced
in the XML file. This is useful if you want to use the same XML file for
multiple domains.  
Make sure to use '/' before and after the variable. Else XML verification will
fail.

* `/POOL_PATH_PLACEHOLDER/` : Will be replaced with the POOL path.
* `/SHARED_PATH_PLACEHOLDER/` : Will be replaced with the SHARED path
  /var/lib/libvirt/shared/<DOMAIN_SOURCE>

## INVENTORYMAKER SSOT FOR SITES

When using any "site-" command, vmh will parse the Inventorymaker SSOT file in
PATH_SITES. In this way you can manage all your domains in a single CSV file.

Vmh will automatically customize the domains based on the variables values in
the CSV file.

Inventorymaker variables that will be parsed:

| IM VARIABLE | TYPE | VMH DOMAIN CHANGE                               |
|-------------|------|-------------------------------------------------|
| CD          | STR  | insert CD/DVD into drive                        |
| CPU         | INT  | replaces number of virtual CPU cores            |
| DISK_SIZE   | INT  | resize the VM's disk to this total size, in GiB |
| DISK_TYPE   | STR  | can be empty, 'CLONE' or 'CHAIN'                |
| HOSTNAME    | STR  | new domain name                                 |
| MAC         | STR  | replaces NICs MAC address (colon separated)     |
| MEM         | INT  | replaces RAM in GiB                             |
| NET         | STR  | replaces NICs network name                      |
| ORDER       | INT  | domain deployment order                         |
| OS          | STR  | Golden Image template name                      |
| STATE       | STR  | exclude hosts where STATE is 'EXC' or 'DOWN'    |

Multible CDs and NICs can be used by adding the variables CD_2 to CD_4,
MAC_2 to MAC_10 and NET_2 to NET_10. Also see the section 'VERSIONING'.

To see the internal structure of the current Inventorymaker SSOT CSV, run:

    $ vmh site-list-pretty <SITE>

## LOGGING

The log file is /var/log/vmhandler.log  
Preferably use the 'log' commands to view the log file.

## EXAMPLES

Start a single, already imported, domain with the name 'myvm'

    $ vmh start myvm

Import the disk image 'my-disk-source' and start a new domain, as an image
chain, with the name 'office'. Also boot/start the domain after that.

    $ vmh import-chain-start office my-disk-source

Connect to a domain in fullscreen mode

    $ virt-viewer-easy myvm f

Take a screenshot of the domain 'myvm' and save it as 'myvm.ppm'

    $ virt-screenshot myvm myvm.ppm

To deploy a site, symlink or create an Inventorymaker CSV file in
PATH_TEMPLATES/vmhsites. Then deploy the site.

    $ ln -s /home/foo-org/gotham-test.csv /var/lib/libvirt/vmhsites/gotham.csv
    $ vmh site-deploy gotham

Export a domain to a new template and compress the disk image.
Set the version number '1' in the new template name.

    $ sudo vmh export-merge libvirt-img-myvm_1 myvm

## Known issues

On Windows XP DISK_SIZE (virt-auto-expand) will not work. This is a bug in
qemu-img / virt-resize. Avoid using DISK_SIZE on Windows XP domains.

## COPYRIGHT

See license file

## SEE ALSO

virsh, virt-sparsify, waitforit, virt-auto-expand, sort-by-version
