---
layout: post
title: "Managing Service Connections in Azure Devops"
hide_title: false
color: rgb(80,140,22)
bootstrap: false
tags: [devops, service connections, azure ad]
---
Housekeeping of App Registrations/SPN's in your Azure AD can be somewhat of a challenge sometimes. In this blogpost I will focus on the following scenario. 

# My problem
Our Azure Devops environment contains a good amount of projects. In each project there is at least one Service Connection defined. A Service Connection is nothing more than an App Registration in your Azure AD with a secret. For a default Service Connection in Azure the expiration date is set 2 years after creation.
- For whatever reason, some (older) projects contain a lot of (>50) Service Connections. Some in use, some don't. 
- The name of the Service Connection is not the same as the App Registration in Azure AD.
- In Azure AD Service Connections from the same Azure DevOps project have the same name. Only a different Application Id...

So how do you find the right Service Connection with, for example, an expired secret inthe Azure DevOps Project so that I can renew the secret?

# What I've tried
- Finding more info in Azure DevOps. No luck.
- Trying the Azure CLI. No luck. Only some simple tasks.
- Making an API-call to Azure DevOps. BINGO!

# The solution (so far). Quick and dirty.
I found the solution on [Stack Overflow](https://stackoverflow.com/questions/72130201/azure-devops-how-to-retrieve-service-connection-information-with-powershell) (where else?) and I want to document it here, because I will probably need this myself in the future.

**Note!** I assume you already have the needed access rights to the project settings / service connections as a user.

## Step 1 - Create a Personal Access Token
Create a personal access token as [described here](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=Windows). Make sure you don't "overpower" it and give a time limit that make sense to you.

## Step 2 - Run the following Powershell code
- Replace `{pat}` with your newly created personal access token.
- Replace `{project}` in `$url` with the project you want get the overview from.
- Run the script.

```powershell
$token = "{pat}"
$token = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes(":$($token)"))
$url="https://dev.azure.com/{organization}/{project}/_apis/serviceendpoint/endpoints?api-version=7.1-preview.4"
$head = @{ Authorization =" Basic $token" }
$endpoints = Invoke-RestMethod -Uri $url -Method GET -Headers $head
$results = $endpoints.value
$results
```
So now you have at least all the data available and you can start digging.
I'm still figuring out how the different entries are build up. But for what I needed it was enough to search on the `AppId` of the App Registration in the output.

I hope this helps. 