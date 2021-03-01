# LAMP Platform

Brown University's LAMP platform deployment repository. This repository
contains all of the code and configuration for deploying the mindLAMP
platform to a cloud hosted Kubernetes cluster.

## Running

Right now, running is semi-manual. To run first start the database, cache,
message queue, and notifications services.

```
$ docker-compose up -d database cache message_queue notifications
```

Confirm the services are up and healthy via `docker-compose ps`. Once the
services are up and healthy, start the rest of the components.

```
$ docker-compose up -d
```

This will start the server and dashboard services. To retrieve the
administrator password on first boot look at the logs for the server.

```
$ docker-compose logs server
```

Assert the server is working properly by making a test curl call.

```
$ curl -k https://localhost:3000/researcher -H "Authorization: Basic admin:YOUR_PASSWORD_HERE"
```

To regenerate the admin password, tear down the services.

```
$ docker-compose down --volumes
```

And bring them back up in the order documented above.

## Services

Below is a brief description of each service.

### Notifications

The notifications service is a mock notifications server. It responds with
200 to the `/push` endpoint. It doesn't do anything else. We needed the
mock notifications service to make sure the server would run.

### Dashboard

The dashboard service is the web frontend to the mindLAMP platform. It
provides access to data and the APIs provided by the server component. In
local development, use the server `localhost:3000` to connect the dashboard
to the local instance of the mindLAMP server.

### Server

The server service is the mindLAMP API. It handles all incoming requests
from the mobile application, and dashboard and stores data in the database.
On first boot the server generates a new administrator password. It
requires the notifications, database, cache, and message queue services to
run.

**NOTE**: The encryption key for the server service must be a 64 hex
character string.

### Database

The database service is the CouchDB database used by mindLAMP. This
container is exposed to localhost for development on port 5984. To get a
graphical interface to the CouchDB instance use this URL:
`http://localhost:5984/_utils`

### Cache

(???) The cache is used to manage the async job queue of the mindLAMP
platform. Used in conjunction with the message queue to send messages to
other microservices in the platform.

### Message Queue

(???) The message queue service provides a means for microservices to query
data from the mindLAMP api. The service subscribes to various different
topics. When a microservice sends a message on a topic corresponding to the
API with an id of data needed for post processing, the server service will
return the data associated with the topic and id provided.
