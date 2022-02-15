# Blameless SLO YAML specs and Samples

*Preview of the Blameless SLO YAML specs*


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



**Resources types**

```yaml
apiVersion: blameless/v1alpha
kind: SLI | Service | SLO | UserJourney | ErrorBudgetPolicy
```

### SLIs

-   SLIs are created first and independently from SLOs.

-   Blameless supports four types of SLIs (`sliType` parameter):

    1. Availability
    2. Latency
    3. Throughput
    4. Saturation

-   Availability SLIs are ratio types of metrics (`ratioMetric`), while the other SLI types are threshold based metrics (`ratioMetric`).

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
queryType: query | metricname
query: <actual query> | <name or path to the metric>
```

For `datadog` and `pingdom`, `metricname` must be selected, otherwise select `query`.

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

-   SLOs are grrouped into a Blameless resource called "UserJourney"

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

### User Journey

| Property    | Description |
| ----------- | ----------- |
| Name        | Required    |
| Description | Optional    |
| Owner       | Required    |

The `Owner` is currently a required parameter.




## Blameless SLO CLI (PREVIEW)

The Blameless SLO CLI supports the following commands
* blameless-slo validate
* blameless-slo deploy
* blameless-slo delete



### Validate

The `validate` command is used to validate `.yaml` specs before deploying the defined SLO resources in Blameless SLO Manager.

```jsx
 blameless-slo validate -s <source> -f <path_to_yaml>
```

Options:
   * **`<source>`**: local | github
   * **`<path_to_yaml>`**: Path to your .yaml file or folder containing multiple .yaml files.

Examples:
```jsx
 blameless-slo validate -s github -f sample1
```


Note: Your targeted Github repository is setup during the initialization of your blameless-slo CLI with the appropriate Github (Personal Access Token) and your Blameless credentials. More information to be provided on this later.



#### Validation output

There are tree types of validations messages: 

- :heavy_check_mark: `[BLAMELESS] SUCCESS`
- :warning: `[BLAMELESS] WARNING`
- :x: `[BLAMELESS] ERROR`

**Examples:**

Example when the validation completed successfully for a selected YAML spec:
```bat
blameless-slo validate -s local -f ./specs/user_journey.yaml
```

:heavy_check_mark: **[BLAMELESS] SUCCESS : ========== Blameless UserJourney Validated Successfully ==========**.

Example when the source type is incorrect (typo):
```bat
blameless-slo validate -s locals -f ./specs/
```

:warning: **[BLAMELESS] WARNING : Please specify type: Allowed options are: github | local**.


Example: Showing the list of syntax errors in the YAML spec 
```bat
blameless-slo validate -s local -f ./specs/sli.yaml
```

:x: **[BLAMELESS] ERROR : ========== Blameless SLI Validation Errors ==========**. <br />
[BLAMELESS] ERROR : 1: "spec.metricSource.mode" must be one of [direct, gcp, lambda] <br />
[BLAMELESS] ERROR : 2: "spec.metricSource.sourceName" is required <br />
[BLAMELESS] ERROR : 3: "spec.metricSource.name" is not allowed <br />



### Deploy

The `deploy` command is used to create/update SLO resources in Blameless. When you execute that command, it first validates the specified yaml files, and if everything is okay, than it will proceed further by deploying resources.

```jsx
 blameless-slo deploy -s <source> -f <path_to_yaml>
```

Options:

   * **`<source>`**: local | github
   * **`<path_to_yaml>`**: Path to your .yaml file or folder containing multiple .yaml files.

