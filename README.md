
# Botfront Helm Charts

This repo contains 2 charts:

- `botfront`: the Botfront platform
- `botfront-project`: The Rasa project connected to Botfront,

At this time you need to install both separately.

## Prerequisites

- Kubernetes 1.12+
- Helm 2.11+
- Persistent volume provisioner support in the underlying infrastructure

## Add the repository

```bash
helm repo add botfront https://botfront.github.io/botfront-helm
```

## Install Botfront

Given that there's quite a few parameters to set, we recommend using a config file. This is a minimal `config.yaml` you could use:

```yaml
# OpenShift only
# --------------
# Uncomment and configure the following if you encounter "Error creating: pods "..." is forbidden: unable to validate against any security context constraint"
# securityContext:
#     runAsNonRoot: true
#     runAsUser: ""
#     fsGroup: ""

botfront:
  version: v0.20.0 # or later
  app:
    # The complete external host of the Botfront application (eg. botfront.yoursite.com). It must be set even if running on a private or local DNS (it populates the ROOT_URL).
    host: botfront.yoursite.om
  ingress:
    # If you want to configure an ingress
    enabled: true
    nginx:
      # Set to true if your cluster uses the nginx-ingress controller
      enabled: true

mongodb:
  # Username of the MongoDB user that will have read-write access to the Botfront database. This is not the root user
  mongodbUsername: username
  # Password of the MongoDB user that will have read-write access to the Botfront database. This is not the root user
  mongodbPassword: password
  # Deploy a MongoDB instance as part of this chart. Set this to false if you want to use an external MongoDB provider, such as Atlas.
  enabled: true
  service:
    ## - if deploying under a different release name or a different namespace,
    name: botfront-mongodb-service
    type: NodePort
  # MongoDB host - if deploying under a different release name or a different namespace, change this: <helm-release-name>-mongodb-service.<namespace>
  mongodbHost: botfront-mongodb-service.botfront
  # MongoDB port (make sure it's a string and not an integer)
  mongodbPort: '27017'
  # MongoDB root username
  mongodbRootUsername: root
  # MongoDB root password
  mongodbRootPassword:

```


```bash
helm install -f config.yaml -n botfront --namespace botfront botfront/botfront
```

You probably need to specify at least the value in **bold**


| Parameter                        | Description                                                                                   | Default                 |
|----------------------------------|-----------------------------------------------------------------------------------------------|-------------------------|
| **`botfront.version`**           | Botfront API Docker image                                                                     | `latest`                |
| **`botfront.app.image.name`**    | Botfront Docker image                                                                         | `botfront/botfront`     |
| **`botfront.app.host`**          | Botfront host (e.g botfront.your-domain.com)                                                  | `nil`                   |
| **`botfront.api.image.name`**    | Botfront API Docker image                                                                     | `botfront/botfront-api` |
| `botfront.ingress.enabled`       | Enable Ingress                                                                                | `true`                  |
| `botfront.ingress.nginx.enabled` | Enable if the `nginx-ingress` controller is installed (and used) on the cluster               | `true`                  |
| `botfront.ingress.tlsSecretName` | Optional. Name of the secret containing the TLS certificate. If not set, SSL will be disabled | `nil`                   |
| `botfront.ingress.tlsHost`       | Optional. Host associated with the TLS certificate (may contain a wildcard)                   | `nil`                   |
| `botfront.imagePullSecret`       | Name of the secret containing the credentials to pull images from a private Docker repo       | `nil`                   |


### Configure MongoDB

Botfront stores its data in a MongoDB database. A MongoDB deployment (`stable/mongodb` chart) is included in this chart and can be optionally installed.
If you're using Botfront in a production environment, consider adding your own deployment, or using a managed service. And configure automated backups :)


| Parameter                      | Description                                                                   | Default                             |
|--------------------------------|-------------------------------------------------------------------------------|-------------------------------------|
| **`mongodb.mongodbUsername`**  | The name of the user accessing the Botfront database (must not be `root`)     | `bfrw`                              |
| **`mongodb.mongodbPassword`**  | The password of the user accessing the Botfront database (must not be `root`) | `nil`                               |
| `mongodb.mongodbHost`          | MongoDB server                                                                | `botfront-mongodb-service.botfront` |
| `mongodb.mongodbPort`          | MongoDB server                                                                | `27017`                             |
| `mongodb.mongodbQueryString`   | MongoDB connection query string                                               | `&retryWrites=true`                 |
| `mongodb.mongodbRootPassword`  | MongoDB `root` password (only required if `mongodb.enabled` is set            | `nil`                               |
| `mongodb.mongodbOplogUsername` | Optional. Considerable database performance gains                             | `nil`                               |
| `mongodb.mongodbOplogPassword` | Optional. Considerable database performance gains                             | `nil`                               |
| `mongodb.enabled`              | Set to `true` to add MongoDB to this deployment                               | `false`                             |


**Important**
If you are using the provided MongoDB deployment, `mongodb.mongodbHost` must follow the following pattern: `<release-name>-mongodb-service.<namespace>`

### Optional: enable and configure Mongo Express

Mongo Express is a web-based client for MongoDB. You can optionally add Mongo Express to your deployment

| Parameter                            | Description                                                   | Default                             |
|--------------------------------------|---------------------------------------------------------------|-------------------------------------|
| `mongo-express.enabled`              | Set to `true` to enable a Mongo Express deployment            | `false`                             |
| `mongo-express.basicAuthUsername`    | The Basic Auth username to access the Mongo Express interface | `nil`                               |
| `mongo-express.basicAuthPassword`    | The Basic Auth password to access the Mongo Express interface | `nil`                               |
| `mongo-express.mongodbAdminPassword` | MongoDB `root` password                                       | `nil`                               |
| `mongo-express.mongodbServer`        | MongoDB server                                                | `nil`                               |
| `mongo-express.hosts[0].host`        | Mongo Express host                                            | `botfront-mongodb-service.botfront` |
| `mongo-express.hosts[0].paths[0]`    | Mongo Express host path. **You must set it to `/`**           | `nil`                               |
| `mongo-express.tls[0].hosts[0]`      | Optional. The host associated to your certificate             | `nil`                               |
| `mongo-express.tls[0].secretName`    | Optional. Secret containing your certificate                  | `nil`                               |


**Important**
If you are using the provided MongoDB deployment, `mongodb.mongodbHost` must follow the following pattern: `<release-name>-mongodb-service.<namespace>`

## Install Rasa


```bash
helm install -n my_project --namespace botfront botfront/ --set graphQLEndpoint=http://<botfront-service>.<botfront-namespace>/graphql



```
| Parameter         | Description                                                                   | Default                                 |
|-------------------|-------------------------------------------------------------------------------|-----------------------------------------|
| `projectId`       | ProjectId                                                                     | `bf`                                    |
| `graphQLEndpoint` | Should have the form `http://<botfront-service>.<botfront-namespace>/graphql` | `nil`                                   |
| `rasa.image`      | Rasa image                                                                    | `botfront/rasa-for-botfront:1.7.1-bf.4` |


## Authenticating to Botfront private repo (Enterprise Edition Customers)

Once you obtain your `key.json` file:

1. Create a `docker-registry` secret in your cluster
```bash
kubectl create secret docker-registry gcr-json-key \
  --docker-server=https://gcr.io \
  --docker-username=_json_key \
  --docker-password="$(cat key.json)" \
  --namespace botfront
```

2. Patch the `default` service account (or the service account pulling images in your pods) **in the namespace Botfront is deployed**.
```
kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "gcr-json-key"}]}' --namespace botfront
```

3. Set `botfront.imagePullSecret` to `gcr-json-key`

