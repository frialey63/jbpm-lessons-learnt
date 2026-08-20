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
should be added automatically, including its relevant parameter values, when it is installed into a project via settings but this is not reliable. (It is recommended to verify parameters by inspection of the main Java file).  The constructor parameters are specified in the `serviceInfo.authInfo` section of `@Wid` descriptor of the WIH main class, and are used to configure empty values or to prompt for values.

### Input Variables

A custom WI has an input parameter `Param` which has been assigned from a process variable `theParam` but when the WI executes the following error is encountered

    Caused by: java.lang.IllegalArgumentException: Workitem declares following required parameter which does not exist: Param

This is because the process variable has not been set correctly in the script (task), it is necessary to utilise the following code

    kcontext.setVariable("theParam", "value");

### Misc

After a custom WIH has been installed it is necessary to add the jar explicitly into the project dependencies.

When using Settings | Custom Tasks Admin, sometimes the upload of the WI jar is reported as being successful but the custom task is not listed for switch on/off.  In this case, it should be possible to still utilise the WI in a project but it is necessary to create the WID manually using the editor, e.g.

	[
	        [
	            "name" : "ChGenericRestGetDefinitions",
	            "displayName" : "ChGenericRestGetDefinitions",
	            "category" : "ch-generic-rest-get-workitem",
	            "description" : "",
	            "defaultHandler" : "mvel: new uk.gov.ch.ChGenericRestGetWorkItemHandler(\"baseUrl\")",
	            "documentation" : "form-xml-to-java/index.html",
	            "parameters" : [
	                                "Method" : new StringDataType(),
	                                "FormXml" : new StringDataType()
	            ],
	            "results" : [
	                                "Result" : new ObjectDataType()
	            ],
	            "mavenDependencies" : [
	                                 "uk.gov.ch:ch-generic-rest-get-workitem:1.0.0-SNAPSHOT"
	            ],
	            "icon" : "ChGenericRestGetDefinitions.png"
	        ]
	]

### CLASSPATH Hell

If a custom WI has dependencies on third party jars then these must be declared (using `compile` scope) in the POM of the work item, 
it is not sufficient to just add it as a separate dependency to the JBPM project in Business Central.

If the dependencies of a custom work item are changed, then following upload of the updated WI jar it is necessary to stop and remove any dependent JBPM project/server before their redeployment.

When a WI is updated is it recommended to increment the project version in the POM before rebuild and upload to Busness Central.

To build a custom WIH project it is necessary to use the JDK (re Eclipse IDE build settings).

Sometimes I cannot make a custom WIH run without classpath errors, even when for an uber jar built by the Maven assembly plugin (and all dependencies and transitive dependencies specified in the POM).

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
