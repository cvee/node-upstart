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
    author          "Chris Verwymeren <cvee@me.com>"
    
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

# Manual Job Control

As an added bonus, Upstart allows for services to be controlled manually using the commands <code>start</code>,  <code>stop</code> and <code>status</code>. For example, an administrator can start, stop and determine the status of the above job definition using the following commands:

    start node-upstart
    stop node-upstart
    status node-upstart

# More Information

## Upstart

* http://upstart.ubuntu.com/

## Upstart Cookbook

* http://upstart.ubuntu.com/cookbook/

## Presentation and Examples

* https://github.com/cvee/node-upstart

## Chris Verwymeren

cvee@me.com
https://twitter.com/cvee