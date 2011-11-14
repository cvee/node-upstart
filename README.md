# Running node.js Apps with Upstart

# Introduction

After developing my first node.js app, I realized the need to run it as a system service. Specifically I wanted to have the app start during boot, run in the background and, if the process died unexpectedly, have it respawn automatically. While researching how to do this on an Ubuntu system I came across Upstart.

# What is Upstart?

The official Upstart website defines Upstart as "an event-based replacement for the /sbin/init daemon which handles starting of tasks and services during boot, stopping them during shutdown and supervising them while the system is running." Through the creation of custom configuration files, called job definitions,  we are able to "daemonize" a node.js app by assigning it behaviors common to system services.

# Upstart Job Definitions

Upstart Job Definitions are are defined in files located in /etc/init. The name of a job is the definition filename without the .conf extension. The following is an example job definition for a node.js app:

    # node-upstart - Example Upstart job definition for a node.js based app
    #
    
    description     "Example Upstart job definition for a node.js based app"
    author          "Chris Verwymeren"
    
    # When to start the service
    start on runlevel [2345]
    
    # When to stop the service
    stop on runlevel [06]
    
    # Prepare the environment
    #   Create directories for logging and process management
    #   Change ownership to the user running the process
    pre-start script
        mkdir -p /var/opt/node
        mkdir -p /var/opt/node/log
        mkdir -p /var/opt/node/run
        chown -R node:node /var/opt/node
    end script
    
    # If the process quits unexpectadly trigger a respawn
    respawn
    
    # Start the process
    exec start-stop-daemon --start --chuid node --make-pidfile --pidfile /var/opt/node/run/node-upstart.pid --exec /home/node/local/node/bin/node -- /home/node/apps/node-upstart.js >> /var/opt/node/log/node-upstart.log 2>&1

The first section provides information about the job definition. In this instance a description of the app is provided along with the name and email address of the author who created the job definition. Similar to most other configuration files, comments are initiated by placing a # symbol at the beginning of a line of text.

    # node-upstart - Example Upstart job definition for a node.js based app
    #
    
    description     "Example Upstart job definition for a node.js based app"
    author          "Chris Verwymeren <cvee@me.com>"

The next section provides information on when to start and stop the job. In this example, the process will be automatically started on the various Multi-User Modes (2-5) and stopped on Halt (0) or Reboot (6).

    # When to start the service
    start on runlevel [2345]
    
    # When to stop the service
    stop on runlevel [06]

<code>pre-start</code> allows the environment to be prepared before a job is started. In this example, the directories for logging and process management are created and assigned ownership.

    # Prepare the environment
    #   Create directories for logging and process management
    #   Change ownership to the user running the process
    pre-start script
        mkdir -p /var/opt/node
        mkdir -p /var/opt/node/log
        mkdir -p /var/opt/node/run
        chown -R node:node /var/opt/node
    end script

Using the respawn command will restart the job if it quits unexpectadly.

    # If the process quits unexpectadly trigger a respawn
    respawn

The last section handles the details of the job command which is run during the "start" action. 

    # Start the process
    exec start-stop-daemon --start --chuid node --make-pidfile --pidfile /var/opt/node/run/node-upstart.pid --exec /home/node/local/node/bin/node -- /home/node/apps/node-upstart.js >> /var/opt/node/log/node-upstart.log 2>&1

In the example above, the job command is run under the node user (<code>--chuid node</code>), a process id file is created at a specific location (--make-pidfile --pidfile /var/opt/node/run/node-upstart.pid) and the node.js app is executed with stdout and stderr logged to a file (--exec /home/node/local/node/bin/node -- /home/node/apps/node-upstart.js >> /var/opt/node/log/node-upstart.log 2>&1).

In the above example, the name of the job definition file is <code>node-upstart.conf</code> therefore the name of the job is <code>node-upstart</code>. When using command line utilites to interact with the job, use the job name.

After creating a job definition save the file to /etc/init. Then, use <code>initctl</code> to verify the job has been recognized by Upstart.

    $ initctl list | grep node-upstart
    node-upstart stop/waiting

# Manual Job Control

As an added bonus, Upstart allows for services to be controlled manually using the commands <code>start</code>,  <code>stop</code> and <code>status</code>. For example, an administrator can start, stop and determine the status of the above job definition using the following commands:

    $ sudo start node-upstart
    node-upstart start/running, process 3410
    $ sudo status node-upstart
    node-upstart start/running, process 3410
    $ sudo stop node-upstart
    node-upstart stop/waiting
    $ sudo status node-upstart
    node-upstart stop/waiting

# More Information

## Upstart

* http://upstart.ubuntu.com/

## Upstart Cookbook

* http://upstart.ubuntu.com/cookbook/

## Runlevel

* http://en.wikipedia.org/wiki/Runlevel

## Presentation and Examples

* https://github.com/cvee/node-upstart

## Chris Verwymeren

* cvee@me.com
* https://twitter.com/cvee
