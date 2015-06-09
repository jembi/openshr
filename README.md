# OpenSHR

The OpenSHR is a shared health record (SHR) that allows clinical data to be stored in a central location within a health information exchange (HIE).

OpenSHR implements the IHE XDS.b profile for receiving and retrieving clinical documents about a particular patient.

Some of the important features include:

* Supports storage of data discretely of supported CDA documents
* Supports on-demand documents
* Extensible CDA documents support

## Setting up the OpenSHR with vagrant and puppet

See the https://github.com/jembi/openmrs-shr-infrastructure repository. Getting up and running is really simple:
```
git clone https://github.com/jembi/openmrs-shr-infrastructure.git
cd openmrs-shr-infrastructure
vagrant up
```
After launching, the SHR will be available on `http://localhost:8082/openmrs`. The default login is `admin` using the password `OpenMRS123`.

## Setting up the OpenSHR without vagrant

We are currently working on bundling a lot of the different components of the OpenSHR together to make it simpler to install. Right now these are the steps you have to go through to get it up and running.

1. To install the OpenSHR you will need to first install OpenMRS v1.11.x. You can find details about how to do that [here](https://wiki.openmrs.org/display/docs/Installing+OpenMRS).
2. Once you get to the OpenMRS configuration web app, you will need to first load in a pre-configured OpenMRS database before continuing. The database is included in this repository and you can load it using the following command: `mysql -u root -p -e "create database openmrs" && mysql openmrs -u root -p < openshr-dump-clean-full-ciel-no-caching-gp-optimisations-05-06-15.sql`. This will setup the database for you with some sane defaults. This is what will be setup for you in this database dump:
  * The OpenMRS database structure with no default data
  * The CIEL dictionary
  * Sane defaults for the SHR module global properties (Clear EPID, ECID root global properties and `change update existing` GP to true so that we can use the same document over and over for testing)
  * Global properties that improve performance
3. Load the SHR openmrs modules in the following order:
  1. [shr-atna-0.5.0.omod](https://github.com/jembi/openmrs-module-shr-atna/releases)
  2. [shr-contenthandler-2.2.0.omod](https://github.com/jembi/openmrs-module-shr-contenthandler/releases)
  3. [xds-b-repository-0.4.5.omod](https://github.com/jembi/openmrs-module-shr-xds-b-repository/releases)
  4. [shr-cdahandler-0.6.0.omod](https://github.com/jembi/openmrs-module-shr-cdahandler/releases)
4. Ensure there are no errors in the logs when loading concepts via liquibase when the cda-handler module starts, if there are you may need to start again...
5. The SHR should be up and running. The default login is set to `admin` with password `OpenMRS123`. You will now need to set some global properties to get the OpenSHR to function correctly:
  * In the XDS-repository module settings, set the ws username and password to an admin user name and password

## Repositories that make up the OpenSHR project

The source code repositories that make up the OpenSHR project can be found here:

* Infrastructure code (puppet and vagrant) that helps you get an environment setup - https://github.com/jembi/openmrs-shr-infrastructure
* Module that exposes the XDS.b interface - https://github.com/jembi/openmrs-module-shr-xds-b-repository
* Module that allows handlers to register support for different types of docuemnts - https://github.com/jembi/openmrs-module-shr-contenthandler
* Module that supports handling of certain CDA documents - https://github.com/jembi/openmrs-module-shr-cdahandler
* Module that supports on-demand documents - https://github.com/jembi/openmrs-module-shr-odd
* Module that provides ATNA audit logging support - https://github.com/jembi/openmrs-module-shr-atna

## Performance tweaks

To get the best performance out of the OpenSHR you may have to make some adjustments to your installation. A list of these can be found here. Many of the global properties are already applied when using the database dump above.

In order of biggest difference noticed:

1. Set GP `search.caseSensitiveDatabaseStringComparison` to false
2. Ensure hibernate batch options are enabled in OpenMRS (see PR https://github.com/openmrs/openmrs-core/pull/1508). In `hibernate.default.properties` file set:
  * `hibernate.jdbc.batch_size=50`
  * `hibernate.order_inserts=true`
  * `hibernate.order_updates=true`
3. Disable OpenMRS validation GP (see PR https://github.com/openmrs/openmrs-core/pull/1503).
  * Set `validation.disable` to true
4. Set the following GPs:
  * `shr-cdahandler.validation.cda` set to false
  * `shr-cdahandler.validation.conceptStructure` set to false
  * `patient.nameValidationRegex` clear value
5. Tune InnoDB by following this guide:
https://www.percona.com/blog/2007/11/01/innodb-performance-optimization-basics/

### Resources

* https://wiki.openmrs.org/display/docs/Performance+Tuning
* http://blog.jhades.org/performance-tuning-of-spring-hibernate-applications/
* https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/5/html/Performance_Tuning_Guide/sect-Performance_Tuning_Guide-Entity_Beans-Batching_Database_Operations.html
* http://www.webdevstuff.com/123/optimizing-mysql-performance-tuning-script.html

### Next steps for performance

* Try get rid of the CIEL concepts we don't need and see if that makes a difference.
* Try one big transaction
* Try MariaDB
* Build something better
* Eventually consistent is ok
