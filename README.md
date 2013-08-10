stormthing
==========

A proverbially good way to quickly stand up storm, without it being a toy configuration. 

Setup Juju **0.7**:

We're using an old version of juju because the latest version does not support local mode.  Local mode is necesary if you don't want to pay for EC2.

The following installation instructions are referenced from _http://askubuntu.com/questions/65359/how-do-i-configure-juju-for-local-usage_

    sudo add-apt-repository ppa:juju/pkgs
    sudo apt-get update
    sudo apt-get install lxc apt-cacher-ng libzookeeper-java zookeeper juju

the following command will fail, but will setup the `~/.juju` directory for you.  Run it

    juju bootstrap


Under `~/.juju` there is a file called `environments.yml`.  In the file, there are 3 `types` which can be `local`, `ec2`, and `openstack`.  Openstack is available.  Will update when we have more info

For development, we use the `local` mode

Now we want to edit `environments.yml`.  Change `type` to *local*

**ex.**

    environments:
      sample:
        type: local
        admin-secret: aLoTofNumb3rsAnDl3tter5SssSasdf
        default-series: precise
        data-dir: /home/peter/.storm/data

`data-dir` is just where storm is going to dump its data.  You shouldn't put this in a temp directory.

Now
    
    juju bootstrap

This should start some stuff, and initialize some other stuff and end with `INFO 'bootstrap' command finished successfully`

Now run 

    juju deploy mysql

and now, if you run 

    juju status

you should get output that is approximately like 

    machines:
      0:
        agent-state: running
        dns-name: localhost
        instance-id: local
        instance-state: running
    services:
      mysql:
        charm: cs:precise/mysql-26
        relations:
          cluster:
          - mysql
        units:
          mysql/0:
            agent-state: pending
            machine: 0
            public-address: null

run 

    juju debug-log

to watch the output.  If, for some reason, things go wrong, exit out, and run `juju destroy-environment` and start from the bootstrap again.  However, this can take a couple minutes

once these are started, run 
    
    juju add-relation wordpress mysql

in order to connect the wordpress and mysql instance.  Now, if you would like to view the wordpress instance, use

    juju expose wordpress

and the wordpress site should be available at the ip specified

----

Once this is up and running, we can try storm.  So first, I tore down the installation (you don't have to, but I didn't need them, and it takes less memory to delete them)

Now I checked out the Storm charm

    bzr branch lp:~maarten-ectors/charms/precise/storm/trunk precise

this should initialize the repository under a directory called precise.  This is important becuase storm looks under particular directories for the charm.  Make sure that the directory (for me it was precise) that you put the repository under matches your operating system.  For more clarification see http://askubuntu.com/questions/65534/having-trouble-locating-charms-pointing-to-store-juju-ubuntu-com-dns-error

Now we deploy the local storm repository

    juju deploy zookeeper
    juju deploy --repository path/to/storm/charm/ local:storm stormmaster #note that this is above the 'precise' directory
    juju deploy --repository path/to/storm/charm/ local:storm stormworker #note that this is above the 'precise' directory



