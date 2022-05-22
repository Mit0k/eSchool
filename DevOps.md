DevOps Internship 2022 (SoftServe) <br />

## How to deploy application via Azure DevOps?

To deploy eSchool Application via Azure DevOps you need to create infrastructure for it. For this purpose you can use this [repo](https://github.com/Mit0k/eSchool-Task-3)<br />
After creating first pipeline you need to:

- Create new pipeline based on this repo
- Select an existing YAML file: `azure-pipelines.yml`
- Change variables in `azure-pipelines.yml`:
  - **Location** , **prefix** - same as on previous pipeline
  - **slotName** - name of slot in which app will be deployed
  - **connection** - name of your Azure Resource Manager service connection
  - **sonarConnection** - name of your SonarQube connection. [How to integrate SonarQube and Azure DevOps?](https://docs.sonarqube.org/latest/analysis/azuredevops-integration/)
- Run

### Notes:

- In addition to Maven test tools сode will be analyzed on each run by SonarQube, with providing reports
- Its a quite simple way to deploy app to Azure because this two repos(this and [this](https://github.com/Mit0k/eSchool-Task-3)) was wrote within one task. <br />
**And that is why you need to read spoiler below:**
<details>
    <summary>Warning! Read me!</summary>


Be careful with changing infrastructure deploying template - to minimize count of code both of repos was written using specific name convention.<br />
So some changes can break everything.<br />
For example:<br />
Look on this two snipets:<br />
**file** `templates\azuredeploy.ps1` in infrastructure deploying [repo](https://github.com/Mit0k/eSchool-Task-3):<br />

```powershell
44  Write-Host "##[debug]Getting resource group"
45  $ResourceGroupNames = @()
46  $ResourceGroupNames += "rg-"+$prefix+"-common-base-"+$Location
47  $ResourceGroupNames +="rg-"+$prefix+"-common-metrics-"+$Location
48  $ResourceGroupNames += "rg-"+$prefix+"-APP-"+$Location
```

And
**file** `azure-pipelines.yml` in this repo:<br />

```yaml
52  - task: AzureRmWebAppDeployment@4
53      inputs:
54        ConnectionType: 'AzureRM'
55        azureSubscription: '$(connection)'
56        appType: 'webAppLinux'
57        WebAppName: 'app-$(prefix)-$(location)'
58        deployToSlotOrASE: true
59        ResourceGroupName: 'rg-$(prefix)-APP-$(location)'
60        SlotName: '$(slotName)'
61        packageForLinux: '$(System.DefaultWorkingDirectory)/target/eschool.jar'
62        RuntimeStack: 'JAVA|8-jre8'
63
```

On first snipet you can see how we create names for resource groups, and on second how we use it in different project. I made this to minimize amount of parameters that will be passed from one project to another ,e.g. now you dont need to pass Resource Group and WebApp name.<br />
On one hand it reduce amount of repeating on expanding code, but on another it can be quite confusing at start. Still i tried to use simple name convention which is mostly the same as Azure [recommend](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations):

```
    [asset type]-[some prefix]-[purpose/module name]-[location]
    pip-eschool-webapp-eastus
```
