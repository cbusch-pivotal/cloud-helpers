# Helps for Pivotal Cloud Foundry Healthwatchs

Helpers to view and alter [Pivotal Cloud Foundry Healthwatch](https://docs.pivotal.io/pcf-healthwatch/1-3/) alert and free chunks via its API.

## Healthwatch API documentation
- [API Index](https://docs.pivotal.io/pcf-healthwatch/1-3/api/index.html)
- [Alerts](https://docs.pivotal.io/pcf-healthwatch/1-3/api/alerts.html)
- [Canary App Health Endpoint](https://docs.pivotal.io/pcf-healthwatch/1-3/api/canary-configuration.html)
- [Free Chunks](https://docs.pivotal.io/pcf-healthwatch/1-3/api/free-chunks.html)


## HEALTHWATCH UAA client

Create a UAA client for Healthwatch necessary for the _Bearer Token_ used in API commands.

1. Target the PAS UAA 

Use `--skip-ssl-validation` as necessary.

```bash
$ uaac target uaa.system.DOMAIN.COM
```

2. Get the client context (login)

```bash
$ uaac token client get admin -s <PAS:UAA:ADMIN_CLIENT_SECRET>
```

3. Create a `healthwatch_admin` client for alert and chunk changes.

```bash
$ uaac client add healthwatch_admin \
  --name healthwatch_admin \
  --secret <PASSWORD> \
  --authorized_grant_types client_credentials,refresh_token \
  --authorities healthwatch.admin,healthwatch.read
  
scope: uaa.none
client_id: healthwatch_admin
resource_ids: none
authorized_grant_types: refresh_token client_credentials
autoapprove:
authorities: healthwatch.read healthwatch.admin
name: healthwatch_admin
required_user_groups:
lastmodified: 1541001384000
id: healthwatch_admin
```

## Retrieve client token

Get the `healthwatch_admin` client token for use in API commands and store in the environment.

1. Get token

```bash
$ uaac token client get healthwatch_admin -s <PASSWORD>
```

2. Save bearer token

```bash
$ export token=$(uaac context | grep access_token | awk '{print $2}')
```

## Working with Free Chunks

View and Change the _Free Chunks_ configuration via [Healthwatch API](https://docs.pivotal.io/pcf-healthwatch/1-3/api/free-chunks.html#get).

1. Get the free chunk configuration

```json
$ curl -G healthwatch-api.system.DOMAIN.COM/v1/free-chunks \
     -H "Authorization: Bearer ${token}"

[{"id":1,"deployment":"healthwatch-default","value":4096.0}]
```

2. Change the free chunk configuration

```json
$ curl healthwatch-api.system.DOMAIN.COM/v1/free-chunks \
     --data "{\"deployment\":\"healthwatch-default\",\"value\":3072.0}" \
     -H "Authorization: Bearer ${token}" \
     -H "Content-Type: application/json"
```

Ensure the change went through

```json
$ curl -G healthwatch-api.system.DOMAIN.COM/v1/free-chunks \
     -H "Authorization: Bearer ${token}"

[{"id":1,"deployment":"healthwatch-default","value":3072.0}]
```

__... or with jq formatting__

```json
$ curl -G healthwatch-api.system.DOMAIN.COM/v1/free-chunks \
     -H "Authorization: Bearer ${token}" | jq .
[
    {
        "id": 1,
        "deployment": "healthwatch-default",
        "value": 3072
    }
]
```

## View _-all-_ Alert configurations


```json
$ curl -G healthwatch-api.system.DOMAIN.COM/v1/alert-configurations \
     -H "Authorization: Bearer ${token}" | jq .

[
    {
        "query": "origin == 'mysql' and name == '/mysql/available' and deployment == 'cf' and job == 'mysql,database'",
        "threshold": {
            "critical": 1.0,
            "type": "EQUALITY"
        }
    },
    ...
```

<details>
<summary>expand for full response...</summary>
<p>

```json
...
    {
        "query": "origin == 'mysql' and name == '/mysql/galera/wsrep_ready' and deployment == 'cf' and job == 'mysql,database'",
        "threshold": {
            "critical": 0.0,
            "warning": 0.9998999834060669,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'locket' and name == 'ActiveLocks'",
        "threshold": {
            "critical": 4.0,
            "type": "EQUALITY"
        }
    },
    {
        "query": "origin == 'locket' and name == 'ActivePresences'",
        "threshold": {
            "critical": 200.0,
            "warning": 150.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'auctioneer' and name == 'AuctioneerFetchStatesDuration'",
        "threshold": {
            "critical": 5.0E9,
            "warning": 2.0E9,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'auctioneer' and name == 'AuctioneerLRPAuctionsFailed'",
        "threshold": {
            "critical": 1.0,
            "warning": 0.5,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'auctioneer' and name == 'AuctioneerLRPAuctionsStarted'",
        "threshold": {
            "critical": 100.0,
            "warning": 50.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'auctioneer' and name == 'AuctioneerTaskAuctionsFailed'",
        "threshold": {
            "critical": 1.0,
            "warning": 0.5,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'gorouter' and name == 'backend_exhausted_conns'",
        "threshold": {
            "critical": 10.0,
            "warning": 5.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'gorouter' and name == 'bad_gateways'",
        "threshold": {
            "critical": 40.0,
            "warning": 30.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'bbs' and name == 'ConvergenceLRPDuration'",
        "threshold": {
            "critical": 2.0E10,
            "warning": 1.0E10,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'bbs' and name == 'CrashedActualLRPs'",
        "threshold": {
            "critical": 20.0,
            "warning": 10.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'Diego.AppsDomainSynced'",
        "threshold": {
            "critical": 1.0,
            "type": "EQUALITY"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'Diego.AvailableFreeChunks'",
        "threshold": {
            "critical": 1.0,
            "warning": 2.0,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'Diego.LRPsAdded.1H'",
        "threshold": {
            "critical": 100.0,
            "warning": 50.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'Diego.TotalAvailableDiskCapacity.5M'",
        "threshold": {
            "critical": 6144.0,
            "warning": 12288.0,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'Diego.TotalAvailableMemoryCapacity.5M'",
        "threshold": {
            "critical": 32768.0,
            "warning": 65536.0,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'Diego.TotalPercentageAvailableContainerCapacity.5M'",
        "threshold": {
            "critical": 0.3499999940395355,
            "warning": 0.4000000059604645,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'Diego.TotalPercentageAvailableDiskCapacity.5M'",
        "threshold": {
            "critical": 0.3499999940395355,
            "warning": 0.4000000059604645,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'Diego.TotalPercentageAvailableMemoryCapacity.5M'",
        "threshold": {
            "critical": 0.3499999940395355,
            "warning": 0.4000000059604645,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'Doppler.MessagesAverage.1M'",
        "threshold": {
            "critical": 1000000.0,
            "warning": 800000.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'gorouter' and name == 'file_descriptors'",
        "threshold": {
            "critical": 60000.0,
            "warning": 50000.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'Firehose.LossRate.1M'",
        "threshold": {
            "critical": 0.009999999776482582,
            "warning": 0.004999999888241291,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'Galera.ClusterStatusSum'",
        "threshold": {
            "critical": 0.9998999834060669,
            "warning": 2.9999001026153564,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'Galera.TotalPercentageHealthyNodes'",
        "threshold": {
            "critical": 0.33320000767707825,
            "warning": 0.9998999834060669,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'health.check.bosh.director.probe.available'",
        "threshold": {
            "critical": 0.4000000059604645,
            "warning": 0.6000000238418579,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'health.check.bosh.director.success'",
        "threshold": {
            "critical": 1.0,
            "type": "EQUALITY"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'health.check.CanaryApp.available'",
        "threshold": {
            "critical": 0.5,
            "warning": 0.5,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'health.check.CanaryApp.probe.available'",
        "threshold": {
            "critical": 0.4000000059604645,
            "warning": 0.6000000238418579,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'health.check.CanaryApp.responseTime'",
        "threshold": {
            "critical": 30000.0,
            "warning": 15000.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'health.check.cliCommand.delete'",
        "threshold": {
            "critical": 1.0,
            "type": "EQUALITY"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'health.check.cliCommand.login'",
        "threshold": {
            "critical": 1.0,
            "type": "EQUALITY"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'health.check.cliCommand.logs'",
        "threshold": {
            "critical": 1.0,
            "type": "EQUALITY"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'health.check.cliCommand.probe.available'",
        "threshold": {
            "critical": 0.4000000059604645,
            "warning": 0.6000000238418579,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'health.check.cliCommand.push'",
        "threshold": {
            "critical": 1.0,
            "type": "EQUALITY"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'health.check.cliCommand.pushTime'",
        "threshold": {
            "critical": 120000.0,
            "warning": 60000.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'health.check.cliCommand.start'",
        "threshold": {
            "critical": 1.0,
            "type": "EQUALITY"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'health.check.cliCommand.stop'",
        "threshold": {
            "critical": 1.0,
            "type": "EQUALITY"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'health.check.OpsMan.available'",
        "threshold": {
            "critical": 0.5,
            "warning": 0.5,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'health.check.OpsMan.probe.available'",
        "threshold": {
            "critical": 0.4000000059604645,
            "warning": 0.6000000238418579,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'healthwatch.ui.available'",
        "threshold": {
            "critical": 0.4000000059604645,
            "warning": 0.6000000238418579,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'ingestor.disconnects'",
        "threshold": {
            "critical": 10.0,
            "warning": 5.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'ingestor.dropped'",
        "threshold": {
            "critical": 20.0,
            "warning": 10.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'ingestor.ingested'",
        "threshold": {
            "critical": 0.009999999776482582,
            "warning": 0.009999999776482582,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'ingestor.ingested.boshSystemMetrics'",
        "threshold": {
            "critical": 0.009999999776482582,
            "warning": 0.009999999776482582,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'gorouter' and name == 'latency'",
        "threshold": {
            "critical": 150.0,
            "warning": 100.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'gorouter' and name == 'latency.uaa'",
        "threshold": {
            "critical": 150.0,
            "warning": 100.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'auctioneer' and name == 'LockHeld'",
        "threshold": {
            "critical": 1.0,
            "type": "EQUALITY"
        }
    },
    {
        "query": "origin == 'bbs' and name == 'LockHeld'",
        "threshold": {
            "critical": 1.0,
            "type": "EQUALITY"
        }
    },
    {
        "query": "origin == 'bbs' and name == 'LRPsExtra'",
        "threshold": {
            "critical": 10.0,
            "warning": 5.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'bbs' and name == 'LRPsMissing'",
        "threshold": {
            "critical": 10.0,
            "warning": 5.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'metrics.published'",
        "threshold": {
            "critical": 0.0,
            "warning": 20.0,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'gorouter' and name == 'ms_since_last_registry_update'",
        "threshold": {
            "critical": 30000.0,
            "warning": 30000.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'redis.counterEventQueue.size'",
        "threshold": {
            "critical": 10000.0,
            "warning": 10000.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'redis.valueMetricQueue.size'",
        "threshold": {
            "critical": 10000.0,
            "warning": 10000.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'rep' and name == 'RepBulkSyncDuration'",
        "threshold": {
            "critical": 1.0E10,
            "warning": 5.0E9,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'bbs' and name == 'RequestLatency'",
        "threshold": {
            "critical": 1.0E10,
            "warning": 5.0E9,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'gorouter' and name == 'responses.5xx'",
        "threshold": {
            "critical": 40.0,
            "warning": 30.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'route_emitter' and name == 'RouteEmitterSyncDuration'",
        "threshold": {
            "critical": 1.0E10,
            "warning": 5.0E9,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'RouteRegistration.MessagesDelta'",
        "threshold": {
            "critical": 50.0,
            "warning": 30.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'uaa' and name == 'server.inflight.count'",
        "threshold": {
            "critical": 200.0,
            "warning": 150.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'SyslogDrain.Adapter.BindingsAverage.5M'",
        "threshold": {
            "critical": 250.0,
            "warning": 200.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'SyslogDrain.Adapter.LossRate.1M'",
        "threshold": {
            "critical": 0.10000000149011612,
            "warning": 0.009999999776482582,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'SyslogDrain.RLP.LossRate.1M'",
        "threshold": {
            "critical": 0.10000000149011612,
            "warning": 0.009999999776482582,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'bosh-system-metrics-forwarder' and name == 'system.cpu.user'",
        "threshold": {
            "critical": 95.0,
            "warning": 85.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'bosh-system-metrics-forwarder' and name == 'system.cpu.user' and job == 'router'",
        "threshold": {
            "critical": 70.0,
            "warning": 60.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'bosh-system-metrics-forwarder' and name == 'system.cpu.user' and job == 'uaa,control'",
        "threshold": {
            "critical": 90.0,
            "warning": 80.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'bosh-system-metrics-forwarder' and name == 'system.disk.ephemeral.percent'",
        "threshold": {
            "critical": 90.0,
            "warning": 80.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'bosh-system-metrics-forwarder' and name == 'system.disk.persistent.percent'",
        "threshold": {
            "critical": 90.0,
            "warning": 80.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'bosh-system-metrics-forwarder' and name == 'system.disk.system.percent'",
        "threshold": {
            "critical": 90.0,
            "warning": 80.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'bosh-system-metrics-forwarder' and name == 'system.healthy'",
        "threshold": {
            "critical": 0.4000000059604645,
            "warning": 0.6000000238418579,
            "type": "LOWER"
        }
    },
    {
        "query": "origin == 'bosh-system-metrics-forwarder' and name == 'system.mem.percent'",
        "threshold": {
            "critical": 95.0,
            "warning": 85.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'gorouter' and name == 'total_requests'",
        "threshold": {
            "critical": 125000.0,
            "warning": 100000.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'gorouter' and name == 'total_routes'",
        "threshold": {
            "critical": 200.0,
            "warning": 100.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'healthwatch' and name == 'uaa.throughput.rate'",
        "threshold": {
            "critical": 15000.0,
            "warning": 12000.0,
            "type": "UPPER"
        }
    },
    {
        "query": "origin == 'rep' and name == 'UnhealthyCell'",
        "threshold": {
            "critical": 0.0,
            "type": "EQUALITY"
        }
    }
]
```
</p>
</details>

## View a _-specific-_ ALERT configuration

```json
$ curl -G "healthwatch-api.system.DOMAIN.COM/v1/alert-configurations"  \
    --data-urlencode "q=origin == 'gorouter' and name == 'total_requests'" \
    -H "Authorization: Bearer ${token}"

[
    {
        "query": "origin == 'gorouter' and name == 'total_requests'",
        "threshold": {
        "critical": 125000,
        "warning": 100000,
        "type": "UPPER"
        }
    }
]
```

## Update a _-specific-_ ALERT configuration

```json
$ curl "healthwatch-api.system.DOMAIN.COM/v1/alert-configurations"  \
    --data "{\"query\":\"origin == 'gorouter' and name == 'total_requests'\", \
             \"threshold\":{\"critical\": 120000,\"warning\":99999,\"type\":\"UPPER\"}}" \
    -H "Authorization: Bearer ${token}" \
    -H "Content-Type: application/json"
    
{"query":"origin == 'gorouter' and name == 'total_requests'","threshold":{"type":"UPPER","critical":120000.0,"warning":99999.0}}
```

Check to ensure data was set.

```json
$ curl -G "healthwatch-api.system.DOMAIN.COM/v1/alert-configurations"  \
    --data-urlencode "q=origin == 'gorouter' and name == 'total_requests'" \
    -H "Authorization: Bearer ${token}" | jq .
[
  {
    "query": "origin == 'gorouter' and name == 'total_requests'",
    "threshold": {
      "critical": 120000,
      "warning": 99999,
      "type": "UPPER"
    }
  }
]
```
