---
title: Convert TICKscript to Flux
description: >
  placeholder
menu:
  v2_0_ref:
    name: TICKscript to Flux
weight: 6
---

placeholder

{{% truncate %}}
| TICKscript Node                         | Flux equivalent                  |
| ---------------                         | ---------------                  |
| [batch](#batch)                         | from, range, filter              |
| [stream](#stream)                       | -                                |
| [alert](#alert)                         | monitor.notify                   |
| [barrier](#barrier)                     | -                                |
| [changeDetect](#changedetect)           | monitor.stateChanges             |
| [combine](#combine)                     | -                                |
| [default](#default)                     | map + if/else + exists operator  |
| [delete](#delete)                       | drop                             |
| [derivative](#derivative)               | derivative                       |
| [eC2Autoscale](#ec2autoscale)           | -                                |
| [eval](#eval)                           | map                              |
| [flatten](#flatten)                     | -                                |
| [from](#from)                           | from                             |
| [groupBy](#groupby)                     | group                            |
| [hTTPOut](#httpout)                     | -                                |
| [hTTPPost](#httppost)                   | http.post                        |
| [influxDBOut](#influxdbout)             | to                               |
| [influxQL](#influxql)                   | -                                |
| [join](#join)                           | join                             |
| [k8sAutoscale](#k8sautoscale)           | -                                |
| [kapacitorLoopback](#kapacitorloopback) | -                                |
| [log](#log)                             | -                                |
| [noOp](#noop)                           | -                                |
| [query](#query)                         | from                             |
| [sample](#sample)                       | time-based (window() + selector) |
| [shift](#shift)                         | timeShift                        |
| [sideload](#sideload)                   | map + variables                  |
| [stateCount](#statecount)               | stateCount                       |
| [stateDuration](#stateduration)         | stateDuration                    |
| [stats](#stats)                         | -                                |
| [swarmAutoscale](#swarmautoscale)       | -                                |
| [uDF](#udf)                             | only custom flux functions       |
| [union](#union)                         | union                            |
| [where](#where)                         | filter                           |
| [window](#window)                       | window                           |
{{% /truncate %}}

## Kapacitor Nodes

### batch
The Kapacitor [BatchNode](https://docs.influxdata.com/kapacitor/latest/nodes/batch_node/)
requires a [QueryNode](#query) and queries data from a specified time range.

Use the [`from()`](/v2.0/reference/flux/stdlib/built-in/inputs/from/),
[`range()`](/v2.0/reference/flux/stdlib/built-in/transformations/range/),
and [`filter()`](/v2.0/reference/flux/stdlib/built-in/transformations/filter/)
functions to accomplish the same thing.

{{< code-tabs-wrapper >}}
{{% code-tabs %}}
[Flux](#)
[TICKscript](#)
{{% /code-tabs %}}
{{% code-tab-content %}}
```js
from(bucket: "example-bucket")
  |> range(start: -10m)
  |> filter(fn: (r) => r._measurement == "errors" and r._field == "value" )
```
{{% /code-tab-content %}}

{{% code-tab-content %}}
```js
batch
  |query('SELECT value from db.rp.errors')
```
{{% /code-tab-content %}}
{{< /code-tabs-wrapper >}}

### stream
_Flux does not currently support streaming data._

### alert
The Kapacitor [AlertNode](https://docs.influxdata.com/kapacitor/latest/nodes/alert_node/)
defines

# **THIS NEEDS TO BE FINISHED**

```js
monitor.check()
monitor.deadman()
```

### barrier
*The Kapacitor [BarrierNode](https://docs.influxdata.com/kapacitor/latest/nodes/barrier_node/)
has no equivalent Flux function.*

### changeDetect
The Kapacitor [ChangeDetectNode](https://docs.influxdata.com/kapacitor/latest/nodes/change_detect_node/)
has no equivalent Flux function. However, the [`monitor.stateChanges`](/v2.0/reference/flux/stdlib/monitor/statechanges/)
does address the specific use case of detecting changes in a status or `_level`
set in the [InfluxDB 2.0 monitoring and alerting](/v2.0/monitor-alert/) workflow.
`monitor.stateChanges` only outputs rows whose `_level` value is different than
the previous row's.

```js
import "influxdata/influxdb/monitor"

monitor.from(start: -1h)
  |> monitor.stateChanges(fromLevel: "any", toLevel: "crit")
  //...
```

### combine
*The Kapacitor [CombineNode](https://docs.influxdata.com/kapacitor/latest/nodes/combine_node/)
has no equivalent Flux function.*

### default
The Kapacitor [DefaultNode](https://docs.influxdata.com/kapacitor/latest/nodes/default_node/)
sets fields and tags on data points if the don't exist.
Use the Flux `map()` function, conditional logic, and the `exists` operator to replicate
this functionality in Flux.

The example below checks to see if the `_field` column exists.
If it does, it preserves the `_field`, but if it doesn't, it sets the field as `defaultFieldKey`.
It then does the same to to set the default field value in the `_value` column.
It also illustrates how to set default tag keys and values.

{{< code-tabs-wrapper >}}
{{% code-tabs %}}
[Flux](#)
[TICKscript](#)
{{% /code-tabs %}}
{{% code-tab-content %}}
```js
// ...
  |> map(fn: (r) => ({
    r with
    _field: if exists r._field then r._field else "value",
    _value: if exists r._value then r._value else 0.0,
    host: if exists r.host then r.host else ""
  }))
```
{{% /code-tab-content %}}

{{% code-tab-content %}}
```js
// ...
  |default()
    .field('value', 0.0)
    .tag('host', '')
```
{{% /code-tab-content %}}
{{< /code-tabs-wrapper >}}

### delete
The Kapacitor [DeleteNode](https://docs.influxdata.com/kapacitor/latest/nodes/delete_node/)
deletes fields and tags from data points.
Use the Flux [`pivot()`](/v2.0/reference/flux/stdlib/built-in/transformations/pivot/)
function to pivot field values under their respective field key column names.
Then use [`drop()`](/v2.0/reference/flux/stdlib/built-in/transformations/drop/)
to drop fields and tags.

```js
// ...
  |> pivot(rowKey:["_time"], columnKey: ["_field"], valueColumn: "_value")
  |> drop(columns: ["field1", "field2", "tag1", "tag2"])
```

### derivative
The Kapacitor [DerivativeNode](https://docs.influxdata.com/kapacitor/latest/nodes/derivative_node/)
computes the derivative of a batch.
Use the [`derivative()`](/v2.0/reference/flux/stdlib/built-in/transformations/aggregates/derivative/)
function to replicate the behavior in Flux.

```js
// ...
  |> derivative(column: "_value", unit: 1s)
```

### eC2Autoscale
*The Kapacitor [EC2AutoscaleNode](https://docs.influxdata.com/kapacitor/latest/nodes/ec2_autoscale_node/)
has no equivalent Flux function.*

### eval
The Kapacitor [EvalNode](https://docs.influxdata.com/kapacitor/latest/nodes/eval_node/)
evaluates expressions on each data point it receives.
Use [`map()`](/v2.0/reference/flux/stdlib/built-in/transformations/map/) to replicate
the behavior in Flux.

```js
// ...
  |> map(fn: (r) => ({
      r with
      error_percent: r.error_count / r.total_count
    }))
```

### flatten
*The Kapacitor [FlattenNode](https://docs.influxdata.com/kapacitor/latest/nodes/flatten_node/)
has no equivalent Flux function.*

### from
The Kapacitor [FromNode](https://docs.influxdata.com/kapacitor/latest/nodes/from_node/)
selects a subset of the data flowing through a [StreamNode](#stream).
While Flux doesn't support

### groupBy

### hTTPOut

### hTTPPost

### influxDBOut

### influxQL

### join

### k8sAutoscale

### kapacitorLoopback

### log

### noOp

### query

### sample

### shift

### sideload

### stateCount

### stateDuration

### stats

### swarmAutoscale

### uDF

### union

### where

### window



## Kapacitor event handlers

Aggregrate
Alerta
Email
Exec
HipChat
Kafka
Log
MQTT
OpsGenie
PagerDuty
Post
Publish
Pushover
Sensu
Slack
SNMP Trap
Talk
TCP
Telegram
VictorOps
