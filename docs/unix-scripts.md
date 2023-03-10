# Unix Scripts in FlashPipe

_FlashPipe_ provides the following Unix scripts for accessing SAP Integration Suite APIs.
- **update_designtime_artifact.sh**
- **deploy_runtime_artifact.sh**
- **sync_to_git_repository.sh**
- **snapshot_to_git_repository.sh**
- **update_integration_package.sh**

These scripts perform the _magic_ that significantly simplifies the steps required to complete the build and deploy steps in a CI/CD pipeline.

The following section describes the usage of the scripts. Since version 2.0.0, input values are passed via environment variables instead of command line arguments.

### 1. update_designtime_artifact.sh
This script is used to create/update a Cloud Integration designtime artifact to the tenant. It provides the following functionalities:
- check existence of artifact to determine if it needs to be created or updated
- create Integration Package (if it does not exist) to store the artifact
- compare contents of artifact in Git repository against tenant to determine if artifact in tenant needs to be updated
- use different `parameters.prop` files to handle different configuration values when deploying multiple copies of artifact to same/different tenants
- create/update designtime artifact
- handle conversion of script collection references when suffix/prefix are used (for deployment of multiple copies in same tenant/different tenants)


#### Usage and environment variable list
```bash
/usr/bin/update_designtime_artifact.sh

Mandatory environment variables:
    HOST_TMN - Base URL for tenant management node of Cloud Integration (excluding the https:// prefix)
    BASIC_USERID - User ID (required when using Basic Authentication)
    BASIC_PASSWORD - Password (required when using Basic Authentication)
    HOST_OAUTH - Host name for OAuth authentication server (required when using OAuth Authentication, excluding the https:// prefix)
    OAUTH_CLIENTID - OAuth Client ID (required when using OAuth Authentication)
    OAUTH_CLIENTSECRET - OAuth Client Secret (required when using OAuth Authentication)
    IFLOW_ID - ID of Integration Flow
    IFLOW_NAME - Name of Integration Flow
    PACKAGE_ID - ID of Integration Package
    PACKAGE_NAME - Name of Integration Package
    GIT_SRC_DIR - Directory containing contents of Integration Flow

Optional environment variables:
    PARAM_FILE - Use to a different parameters.prop file instead of the default in src/main/resources/
    MANIFEST_FILE - [DEPRECATED] Use a different MANIFEST.MF file instead of the default in META-INF/
    WORK_DIR - Working directory for in-transit files (default is /tmp if not set)
    HOST_OAUTH_PATH - Specific path for OAuth token server, e.g. example /oauth2/api/v1/token for Neo environments (default is /oauth/token if not set for CF environments)
    VERSION_HANDLING - [DEPRECATED] Determination of version number during artifact update
    SCRIPT_COLLECTION_MAP - Comma-separated source-target ID pairs for converting script collection references during upload/update

NOTE: Encapsulate values in double quotes ("") if there are space characters in them
```

#### Example (OAuth for Cloud Foundry)
```bash
/usr/bin/update_designtime_artifact.sh

Environment variables set before call:
    HOST_TMN: ***.hana.ondemand.com
    HOST_OAUTH: ***.authentication.<region>.hana.ondemand.com
    OAUTH_CLIENTID: <clientid>
    OAUTH_CLIENTSECRET: <clientsecret>
    IFLOW_ID: GroovyXMLTransformation
    IFLOW_NAME: "Groovy XML Transformation"
    PACKAGE_ID: FlashPipeDemo
    PACKAGE_NAME: "FlashPipe Demo"
    GIT_SRC_DIR: "FlashPipe Demo/Groovy XML Transformation"
    SCRIPT_COLLECTION_MAP: "DEV_Common_Scripts=Common_Scripts"
```

#### Example (OAuth for Neo)
```bash
/usr/bin/update_designtime_artifact.sh

Environment variables set before call:
    HOST_TMN: ***.hana.ondemand.com
    HOST_OAUTH: oauthasservices-<tenantid>.<region>.hana.ondemand.com
    HOST_OAUTH_PATH: /oauth2/api/v1/token
    OAUTH_CLIENTID: <clientid>
    OAUTH_CLIENTSECRET: <clientsecret>
    IFLOW_ID: GroovyXMLTransformation
    IFLOW_NAME: "Groovy XML Transformation"
    PACKAGE_ID: FlashPipeDemo
    PACKAGE_NAME: "FlashPipe Demo"
    GIT_SRC_DIR: "FlashPipe Demo/Groovy XML Transformation"
```

### 2. deploy_runtime_artifact.sh
This script is used to deploy Cloud Integration designtime artifact(s) to the runtime. It will compare the version of the designtime artifact against the runtime artifact before executing deployment if there are differences.


#### Usage and environment variable list
```bash
/usr/bin/deploy_runtime_artifact.sh

Mandatory environment variables:
    HOST_TMN - Base URL for tenant management node of Cloud Integration (excluding the https:// prefix)
    BASIC_USERID - User ID (required when using Basic Authentication)
    BASIC_PASSWORD - Password (required when using Basic Authentication)
    HOST_OAUTH - Host name for OAuth authentication server (required when using OAuth Authentication, excluding the https:// prefix)
    OAUTH_CLIENTID - OAuth Client ID (required when using OAuth Authentication)
    OAUTH_CLIENTSECRET - OAuth Client Secret (required when using OAuth Authentication)
    IFLOW_ID - Comma separated list of Integration Flow IDs

Optional environment variables:
    DELAY_LENGTH - Delay (in seconds) between each check of IFlow deployment status (default to 30 if not set)
    MAX_CHECK_LIMIT - Max number of times to check for IFlow deployment status (default to 10 if not set)
    COMPARE_VERSIONS - Perform version comparison of design time against runtime before deployment. Allowed values: true (default), false
    HOST_OAUTH_PATH - Specific path for OAuth token server, e.g. example /oauth2/api/v1/token for Neo environments (default is /oauth/token if not set for CF environments)
```

#### Example (OAuth for Cloud Foundry)
```bash
/usr/bin/deploy_runtime_artifact.sh

Environment variables set before call:
    HOST_TMN: ***.hana.ondemand.com
    HOST_OAUTH: ***.authentication.<region>.hana.ondemand.com
    OAUTH_CLIENTID: <clientid>
    OAUTH_CLIENTSECRET: <clientsecret>
    IFLOW_ID: GroovyXMLTransformation
```

#### Example (OAuth for Neo)
```bash
/usr/bin/deploy_runtime_artifact.sh

Environment variables set before call:
    HOST_TMN: ***.hana.ondemand.com
    HOST_OAUTH: oauthasservices-<tenantid>.<region>.hana.ondemand.com
    HOST_OAUTH_PATH: /oauth2/api/v1/token
    OAUTH_CLIENTID: <clientid>
    OAUTH_CLIENTSECRET: <clientsecret>
    IFLOW_ID: GroovyXMLTransformation
```

### 3. sync_to_git_repository.sh
This script is used to sync Cloud Integration designtime artifacts and integration package details (optional) from a tenant to a Git repository. It will compare any differences (new, deleted, changed) in files from tenant and commit/push to the Git repository.


#### Usage and environment variable list
```bash
/usr/bin/sync_to_git_repository.sh

Mandatory environment variables:
    HOST_TMN - Base URL for tenant management node of Cloud Integration (excluding the https:// prefix)
    BASIC_USERID - User ID (required when using Basic Authentication)
    BASIC_PASSWORD - Password (required when using Basic Authentication)
    HOST_OAUTH - Host name for OAuth authentication server (required when using OAuth Authentication, excluding the https:// prefix)
    OAUTH_CLIENTID - OAuth Client ID (required when using OAuth Authentication)
    OAUTH_CLIENTSECRET - OAuth Client Secret (required when using OAuth Authentication)
    PACKAGE_ID - ID of Integration Package
    GIT_SRC_DIR - Base directory containing contents of Integration Flow(s)

Optional environment variables:
    INCLUDE_IDS - List of included IFlow IDs
    EXCLUDE_IDS - List of excluded IFlow IDs
    DRAFT_HANDLING - Handling when IFlow is in draft version. Allowed values: SKIP (default), ADD, ERROR
    DIR_NAMING_TYPE - Name IFlow directories by ID or Name. Allowed values: ID (default), NAME
    COMMIT_MESSAGE - Message used in commit
    SCRIPT_COLLECTION_MAP - Comma-separated source-target ID pairs for converting script collection references during sync
    NORMALIZE_MANIFEST_ACTION - Action for normalizing IFlow ID & Name in MANIFEST.MF. Allowed values: NONE (default), ADD_PREFIX, ADD_SUFFIX, DELETE_PREFIX, DELETE_SUFFIX
    NORMALIZE_MANIFEST_PREFIX_SUFFIX - Prefix/suffix used for normalizing IFlow ID & Name in MANIFEST.MF
    SYNC_PACKAGE_LEVEL_DETAILS - Sync details of Integration Package. Allowed values: NO (default), YES
    NORMALIZE_PACKAGE_ACTION - Action for normalizing Package ID & Name package file. Allowed values: NONE (default), ADD_PREFIX, ADD_SUFFIX, DELETE_PREFIX, DELETE_SUFFIX
    NORMALIZE_PACKAGE_ID_PREFIX_SUFFIX - Prefix/suffix used for normalizing Package ID
    NORMALIZE_PACKAGE_NAME_PREFIX_SUFFIX - Prefix/suffix used for normalizing Package Name
    WORK_DIR - Working directory for in-transit files (default is /tmp if not set)
    HOST_OAUTH_PATH - Specific path for OAuth token server, e.g. example /oauth2/api/v1/token for Neo environments (default is /oauth/token if not set for CF environments)
```

#### Example (OAuth for Cloud Foundry)
```bash
/usr/bin/sync_to_git_repository.sh

Environment variables set before call:
    HOST_TMN: ***.hana.ondemand.com
    HOST_OAUTH: ***.authentication.<region>.hana.ondemand.com
    OAUTH_CLIENTID: <clientid>
    OAUTH_CLIENTSECRET: <clientsecret>
    PACKAGE_ID: FlashPipeDemo
    GIT_SRC_DIR: "FlashPipe Demo"
    SYNC_PACKAGE_LEVEL_DETAILS: YES
    NORMALIZE_PACKAGE_ACTION: DELETE_PREFIX
    NORMALIZE_PACKAGE_ID_PREFIX_SUFFIX: DEV
```

#### Example (OAuth for Neo)
```bash
/usr/bin/sync_to_git_repository.sh

Environment variables set before call:
    HOST_TMN: ***.hana.ondemand.com
    HOST_OAUTH: oauthasservices-<tenantid>.<region>.hana.ondemand.com
    HOST_OAUTH_PATH: /oauth2/api/v1/token
    OAUTH_CLIENTID: <clientid>
    OAUTH_CLIENTSECRET: <clientsecret>
    PACKAGE_ID: FlashPipeDemo
    GIT_SRC_DIR: "FlashPipe Demo"
    SYNC_PACKAGE_LEVEL_DETAILS: YES
    NORMALIZE_PACKAGE_ACTION: DELETE_PREFIX
    NORMALIZE_PACKAGE_ID_PREFIX_SUFFIX: DEV
```

### 4. snapshot_to_git_repository.sh
This script is used to capture a snapshot of the Cloud Integration tenant's artifacts and integration package details (optional) to a Git repository. It will compare any differences (new, deleted, changed) in files from tenant and commit/push to the Git repository.


#### Usage and environment variable list
```bash
/usr/bin/snapshot_to_git_repository.sh

Mandatory environment variables:
    HOST_TMN - Base URL for tenant management node of Cloud Integration (excluding the https:// prefix)
    BASIC_USERID - User ID (required when using Basic Authentication)
    BASIC_PASSWORD - Password (required when using Basic Authentication)
    HOST_OAUTH - Host name for OAuth authentication server (required when using OAuth Authentication, excluding the https:// prefix)
    OAUTH_CLIENTID - OAuth Client ID (required when using OAuth Authentication)
    OAUTH_CLIENTSECRET - OAuth Client Secret (required when using OAuth Authentication)
    GIT_SRC_DIR - Base directory containing contents of artifacts (grouped into packages)

Optional environment variables:
    DRAFT_HANDLING - Handling when IFlow is in draft version. Allowed values: SKIP (default), ADD, ERROR
    COMMIT_MESSAGE - Message used in commit
    SYNC_PACKAGE_LEVEL_DETAILS - Sync details of Integration Package. Allowed values: NO (default), YES
    WORK_DIR - Working directory for in-transit files (default is /tmp if not set)
    HOST_OAUTH_PATH - Specific path for OAuth token server, e.g. example /oauth2/api/v1/token for Neo environments (default is /oauth/token if not set for CF environments)
```

#### Example (OAuth for Cloud Foundry)
```bash
/usr/bin/snapshot_to_git_repository.sh

Environment variables set before call:
    HOST_TMN: ***.hana.ondemand.com
    HOST_OAUTH: ***.authentication.<region>.hana.ondemand.com
    OAUTH_CLIENTID: <clientid>
    OAUTH_CLIENTSECRET: <clientsecret>
    GIT_SRC_DIR: "TrialTenant"
```

#### Example (OAuth for Neo)
```bash
/usr/bin/snapshot_to_git_repository.sh

Environment variables set before call:
    HOST_TMN: ***.hana.ondemand.com
    HOST_OAUTH: oauthasservices-<tenantid>.<region>.hana.ondemand.com
    HOST_OAUTH_PATH: /oauth2/api/v1/token
    OAUTH_CLIENTID: <clientid>
    OAUTH_CLIENTSECRET: <clientsecret>
    GIT_SRC_DIR: "TrialTenant"
```

### 5. update_integration_package.sh
This script is used to create/update a Cloud Integration `integration package` to the tenant. It provides the following functionalities:
- check existence of package to determine if it needs to be created or updated
- compare contents of package in Git repository against tenant to determine if package in tenant needs to be updated
- create/update integration package

#### Usage and environment variable list
```bash
/usr/bin/update_integration_package.sh

Mandatory environment variables:
    HOST_TMN - Base URL for tenant management node of Cloud Integration (excluding the https:// prefix)
    BASIC_USERID - User ID (required when using Basic Authentication)
    BASIC_PASSWORD - Password (required when using Basic Authentication)
    HOST_OAUTH - Host name for OAuth authentication server (required when using OAuth Authentication, excluding the https:// prefix)
    OAUTH_CLIENTID - OAuth Client ID (required when using OAuth Authentication)
    OAUTH_CLIENTSECRET - OAuth Client Secret (required when using OAuth Authentication)
    PACKAGE_FILE - Path to location of package file

Optional environment variables:
    PACKAGE_ID - Use package ID from variable rather than from file
    PACKAGE_NAME - Use package Name from variable rather than from file
    WORK_DIR - Working directory for in-transit files (default is /tmp if not set)
    HOST_OAUTH_PATH - Specific path for OAuth token server, e.g. example /oauth2/api/v1/token for Neo environments (default is /oauth/token if not set for CF environments)
```

#### Example (OAuth for Cloud Foundry)
```bash
/usr/bin/update_integration_package.sh

Environment variables set before call:
    HOST_TMN: ***.hana.ondemand.com
    HOST_OAUTH: ***.authentication.<region>.hana.ondemand.com
    OAUTH_CLIENTID: <clientid>
    OAUTH_CLIENTSECRET: <clientsecret>
    PACKAGE_FILE: "<path_to_file>/FlashPipeDemo.json"
```

#### Example (OAuth for Neo)
```bash
/usr/bin/update_integration_package.sh

Environment variables set before call:
    HOST_TMN: ***.hana.ondemand.com
    HOST_OAUTH: oauthasservices-<tenantid>.<region>.hana.ondemand.com
    HOST_OAUTH_PATH: /oauth2/api/v1/token
    OAUTH_CLIENTID: <clientid>
    OAUTH_CLIENTSECRET: <clientsecret>
    PACKAGE_FILE: "<path_to_file>/FlashPipeDemo.json"
```
