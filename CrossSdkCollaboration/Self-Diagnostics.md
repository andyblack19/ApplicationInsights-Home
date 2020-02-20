# Application Insights SDK Self-Diagnostics

## Introduction
We want to improve supportability across Application Insights SDKs by introducing a unified configuration.

This proposal is to introduce a means to enable temporary logging in production environments. Temporary logging instructions would supercede any logging configuration in the SDK, to enable DevOps to take control.

Permanent changes to logging would belong in the SDK's configuration (config file or code-based initialization), but may use this same schema.

### Challenges

- We cannot depend on Customers being able to run additional executables in production to collect product logs.
    - Azure for example requires additional signing to run executables.
- Customers do not want to re-deploy applications to change a configuration. They would like to enable/disable settings in their production environments.


## Proposal
- Provide a tool-less method for collecting logs.
- Environment Variables would enable customers to configure the SDK without a redeploy.
  - Note that this requires restarting an application because our SDKs are not monitoring these settings for changes.
- [Stretch Goal] Turn on/off diagnostics without re-starting an application.
  - Note that this may require a larger engineering effort for some SDKs.

## Technical Specification

### Control Plane

Each product is free to define unique ways to set a configuration (aka "control plane"). But we are asking all SDKs to support Environment Variables for the sake of consistency.

#### Environment Variable
- `APPLICATIONINSIGHTS_SELF_DIAGNOSTICS`


### Configuration String

- The configuration string will consist of a list of key-value pairs separated by semicolon:
`key1=value1;key2=value2;key3=value3`
- Reserved characters
    - `=` equal sign
    - `;` semi colon
- The configuration string will be case insensative. 
- Keys will not be order dependent and are expected to appear only once.
- The full length will not exceed XXX characters. **TODO: Needs a decision**

### Schema


- `Destination`
    - Values: **File**, **ETW**, **Console**, **None**
    - Specifying a destination will "turn-on" temporary logging. This can override anything in the SDK's normal configuration. The destination will define where logs are sent and the value will define the sub keywords.

To revert to the SDK's previously configured logging:
- Destination will be set to None.
- The Environment Variable would be set to string.Empty.
- The Environment Variable would be deleted.
    
#### "Destination=File" schema
All SDKs would support File logging to provide a basic and consistent supportability experience.

- [Optional] `Path`
    - Value: The full path to a directory that the SDK will have write access.
    - Default Value: "%TEMP%"
- [Optional] `Level`
    - Value: Verbose, Information, Warning, Error
    - Default Value: Verbose
    - Reference: https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.tracing.eventlevel?view=netframework-4.8
- [Optional] `MaxSize`
    - Value: Integer specifying max size in either kilobytes or megabytes? **TODO: Needs a decision**
    - Default: no max size? **TODO: Needs a decision**

##### File Name
`ApplicationInsightsLog_{DateTime.UtcNow.ToInvariantString("yyyyMMdd_HHmmss")}_{process.ProcessName}_{process.Id}.log`

This format is proposed because multiple SDKs may read the system's environment variable. This format will help users identify the program or process writing to this file.

##### Full Example
`Destination=File;Path=C:/Temp;Level=verbose;MaxSize=50`

##### Minimum Valid Example
`Destination=File`

##### Commitments
- DotNet: Yes
- Java: Yes
- JavaScript: Not Applicable
- Node: ???
- Python: ???
- App Service Extensions: ???


#### "Destination=ETW" schema

**TO BE DEFINED**

The DotNet SDKs have ETW logging always on. Any changes to this would be a breaking change.

#### "Destination=Console" schema

**TO BE DEFINED**

One option would be to support [Sysinternals DebugView](https://docs.microsoft.com/en-us/sysinternals/downloads/debugview).
DotNet could use `Trace.WriteLine("Hello World")`
