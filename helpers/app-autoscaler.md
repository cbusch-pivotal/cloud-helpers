# APP AUTOSCALER

The Pivotal Cloud Foundry App Autoscaler plugin all the user to configure PAS (Pivotal Application Service) Autoscaler service instances.

## App Autoscaler Documentation
- [Scaling an Application Using App Autoscaler](https://docs.pivotal.io/pivotalcf/2-1/appsman-services/autoscaler/using-autoscaler.html)
- [Using the App Autoscaler CLI](https://docs.pivotal.io/pivotalcf/2-1/appsman-services/autoscaler/using-autoscaler-cli.html)


## Install the App Autoscaler plugin within the CF CLI
- Download the [CLI plugin](https://network.pivotal.io/products/pcf-app-autoscaler/) for your desktop environment (Mac, Linux, Windows, etc.)

__Install the plugin, as such for the Mac__

```bash
$ cf install-plugin autoscaler-for-pcf-cliplugin-macosx64-binary-2.0.40
Do you want to install the plugin autoscaler-for-pcf-cliplugin-macosx64-binary-2.0.40? [yN]: y
Installing plugin App Autoscaler...
OK
Plugin App Autoscaler 2.0.40 successfully installed.
```


## App Autoscaler CLI commands

```bash
$ cf plugins | grep 'App Autoscaler'
App Autoscaler  2.0.40  autoscaling-apps             Displays apps bound to the autoscaler
App Autoscaler  2.0.40  autoscaling-events           Displays previous autoscaling events for the app
App Autoscaler  2.0.40  autoscaling-rules            Displays rules for an autoscaled app
App Autoscaler  2.0.40  autoscaling-slcs             Displays scheduled limit changes for the app
App Autoscaler  2.0.40  configure-autoscaling        Configures autoscaling using a manifest file
App Autoscaler  2.0.40  create-autoscaling-rule      Create rule for an autoscaled app
App Autoscaler  2.0.40  create-autoscaling-slc       Create scheduled instance limit change for an autoscaled app
App Autoscaler  2.0.40  delete-autoscaling-rule      Delete rule for an autoscaled app
App Autoscaler  2.0.40  delete-autoscaling-rules     Delete all rules for an autoscaled app
App Autoscaler  2.0.40  delete-autoscaling-slc       Delete scheduled limit change for an autoscaled app
App Autoscaler  2.0.40  disable-autoscaling          Disables autoscaling for the app
App Autoscaler  2.0.40  enable-autoscaling           Enables autoscaling for the app
App Autoscaler  2.0.40  update-autoscaling-limits    Updates autoscaling instance limits for the app
```

## Overview of setting up the app autoscaler
1. Ensure the "app-autoscaler" service has been created in the specified org/space.
2. Bind the app-autoscaler service to the app. Multiple apps within the space can be bound to a single app-autoscaler service and have specific autoscaling settings for each app bound to that service.
3. Use a autoscaler.yml to define and set autoscaler features.
4. Enable or disable autoscaling for the app.

***************

#### 1. Create an instance of the app autoscaler

```bash
$ cf create-service app-autoscaler standard autoscaler
```

#### 2. Bind the autoscaler to the apps

A single instance of an _Autoscaler_ service can be bound to multiple applications within the same space. Each app binding is managed separately so that all apps bound to the service instance are not forced to use the same settings.

```
$ cf bind-service attendees autoscaler
```

#### 3. Use an autoscaler yaml definition to provide scaling configuration
- min and max instances
- rules for HTTP, CPU, Memory, etc.

> NOTE: If you set the rules first in the Autoscaler Manager in the Apps Manager UI, you can then use the "cf autoscaling-rules <app name>" command to view them and use them as a template in your autoscaler.yml file later.

__Display current autoscaling rules___

```bash
$ cf autoscaling-rules attendees
Presenting autoscaler rules for app attendees for org busch / space test as chris
OK
Rule Guid     Rule Type      Rule Sub Type   Min Threshold   Max Threshold
9769b801-...  http_latency   avg_99th        10              20
b63172b8-...  cpu                            20              80
```

__Use an__ `autoscaler.yml` __definition to set autoscaler properties__

```bash
$ cf configure-autoscaling autoscaler.yml
```

### An example autoscaler.yml

```yaml
---
instance_limits:
  min: 2
  max: 4
rules:
  - rule_type: cpu
    threshold:
      min: 20
      max: 80
  - rule_type: "http_latency"
    rule_sub_type: "avg_99th"
    threshold:
      min: 10
      max: 20
scheduled_limit_changes:
  - recurrence: 10
    executes_at: "2018-11-28T00:00:00Z"
    instance_limits:
      min: 10
      max: 20
```


### 4. Enabling or disabling app autoscaler

__Enable autoscaler for the "attendees" app__
```
$ cf enable-autoscaling attendees
```

__Disable autoscaler for the "attendees" app__
```
$ cf disable-autoscaling attendees
```