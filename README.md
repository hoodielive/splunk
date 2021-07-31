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

Configuration Files
- Splunk runs on configuration (.conf) files.
	Every behavior and function within Splunk is defined in a .conf file.
- You may have multiple copies of the same configuration file. They are evaluated by Splunk based on precedence.

3 Common Configuration Files in Splunk:
1. inputs.conf - governs data input such as forwarders and file system monitoring. Host values of the data coming in. Define or specify which iindexers should store events.
2. props.conf - governs indexing property behavior. For example, timestamp recognition, anonymizing sensitive data, definining basic search time field extractions.
3. transform.conf - settings and values tat govern data transformation. Supports regex.
Note: Splunks documentation is excellent and does a great job of explanining the purpose of its various configuration files. See: https://docs.splunk.com/Documentation/Splunk/latest/Admin/Listofconfigurationfiles

To understand how configuration files work, we must first understand their place in the Splunk filesystem:

- Splunk utilizes the same file structure as Windows, Linux and Mac.
- /opt/splunk
- /opt/splunk/bin - containers splunk binaries to execute commands like start|stop|restart.
- /opt/splunk/var/lib - contains information relative to splunk indexes, import splunk system stuff.
- /opt/splunk/etc - contains 3 paths:
	1. System (contains a default and local directory).
	2. Apps (contains search, launcher, and others apps. Search has a default, local and metadata directory structure).
	3. User (contains users).

Note: Defaults ( are unaltered files whereas local is user-defined and thus altered files ). Further, System (Global context), Apps (App context) and Users (User context).

Note: Never mess around with default files. Make all your custom configuration in the local directories respectively.

Understanding Layering and Precedence
- Configuration files have precedence based on Splunk internal logic.

Configuration File Context
- Global
- App|User specific

In the Global context:

	System local directory
		App Local directories
			App default directories
			 System default directory

In the App|User context:

	User directories for current user
		App directories for currently running app (local, followed by default)
			App directories for all other apps (local, followed by default)
				System directories (local, follwed by default)

What's inside the configuration files?
1. Stanzas
2. Attribute  = value pairs

For example:
app.conf:

```bash
[id]
group = <group-name>
name = <app-name>
version = <version-number>
```

### btool
Use btool to examine configuration settings. Namely, merged configurations (from various file contexts) and troubleshooting.

```bash
# $Splunk_HOME/bin

# ./splunk cmd btool <configuration file prefix> list

./splunk cmd btool transforms list # this shows transforms list 'globally'


# Show specific settings apps are using.
# ./splunk cmd btool --app=app_name <conf file prefix> list

./splunk cmd btool --app=search transforms list
./splunk cmd btool transforms list --debug

# Check for typos in stanzas and settings.
./splunk cmd btool check

```

Use btool to:
1. Investigate global configuration values.
2. Investigate configuration values in a single app.
3. Learn the source of configuration values.
4. Check for typos in stanza setting names.


### Indexes

Describe Index Structure:
1. Repository of Splunk events (a place to put the data).
2. Built-in or custom.
3. Indexes contain 3 types of data:
	 1. Raw data in compressed form.
	 2. Indexes that point to raw data.
	 3. Metadata :: data that describes data: timestamp, host value or sourcetype value.

There are 2 main types of indexes in Splunk:
1. Event
	- the default type of index.
	- can handle any type of data.
2. Metrics
  - Optimized to store and retrieves metric data.
	- Timestamps, Metric name, Values and Dimensions.
	- It is structured kind of like DNS hiearchy.
	- Data Warehousing.

How splunk processes data:
1. Data - any kind of 'raw' data.
2. Process - transforms incoming data into 'events'. It adds the following fields. 
	 	- host, source, sourcetype
		- Character set encoding
		- Line breaks
		- Timestamps
		- Metadata
3. Index - indexes are stored in buckets.
	 - $SPLUNK_HOME/var/lib/splunk/*
	 - main,_internal,_audit
	 - main: indexes go here.
	 - \_internal: internal logs and metrics
4. Bucket - directories on the file system organized by age.
	 - Hot (hotPath)
	 - Warm (warmPath)
	 - Cold (coldPath)
	 - Frozen (frozenPath)
	 - Thawed (thawedPath)
