# Splunk

Regex|Splunk

Identifying Splunk Component:

1. Ingests Data.
2. Parses, indexes and stores data.
3. Runs searches on indexed data.


### Processing
1. Search Head
2. Indexer
3. Forwarder

### Management

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
2. Process - transforms incoming data into 'events'.
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

Note: There are 6 buckets in a fishbucket.

- Newly indexed data goes to the hotPath.
- An index has 1 or more hot buckets.
- Then it moves to warm bucket because it has no active writes to it.
- An index will have 1 or more warm buckets.
- The coldPath is in a different location of the file structure because it is data that is rarely accessed.
- An index has many cold buckets.
- Frozen is rolled from cold and deleted by default but you can override this default and archive them.
- If you restore any of the archived data it goes to the thawed path.

Buckets:

```bash
Hot (hotPath): $SPLUNK_HOME/var/lib/splunk/defaultdb/db/*
Warm (warmPath): $SPLUNK_HOME/var/lib/splunk/defaultdb/db/*
Cold (coldPath): $SPLUNK_HOME/var/lib/splunk/defaultdb/colddb/*
Frozen (frozePath): Location that you specify.
Thawed (thawedPath): $SPLUNK_HOME/var/lib/splunk/defaultdb/thaweddb/*
```

### Check Data Integrity

Splunk's double hash.
	Computes a hash on a newly indexed data set.
	Computes another hash on the same data when it moves buckets.
	Stores both has files in the /rawdata directory.

source> index(level 1 hash)> Hot Path (level 2 hash)> Warm Path.

Command Line Options
1. Check hashes to validate data.
```bash
./splunk check-integrity -bucketPath [ bucket path ] [ -verbose ]
```
2. Configure data integrity control.
```bash
enableDataIntegrityControl=true
```
3. Regenerate hashes.
```bash
./splunk generate-hash-files -bucketPath [ bucket path ] [ verbose ]
```

### index.conf Options:

Can be configured with:
1. Global.
	- Defined either at the beginning of a file or in the [default] stanza.
	- Each index.conf file has only one [default] stanza.
2. Per index.
	- Options under an [<index>] stanza.
	- A few of the many options:
		1. Set bucket paths.
		2. Set database sizes.
		3. Specify event or metric data types.
3. Per provider family.
	- Options for External Resource Providers (ERPs).
	- For example: Hadoop.
	- All provider stanzas begin with [provider: ].
4. Per provider.
	- Properties that are common to multiple providers.
	- All properties that cna be used in a family can be used in a provider.
	- Stanzas for provider families being with [provider-family: ].
	- For example: Hadoop, Spark, Hive.
5. Per virtual index.
	- Common for Hadoop family.
	- Let's Splunk access data stored in external systems and push computations to those systems.

### The Fish Bucket

This guys keeps track of which files and which parts of files have already been indexed.
- Used for deduplication.
- Contains seek pointers and crcs.

Exercise:
1. Create an index.
- Go to settings and click index.
- Click new index - top right corner.
- Configure and save.
2. Apply a data retention policy.
```bash
nvim /opt/splunk/etc/system/local/indexes.conf

[history]
frozentimePeriodInSecs = 604800 (7 days)

[_internal]
frozentimePeriodInSecs = 2592000 (30 days)

[_thefishbucket]
frozentimePeriodInSecs = 4419200 (almost 28 days)

# Altering these periods make the retention levels go up or down respectively.
```
3. Explore buckets in the file system.
- /opt/splunk/var/lib/splunk/defaultdb/db
4. Check hashes to validate data.
```bash
./splunk check-integrity -bucketPath /var/lib/splunk/defaultdb/db/ -verbose
```

### User management

Splunk is a role (collection of user capabilities).
- Users are not assigned to individual capabilites.
- Splunk has 4 default roles:
  1. admin
	2. poweruser
	3. user
	4. can_delete

Capabilities
1. Assigned to roles.
2. Additive in nature.
3. Can be used to granularly manage users in Splunk.
4. Can be added or removed in Splunk web, or in authorize.conf

### Splunk Authenticaton Management

LDAP quick review:
1. Defines a protocol to authenticate to, access, and update objects in an X.500 style directory.
2. DN = Distinguished Name, an entry's unique identifier usually composed of multiple attributes.
3. CN = Common | canonical Name.
4. OU = Organizational Unit.
5. DC = Domain component.
6. SN = Surname.

```bash
# attributes
dn: cn=Jon Doe,dc=rhce,dc=lab
cn: Jon Doe
sn: Doe
```

In order for Splunk to communicate and authenticate with LDAP, its need to send a 'BIND' packet and this ldap bind allows the client to authenticate to a X.500 directory server and verify identity of users as well as map users and groups from ldap to splunk users and roles.

Options?
1. Splunk web.
	1. Create an LDAP strategy.
	2. Map LDAP groups to Splunk roles.
	3. Specify the connection order.
2. Edit authentication.conf file directly.

LDAP connection settings:
- Host
		1. IP address.
		2. fqdn.
- Port
		1. Port 389 or 636
- Bind DN
		1. Bind LDAP service to Splunk. Use a _svc_ acct for this.

User settings:
1. User base DN
2. User base filter
3. User name attribute
4. Real name attribute
5. Email attribute
6. Group mapping attribute

Group Settings:
1. Group base DN
2. Static group search filter
3. Group name attribute
4. Static member attribute

Dynamic Group:
1. Dynamic member attribute
2. Dynamic group search filter


MFA
Scripted Authentication API (SAML)
DUO and RSA

ADDS
- Server Name.
- Go to tools and ADDS.
- Explore OUs.
- fqdn.

### Getting Data Into Splunk (Pipelining)

1. Inputs
2. Parsing
3. Indexing
4. Search

### General Input Categories

	- File and directory inputs
		- Monitor files and directories.
		- Locally and remotely.
		- Monitor compressed files.
	- Upload
		- Upload files to Splunk.
		- Used for one-time analysis.
	- MonitorNoHandle
		- Available for Windows hosts only.
		- Monitors files and directories that the system rotates automatically.

### Network Inputs

	- Data from TCP and UDP.
	- syslog
	- Data from SNMP events.

### Windows inputs

	- Windows event logs.
	- Registry.
	- Active Directory.
	- WMI.
	- Performance monitoring (perfmon).

### Other Data sources

	- Metrics
	- FIFO queues Scripted inputs (API(s), Message queues)
	- Modular inputs (build functionality for ingesting sensitive data/security oriented)
	- HTTP event collector (web related content)

### Basic Settings for Inputs: (Ways to configure)

1. Through an app.
	 - Many apps have preconfigured inputs.
2. Splunk web.
 	 - Settings> data inputs.
	 - Settings> add data.
3. CLI
	 - For example: ./splunk add monitor <path>.
4. Through inputs.conf
	 - Add a stanza for each input.
5. Guided Data Onboarding (GDO)
	 - This is a data input wizard.
	 - Somewhat new for Splunk.

### Types of Forwarders

1. Universal (replaced the light forwarder).
	 - Collects data from a data source and forwards it to a receiver.
	 - This is its only job.
	 - The source 'could' be another 'forwarder' and so could the receiver.
	 - Most usual cases is that we are usually forwarding to an indexer or a splunk searchhead.
	 - Installed separately.
	 - It is NOT a Splunk Enterprise instance.
	 - It has NO license.
	 - It does not 'search' indexes nor does it parse any data.
	 - It simply takes the data and forwards the data.
2. Heavy
	 - Advanced type of forwarder.
	 - Full installation of Splunk Enterprise and you apply a forwarder 'license' to it.
	 - It parses data and can also route data on criteria such as source.
	 - Use cases: db data to a specific indexer and then you want to route all web data to another indexer.
	 - Can index data locally.

### Configure Universal forwarder.

1. Configure receiving on a Splunk Enterprise instance.
2. Download and install the UF on the source system.
3. Start service.
4. Configure it to send data.
5. Configure it to collect data from the host system.


### Distributed Search

Distributed search provides a way to scale your deployment by separating the search management and presentation layer from the indexing and search retrieval layer.

- There are two necessary components for distributed searching:
  1. Search heads.
	2. Indexers, also called search peers.

Types:
- Horizontal scaling
		- expand environment by adding search heads to increase performance.
- Access control
		- control access to indexed data through access control.
- Geo-dispersed data
		- content derived from geographical locations.

Configuration of Distributed Search

Machines:
- Virtual, physical or cloud machines.
- Splunk Enteprise installed on all machines.

Network:
- All machines must be able to communicate with each other.
- All machines need to have a FQDN - use Static IP addresses.
- DNS server or manual hostname/hosts entries.
- Firewalls configured properly.
- Minimum of one deployer and 3 cluster members.

CLI commands
- In order to add.

1. Requirements
2. Deployer
3. Splunk Installation
4. Initialization
5. Captain
6. Post Deployment

Configuring the Deployer:
1. Set security key
2. Label the cluster

Edit servers.conf in $SPLUNK_HOME/opt/etc/system/local
Add stanzas
```bash
[shclustering]
pass4SymmKey = <security key>
shcluster_label = <name your cluster>
```

### Staging

3 Phases of the Splunk Indexing Process:
1. Data Input - Forwarding, monitoring, network, scripted.
2. Parsing - Breaks the data stream into individual events.
3. Indexing - Writes the parsed data into index buckets.

Input Options:
1. Source to Splunk
	- Files and Directories
	- Network Events
	- Windows Sources
	- Other Data Sources

### Configuring Forwarders

1. Configure receiving on an indexer or cluster.
2. Download and install the universal forwarder.
3. Start the universal forwarder and accept the license agreement.
4. Configure the universal forwarder to send data.
5. Configure the unversal forwarder to collect data fro mthe host it is on.

4 Ways to configure:
1. Gui (windows)
2. CLI
3. Configuration Files
4. Deployment Server

### Configuring Heavy Forwarders

1. Basic configuration:
   1. Install Splunk Enterprise.
	 2. Apply a forwarder license.
	 3. Set up forwarding:
	 		1. Settings > Forwarding and Receiving.
			2. Add New > Configure Forwarding.
			3. Edit outputs.conf directly.
2. Advanced Configuration:
	 1. Heavy forwarders can index data before they forward it.
	 2. Through Splunk web:
	    1. Settings> Forwarding and Receiving.
			2. Check the box> Store local copy.

Additional Forwarder Options:
1. Load-Balancing (distributes data over multiple instances) but this requires you configuring distributed searching.


### SPL Queries
1. Will have key-value pairs.
   - index=main
   - sourcetype=access_combined_wcookie
   - status !=200
2. Literal Strings.
3. Commands.
6. Comparison Operators.
4. Functions and Options.
5. Logical Operators (Booleans).
6. Comparison Operators.
7. Clauses (grouping by fields). Or As to rename a field.
8. The Search Pipeline (Pipe).
Note: Output of the left side of pipe becomes the input to the right side of the pipe.

### Searches
