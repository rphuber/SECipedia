This is for general considerations when setting up
	In addition, please see aws_rds_setting_up_aurora.md

Bad practice to have databases open to the public.
	Open them up to a private VPC and create security groups and roles based on that
		Make sure to create your own security group for the DB

Relational (Online Transaction Processing-OLTP)
	Scenario:
		What if we have an EC2 instance connecting to an RDS mySQL db?
			We need to connect the security group of the EC2 instance (think of outbound) to the security group of the rds service (think of inbound)
				We accomplish this in a VPC
					This step is needed so the two items can communicate with eachother
					Look at VPC.md

	Relational Database Service (RDS)
		Storage
			Magnetic
				aka Standard Storage
				
				Ideal for applications with light or burst I/O requirements

				Cost effective, but slower than SSD

			General Purpose (SSD)
				aka gp2

			Provisioned IOPS (input/output operations per second)
				Designed for I/O intensive workloads (e.g. database loads)
					These types of loads are sensitive to storage performance and consistency in random access I/O throughput.

		Monitoring
			Options
				Cloudwatch
					Monitors the performance and health of a DB instance
					Performance charts are shown in the RDS console.

					You can also subscribe to Amazon RDS events to be notified when certain events occur: Changes to a DB instance, etc.



		AWS provides SDKs for their RDS services
			You can utilizes these SDKs instead of the RDS's SOAP and Query APIs.
				These SDKs provide basic functions that aren't in the SOAP and Query APIs, including:  Request authentication, request retries, and error handling.

		RDS manages backups, software patching, automatic failure detection, and recovery.

		This is a managed service, in other words, you're not allowed shell access to DB instances.  You're restricted to certain system procedures and tables that require advanced privileges.
			If you REALLY want this shell access, you can always host a database on an EC2 instance.

			Although you can't SSH into the instance, you are allowed to use practically any other app that you'd normally use (think MySQL workbench, etc.)

		You get high availability with a primary instance and a synchronous secondary instance that you can failover to when problems occur.  You can use a MySQL read replicas to increase read scaling.
			For tests ENVs, this isn't necessary

		You can run your DB instances in a VPC for extra control.

		Security Groups
			If you're attaching the RDS to an EC2 instance, use the EC2 instances security group (as the identifier) when connecting to the RDS's security group.
				This will allow access from any entity in the EC2's security group.

			RDS uses DB security groups, VPC security groups, and EC2 security groups.

				DB security group (dont use this)
					controls access to a DB that's not in a VPC

				VPC security group
					In the real world, this is the option that you want.

					Controls access to a DB instance inside a VPC

					Think of this level of access as occuring at the instance level


				EC2 Security Group (dont use this)
					Controls access to an EC2 instance and can be used with a DB instance.





		Has the ability to add a managed memcached or redis-compatible in-memory cache
		to speed up your database access

		DB Parameter Group
			Each DB engine has a set of parameters that control the behavior of the db

				You manage the configuration of a DB engine by using this parameter group.

			A DB parameter group contains engine configurations values that can be applied to one or more DB instances of the same instance type.

			AWS applied a default DB parameter group if you don't specify a DB parameter group when you create a DB instance.



		Storage Capacity
			For each DB instance, you can select from 5 GB to 6 TB of associated storage capacity.

		MySQL
			You can resize your instance (cpu/HD) on the fly.

		Microsoft SQL Server
			Can't change the HD space on the fly
				You either initially create the db with the appropriate HD space
				for future endeavors OR once you get to a certain theshold you
				export all of database and create a new one with increased storage
					CON: Outage when migrating over DB
			When created, it doesn't automatically have a db  

		You can change the password

		Resore DB instance to a point in time
			If you do this, it will create an entirely new instance with a new endpoint

		Amazon doesn't give you a public/private IP for your DB, you get an DNS endpoint
		You only have control over the DB, not the server itself
			EX: You can't patch, ssh or rdp into the underlying server, etc.  You can say when the maintenance will occur, but that's pretty much it

		Maximum backup window (time to keep a backup snapshot) 35 days

		Multi-AZ deployment
			Runs a DB instance in several AZ's.

			AWS automatically provisions and maintains a synchronous standby replica to provide data redundancy, failover support, eliminate I/O freezes, and minimize latency spikes during system backups.

			Make sure to utilize the ENDPOINT DNS name when doing this type of deployment.  If a SQL server goes down, AWS will automatically remap the DNS into an instance in a new AZ.


			Only MySQL can be utilized for read replicas
		
		MySql

		Sql Server
			From Microsoft, you can bring your own license
		Postgres
		Oracle
			You can bring your own liceneses or you can use Amazon's licenses
		
		
		Setting up an RDS instance
			Create a security group
	NoSQL
		DynamoDB
			vs Mongo
				You can't have embedded data structures in Dynamo (key value pairs aka objects).
			Always stored on SSD 

			Creating an instance
				Hash Attribute Key
					Primary Key, can be UserID, etc.

			Spread Across 3 geographically distinct data centers automatically
				Eventually Consistent Reads (Default)
					Consistency across all copies of data is usually reached within a second.  So if the data is written and then read from another datacenter, there should be a roughly second delay.
						When do you do query the DB, there might be a chance that the query is routed to an AZ datacenter that isn't updated (~second delay)
						This gives best read performace vs. Strongly Consistent Reads
				Strongly Consistent Reads
					Returns a result that reflects all writes that received a successful response prior to the read
						I.e. If a write 
					This is a more-or-less gaurentee that the data isn't stale when it's read
			Pricing
				Wont need to do calculations on the exam, but just remember the main breakdown on how pricing in Dynamo is done (read/write throughput capacity and Storage)
				Provisional Throughput Capacity
					Write Throughput $.0065 per hour for every 10 units
					Read Throughput $.0065 per hour for every 50 units

					Read Unit - 1 read a second
					Write Unit - 1 write a second
				Storage costs
					$.25Gb/month 

				Ex: Assume that your application needs to perform 1 million writes
				and 1 million reads per day, while storing 3 GB of data.

					11.6 writes/reads per second
						Need 12 read units
						Need 12 write units

					On-demand pricing in US East Region
						12 Write Capacity Units would cost $.1872 per day
						12 Read Capacity Units would cost $.0374 per day
							$.2246 per day

					Storage costs $.25 per GB per month
						Assuming a 30-day month, your 3 GB would cost you
						3 * .25 / 30 = $.025/day

					So
					.2246 + .025 = $.2496/day or $7.50/month 

	Data Warehousing Databases (Online Analytics Programming-OLAP)
		These used to be relational in structure, but now these are more of their own types

		Used for business intelligence and large scale reporting
			Ex: Give me the net profit for all apple computers in Japan
				Huge numbers of inputs need to go into this calculation

		Redshift
			COlumnar Data Storage
				Instead of storing data as a series of rows, Redhsift organizes
				the data by column.
					Row-based systems
						Ideal for transaction processing
					Column-based systems
						Ideal for data warehousing and analytics where queries
						often involve aggregates performed over large data sets.
						Since only the columns involved in the queries are 
						processed and columnar data is stored sequentially on the storage media, column-based systems require far fewer
						I/Os, which greatly improces query performance.

						Advanced Compression
							Because of the sequential way that the column-based data
							is stored on disk.

							The data-types are exactly the same in each column

							Redshift doesn't require indexes or materialized views
							thus it uses less space

							As data is loaded into an empty table, Redshift automatically samples your data and selects the most appropriate compression scheme
			Backup
				The whole cluster can only be in one AZ,
					You can restore a snapshot to a new AZ in the event of an outage
			Security
				Encrypted in transit using SSL
				Encrypted at rest using AES-256 encryption
					Key managament by default is taken care of
					You can manage your own keys through Hardware Security Modules (HSMs) or AWS Key Management Service (if needed)

			Pricing
				Compute Node Hours
					aka: Total number of hours you run across all your compute nodes for the billing period

					You are billed for 1 unit per node per hour

					Ex: 3-node data warehouse cluster running persistently for an entire month would invur 2,160 instance hours
						You wont be charged for leader node hours, only compute nodes.
							You will pay if you are using 1 node (by default this combines a leader node and a compute node)
				Backup

				Data transfer
					Only within a VPC, not outside it

			Massively Parallel Processing (MPP)
				Automatically distributes data and query load across all nodes.
			Can start with a single node
				160 Gb

				Would include the functionality of a compute node and a leader node
			Can scale to multi-node
				In these configs
					leader node - Managaes client connections and receives queries
					compute node - Store data and perform queries and computations
						Max: 128 Compute Nodes  

			10x faster than traditional solutions



	ElatiCache
		Web service that makes it easy to deploy, operate, and scale an in-memory cache in the cloud

		If you have a service that is making repeated calls to the db, then we can cache that information via elastiCache.
			Significantly improves latency and throughput for many read-heavy application workloads
				Ex: social networking, gaming, media sharing, Q&A portals
			Compute-intensive workloads
				Ex: recommendation engine

		ElastiCache supports
			Memcached
				Widely adopted memory object caching system
				ElastiCache is protocol compliant with memcached, so popular tools
				that you use today with existing memcached envs will work with the
				ElastiCache service
			Redis
				Open source in-memory key-value store that supports data structures
				such as sorted sets and lists.

				Redundancy
					ElastiCache supports Master/Slave replication

					Allows Multi-AZ redundancy

