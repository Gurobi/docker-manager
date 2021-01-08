![Logo](https://www.gurobi.com/wp-content/uploads/2018/12/logo-final.png "Gurobi Optimization")
# Quick reference
Maintained by: [Gurobi Optimization](https://www.gurobi.com)

Where to get help: [Gurobi Support](https://www.gurobi.com/support/), [Gurobi Documentation](https://www.gurobi.com/documentation/)

# Supported tags and respective Dockerfile links
## Simple Tags
* [9.1.1, latest](https://github.com/Gurobi/docker-manager/blob/master/9.1.1/Dockerfile)

# Quick reference (cont.)

Supported architectures: linux amd64

Published image artifact details: https://github.com/Gurobi/docker-manager

Gurobi images:
- [gurobi/optimizer](https://hub.docker.com/repository/docker/gurobi/optimizer): Gurobi Optimizer (full distribution)
- [gurobi/python](https://hub.docker.com/repository/docker/gurobi/python): Gurobi Optimizer (Python API only)
- [gurobi/modeling-examples](https://hub.docker.com/repository/docker/gurobi/modeling-examples): Optimization modeling examples (distributed as Jupyter Notebooks)
- [gurobi/compute](https://hub.docker.com/repository/docker/gurobi/compute): Gurobi Compute Server
- [gurobi/manager](https://hub.docker.com/repository/docker/gurobi/manager): Gurobi Cluster Manager

# What is `gurobi/manager`?
The Gurobi Optimizer is the fastest and most powerful mathematical programming solver available 
for your LP, QP and MIP (MILP, MIQP, and MIQCP) problems. 
More info at the [Gurobi Website](https://www.gurobi.com/products/gurobi-optimizer/).

The Gurobi Cluster Manager is a new server component that helps to manage your Compute
Server. It provides better security through user authentication and API keys. 
It also expands the capabilities of Compute Server cluster nodes with unified management of 
interactive and non-interactive optimization tasks. 
[Gurobi Compute Server Documentation](https://www.gurobi.com/documentation/current/remoteservices/index.html)
 
## Getting a Gurobi license

Please contact your sales representative at [sales@gurobi.com](mailto:sales@gurobi.com) to discuss licensing options. 
Note that standard license types (NODE, Academic) do not work with Docker.

## Using the client license

You will need to specify a set of properties to connect to a license server.  You have a few options:

* Mounting the client license file:
You can store connection parameters in a client license file (typically called `gurobi.lic`) 
and mount it to the container. This option provides a simple approach for testing.

* Setting parameters through environment variables: Please contact Gurobi support for details.

 
## How to use this image?
### Start a `cluster manager` server instance
`$ docker run gurobi/manager --database=MONGO_DB_URL`

... where `MONGO_DB_URL` is a valid Mongo Database URL

>Note: The `Cluster Manager` needs a database and at least
one node to be able to optimize models.

### ... via docker stack deploy or docker-compose
Example `docker-compose.yml` for a cluster manager:

```
version: '3.1'
services:

  mongodb:
    image: mongo:latest
    restart: always
    volumes:
      - mongodb:/data/db
    networks:
      - back-tier

  manager:
    image: gurobi/manager:latest
    restart: always
    depends_on:
      - "mongodb"
    ports:
      - "61080:61080"
    command: --database=mongodb://mongodb:27017
    networks:
      - back-tier
  
  compute:
    image: gurobi/compute:latest
    restart: always
    depends_on:
      - manager
    command: --manager=http://manager:61080
    networks:
      - back-tier
    volumes:
      - ./gurobi.lic:/opt/gurobi/gurobi.lic:ro

volumes:
  mongodb:

networks:
  back-tier:

```

Run `docker-compose up compute`

>You can scale `gurobi/compute` instances using the [scale](https://docs.docker.com/compose/reference/scale/) parameter in docker-compose:
> `$ docker-compose up --scale compute=2`

Then you can go to http://localhost:61080/manager and login using the default credentials:
```
    standard user: gurobi / pass
    administrator: admin / admin
    system administrator: sysadmin / cluster

```

# License

View [End User License Agreement](https://www.gurobi.com/wp-content/uploads/2020/11/EULA_standard.pdf) for the software contained in this image.

As with all Docker images, these likely also contain other software which may be under other 
licenses (such as Bash, etc from the base distribution, along with any direct or indirect 
dependencies of the primary software being contained).

As for any pre-built image usage, it is the image user's responsibility to ensure that any use 
of this image complies with any relevant licenses for all software contained within.

