# Applicational AutoScaling 
[back](../README.md)

For applicational scaling we would use nomad sherpa autoscaler since is ideal for smaller, development or KISS setups, it runs on an interval time and scrape's resource usage of all job groups which have an active scaling policy.

To set scaling policies at the hcl metadata level we can define the following settings:

```
meta {
    sherpa_enabled = true
    sherpa_max_count = 4
    sherpa_min_count = 1
    sherpa_scale_out_count = 1
    sherpa_scale_in_count = 1
    sherpa_scale_out_cpu_percentage_threshold = 75
    sherpa_scale_out_memory_percentage_threshold = 75
    sherpa_scale_in_cpu_percentage_threshold = 30
    sherpa_scale_in_memory_percentage_threshold = 30
    }
```

In this case the autoscaler will iterate stored policies, performing calculations to figure out  job group CPU and memory usage and for the example if consumption is above 75% or below 30%, the scaler will request scaling of the job group as long as the scaling policies limits are not violated.


## Scale API and external metrics.

There is also available an API that we can use to scale jobs based on custom metrics, for that we need to build a process that if we reach certain number of conditions an api call is triggered in order to scale the job out or in.


| Method   | Path                         |
| :--------------------------- | :--------------------- |
| `PUT`    | `/v1/scale/out/:job_id/:group`              | `200 application/binary` |

#### Parameters

* `:job_id` (string: required) - Specifies the ID of the job and is specified as part of the path.
* `:group` (string: required) - Specifies the group name within the job and is specified as part of the path.
* `count` (int: 0) - Specifies the count which to scale the job group by. If this is not passed, Sherpa will attempt to use the value within the scaling policy.

### Sample Request

```
$ curl \
    --request PUT \
    --data @payload.json \
    http://127.0.0.1:8000/v1/scale/out/my-app/my-app-group?count=2
```

