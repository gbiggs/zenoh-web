---
title: "Troubleshooting"
weight : 1030
menu:
  docs:
    parent: getting_started
---

## Activate logging

Activating the zenoh logging can provide useful information for any troubleshooting. The zenoh router (`zenohd`) and all the zenoh APIs (except zenoh-pico) are developped with a Rust code base. Logging is controlled via the `RUST_LOG` environment variable that can typically be defined with the desired logging level amongst:
 - `error` - this is the default level if `RUST_LOG` is not defined
 - `warn`
 - `info`
 - `debug`
 - `trace`
 - `off` - to disable all logging

More advanced logging directives can be defined via the `RUST_LOG`. For more details see this [page](https://docs.rs/env_logger/latest/env_logger/#enabling-logging).  
Note that the logs are written on `stderr`.

------
## Known troubles with the zenoh router (`zenohd`)

### Segmentation fault at startup

The router is likely trying to load an incompatible plugin.

To check this, [activate the logs](#activate_logging) at `debug` level and look for such logs:
```C#
[2021-08-24T15:24:06Z DEBUG zenohd] loaded plugin: webserver from "/Users/ato/.zenoh/lib/libzplugin_webserver.dylib"
[2021-08-24T15:24:06Z DEBUG zenohd] loaded plugin: rest from "/Users/ato/eclipse-zenoh/zenoh/target/debug/libzplugin_rest.dylib"
[2021-08-24T15:24:06Z DEBUG zenohd] loaded plugin: storages from "/Users/ato/eclipse-zenoh/zenoh/infra/zenoh/target/debug/libzplugin_storages.dylib"
```
Here you can see all the plugins libraries that have been loaded by `zenohd` at startup. You must check if each of those are using the same zenoh version as dependency than `zenohd`.  
To assess which one is causing troubles, you can also move or rename all the libraries but one and test if `zenohd` is correctly loading this one. Then repeat the process for each library.


------
## Known troubles with the APIs

### _"Error treating timestamp for received Data"_

If you get such log:
```C#
[2021-08-23T08:44:47Z ERROR zenoh::net::routing::pubsub] Error treating timestamp for received Data (incoming timestamp from E74B6FF3D82D49BEA11B8F1BD0AC4C7A exceeding delta 100ms is rejected: 2021-08-23T08:44:47.299498897Z vs. now: 2021-08-23T08:44:47.069513846Z): drop it!
```

It means your zenoh application on your local host received a publication from another remote host with a system time that is drifting in the future for more than 100ms wrt. the local system time. You should synchronize the time between your hosts using a tool such as [PTP](http://linuxptp.sourceforge.net/) or [NTP](http://www.ntp.org/).
