geoshape_geonode Deployment
===========================
Deploys an initial version of the GeoShape/Rogue version of GeoNode.

Requirements
------------
####Cookbooks
Chef community cookbook requirements are met through the mu repository and this repository

####Credentials
GeoShape uses Chef Vault for credentials.  The platform.mu server must have a vault called geoshape_geonode present before deployment with two items ... geoshape_os and unison_os that hold account credentials.  Create them on a one-time basis like this:

`knife vault create geoshape_geonode geoshape_os '{"username": "rogue", "password": "somepass"}'`

`knife vault create geoshape_geonode unison_os '{"username": "unison", "password": "somepass"}'`
	
If you need to add an existing node to the vault later try this:

`knife vault update geoshape_geonode unison_os --search name:GEOSHAPE-DEV-2014121418-ZI-GEONODE-IG5`

`knife vault update geoshape_geonode geoshape_os --search name:GEOSHAPE-DEV-2014121418-ZI-GEONODE-IG5`

Where the name is the MU-ID/Nodename of the desired node

And to check your vaults:
`knife vault edit geoshape_geonode geoshape_os`

Current Capabilities
--------------------
- Deploys a single self-contained instance of GeoShape
- Can target an existing VPC or create own VPC
- Deploys in a 1:1 autoscale group/pool for higher availability

Current Limitations
-------------------
- No load balancing, although an ELB is created in the master version of deployment
- Deploys in a public subnet at this time
- Runs in Ubuntu only

Planned Enhancements
--------------------
- Add CentOS.  Currently the platform.mu version runs on ubuntu 
- Add Ubuntu hardening
- Complete production baskets of kittens (BOKs) in VPC.  Currently only dev works
- Clean up artifacts.  Many artifacts in applications are hints for future development and do not yet function
- Deploy servers in private subnet behind elastic load balancer -- currently deploying in public subnet and ELB is present but not yet used
- Move community cookbooks to a berkshelf implementation once integrated in platform.mu

Attributes
-----------
#### postgres overrides for Platform.mu
The platform.mu provisioning tool already contains the postgres cookbook; the necessary attributes for version 9.3 have been overridden in rogue's attributes file:

- `node.normal.postgresql.enable_pgdg_apt = true`

- `# Override pgsql version and dependent attributes, per https://www.chef.io/blog/2013/12/03/doing-wrapper-cookbooks-right/`
- `# These will need to be made deployment-specific to handle CentOS later on. Note directory from arahav`
- `default['postgresql']['version'] = '9.3'`
- `default['postgresql']['client']['packages'] = ["postgresql-client-#{node['postgresql']['version']}","libpq-dev"]`
- `default['postgresql']['server']['packages'] = ["postgresql-#{node['postgresql']['version']}"]`
- `default['postgresql']['contrib']['packages'] = ["postgresql-contrib-#{node['postgresql']['version']}"]`
- `default['postgresql']['dir'] = "/var/lib/postgresql/#{node.postgresql.version}/main"
default['postgresql']['server']['servicename'] = "postgresql-#{node['postgresql']['version']}"`


Usage
-----
#### Carry out a master deployment that creates a VPC and deploys into it
Expected time 34 minutes

`mu-deploy  /opt/mu/geocloud_platform/applications/geoshape_geonode/master.json`

Same, but exclude zone us-east-1a -- needed in some older accounts with AZs that do not support VPC subnets:

`mu-deploy  /opt/mu/var/geocloud_platform/applications/geoshape_geonode/master.json -p azskip=us-east-1a -p appname=fullshape`

#### Deploy into an existing dev VPC
The target VPC has the MU-ID of ALLZONEGEONODE-DEV-2015042309-RD:

Note that for now you need to discover (through the deploy's BOK) and insert the vpc_name

`mu-deploy /opt/mu/var/geocloud_platform/applications/geoshape_geonode/master.json -p azskip=us-east-1a -p deploy_id=ALLZONEGEONODE-DEV-2015042309-RD -p vpc_name=allzonegeonode-vpc '' 



Errata
------
- Currently the chef pg gem build fails due to a defect in the Rogue recipe.  Our recipe works around and recovers from the failure
- Currently the initial full chef run fails when Rogue tries to connect to the database.  May be a timing issue; the automatic retry seems to repair this problem

