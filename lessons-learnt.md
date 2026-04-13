# JBPM (and Business Central) Lessons Learnt

JBPM 7.61.0.Final (last version on Docker Hub)

## Rest Interface

The Swagger documentation for the interface is available at URL `http://localhost:8080/kie-server/docs/` with Business Central running.

## Settings

### Data Sources

A data source can be defined in the Business Central settings but it is necessary to inspect the file `/opt/jboss/wildfly/standalone/configuration/standalone.xml` to determine its JNDI name.
This JNDI name can then be used to construct an ExecuteSQL work item.

## Custom Work Items

The custom work items (WI) should be viewed as example and/or starter projects which may contain bugs (e.g. Archive does not properly process archive entries). 
Work item definition (WID) metadata (defined a the top of the main Java file of the WI) are often not 100% accurate especially for the parameter data types.

Before attempting to use one of these WI it is vital to inspect the source code in order to void frustrating usage failures.

https://github.com/kiegroup/jbpm-work-items

### Constructor Parameters

Some custom WI have constructor parameters which need to be set when the work item handler (WIH) is added to the JBPM project in the deployment settings.  The WIH
should be added automatically, including its relevant parameter values, when it is installed into a project via settings but this is not reliable. (It is recommended to verify parameters by inspection of the main Java file).

### CLASSPATH Hell

If a custom WI has dependencies on third party jars then these must be declared (using `compile` scope) in the POM of the work item, 
it is not sufficient to just add it as a separate dependency to the JBPM project in Business Central.

If the dependencies of a custom work item are changed, then following upload of the updated WI jar it is necessary to stop and remove any dependent JBPM project/server before their redeployment.

When a WI is updated is it recommended to increment the project version in the POM before rebuild and upload to Busness Central.

### Maven Archetype

When creating a new custom WI from scratch it is important to generate it from the Maven archetype, do not be tempted to clone and edit an existing project.  When this was tried once, 
the Business Central work item importer became completely inoperative!

### Naming

Do not embed number(s) into the name of a work item.  The WI will import but if the WID uses the same base name (which it should for consistency) then embedded numbers will cause the work item to not be shown in the
Business Central task selection dialog and hence it cannot be added to a process diagram!

## Business Rules (DRL Files)

In order to execute business rules it is necessary to specify the rule flow group in the DRL

		rule "Match CS01"
		    ruleflow-group "cs01-rfg"

and to reference this is the rules task.
