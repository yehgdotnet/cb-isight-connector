# Carbon Black - iSIGHT Connector

For more information on iSIGHT and a special license offer for Carbon Black customers, see http://info.isightpartners.com/upgrade/carbonblack.

## Installation Quickstart

As root on your Carbon Black server:
```
cd /etc/yum.repos.d
curl -O https://opensource.carbonblack.com/release/x86_64/CbOpenSource.repo
yum install python-cbisight-connector
```

Once the software is installed via YUM, then copy the `/etc/cb/integrations/isight/isight.config.template` 
file to `/etc/cb/integrations/isight/isight.config`. Edit this file and place your API and secret key 
into the `iSightRemoteImportPublicKey` and `iSightRemoteImportPrivateKey` options. Also place an API token
associated with an administrative Cb user into the `carbonblack_server_token` option.

Run the integration once to make sure the configuration worked:
```
/usr/share/cb/integrations/isight/isight
```

Any errors will be logged into `/var/log/cb/integrations/isight/isight.log`.

## Introduction

This document describes how to install and use the Carbon Black iSIGHT Connector.  This connector allows for the importing of iSIGHT threat intelligence feeds and tags documents matching any threat intelligence feeds in the Carbon Black database. The iSIGHT connector uses the ThreatScape v2 API as described at http://www.isightpartners.com/doc/sdk-bp-docs/#/ to retrieve threat intelligence from iSIGHT. The connector will create a Carbon Black feed for any iSIGHT threat intelligence hits, and queries for new threat indicators from iSIGHT’s ThreatScape API every hour by default.

*Note: This is a COMMUNITY SUPPORTED add-on. Please see the end for support options.*

The following IOC types are extracted from iSIGHT ThreatScape:
* MD5 Hashes
* Domain Names
* IP Addresses
* IP Address Ranges

Future versions of the iSIGHT connector will expand the IOC types to include registry key, filenames, and other indicator types supported by the ThreatScape API.

## Requirements

This Carbon Black iSIGHT Connector has the following requirements: 
* *Carbon Black Enterprise Server 5.0 (or greater)* - this integration leverages API calls and feed functionality available in Carbon Black 5.0 and newer.  In order to check the version, you can run the following rpm command on your server: 

```
[root@localhost ~]# rpm -qa | grep cb-enterprise
cb-enterprise-5.0.0.150122.1654-1.el6.x86_64
```

* *Access to iSIGHT ThreatScape API* - Request an API access key from iSIGHT Partners.

## Configuration

You'll need to place a configuration file in the following location: `/etc/cb/integrations/isight/isight.config`

A sample file is provided in `/etc/cb/integrations/isight/isight.config.example`, so you can rename the file with the following command: 

```
mv /etc/cb/integrations/isight/isight.config.example /etc/cb/integrations/isight/isight.config
```

The example configuration file is placed here along with the comments it contains:

```
[cb-isight]
# These credentials come from iSIGHT; iSightRemoteImportPublicKey is the API key provided by iSIGHT.
# iSightRemoteImportPrivateKey is the secret key provided by iSIGHT.
#
iSightRemoteImportPublicKey=
iSightRemoteImportPrivateKey=

# URL of iSight REST API endpoint
# Effective 15-Oct-2014, mysight-api.isightpartners.com is deprecated in favor of api.isightpartners.com
#
iSightRemoteImportUrl=https://api.isightpartners.com

# By default, the iSIGHT connector will only access the /view/iocs endpoint to retrieve the
# latest indicators from iSIGHT. This does not provide a threat score for each indicator, as this
# information is in the report itself. If iSightGetReports is set to true, the full reports will
# be downloaded in order to retrieve the threat score for each indicator. This requires your
# account to be provisioned for access to the /report endpoint and may incur additional cost.
iSightGetReports=false

# Number of days (relative to today) to back-pull reports from
#
iSightRemoteImportDaysBack=80

# API token for the Carbon Black server
# This is used to set up the iSIGHT feed and update the Cb feed every time a new version is downloaded
# from the iSIGHT APi
carbonblack_server_token=

# If you need to use an HTTPS proxy to access the iSIGHT API server, uncomment and configure the https_proxy
# variable below.
# https_proxy=http://proxyuser:proxypass@proxyhostname:proxyport

```

*To download reports from iSIGHT, you must set `iSightRemoteImportPublicKey`, `iSightRemoteImportPrivateKey` 
and `iSightRemoteImportUrl`. The URL should not have to change; the default will be correct for 
iSIGHT ThreatScape customers. The Public and Private key fields must be set to the API key and 
secret key, respectively, provided by iSIGHT.*

*You must also fill in an API token for your Carbon Black server in `carbonblack_server_token`. You can
access your API token by logging in as an administrative user on the Cb web user interface, then 
clicking your name in the top right corner, and selecting "Profile". You will see "API token" on the left
hand side. Clicking this will reveal the API token associated with the current user. Copy and paste
this token after the equal sign on the `carbonblack_server_token` line in the configuration file.* 

## Execution

By default the Linux cron daemon will run this integration every hour to check for new data from the iSIGHT ThreatScape API server.  When it runs it will use the current settings found in `/etc/cb/integrations/isight/isight.config`, so make sure you are careful when changing any of those settings.

When you first install the connector, you might not want to wait until the hour mark for the job to run.  In this case, you can force the connector to run manually.  As either root or cb user on the Carbon Black Server, execute the following command:

```
/usr/share/cb/integrations/isight/isight -c /etc/cb/integrations/isight/isight.config
```

It is perfectly fine to do this because the script will only allow one copy of itself to run at a time, so you don't have to worry about the cron daemon attempting to run this while your manual instance is still executing.  

Note #1: this script can take a long time to run depending on the amount of data available from the ISIGHT services you have configured. 

Note #2: this script logs everything to `/var/log/cb/integrations/isight/isight.log`, so you will see very little output when you run it manually.

## Troubleshooting

If you suspect a problem, please first look at the iSIGHT connector logs found here: `/var/log/cb/integrations/isight/isight.log`

(There might be multiple files as the logger "rolls over" when the log file hits a certain size).

## Contacting Bit9 Developer Relations Support

Web: https://community.bit9.com/groups/developer-relations
E-mail: dev-support@bit9.com

### Reporting Problems

When you contact Bit9 Developer Relations Technical Support with an issue, please provide the following:

* Your name, company name, telephone number, and e-mail address
* Product name/version, CB Server version, CB Sensor version
* Hardware configuration of the Carbon Black Server or computer (processor, memory, and RAM) 
* For documentation issues, specify the version of the manual you are using. 
* Action causing the problem, error message returned, and event log output (as appropriate) 
* Problem severity

