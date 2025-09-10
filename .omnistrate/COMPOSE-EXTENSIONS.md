# Compose specification

Omnistrate extends the docker-compose specification, which allows custom extensions using the syntax `x-`, and supports a subset of the standard specifications.

Omnistrate leverages the standard spec and those extensions together to complete the spec for your SaaS. We will go over the extensions below in more detail.

Automatic Docker Compose Generation

When using [build from repo](../../getting-started/build-from-repo/) or [build from image](../../getting-started/build-from-image/) approaches, Omnistrate will automatically generate a docker compose file for you if one doesn't already exist in your project. This generated compose file will include the necessary configuration based on your container image and any environment variables you specify during the build process.

Here is a list of the supported and unsupported specifications, to better understand what you can and cannot specify via docker-compose on the Omnistrate platform and what you can expect to be supported in the future.

For more details on the compose specification, please refer to the [official docker-compose documentation](https://docs.docker.com/compose/compose-file/compose-file-v3/).

## Supported compose tags

This is a list of specs supported in the compose specification, including native and custom tags.

### Native tags

Here are the native tags Omnistrate supports either natively or with an optimized implementation:

- [image](https://docs.docker.com/compose/compose-file/compose-file-v3/#image) - native support for all public registries and private registries with credentials

- [expose](https://docs.docker.com/compose/compose-file/compose-file-v3/#expose) - native support for exposing ports

- [ports](https://docs.docker.com/compose/compose-file/compose-file-v3/#ports) - native support for port mapping

- [volumes](https://docs.docker.com/compose/compose-file/compose-file-v3/#volumes) - native support for volumes and volume mapping, configuration, and management

- [environment](https://docs.docker.com/compose/compose-file/compose-file-v3/#environment) - native support for environment variables

- [Env files](https://docs.docker.com/compose/how-tos/environment-variables/set-environment-variables/#use-the-env_file-attribute) - native support for specifying environment variables in a separate file. For example:

  ```
  env_file:
    - .env
  ```

You can do [variable interpolation](https://docs.docker.com/compose/how-tos/environment-variables/variable-interpolation/) using the syntax `${VARIABLE_NAME}`. For example, if your `.env` file contains the environment variable `DB_HOST=YOUR_DB_HOST`, you can use it in your compose file like this:

```
environment:
  - DB_HOST=${DB_HOST}
```

- [depends_on](https://docs.docker.com/compose/compose-file/compose-file-v3/#depends_on) - native support for service dependencies

- [entrypoint](https://docs.docker.com/compose/compose-file/compose-file-v3/#entrypoint) - native support for entrypoint scripts

- [commands](https://docs.docker.com/compose/compose-file/compose-file-v3/#command) - native support for commands

- [init](https://docs.docker.com/compose/compose-file/compose-file-v3/#init) - tag is ignored and supported through cluster_init action hooks, which runs as a job.

- [container_name](https://docs.docker.com/compose/compose-file/compose-file-v3/#container_name) and [hostname](https://docs.docker.com/compose/compose-file/compose-file-v3/#domainname-hostname-ipc-mac_address-privileged-read_only-shm_size-stdin_open-tty-user-working_dir) - tags are ignored, Omnistrate has a fully managed DNS service that doesn't require any explicit configuration to work.

- [network](https://docs.docker.com/compose/compose-file/compose-file-v3/#network) and [port mapping](https://docs.docker.com/compose/compose-file/compose-file-v3/#ports) `host:container` - tag is ignored as Omnistrate auto-configures network and ports as part of the tenancy model.

- [tmpfs](https://docs.docker.com/compose/compose-file/compose-file-v3/#tmpfs) - Omnistrate already creates a default tmpfs volume

- [logging](https://docs.docker.com/compose/compose-file/compose-file-v3/#extension-fields) - Omnistrate offers its own managed logging system as integration that can be enabled by adding the `x-omnistrate-integrations` custom spec followed by the `omnistrateLogging` item

- [read only volumes](https://docs.docker.com/compose/compose-file/compose-file-v3/#volumes) - currently ignored

- [healthcheck](https://docs.docker.com/compose/compose-file/compose-file-v3/#healthcheck) - achieved via `x-omnistrate-action-hooks`

- [user](https://docs.docker.com/compose/compose-file/compose-file-v3/#domainname-hostname-ipc-mac_address-privileged-read_only-shm_size-stdin_open-tty-user-working_dir) definition - achieved via `SECURITY_CONTEXT` environment variables according to the UID and GID.

- Shared volumes (efs) - currently mapping the volume to the container path

- [ulimits](https://docs.docker.com/compose/compose-file/compose-file-v3/#ulimits)

- [Capabilities](https://docs.docker.com/compose/compose-file/compose-file-v3/#cap_add-cap_drop)

- [Sysctl](https://docs.docker.com/compose/compose-file/compose-file-v3/#sysctls)

- [labels](https://docs.docker.com/reference/compose-file/build/#labels) - allows you to specify custom metadata for resources, such as custom resource names or descriptions. For example:

  ```
  labels:
    - "name=Your Resource Name"
    - "description=Your Resource Description"
  ```

In addition, Omnistrate also supports several custom tags as follows.

### Custom Tags

#### x-omnistrate-mode-internal

`x-omnistrate-mode-internal` tag allows you to tag any Resource in the compose specification as internal.

Here is an example:

```
x-omnistrate-mode-internal: true
```

- Possible values are true or false.
- Default value is false
- If set to true, this Resource will be internal and won't be exposed to your customers.
- If set to false, the Resource will be exposed to your customers to configure and provision them

#### x-omnistrate-api-params

`x-omnistrate-api-params` allows you to define API params in addition to environment variables.

Here is an example:

```
x-omnistrate-api-params:
  - key: writerInstanceType
    description: Writer Instance Type
    name: Writer Instance Type
    type: String
    modifiable: true
    required: true
    export: true
```

- It contains collection of API parameters
- Each API parameter can be configured to specify the name, description, type and a few other properties.
- **Name**: Name of the parameter
- **Description**: Description for the parameter
- **Type**: Type of the parameter. The valid types are described on the [api parameters build guide](../api-params/#parameter-types)
- **Required**: specify if this field is a required parameter for the user of this SaaS Product
- **Export**: configures if this field will be returned as part of the describe call on this component to your users
- **Modifiable**: configure if this field is modifiable once configured
- **Default Value**: allows you to specify a default value for this field
- **Options**: allows you to specify a list of options for the user to choose from for this field
- **Labeled Options**: allows you to specify a list of options for the user to choose from for this field with a label
- **Limits**: allows you to specify the minimum and maximum values for this field
- **Regex**: allows you to specify a regex pattern to validate the input for this field

Here is an example that defines a parameter with options to choose from:

```
x-omnistrate-api-params:
  - key: writerInstanceType
    description: Writer Instance Type
    name: Writer Instance Type
    type: String
    modifiable: true
    required: true
    export: true
    options:
      - t4g.small
      - t4g.medium
      - t4g.large
```

Alternatively, you can label each option with a more user-friendly name:

```
x-omnistrate-api-params:
  - key: writerInstanceType
    description: Writer Instance Type
    name: Writer Instance Type
    type: String
    modifiable: true
    required: true
    export: true
    labeledOptions:
      Small: t4g.small
      Medium: t4g.medium
      Large: t4g.large
```

In addition, you can configure limits for a parameter. Here's an example for a DB password parameter (string) and a DB port parameter (integer):

```
x-omnistrate-api-params:
    - key: postgresqlRootPassword
      description: Postgresql Root Password
      name: Postgresql Root Password
      type: String
      modifiable: true
      required: true
      export: true
      limits:
        minLength: 8
        maxLength: 64

    - key: postgresqlPort
      description: Postgresql Port
      name: Postgresql Port
      type: Float64
      modifiable: true
      required: true
      export: true
      limits:
        min: 1024
        max: 65535
```

You can also define a default value for a parameter. Here's an example for a DB username parameter:

```
x-omnistrate-api-params:
    - key: postgresqlUsername
      description: Postgresql Username
      name: Postgresql Username
      type: String
      modifiable: true
      required: true
      export: true
      defaultValue: postgres
```

You can also define a regex pattern to validate the input for a parameter. Here's an example for a DB password parameter:

```
x-omnistrate-api-params:
    - key: postgresqlPassword
      description: Default DB Password
      name: Password
      type: String
      modifiable: false
      required: true
      export: false
      regex: ^[a-zA-Z0-9!@#$%^&*()_+]{8,32}$
```

Note

Please note that all the environment variables are **automatically** added as API parameters if `x-omnistrate-api-params` is not specified

#### x-omnistrate-actionhooks

`x-omnistrate-actionhooks` allows you to configure action hooks for a given Resource.

Here is an example:

```
x-omnistrate-actionhooks:
  - scope: CLUSTER
    type: INIT
    commandTemplate: >
      PGPASSWORD={{ $var.postgresqlRootPassword }} psql -U postgres
      -h writer {{ $var.postgresqlDatabase }} -c "create extension vector"
```

Action hooks allows you to inject custom code at different phase of the lifecycle of your control plane operations. As an example, in the above case, we are enabling vector extension for Postgres on cluster initialization

For more details and examples on action hooks, please see [here](../actionhooks/)

#### x-omnistrate-integrations

`x-omnistrate-integrations` allows you to add integrations to your service.

Here is an example:

```
x-omnistrate-integrations:
  - omnistrateLogging
  - omnistrateMetrics
```

In the above example, we are enabling omnistrate native metrics and logging extensions.

For the full list of integrations, please see [here](../integrations/)

#### x-omnistrate-compute

`x-omnistrate-compute` allows you to customize the compute parameters of a Resource.

You can customize the following:

- number of replicas
- instance type across cloud providers
- size of the root volume (GiB) across instances

Here is an example:

```
x-omnistrate-compute:
  replicaCountAPIParam: numReplicas
  instanceTypes:
    - cloudProvider: aws
      name: t4g.small
    - cloudProvider: gcp
      name: e2-medium
    - cloudProvider: azure
      name: Standard_B2als_v2
  rootVolumeSizeGi: 10
```

In the above example, `replicaCountAPIParam` is configured dynamically using `numReplicas` parameter i.e. the users of your service can decide how many replicas they want. In the same way `instanceTypes` can also be dynamic if you want your customers to specify the machine type among the list of options. For `rootVolumeSizeGi`, you can specify any integer between 10 and 16384 (up to 16 TB).

Note that the actual value of each of the fields could be static or dynamic.

#### x-omnistrate-storage

`x-omnistrate-storage` allows you to customize the storage parameters of a Resource.

Here is an example:

```
x-omnistrate-storage:
  aws:
    instanceStorageType: AWS::EBS_GP3
    instanceStorageSizeGi: 100
    instanceStorageIOPSAPIParam: instanceStorageIOPS
    instanceStorageThroughputAPIParam: instanceStorageThroughput
  gcp:
    instanceStorageType: GCP::PD_BALANCED
    instanceStorageSizeGi: 100
  azure:
    instanceStorageType: AZURE::STANDARD_SSD
    instanceStorageSizeGi: 100
```

You can customize the following:

- storage type from block device to blobs or both
- size of the volume
- storage IOPS
- storage throughput

Like above, the actual value of each of the fields could be static or dynamic.

#### x-omnistrate-capabilities

`x-omnistrate-capabilities` allows you to add capabilities to your Resources.

Here is an example:

```
x-omnistrate-capabilities:
  httpReverseProxy:
    targetPort: 80
  enableMultiZone: true
  enableEndpointPerReplica: true
  autoscaling:
    maxReplicas: 1
    minReplicas: 1
  serverlessConfiguration:
    enableAutoStop: true
    minimumNodesInPool: 5
    targetPort: 3306
```

To learn more about capabilities, please see [this page](../../runtime-guides/overview/)

#### x-omnistrate-load-balancer

`x-omnistrate-load-balancer` allows you to configure a load balancer for your Resources in the compose specification file adding L7-L4 capabilities.

Adding L7 load balancing capabilities to your Resources allows to route traffic based on the URL path, while L4 routes traffic based on the port number.

Here is an example:

```
version: "3"
x-omnistrate-integrations:
  - omnistrateLogging
  - omnistrateMetrics
x-omnistrate-load-balancer:
  https:
    - name: PGAdmin
      description: L7 Load Balancer for PGAdmin - New
      paths:
        - associatedResourceKey: admin
          path: /
          backendPort: 80

  tcp:
    - name: Writer
      description: L4 Load Balancer for Writer
      ports:
        - associatedResourceKeys:
            - writer
          ingressPort: 5432
          backendPort: 5432

services:
  Admin:
    ...

  Writer:
    ...

  Reader:
    ...
```

#### x-omnistrate-service-plan

`x-omnistrate-service-plan` allows you to configure the Plan for your SaaS in the compose specification file.

Here is an example:

```
x-omnistrate-service-plan:
  name: 'Mysql Free Tier'
  tenancyType: 'OMNISTRATE_DEDICATED_TENANCY'
  deployment:
    hostedDeployment:
      awsAccountId: 'xxxxxxxxxxx'
      awsBootstrapRoleAccountArn: 'arn:aws:iam::xxxxxxxxxxx:role/omnistrate-bootstrap-role'
      gcpProjectId: 'test-account'
      gcpProjectNumber: 'xxxxxxxxxxx3'
      gcpServiceAccountEmail: 'bootstrap.service@gcp.test.iam'
      azureSubscriptionId: 'xxxxxxxx-xxxx-xxx-xxxx-xxxxxxxxxx'
      azureTenantId: 'xxxxxxxx-xxxx-xxx-xxxx-xxxxxxxxxx'
  pricing:
    - dimension: cpu
      unit: cores
      timeUnit: hour
      price: 0.01
    - dimension: memory
      unit: GiB
      timeUnit: hour
      price: 0.05
    - dimension: storage
      unit: GiB
      timeUnit: hour
      price: 0.02
    - dimension: replica
      timeUnit: hour
      price: 0.10
  metering:
    s3BucketARN: arn:aws:s3:::my_billing_bucket_name
    s3BucketRegion: us-west-2
    gcsBucketName: my-billing-bucket-name
  billingProviders:
    - name: stripe
      externalProductID: "prod_123"
      enablePaywall: false
      isDefault: true
    - name: BYO Billing Provider
  maxNumberOfInstancesAllowed: 10
```

In the above example, we are configuring the Plan for the SaaS Product. The Plan contains the following fields:

- `name`: The name of the Plan.
- `tenancyType`: The tenancy type of the Plan. Possible options are:
- `OMNISTRATE_DEDICATED_TENANCY`: The infrastructure provisioned for a deployment (VMs) are dedicated to a single customer deployment.
- `OMNISTRATE_MULTI_TENANCY`: The infrastructure provisioned for a deployment (VMs) are shared among multiple customers and deployments.
- `CUSTOM_TENANCY`: Infrastructure is provisioned by Omnistrate but the affinity of the deployment to the infrastructure is configured in the Helm Chart / Operator configuration.
- `deployment`: The deployment type of the Plan. Possible options are:
- `hostedDeployment`: The Plan is deployed on your account.
- `byoaDeployment`: The Plan is deployed on your customer's account.
- Omitted: Omnistrate will host the Plan for you.
- `pricing`: The usage-based pricing model of the Plan. It will be used to generate invoices at the end of the month. Make sure to enable tenant billing first before using this feature. For a detailed guide, see [here](../../fin-ops-guides/billing/). Please specify the pricing for the following dimensions:
- `cpu`: The pricing is based on the CPU usage.
  - `unit`: (Optional) For now we only support `cores`. Defaults to `cores` if omitted.
  - `timeUnit`: (Optional) For now we only support `hour`. Defaults to `hour` if omitted.
- `memory`: The pricing is based on the amount of memory (RAM) used.
  - `unit`: (Optional) For now we only support `GiB`. Defaults to `GiB` if omitted.
  - `timeUnit`: (Optional) For now we only support `hour`. Defaults to `hour` if omitted.
- `storage`: The pricing is based on the amount of allocated storage.
  - `unit`: (Optional) For now we only support `GiB`. Defaults to `GiB` if omitted.
  - `timeUnit`: (Optional) For now we only support `hour`. Defaults to `hour` if omitted.
- `replica`: The pricing is based on the Replica usage.
  - `unit`: (Optional) For now we only support `replicas`. Defaults to `replicas` if omitted.
  - `timeUnit`: (Optional) For now we only support `hour`. Defaults to `hour` if omitted.
- `metering`: The metering configuration of the Plan. You can meter your infrastructure to capture the usage per customer, aggregate and store them at a defined location in your account. For a detailed step-by-step guide on configuring your bucket with the appropriate policy for storing metering data, please see [here](../../fin-ops-guides/billing/).
- `s3BucketARN`: The ARN of the S3 bucket where the metering data will be stored.
- `s3BucketRegion`: The region of the S3 bucket. e.g. `us-east-1`, `us-west-2`, etc.
- `gcsBucketName`: The name of the GCS bucket where the metering data will be stored.
- `billingProviders`: Configure billing providers for your service plan. Each provider supports different payment methods and billing integrations. All specified providers must be enabled for tenant billing at the account level.
- `name`: The billing provider name. Use `stripe` (case-insensitive) for Stripe integration or the name you set for your bring-your-own (BYO) billing provider.
- `isDefault`: (Optional) Whether this provider is the default. Only one provider can be set as default. Defaults to `false`.
- `externalProductID`: (Stripe only) You can specify the billing product ID for your Plan. This can be used to identify the Plan in your billing system. The billing product ID is a unique identifier for the Plan and is used to track usage and generate invoices.
- `enablePaywall`: (Stripe only) Whether a valid payment method is required for your customers to create resource instances. Possible options are:
  - `true`: Your customers must provide a valid payment method before consuming the service.
  - `false`(default): Your customers can consume the service without providing a valid payment method.
- `maxNumberOfInstancesAllowed`: The maximum number of instances allowed per tenant. This limit is applied at the tenant organization level, e.g. each tenant organization can create up to `N` instances of this Plan. Possible options are:
- `-1`(default): No limit on the number of instances that can be created.
- `N`: Any positive integer.

Note

For `deployment`, please don't forget to replace the account numbers, project id and other information with your own account information

- It contains cloud provider config
- You can choose to specify one cloud providers or all the available cloud providers
- If you choose to only configure one cloud provider (say AWS), your service will only be launched in AWS

#### x-omnistrate-job-config

`x-omnistrate-job-config` allows you to configure a Resource as a job that runs one time during create, update, or modification operations of a deployment.

When a Resource is configured with job settings, it will execute once during the deployment lifecycle and complete its task. This is useful for initialization scripts, data migrations, setup tasks, or any one-time operations that need to occur as part of your SaaS Product deployment.

Here is an example:

```
services:
  hello-world:
    build:
      context: ./jobs/hello-world
      dockerfile: Dockerfile
    environment:
      RAY_ADDRESS: "ray://{{ $var.rayClusterAddress }}:10001"
      SCRIPT_PATH: "submit_job.py"
    deploy:
      resources:
        reservations:
          cpus: "0.1"
          memory: 256M
        limits:
          cpus: "0.5"
          memory: 1G
    privileged: true
    platform: linux/amd64
    volumes:
      - source: ./jobs/hello-world
        target: /app
        type: bind
      - source: ./tmp
        target: /tmp
        type: bind
    x-omnistrate-compute:
      replicaCountApiParam: numReplicas
    x-omnistrate-job-config:
      backoffLimit: 0
      activeDeadlineSeconds: 3600
```

In the above example, we are configuring a job-based Resource with the following job-specific configurations:

- `backoffLimit`: The number of retries before considering the job as failed. Set to `0` to disable retries.
- `activeDeadlineSeconds`: The maximum duration (in seconds) the job is allowed to run before it is terminated. In this example, the job will timeout after 1 hour (3600 seconds).

The job will execute once when:

- A new deployment is created
- An existing deployment is updated
- The deployment is modified (e.g., configuration changes)

This makes jobs ideal for:

- Database initialization and schema setup
- Data migration scripts
- Configuration validation
- External system integration setup
- Batch processing tasks

Note

Jobs are designed to run to completion and then terminate. They are not meant for long-running services. If you need a long-running service, use a regular Resource without the `x-omnistrate-job-config` extension.

#### x-omnistrate-image-registry-attribute

`x-omnistrate-image-registry-attributes` allows you to configure image registries

Here is an example:

```
x-omnistrate-image-registry-attributes:
  docker.io:
    auth:
      username: username
      password: password
```

Note

For private image registries, username and password are required. We will only download your images in your account (hosted mode) or in your customers' account (BYOA mode)
