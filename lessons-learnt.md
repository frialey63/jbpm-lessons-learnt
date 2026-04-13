# JBPM Lessons Learnt

JBPM 7.61.0.Final (last version on Docker Hub)

# Rest Interface

The Swagger documentation for the interface is available at URL `http://localhost:8080/kie-server/docs/`

## Settings

### Data Sources

A data source can be defined in the Business Central settings but it is necessary to inspect the file `/opt/jboss/wildfly/standalone/configuration/standalone.xml` to determine its JNDI name.

## Custom Work Items

The custom work items should be cviewed as examples and/or starter projects which may contain bugs (e.g. Archive Workitem does not properly process archive entries), 
for example the WID metadata (defined a the top of the main WIH Java file) are often not 100% accurate especially for the parameter data types.

Before attempting to use one of these examples it is vital to inspect the source code in order to void frustrating usage failures.

https://github.com/kiegroup/jbpm-work-items

### Constructor Parameters

Some custom work items have constructor parameters which need to be set when the WIH is added to the JBPM project in the deployment settings.  The WIH with parameters (placeholders or values)
should be added automatically when it is installed into a project via the settings. (However, it is recommended to check these by inspection of the Java source code).

### CLASSPATH Hell

If a custom work item has dependencies on third party jars then these must be declared (using compile scope) in the POM of the work item, 
it is not sufficient to just add it as a separate dependency to the JBPM project in Business Central.

If the dependencies in a custom work item POM are changed, following upload of the updated WI it is necessary to stop and remove any dependent JBPM project/server before its redeployment.

When a WIH is updated is it recommended to increment the project version in the POM before rebuild and uploading to Busness Central.

### Maven Archetype

When creating a new custom work item from scratch it is important to generate it from the Maven archetype, do not be tempted to clone an existing project.  When this was tried once, 
the Business Central work item importer became completely broken!

### Naming

Do not embed number(s) into the name of a work item.  The WI will import but if the WID uses the same name (which it should for consistency) then embedded numbers will cause the WI to not be shown in the
Business Central task selection dialog and hence it cannot be added to a process diagram!

## DRL Files

In order to execute the business rules it is necessary to specify the rule flow group in the DRL

		rule "Match CS01"
		    ruleflow-group "cs01-rfg"

and to reference this is the rules task.
