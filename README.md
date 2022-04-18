# Blameless SLO YAML specs and Samples


The latest specification documentation can be found under the [/spec](https://github.com/blamelesshq/blameless-slo-samples/tree/main/specs) directory. 

For more information about **Blameless SLO Manager**

-   https://docs.blameless.com/features/slo/slo-guide-intro

For more information about the Blameless SLO API Endpoints

-   [Blameless SLO API doc](https://blameless.blameless.io/api/v1/services/SLOService/)
-   [Blameless SLO Timeseries docs](https://blameless.blameless.io/api/v1/services/SLOTimeSeriesService/doc)



## The Blameless SLO data model

There are 5 resource types in 2 categories:

1. **Objectives**

-   User Journey (groups of SLOs)
-   SLO
-   Service (group of SLIs)
-   SLI

2. **Notification**

-   Error Budget Policy


### Resources types

```yaml
apiVersion: blameless/v1alpha
kind: SLI | Service | SLO | UserJourney | ErrorBudgetPolicy
```

### Naming resources

* SLO resources are uniquely identified by their name (string) provided in each YAML specification. The name of a resource is specified under the `metadata.name` property.
```yaml
metada:
    name: string # unique name for resource
```

* SLO resources refer to each other using this unique name. For example, an SLI must refer to a service by its name under `metadata.service`:
```yaml
metadata:
    service: string     # name of the service to associate this SLI with
```

* Naming resources

### SLIs

-   SLIs are created first and independently from SLOs.

-   Blameless supports four types of SLIs (`sliType` parameter):

    1. Availability
    2. Latency
    3. Throughput
    4. Saturation

-   Availability SLIs are ratio types of metrics (`ratioMetric`), while the other SLI types are threshold based metrics (`thresholdMetric`).

-   SLIs are grouped in a Blameless resource called `Service`.

**Structure of a metric**

```yaml
source: string # data source
queryType: string # a name for the type of query to run on the data source
query: string # the query to run to return a metric
metadata: # optional, allows data source specific details to be passed
```

**Possible values**

```yaml
source: prometheus | newrelic | datadog | pingdom
queryType: query
query: <actual query> | <pingdom check id>           # must be a string
```

For pingdom, the check id of the metric should be provided for now. It can be found in Pingdom in the URL pointing to the metric:

> Example: `my.pingdom.com/reports/uptime#checkid=1234567`

For Availability type SLIs, 2 metrics must be specified:

```yaml
ratioMetric:
    good: # the numerator
        source: string     # data source for the "good" numerator
        queryType: string  # a name for the type of query to run on the data source
        query: string      # the query to run to return the numerator
        metadata:          # optional, allows data source specific details to be passed
    total: # the denominator
        source: string     # data source for the "total" denominator
        queryType: string  # a name for the type of query to run on the data source
        query: string      # the query to run to return the denominator
        metadata:          # optional, allows data source specific details to be passed
```

For other SLI types, only 1 metric must be specified

```yaml
thresholdMetric:           # represents the metric used to inform the SLO in the objectives stanza
    source: string         # data source for the metric
    queryType: string      # a name for the type of query to run on the data source
    query: string          # the query to run to return the metric
    metadata:              # optional, allows data source specific details to be passed
```

### SLOs

-   One or more SLOs can be created on the same SLI.

-   SLOs are grouped into a Blameless resource called "UserJourney"

#### SLO status

The SLO status is a required SLO setting which indicates if the attached Error Budget Policy (EPB) is active and if the SLO can be edited or not.

| SLO status  | EBP active | Editable |
| ----------- | ---------- | -------- |
| development | no         | yes      |
| testing     | yes        | yes      |
| active      | yes        | no       |

### Service

| Property    | Description |
| ----------- | ----------- |
| Name        | Required    |
| Description | Required    |
| Notes       | Optional    |
| Owner       | Required    |

### User Journey

| Property    | Description |
| ----------- | ----------- |
| Name        | Required    |
| Description | Optional    |
| Owner       | Required    |

The `Owner` is currently a required parameter.




## Blameless SLO CLI

The Blameless SLO CLI supports the following commands to manage your [SLO resources](#the-blameless-slo-data-model) in your own Blameless instance.
```unix
$ blameless-slo validate
$ blameless-slo deploy
$ blameless-slo delete
```

* Using the [deploy](#deploy) command option, You can create and update multiple SLO resources at the same time by grouping your SLO YAML files under the same directory or one SLO resource (1 yaml file) at a time.

* As you develop your SLO specifications for your services, you can use the [validate](#validate) command option prior to deploying to your Blameless instance.

* If you need to remove your SLO resources from Blameless SLO Manager, you can use the [delete](#delete) command option.

* If you have an SLO resource refering to a resource which does not exist in your Blameless instance under SLO Manager (e.g. SLI --> Service or SLO --> User Journey), the Blameless SLO CLI will report the issue and it will not create the new SLO resource.

* Check the [Naming resources](#naming-resources) section to make sure you are appropriately naming your resources before you create or update SLO resources in SLO Manager on your Blameless instance.


### Install the CLI

The Blameless SLO CLI is publicly available as an [NPM package](https://www.npmjs.com/package/@blamelesshq/blameless-slo) which can be installed as followed:
```unix
$ npm install -g @blamelesshq/blameless-slo
```

### Initial setup

> Contact Blameless Customer Success ([cs@blameless.io](mailto:cs@blameless.io)) to obtain your `Client ID` and `Client Secret` which is associated to your Blameless instance: <instance_name>.blameles.io

The following environment variables must be set, before you can execute the `blameless-slo` command:
```unix
$ export BLAMELESS_OAUTH_CLIENT_ID=<contact_blameless_cs>
$ export BLAMELESS_OAUTH_CLIENT_SECRET=<contact_blameless_cs>
$ export BLAMELESS_OAUTH_AUDIENCE=<instance_name>.blameless.io
$ export BLAMELESS_TENANT_DOMAIN=<instance_name>.blameless.io
```

You will be reminded to set those environment variables if one or more of those have not been set:
```unix
$ blameless-slo

[BLAMELESS] ERROR : Config is missing. Please set correct config values via environment variables.
         Required Env Vars:
         BLAMELESS_OAUTH_CLIENT_ID
         BLAMELESS_OAUTH_CLIENT_SECRET
         BLAMELESS_OAUTH_AUDIENCE
         BLAMELESS_TENANT_DOMAIN
```


### Validate

The `validate` command is used to validate `.yaml` specs before deploying the defined SLO resources in Blameless SLO Manager.

```jsx
blameless-slo validate -f <path_to_yaml>
```

Options:
   * **`<path_to_yaml>`**: Path to your .yaml file or folder containing multiple .yaml files.


Examples:
```jsx
blameless-slo validate -f sample1
```



#### Validation output

There are tree types of validations messages: 

- :heavy_check_mark: `[BLAMELESS] SUCCESS`
- :warning: `[BLAMELESS] WARNING`
- :x: `[BLAMELESS] ERROR`

**Examples:**

Example when the validation completed successfully for a selected YAML spec:
```bat
blameless-slo validate -f ./specs/user_journey.yaml
```

:heavy_check_mark: **[BLAMELESS] SUCCESS : ========== Blameless UserJourney Validated Successfully ==========**.

Example when the option is incorrect (typo):
```bat
blameless-slo validate -e ./specs/
error: required option '-f,--filePath <file_name>' not specified
```

Example: Showing the list of syntax errors in the YAML spec 
```bat
blameless-slo validate -f ./specs/sli.yaml
```

:x: **[BLAMELESS] ERROR : ========== Blameless SLI Validation Errors ==========**. <br />
[BLAMELESS] ERROR : 1: "spec.metricSource.mode" must be one of [direct, gcp, lambda] <br />
[BLAMELESS] ERROR : 2: "spec.metricSource.sourceName" is required <br />
[BLAMELESS] ERROR : 3: "spec.metricSource.name" is not allowed <br />



### Deploy

The `deploy` command is used to create/update SLO resources in Blameless. When you execute that command, it first validates the specified yaml files, and if everything is okay, it will proceed further by deploying those resources.

* When multiple SLO resources are provided as input, the Blameless SLO CLI automatically makes sure to create resources in the following order in your Blamelesss instance: Service > SLI > User Journey > Error Budget > SLO.

* SLO resources are uniquely identified by the name provided in your SLO specification.

```jsx
 blameless-slo deploy -f <path_to_yaml>
```

Options:

   * **`<path_to_yaml>`**: Path to your .yaml file or folder containing multiple .yaml files.


### Delete

The `delete` command is used to delete SLO resources from Blameless SLO Manager. When you execute that command, it first validates the specified yaml files, and if everything is okay, it will proceed further by deleting those resources.

```jsx
 blameless-slo deploy -f <path_to_yaml>
```

Options:

   * **`<path_to_yaml>`**: Path to your .yaml file or folder containing multiple .yaml files.
