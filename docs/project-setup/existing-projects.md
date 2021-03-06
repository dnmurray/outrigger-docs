# Existing Projects

To add Outrigger support to an existing project, you'll need to understand how to create:

* Docker Compose file(s)
* Custom Domain Names for Project Services 
* Local code sharing with your containers
* Persistent data storage
* An (*optional*) `.outrigger.yml` file

## Docker Compose file(s)

Outrigger is intended to work with either raw Docker commands or Docker Compose files, and we find things are best when
orchestrating your local environments using Docker Compose.

Be sure to read the [Docker Compose compose file documentation](https://docs.docker.com/compose/compose-file/)
so that you have a handle on how to setup the needed configuration.

### Define Project Services

The essential point is that you will need to define a Docker Compose Service for each of the components in your architecture.
Web/API Server, database, cache/key-value store, search server, proxy cache, mail server, etc. Each of those will be configured
as a service in your `docker-compose.yml` file.

Please see the [Outrigger Examples Repository](https://github.com/phase2/outrigger-examples) to see setups of various 
technologies within Outrigger.  Specifically, the [Drupal 8 example](https://github.com/phase2/outrigger-examples/tree/master/drupal8) 
covers all of the high points.

### Build Container

In addition to containers for all of your project services, we find it useful to have a container we refer to as the 
`build` container. This container provides the tools supporting development of your project. 

The default build image for Outrigger is `outrigger/build`. It includes tools supporting a wide variety of modern
web development such as `node`, `npm`, `composer`, `grunt`, `gulp`, `bower`, etc.

We often configure the build container and associated command containers in a file named `build.yml`. The Drupal 8
example mentioned before contains a sample a [build.yml](https://github.com/phase2/outrigger-examples/blob/master/drupal8/build.yml)
file.

### Choosing Images For Project Services

Any valid Docker Image can work with Outrigger. We also supply [a collection of crafted Docker
images](../project-setup/docker-images.md) that are easily configured and optimized. An example for
an Apache / PHP image based on PHP7 is **outrigger/apache-php:php70** or **php:7.1.1-apache**  

!!! important "Use Image Tags"
    When selecting and image to use, be sure to also specify a tag for that image. When you do not
    specify a tag, the default tag of *latest* is used and it is easy to get out of sync with other
    team members or simply get unwanted changes if you do not explicitly specify an image tag.
        
### General Config

Once you set up your `docker-compose.yml` file, you'll likely need to customize your services for a variety of reasons. 
Many images will use environment variables and/or volume mounts to customize the running container configuration. See
[Customizing Container Configuration](../common-tasks/customizing-container-configuration.md) to learn more about 
customizing your configuration for your project. 

## Custom Domain Names for Project Services

See [DNS Resolution](../common-tasks/dns-resolution.md) to learn how to use Docker Labels to specify
custom domain names for your project's services.

## Sharing code into and across containers

The straightforward way to get code from your host into containers is to use an NFS volume mount.  This allows you, for
example, to mount the current directory (`.`) to a specific place inside a container (`/var/www`) with a simple volume 
mount specification in your `docker-compose.yml` file. See the [Volume Mount](https://docs.docker.com/compose/compose-file/#volumes) 
documentation on volume mount specifications. For example:

    .:/var/www

If filesystem performance for builds or filesystem notifications for watches are what you need, check out the 
[Filesystem Sync](./filesystem-sync.md) documentation to guide you through setup.

## Persistent Data Storage

And finally we have a Persistent Data Volume, `/data`, that lives within the Docker Machine. Be sure to read about the 
Persistent Data Volume (`/data`) in the [Key Concepts](./key-concepts.md) documentation. Often data may need to persist across 
container runs while needing some level of performance but not need to be shared directly with the host computer. 
Things like the `/var/lib/mysql` data directory are often in this category as you may want to export a DB to your host 
computer, but you very rarely want the raw MySQL data files.  For these cases a volume mount from `/data` into your containers
is what you need.  For example:

    /data/project-one/local/mysql:/var/lib/mysql

## .outrigger.yml

Finally, you can customize the `rig project run` command with common or project specific scripts to provide a considerable
upgrade to the developer experience. See [Project Configuration](./project-configuration.md) for details on how to use
an `.outrigger.yml` file in your project directory to have project specific configuration integrated into `rig`.
