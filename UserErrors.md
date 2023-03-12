## User Errors in Azure-SSIS integration

As the documentation suggests, UserErrors can be either Vnet,network related issues, SQL DB related, or custom setup (script) related. UserErrors stem from user side and are not related to Azure.

Vnet, networking related issues are very common, as many users fail to setup the rather complicated networking configurations required by standard VNet injection of SSIS IR. You might get an error such as MisconfiguredDnsServerOrNsgSettings, or Forbidden. In the former case, you are setting up either DNS or NSG incorrectly, but also UDR should be taken into account. One important thing is not to forget to separate IR and sql db into different subnets in the Vnet, and making sure an NSG is both configured for these subnets (setting up one NSG is not enough, as both sides need to be able to communicate).

CustomSetupScriptFailure is quite common, as it is easy to make a mistake and mess up this process. This error can have subcodes indicating their cause such as CustomSetupScriptBlobContainerInaccessible and CustomSetupScriptExecutionFailure. Since custom scripts are long and complicated, testing is required if the suggestion of the error message is not helpful.

DB related errors are either a message suggesting a network problem, a timeout, or a catalog related subcode such as CatalogDbAlreadyExist or CatalogDbCreationFailure.

The first make up the majority, and it is UserErrorConnectSsisDBFailure. It is accompanied with a message like   can also be seen as a performance error, because the SQL DB cannot handle the load coming from SSIS, usually scaling the DB to a higher tier resolved the issue. Setting a retry and timeout is also recommended, but first scaling should be done. Deleting the log count if it is massive is one the last steps to be done if all else do not work. However, it is most likely that it is a resourcing related problem on the DB side.

PackageExecutionErrors (UserErrorSsisPackageExecutionFailed) are most common UserErrors in Azure-SSIS and tend to be transient. They are likely to be caused by internal SSISDB configuration, or driver (OLEDB), that is why troubleshooting via checking the package execution logs in catalogDB is recommended as first step (As users are required to deploy their SSIS packages in the SSIS catalogdb hosted on managed sql or azure sql). They are different from others listed above in that they are from an inherent cause in SSIS system, and not caused by the IR. If this is permenant, then it suggests there is something wrong with your package, such as syntax.



For other minor errors see the documentation : https://learn.microsoft.com/en-us/azure/data-factory/ssis-integration-runtime-management-troubleshoot
