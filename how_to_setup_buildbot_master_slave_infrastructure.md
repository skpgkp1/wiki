## How to set up buildbot master/slave infrastructure

### Name used in this document:

    Buildbot master server name: (say masterserver)
    Buildbot slave server name: (say slaveserver)
    Slave user name: (say slaveuser1)
    Slave password: (say slavepass1)

## Buildbot master configuration

### Install virtualenv and create sandbox 

    $sudo easy_install virtualenv

    $cd /local/foo
    
    $mkdir buildbot-work

    $cd buildbot-work

    $virtualenv sandbox

    $source sandbox/bin/activate

### Install dependency packages for master

    $pip install --upgrade setuptools

    $pip install buildbot

    $pip install buildbot[tls]

    $pip install buildbot_pkg

    $pip install mock

    $pip install buildbot-www

    $pip install buildbot-grid-view

    $pip install buildbot-waterfall-view

    $pip install buildbot-console-view

### Create buildbot master

    $buildbot create-master master

    $cd master

    $cp master.cfg.sample master.cfg
    
    $ sdiff -s master.cfg.sample master.cfg
    c['workers'] = [worker.Worker("example-worker", "pass")]      | c['workers'] = [worker.Worker("slaveuser1", "slavepass1"
          workernames=["example-worker"],                         |       workernames=["slaveserver"],
    c['buildbotURL'] = "http://localhost:8010/"                   | c['buildbotURL'] = "http://masterserver:8010/"

    $buildbot start .


## Buildbot slave configuration

### Install virtualenv and create sandbox 

    $sudo easy_install virtualenv

    $mkdir buildbot-work

    $cd buildbot-work

    $virtualenv sandbox

    $source sandbox/bin/activate

### Install dependency packages for slave

    $pip install --upgrade setuptools

    $pip install buildbot

    $pip install buildbot[tls]

    $pip install buildbot_pkg

    $pip install mock

    $pip install buildbot-www

    $pip install buildbot-grid-view

    $pip install buildbot-waterfall-view

    $pip install buildbot-console-view
    
    $pip install buildbot-slave
 
### Create buildbot slave

    $buildslave create-slave /local/foo/buildbot-work/sandbox/slave masterserver:9989 slaveuser1 slavepass1
    
    $cd /local/foo/buildbot-work/sandbox/slave
    
    $buildslave start
    
