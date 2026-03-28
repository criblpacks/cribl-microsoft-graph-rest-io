# Microsoft Graph REST Collector IO
----
## About this Pack
This pack is built as a complete SOURCE + DESTINATION solution (identified by the IO suffix). Data collection and delivery happen entirely within the Pack's context - you can choose how data arrives at a DESTINATION:
*  *Send to Worker Group Routes* (the default): data is sent to the top-level Worker Group Routes.
*  *Default Destination*: data is sent to the Worker Group's [Default Destination](https://docs.cribl.io/stream/destinations-default/). 
*  *In-Pack Destination*: data is sent to one or more Destinations configured within the Pack.

This Pack is designed to collect, process, and output Microsoft Azure Graph data via the Azure Graph REST API. It currently supports the following endpoints:
* [Azure (MSEntra) Signins](https://learn.microsoft.com/en-us/graph/api/signin-list?view=graph-rest-1.0&tabs=http)
* [Azure User Details](https://learn.microsoft.com/en-us/graph/api/user-list?view=graph-rest-1.0&tabs=http)
* [Azure Device Details]()
* [Azure Security Alerts V2](https://learn.microsoft.com/en-us/graph/api/security-list-alerts_v2?view=graph-rest-1.0&tabs=http)
* [Legacy Azure Security Alerts](https://learn.microsoft.com/en-us/graph/api/alert-get?view=graph-rest-1.0&tabs=http) - deprecated and will be [***removed*** April 2026](https://learn.microsoft.com/en-us/graph/api/resources/security-api-overview?view=graph-rest-1.0&viewFallbackFrom=graph-rest-v1.0&preserve-view=true#alerts)

The Pack includes OCSF and Splunk output processing. OCSF data is mapped to the following Classes:
* Azure Signins - [Authentication [3002] Class](https://schema.ocsf.io/1.4.0/classes/authentication)
* Azure User Details - [User Inventory Info [5003] Class](https://schema.ocsf.io/1.4.0/classes/user_inventory)
* Azure Device Details - [Device Inventory Info [5001] Class](https://schema.ocsf.io/1.4.0/classes/inventory_info)
* Azure Alerts (all) - [Detection Finding [2004] Class](https://schema.ocsf.io/1.4.0/classes/detection_finding)

Splunk data is mapped to the following sourcetypes - these are the sourcetypes used by the [Splunk Add on for Microsoft Azure](https://splunkbase.splunk.com/app/3757):
* Azure Signins: `sourcetype=azure:aad:signin`
* Azure Users: `sourcetype=azure:aad:user`
* Azure Device Details:`sourcetype=azure:aad:device`
* Azure Alerts V2: `sourcetype=ms:graph:security:alerts:v2`


## Deployment

* Every bundled Source within this pack adds a hidden field: `__packsource`. This field allows for simplified routing based on the Pack source.
* This pack is configured by default to use the Destination *Send to Worker Group Routes*. You *must* add either a Worker Group Route or rely on the Default Destination.
* To explicitly use the Worker Group's *Default Destination*, change the Pack's Routes to *default:default*. The Pack will then route the data to the destination currently set as the Default on the Worker Group.

### Configure the Collectors
* Obtain a ```Tenant ID```, ```Client ID``` and ```Client Secret``` from your Azure/Microsoft Entra Administrator. These credentials must have the following permissions in order for all the Collectors to work:
  * AuditLog.Read.All
  * Device.Read.All
  * Policy.Read.All
  * SecurityAlert.Read.All
  * SecurityEvents.Read.All
  * User.Read.All
* Update the variables for ```Tenant ID```, ```Client ID``` and ```Client Secret``` with the correct values.
* Perform a Run > Preview to verify that each Collector works correctly.
* Schedule the Collectors and adjust the schedule as needed. Collectors requiring State Tracking should already have it enabled. Default schedules for each are:
   *  Signins: Every 5 minutes with default State Tracking
   *  Alerts V2: Every 5 minutes with default State Tracking
   *  Users: Once/day - this endpoint is configured to retrieve all available Users so keep that in mind.
   *  Devices: Once/day - this endpoint is configured to retrieve all available Devices so keep that in mind.

### Configure Output Format

Each data type can be configured to output data in either normalized JSON, OCSF, or Splunk (`_raw` + Splunk fields) format. Enable *only one* format for each of the following pipelines:
* ```cribl_azure_graph_signins```
* ```cribl_azure_graph_users```
* ```cribl_azure_graph_devices```
* ```cribl_azure_graph_alerts_v2```

### Configure your Destination/Update Pack Routes
To ensure proper data routing, you must make a choice: retain the current setting to use the Default Destination defined by your Worker Group, or define a new Destination directly inside this pack and adjust the pack's route accordingly.

### Commit and Deploy
Once everything is configured, perform a Commit & Deploy to enable data collection.

### Variables

The Pack has the following variables:
* `microsoft_graph_default_splunk_index`: Default index for the Splunk output - defaults to `azure`.
* `microsoft_graph_tenant_id`: Your Microsoft Tenant ID
* `microsoft_graph_client_id`: Your Microsoft Client ID
* `microsoft_graph_client_secret`: Your Microsoft Client Secret

## Upgrades

Upgrading certain Cribl Packs using the same Pack ID can have unintended consequences. See [Upgrading an Existing Pack](https://docs.cribl.io/stream/packs#upgrading) for details.

## Release Notes

### Version 2.0.0
* Updated Route Destinations to "Send to Worker Group Routes". See above for details.

### Version 1.1.1
- Removed included Splunk HEC Destination
- Pagination fix for AlertsV2
- Reduced AlertsV2 backfill from 30 days to 7 days

### Version 1.1.0
- Converted Collectors to use variables wherever possible
- State tracking fixes for Alerts V2 and Signins

### Version 1.0.1
- State Tracking fix for `in_azure_graph_security_alerts_v2` Collector - updated the `filter` request parameter. 
- Updated `source` and `sourcetype` for `cribl_azure_graph_alerts` and `cribl_azure_graph_alerts_v2` pipelines. 

### Version 1.0.0
- Initial release

## Contributing to the Pack

To contribute to the Pack, please connect with us on [Cribl Community Slack](https://cribl-community.slack.com/). You can suggest new features or offer to collaborate.

## License
This Pack uses the following license: [Apache 2.0](https://github.com/criblio/appscope/blob/master/LICENSE).
