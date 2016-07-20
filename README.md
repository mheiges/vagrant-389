# vagrant-389_ebrc

Warning: This project will not be useable by anyone outside EBRC.

This Vagrant project prepares two Virtualbox virtual machines for
installation of 389 Directory Server per EuPathDB BRC specifications. It
requires access to EBRC's source code repository for complete
installation.

These VMs are intended for developing Puppet manifests for 389
management and for pre-flighting 389 configuration changes before
deploying on the production servers.

## Acquire project

    git@github.com:mheiges/vagrant-389_ebrc.git

## Setup the `scratch` directory

The `scratch` directory is empty when initially cloned from the git
repo. It needs things added before you can install and use the 389
software. The `scratch` directory is mounted on the guest as
`/vagrant/scratch/`.

### Acquire EBRC Puppet manifests

    cd scratch
    git clone git@git.apidb.org:puppet4.git puppetlabs

### Sample directory data

Place data for import into the directory as `sampledata.ldif` in
`scratch` (specifically this file name). For example, use a backup from
`/var/lib/dirsrv/slapd-ds4/ldif/daily.ldif-userRoot` on a production
server. This data will be imported by the `setupLdapInstance` script.

## 389 Development

From with in the vagrant project directory, switch to the appropriate
branch for the vagrant project (this is not the `scratch/puppetlabs`
branch). This gives you the proper Vagrantfile.

    git checkout 389

    vagrant up dsa
    vagrant up dsb

    vagrant ssh dsa
    vagrant ssh dsb

On each dsa and dsb run puppet to install 389 DS according to EBRC's
specifications.

    sudo /opt/puppetlabs/bin/puppet apply \
      --environment=savm \
      /etc/puppetlabs/code/environments/savm/development/fedora_ds.pp 

You will probably need to `vagrant reload dsa` and `vagrant reload dsb`
after puppet provisioning to reset the Landrush plugin configuration so
the hostname for the guests resolve.

The interactive command `sudo -E ~vagrant/setupLdapInstance` can be used to
setup an instance, or do it manually according to
https://wiki.apidb.org/index.php/389DirectoryServerInstallation

    sudo -E ~vagrant/setupLdapInstance


## Setup Multi-master replication

To setup MMR between `dsa` and `dsb`

    sudo -i

    ~root/fedora-ds/Directory/bin/mmr.pl --host1 dsa.vm.apidb.org \
             --host2 dsb.vm.apidb.org  \
             --host1_id 2 \
             --host2_id 4 \
             --bindpw gas1gacdo   \
             --repmanpw bohozo \
             --base dc=apidb,dc=org  \
             --create  \
             --with-ssl



## Test Multi-master replication

    ldapsearch -ZZ -LLL -x -H ldap://dsa.vm.apidb.org \
      -D "cn=directory manager" \
      -w gas1gacdo \
      -b "uid=plasmoalpha,ou=People,dc=apidb,dc=org" description
  
    ldapsearch -ZZ -LLL -x -H ldap://dsb.vm.apidb.org \
      -D "cn=directory manager" \
      -w gas1gacdo \
      -b "uid=plasmoalpha,ou=People,dc=apidb,dc=org" description

Change `description` through `dsa` and ldapsearch again to confirm
change is replicated.

    /usr/bin/ldapmodify -x -H ldap://dsa.vm.apidb.org \
      -D "cn=directory manager"  -w gas1gacdo  <<EOF
    dn: uid=plasmoalpha,ou=People,dc=apidb,dc=org
    changetype: modify
    replace: description
    description: new description
    EOF

Change `description` attribute through `dsb`.

    /usr/bin/ldapmodify -x -H ldap://dsb.vm.apidb.org \
      -D "cn=directory manager"  -w gas1gacdo  <<EOF
    dn: uid=plasmoalpha,ou=People,dc=apidb,dc=org
    changetype: modify
    replace: description
    description: another description
    EOF
