# Splunk
Regex|Splunk

Identifying Splunk Component:

1. Ingests Data.
2. Parses, indexes and stores data.
3. Runs searches on indexed data.


## Processing
1. Search Head
2. Indexer
3. Forwarder

## Management
1. Monitoring Console
2. Deployment Server
3. License Server/Master
4. Cluster Master
5. Deployer

### Splunk Licensing
1. You license data ingested per day, not data stored.
2. Daily indexing volume is measured from midnight to midnight by the clock on the license master.

Standard Enterprise Liscense
- Obtained by a sales represenative.

Enterprise Trial
- 500MiB a day, after 60 days converts to a free license.

Sales Trial
- This is a license that is obtained for the simple purpose of 'POC'.

Dev/Test
- Restricted to non-production Splunk staging environments :: DevOps.

Free (see Enterprise Trial) :: Limited Feature Set
- Can't set up alert.
- No login.
- No alerts.
- No distributed architecture.

Industrial IoT
- Sensors.
- Gives access to Splunk Enterprise and a specific set of features and apps that are relative ot IoT.

Forwarder License
- Used to set up for Heavy Forwarder.

License Violations
- If you exceed your licensed daily volume in 1 calendar day you get a Warning.
- If you exceeed your licensed daily volume 5 or more times in a 30 day period, you are in Violation.
- If you are in violation, Splunk will send you messages but will not interrupt or disrupt search functionality. Prior to v5 it did.

Distributed Licensing
- Setup a License Manager.
- License pools are created from license stacks.
- Pools are sized for specific purposes.
- Managed by License manager.
- Indexers and other Splunk Enterprise instance are assigned to a pool.
- License groups are a stack of licenses bound together.
- Only Enterprise|Sales trial licenses can be stacked.
- Whereas Enterprise Trial, Free and Forwarders cannot.
- When we purchase licenses we receive a license file from the representative and we install them on the license manager.

### Splunk Configuration Files
- Describe Splunk configuration directory structure.
- Understand configuration layering.
- Understand configuration precedence.
- Use btool to examine configuration settings.
