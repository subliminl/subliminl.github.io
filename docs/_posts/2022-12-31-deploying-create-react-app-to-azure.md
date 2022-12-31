---
layout: post
title:  "Deploying a create-react-app production build to Azure"
date:   2022-12-30 18:54:22 +0100
categories: update
description: "Wherein I make some tweaks to Github Actions."
---
So I have a little ``create-react-app`` site, and I want it hosted on Azure, on my custom domain, secured with SSL.

I found Microsoft’s [instructions](https://azure.microsoft.com/nl-nl/blog/deploy-to-azure-using-github-actions-from-your-favorite-tools/) for deploying using Github Actions.

So now whenever I commit changes it will automatically compile and deploy to Azure. Cool. Job done.

However, there are some issues. 

The script that Microsoft provides deploys the development build, not the production build.

Not only are we uploading everything including `node_modules`, we're doing it one file at a time. This takes quite some time, and uses up precious Github storage.

What we really want to do is upload the contents of the `build` directory.
So, I will configure Github actions to deploy just the production build using a zip file.

Let’s start by creating a new React app:

Now it’s time to set up our Azure app service.
1.	Got to ``portal.azure.com``.
2.	Click **Create a Resource**.
3.	Click Web **App > Create**.
4.	Select or create a subscription and resource group.
5.	Set up your instance details.
    *	**Publish:** Code
    *	**Runtime stack:** If unsure, choose the latest Node package.
    *	**Operating system:** Linux. On Azure, Linux apps are around 25% the cost of Windows apps.
    *	**Region:** As appropriate.
6.	Set up your pricing plan. We want the cheapest option that provides SSL and allows a custom  domain, so we’re going to go for a Basic B1 plan. This should be around 13USD. If it’s higher, make sure you've selected Linux as the operating system. 
 
    <img class='centered' src='/assets/images/azure-plans.png' />

7.	Click **Next: Deployment**.
8.	Enter your Github account details.
9.	Select the Organisation, Repository and Branch for your project.
10.	Click **Review + create**.

Now let’s go back to VS Code.

1.	In a terminal window, run `git fetch` to get the latest files from your repo. 
2.	You’ll see a new .github\workflows folder with a yml file. Open the yml file.
3.	Looks for the steps section.
4.	The second step should be called “npm install, build and test”. If, like me, you haven’t set up any tests, you can remove the whole “npm run test –if-present” line.
5.	Now we need to add an extra step to zip the artifact. Insert the step between the npm install and upload like this:

```yml
    - name: npm install, build, test
```

The name is not important. The command says to zip all the files in the root directory (./*) into release.zip, silently (-q) and including subfolders (-r).
6.	Now in the “Upload artifact for deployment job” we change the path from . to release.zip.
 
7.	Save your changes. 




