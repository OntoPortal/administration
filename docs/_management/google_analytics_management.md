---
title: Google Analytics Management
layout: default
description: Setting up Google Analytics tracking for your Appliance
weight: 50
status: In progress
---

# Google Analytics Management

## Adding Google Analytics to the {{site.opva}}

BioPortal uses the Google API to process our website data. 

If you want to use the Google in your OntoPortal site,
you will need a Google developer account to use the Google API,
and a Google Analytics account to see the analytics.
The Google API is used to process the web site's data.

### Google Developer Account

You can create an account at `https://console.developers.google.com/project`.
Use the following steps to do so.

* Create an account
* Generate credentials (p12 key & email) and save it for later
  * API Manager > Credentials > Add credentials > Service account
  * Note: default password of the p12 keys is notasecret
* Enable Analytics API
  * API Manager > Overview > Advertising APIs > Analytics API
* Copy p12 keys to the appliance (in the /srv directory for example)
  * Example: `scp mybioportal-ffad56jdic91ky.p12 {my_appliance_hostname}:/srv/`

### Google Analytics Account

You use Google Analytics to get your website's data. 

* Create an account at `https://www.google.com/analytics/web`
* Add the "tracking ID" of this account in `/srv/rails/BioPortal/current/config/bioportal_config_production.rb`
  * Example: `$ANALYTICS_ID = "UA-63691424-3"`
* Give permission to your developer credential (previously generated at the Google Developer Account step) to be able to retrieve the data
  * Admin tab > Property (typically, the website you are doing analytics on) > User Management
  * Add permissions for the google developer credential email (also generated in the previous section)
* Add the following in `/srv/ncbo/ontologies_api/current/config/environments/appliance.rb` and `/srv/ncbo/ncbo_cron/config/appliance.rb`

```
 LinkedData.config do |config|
  config.enable_ontology_analytics = true
 end
 NcboCron.config do |config|
  config.enable_ontology_analytics = true
  config.cron_ontology_analytics   = "30 */4 * * *"
  # Google Analytics config
  config.analytics_service_account_email_address = "account-1@mybioportal-1145.iam.gserviceaccount.com" # The email address you get when creating the google developer account
  config.analytics_path_to_key_file              = "/srv/agroportal-ffad56jdic91ky.p12" # you have to get this file from the first step
  config.analytics_profile_id                    = "ga:111821946" # replace with your ga view id
  config.analytics_app_name                      = "mybioportal"
  config.analytics_app_version                   = "1.0.0"
  config.analytics_start_date                    = "2015-10-01"
  config.analytics_filter_str                    = ""
 end
 ```

## Tests using API explorer

Relevant links are
* https://developers.google.com/apis-explorer/#search/analytics/m/analytics/v2.4/analytics.data.get
* https://ga-dev-tools.appspot.com/query-explorer/

```
 id = ga:111821946
 start-date = 30daysAgo
 end-date = yesterday
 metrics = ga:pageviews
 dimensions = ga:pagePath,ga:year,ga:month
 filters
```

## Removing Analytics from OntoPortal UI

In `/srv/rails/BioPortal/current/app/views/home/index.html.haml` add:

```
 display: none;
 %fieldset{style: "display: none;"}
  %legend
   Ontology Visits #{"in full #{$SITE} " if at_slice?} (#{@analytics.date.strftime("%B %Y")})
```

In `/srv/rails/BioPortal/current/app/views/ontologies/_visits.html.haml` add:

```
 display:none;
 #visits_content{:style => "margin: 2em 1em 0;"}
 ```