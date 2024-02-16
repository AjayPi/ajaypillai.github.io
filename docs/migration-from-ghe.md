# GitHub EMU - Migration from GitHub Enterprise

## Preliminary steps

<ul>
  <li>Determine if you already have a "The Product" Project workspace.</li>
  <li>Determine if the GitHub EMU Org already exists inside your "The Product" Project workspace.</li>
  <ul>
    <li>If both of the above exist, great. You can continue with the steps below.</li>
    <li>If not, you can <u>Create an EMU Org</u> and <u>assign roles</u> for the org.
</li>
  </ul>
  <li>Make sure that you are the GHE Org owner of source repos.</li>
  <li>Determine if you are using open source actions (synced-actions).</li>
  <ul>
    <li>If so, you can <a href ="mailto: pillai.ajay.m@gmail.com">contact us</a> for these to be whitelisted in EMU.</li>
  </ul>
  <li>Review GitHub runners code and plan for making changes.</li>
  <ul>
    <li>EMU, "The Product" GHE runners have different labels and different connectivity to on-prem servers, please refer to <a href="#2">"Runner connectivity matrix"</a> below and plan for runner code changes</li>
  </ul>
  </ul>
    

## Prerequisites
  
### Migration tooling

<ul>
    <li>GitHub CLI - https://github.com/cli/cli#installation </li>
    <li>GitHub CLI extension gh-gei - https://github.com/github/gh-gei#using-the-gei-cli</li> 
  </ul>
    
GitHub CLI and the gh-gei extension are available for all platforms including Windows. However, the interactive script referenced in <a href="#1">Step 0</a> of the section below was created with Linux/Mac in mind. It may be able to execute on Windows Subsystem for Linux (WSL) but that hasn’t been tested.
  
### Permissions to use the tool to migrate the repositories.

<b>Source organization</b> (github.comp-the-product.com):
<ul>
  <li>Organization owner role</li>
  <li><a href="https://docs.github.com/en/migrations/using-github-enterprise-importer/preparing-to-migrate-with-github-enterprise-importer/managing-access-for-github-enterprise-importer#personal-access-tokens-for-github-products">Personal Access Tokens</a> (classic, NOT a fine-grained personal access token) with following permissions:</li>
  	
<ul>
    <li>read:org</li>
    <li>repo (all permissions)</li>
</ul>
</ul>
    
<b>Destination organization</b> (EMU github.com):
  <ul>
    <li><a href="https://docs.github.com/en/migrations/using-github-enterprise-importer/preparing-to-migrate-with-github-enterprise-importer/granting-the-migrator-role-for-github-enterprise-importer#about-the-migrator-role">Migrator role for GitHub Enterprise Importer</a> (by default assigned to project admin, if you want to delegate this to a team member, please <a href ="mailto: pillai.ajay.m@gmail.com">contact us</a>.</li>
    <li><a href="https://docs.github.com/en/migrations/using-github-enterprise-importer/preparing-to-migrate-with-github-enterprise-importer/managing-access-for-github-enterprise-importer#personal-access-tokens-for-github-products">Personal Access Tokens</a> (classic, NOT a fine-grained personal access token) with following permissions:</li>

<ul>
      <li>read:org</li>
      <li>repo (all permissions)</li>
      <li>workflow (all permissions)</li>
   </ul>
	    
<li>Since GitHub EMU is OpenID Connect (OIDC) authenticated, ensure SSO authorization is enabled for the destination organization when creating the Personal Access Token. See image below for reference.</li>
</ul>


    
### Analysis & Planning
    
<ul>
  <li>Not everything can be migrated, therefore it is important to take note of items that will need to be transferred over manually.</li>
  <li>Check the documentation <a href="https://docs.github.com/en/early-access/enterprise-importer/understanding-github-enterprise-importer/migration-support-for-github-enterprise-importer#github-enterprise-server-migration-support">here</a> for the latest updated list of migration limitations.</li>
  <li><a href="https://github.com/github/gh-migration-analyzer">GitHub Migration Analyzer</a> is a self service tool that analyzes repositories and provides information that would be helpful in prioritizing and planning the migration. Some of items like Secrets, GitHub Advanced Security Alerts, Teams, Large File Storage (LFS) objects, Webhooks and Actions workflows are discussed in Step 5 of the procedure below.</li>
  </ul>

### Limitations	
  <ul>
    <li>GitHub Migration Analyzer supports analyses at only organization level on GitHub Enterprise Server (github.comp-the-product.com), it cannot be used to analyze single repositories.</li>
  </ul>
	
<details>
  <summary>Analyzing Steps</summary>

<ol>
  <li>Environment prerequisites: <a href ="https://nodejs.org/">Node.js</a> runtime environment, version 14 or greater.</li>
  <li>Clone GitHub Migration Analyzer.</li>
</ol>
	
<pre> 
<code>
git clone https://github.com/github/gh-migration-analyzer.git
cd(pathname of migration analyzer directory)
</code>
</pre>

<ol start="3">	
<li>Install dependencies</li>
</ol>

```
npm install  
```

<ol start="4">	
<li>Generate GitHub PAT with org:read scope and store in Environment variable.</li>
</ol>

```
export GH_PAT=<Replace with PAT from the source org>  
```

<ol start="5">
<li>Execute the analyzer</li>
 </ol>
 
<pre>
<code>
node src/index.js GH-org -o (ORGName) -s (GHES GraphQL Endpoint) 
(ORGName) : Source org name. for e.g. com-theproduct
(GHES GraphQL Endpoint) : The GraphQL endpoint. for e.g.: https://github.tri-ad.tech/api/graphql or https://github.com-"The Product".com/api/graphql 
</code>
</pre>

<ol start="6">
<li>Result files in CSV format will be located in ${project-root}${org-name}-metrics example: org-metrics.csv repo-metrics.csv.</li>
<li>Review the results to create a repository migration list.</li>

</ol>
</details>	

## Migration
  
This is the recommended method. It can be done by repository owners and by organization owners.

### Requirements

<ul>
	<li>GHE: Org owner role, PAT (Personal Access Token)</li>
	<li>EMU: Migrator role, PAT (Personal Access Token)</li>
	<li><b>For GitHub version < 3.8.0 ONLY</b> - AWS S3 bucket  (as a temporary storage). To find out what version your GitHub Server is running, scroll down to the bottom of the page on your GitHub homepage. The version should be mentioned in the footer.</li>
</ul>

<h3>Migration Steps</h3>
</br>

<p id="1"> </p>
<details>
	<summary>Step 0 - Intermediate Storage <i>(Only for GitHub version < 3.8.0)</i></summary>
	
The migration process creates an archive of the repository and stores it in a temporary location that must be reachable over the internet. GitHub recommends using AWS S3. Review the documentation here to set up this intermediate storage. Skip this step if your GitHub version is >= 3.8.0.

We recommend using S3 in your existing AWS account(s). If you do not have your own AWS account please <a href ="mailto: pillai.ajay.m@gmail.com">contact us</a> to set-up one.

<b>Steps for S3 to comply with "The Company" AWS SSO</b> (i.e. without using IAM users):
<ul>
  <li>Create a temporary S3 bucket in your AWS account with a distinguishable name (it needs to be cleaned up later):
<ul>
   <li>(ex) gh-gei-<your account ID> </li>
</ul>
   <li>Create a custom role in the account with AmazonS3FullAccess. Memo the role's ARN for the next step ( should look like: arn:aws:iam::NNNNN:role/XXXXX )</li>
   <li>Ensure assume-role is working and returns temporary credentials. These credentials will be used in Step 3:</li>
	
<pre>
<code>
aws sso login --profile=<your profile>
aws sts assume-role --profile=<your profile> \
  --role-arn <your ARN> \
  --role-session-name gh-gei
{
    "Credentials": {
        "AccessKeyId": "...",
        "SecretAccessKey": "...",
        "SessionToken": "...",
    },
    ...
}
</code>
</pre>
	
<b>Note</b>: The assume-role session's lifetime is 1 hour (3600 seconds) by default. For larger repo migrations, you may want to extend it with --duration-seconds.
If you AWS setup doesn’t have SSO, then you can skip the above steps and use the IAM users

</details>

<details>
	<summary>Step 1 - Install the necessary tools</summary>

Skip to Step 2 if the tools are installed.

The repo migration process requires the <a href="https://github.com/cli/cli#installation">GitHub CLI</a> and the <a href="https://github.com/github/gh-gei#using-the-gei-cli">gh-gei extension</a> installed locally. Please follow the links for install instructions.

</details>


<details>
	<summary>Step 2 - Test access to the destination organization</summary>

Skip to Step 3 if you’re already logged in and have tested that you have permissions to access the destination organization.

Login to GitHub.com (EMU)
	
```
gh auth login
```

Alternatively, you can set EMU's PAT in GH_TOKEN:
	
```
export GH_TOKEN=<Replace me with PAT from the target GitHub EMU org>
```

Confirm github.com access
	
```
gh repo list <destination organization>
```

If you are able to see a non-error response after executing the command, you have successfully tested your access.
</details>

<details>
	<summary>Step 3 - Configure environment variables</summary>

Since EMU does not support Org owner privileges, please <a href ="mailto: pillai.ajay.m@gmail.com">contact us</a> to request for a <a href="https://docs.github.com/en/migrations/using-github-enterprise-importer/preparing-to-migrate-with-github-enterprise-importer/granting-the-migrator-role-for-github-enterprise-importer#about-the-migrator-role">migrator role for GitHub Enterprise Importer</a>.

	For Linux/Mac users:
	
```
export GH_SOURCE_PAT=<Replace with PAT from the source org>
export GH_PAT=<Replace me with PAT from the target GitHub EMU org>
```

For PowerShell users:
	
```
$env:GH_SOURCE_PAT="<Replace with PAT from the source org>"
$env:GH_PAT="<Replace me with PAT from the target GitHub EMU org>"
```

<b>(Only for GitHub version < 3.8.0)</b> Set the temporary S3 bucket name (created in Step 0):
	
```
export AWS_BUCKET_NAME="<name of the S3 bucket>"
```
<b>(Only for GitHub version < 3.8.0)</b> Set credentials (retrieved in Step 0):
	
```
export AWS_ACCESS_KEY=<AccessKeyId in Step 0>
export AWS_SECRET_KEY=<SecretAccessKey in Step 0>
export AWS_SESSION_TOKEN=<SessionToken in Step 0>
```

</details>

<details>
	<summary>Step 4 - Migrate</summary>

To migrate a single repository, run the following command.

<b>For GitHub version < 3.8.0</b>
```
gh gei migrate-repo \
--github-source-org SOURCE \
--source-repo CURRENT-NAME \
--github-target-org DESTINATION \
--target-repo NEW-NAME \
--ghes-api-url GHES_API_URL \
--aws-bucket-name "${AWS_BUCKET_NAME}" \
--aws-access-key "${AWS_ACCESS_KEY}" \
--aws-secret-key "${AWS_SECRET_KEY}" \
--aws-session-token "${AWS_SESSION_TOKEN}" \
--aws-region ap-northeast-1
```
<b>For GitHub version >= 3.8.0</b>
```
gh gei migrate-repo \
--github-source-org SOURCE \
--source-repo CURRENT-NAME \
--github-target-org DESTINATION \
--target-repo NEW-NAME \
--ghes-api-url GHES_API_URL \
```
<b>Note</b>: "The Product" URL: “https://github.com-"The Product".com/api/v3”

To migrate multiple repositories, use the GitHub CLI to generate a migration script. The resulting script will contain a list of migration commands written in Powershell, one per repository. If you’re performing the migration on a Mac/Linux, you would need to install Powershell for those platforms. Instructions to install Powershell on a <a href="https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-macos?view=powershell-7.3">mac</a> and <a href="https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-linux?view=powershell-7.3">Linux</a>.

<b>For GitHub version < 3.8.0</b>	
```
gh gei generate-script \
--github-source-org SOURCE  \
--github-target-org DESTINATION \
--output FILENAME \
--ghes-api-url GHES_API_URL
--aws-bucket-name AWS_BUCKET_NAME
```
	
<b>For GitHub version >= 3.8.0</b>	
```
gh gei generate-script \
--github-source-org SOURCE  \
--github-target-org DESTINATION \
--output FILENAME \
--ghes-api-url GHES_API_URL
```
	
</details>

<details>
	<summary>Step 5 - Follow-up tasks</summary>

At this point while the actual repository and its contents may be migrated to the new location, there are some additional tasks that need to be followed to ensure that the repository is ready for use.

<ul>
  <li>Review the migration log for each migrated repository. </li>

For more information, see <a href="https://docs.github.com/en/enterprise-server@3.7/migrations/using-github-enterprise-importer/completing-your-migration-with-github-enterprise-importer/accessing-your-migration-logs-for-github-enterprise-importer">Accessing your migration logs</a>. If the logs contain warnings or errors, refer to the <a href="https://docs.github.com/en/enterprise-server@3.7/migrations/using-github-enterprise-importer/completing-your-migration-with-github-enterprise-importer/accessing-your-migration-logs-for-github-enterprise-importer">troubleshooting guide</a>.
</br>
	
<b>Note</b>: All repositories are migrated as private. If you were using a shared PAT and cannot see the migrated repository in the target org, please <a href ="mailto: pillai.ajay.m@gmail.com">contact us</a> for access.

 <li>Review items that are not migrated due to limitations of the importer program.</li>

 <ul>
   <li><b>Team access</b></li>

When performing a repo-by-repo migration, as outlined in this document, teams and team access are not transferred over. Instead, this step will need to be manually performed by the Operations team. This is a temporary solution until "The Product" integration is implemented, at which point teams will be able to manage their own access permissions without manual intervention.

For the time being, please <a href ="mailto: pillai.ajay.m@gmail.com">contact us</a> to perform this step.

   <li><b>Secrets</b></li>

Secrets are encrypted environment variables that are encrypted and made available for use in GitHub Actions workflows. They are not migrated by the importer tool. To avoid breaking workflows, they would need to be manually copied over to the destination repository. Review any organization, repository or repository environment secrets that may exist in the source repository.

Workflow secrets are encrypted and GitHub currently provides no way (programmatic or otherwise) to retrieve the unencrypted value. Therefore, in order to manually recreate the secrets in the destination repository, you would need the unencrypted secret stored somewhere else like 1Password. If you don’t have access to the unencrypted secret, consider recreating the token with the same scopes/permissions from the source that it belongs to, store a copy in an out-of-band secret management system like 1Password so that it can be used in an emergency and then add it to the destination repository using steps outlined in <a href="https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository">Creating encrypted secrets for a repository</a>. Alternatively, <a href="https://cli.github.com/manual/gh_secret">GitHub CLI</a> can also be used to re-add secrets.

   <li><b>Webhooks</b></li>
	 
All active webhooks are migrated to the destination repository but disabled by default. Please refer to steps for <a href="https://docs.github.com/en/migrations/using-github-enterprise-importer/understanding-github-enterprise-importer/migrating-between-github-products-with-github-enterprise-importer#enabling-webhooks">Enabling webhooks</a>.

   <li><b>Workflows</b></li>
	 
Workflows are migrated to the destination repository but disabled by default to avoid accidental triggering. Workflow history is not migrated due to the limitations of the importer program. To use self-hosted runners to execute the workflows, refer to the guide on <a href="https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners">Adding self-hosted runners</a> and <a href="https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/using-self-hosted-runners-in-a-workflow">Using self-hosted runners</a> in a workflow. EnTec runners offering are available <a href="https://portal.com-"The Product".com/docs/default/component/"The Product"-welcome-guides/ja/features-and-tools/planned-development-tools/github-emu-actions/github-emu-runners/">here</a>.

   <li><b>Mannequins</b></li>
	 
After you run a migration with GitHub Enterprise Importer, all user activity in the migrated repository (except Git commits) is attributed to placeholder identities called mannequins. You can reattribute the history for each mannequin to an organization member by sending an attribution invitation with the GitHub CLI or in your browser. If you use the GitHub CLI, you can reclaim mannequins in bulk. For more information, refer to the guide on <a href="https://docs.github.com/en/enterprise-server@3.7/migrations/using-github-enterprise-importer/completing-your-migration-with-github-enterprise-importer/reclaiming-mannequins-for-github-enterprise-importer#reclaiming-mannequins-with-the-github-cli-recommended">Reclaiming mannequins</a>.

<b>NOTE</b> - please ask you organization administrators to perform this task

   <li><b>Branch Protection Rules</b></li>
	 
Any rules that apply to <b>teams</b> are not migrated. This is because teams and team access is not migrated, as mentioned in bullet 1 above. However, branch protections themselves do get migrated. Please refer to the guide on manually <a href="https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule">Managing a branch protection rule</a> in the destination repository to add any teams as required that may have been recreated.

   <li><b>GitHub Large File Storage (LFS)</b></li>
	 
GitHub Enterprise Importer does not migrate Git LFS objects. If the source repository uses Git LFS, you can manually push Git LFS objects to the migrated repository locally. At a high level, these are the steps that you would need to follow:
	<ul>
	<li>Clone the source repo to local</li>
	<li>Add the URL of the migrated repo to your local, LFS-enabled repo using the following:</li>
		
<pre>
<code>
git remote add github git@github.com:OWNER/REPO_NAME.git
</code>
</pre>

<li>Fetch all LFS objects locally</li>

```		 
git lfs fetch origin -all
```

<li>Push LFS objects to destination repository</li>

```	
git lfs push github -all
```
		 

See more information on <a href="https://docs.github.com/en/repositories/creating-and-managing-repositories/duplicating-a-repository#mirroring-a-repository-that-contains-git-large-file-storage-objects">Mirroring a repository that contains Git LFS objects</a>.
</ul>
	 		 
<li><b>GitHub Pages</b></li>

While the Importer <a href="https://docs.github.com/en/enterprise-server@3.7/migrations/using-github-enterprise-importer/understanding-github-enterprise-importer/migration-support-for-github-enterprise-importer#github-enterprise-server-migration-support">supports Pages settings</a>, our EMU policy may block them from importing. Check the status in a migration log which is attached as an Issue in the migrated repo (example below) and, if blocked, you'll need to re-enable the settings manually.

```
[2023-06-22T06:55:27Z] INFO -- Import started
[2023-06-22T06:55:28Z] WARN -- Page visibility is set to public but the organization's settings do not allow public pages.
[2023-06-22T06:55:41Z] INFO -- Import complete
```

<li><b>GitHub Advanced Security</b></li>

Refer to GituHb advanced security section below.

<li><b>GitHub Actions Workflow</b></li>

Update the references of synced-actions to OSS and marketplace location

```
uses: synced-actions/aws-actions-configure-aws-credentials@v1
#to the following
uses: aws-actions/configure-aws-credentials@v1
```

<li><b>GitHub Applications</b></li>
<ul>

   <li>Custom GitHub applications developed by the development teams needs to be re-configured on the EMU. Network connectivity should be revalidated between the custom GitHub application and GitHub.com</li>
   <li>First or Third party GitHub applications needs to be re-configured again after the migration on the EMU.</li>
	 </ul>
	</ul>
</details>
	
<details>
	<summary>Step 6 - Clean up</summary>
	
<ol>
   <li>Archive the repository in GitHub Enterprise and update the README to point to the new repository location. Please refer to <a href="https://docs.github.com/en/enterprise-server@3.7/repositories/archiving-a-github-repository/archiving-repositories">Archiving repositories</a> for instruction.</li>
   <li>Update any documentation that may be referencing the repository.</li>
</ol>
	

<b>IMPORTANT - For GitHub version < 3.8.0:</b> The import tool <a href="https://docs.github.com/en/early-access/enterprise-importer/migrating-repositories-with-github-enterprise-importer/migrating-repositories-from-github-enterprise-server-to-github-enterprise-cloud#setting-up-an-aws-s3-storage-bucket"><b>DOES NOT</b></a> clean up the intermediate storage (AWS S3) that is used for migrating repositories. The storage bucket would have to be manually cleaned up once all the organizations and repositories are migrated.

</details>
	

### GitHub Advanced Security
<ul>
	<li>If you enabled GitHub Advanced Security for the destination organization before migrating repositories, the settings for individual features were migrated. If not, you'll need to re-enable individual features after the migration. For more information please refer to the guide on <a href="https://docs.github.com/en/enterprise-cloud@latest/migrations/using-github-enterprise-importer/understanding-github-enterprise-importer/migrating-between-github-products-with-github-enterprise-importer#managing-github-advanced-security">Managing GitHub Advanced Security</a>.</li>
	<li>Code scanning alerts are not migrated by GitHub Enterprise Importer.  However, the alerts are available as SARIF data in the source repository.</li>
	<li>The <b>state</b> and <b>resolution</b> of CodeQL code scanning and secret scanning alerts are not migrated. After the repository is migrated, and re-enable Secrets Scanning and CodeQL Scanning For secret scanning, a scan of the entire repository will be performed. After the scan is complete, all alerts will be populated, but without remediation states. For migrating state and resolution of secret scanning alerts you can use --migrate-secret-alerts .</li>

```
gh gei migrate-secret-alerts --source-org <source_org> --source-repo <source_repo> --target-org <target_org> --target-repo <targe_repo>
```

<li>For CodeQL Code Scanning alerts, they will populate differently from the original alerts in the source repository.</li>
	<ul>
		<li>Alerts will only include the detection and the latest state of the alert, not the entire timeline from the source repository.</li>
		<li>Alerts will only be identified as open or fixed. Other remediation states, such as dismissed and reopened, will be lost.</li>
		<li>The dates for all events on the alert will be the date of the API call, not the dates when the events originally occurred on the source repository.</li>
		<li>All actors, such as the alert creator, will change to the owner of the personal access token used for the API call.</li>
</ul>
	
<li>For migrating state and resolution of code scanning alerts you can use --migrate-code-scanning-alerts  with gei .</li> 

```		
gh gei migrate-code-scanning-alerts --source-org <source_org> --source-repo <source_repo> --target-org <target_org> --target-repo <targe_repo>
```

<li>When Dependabot alerts and the dependency graph are enabled, Dependabot alerts will be rebuilt from the current state of the default branch. Remediation states of these alerts are not migrated, and any previous alerts are also not migrated. Currently gei  doesn't support migration of dependabot alerts.</li>
<li>If you have custom triggers for CodeQL Code Scanning as opposed to default  CodeQL configuration, review and validate those custom triggers.</li>

<b>Note</b>: GitHub Advanced Security license is per committer and its license is bound to a particular committer for 90 days, if there are no further commits from that particular committer then license is moved back to the pool.
	
<p id="2"> </p>

</br>
<h3>EnTec Runner connectivity matrix</h3>

</br>
EMU, "The Product" GHE runners have different connectivity to on-prem servers.
</br>
<b>Kubernetes runners</b>
</br>
<table>
	<tr>
		<th></th>
		<th>EMU</th>
		<th>"The Product" GHE</th>
		<th>Notes</th>
		</tr>
	<tr>
		<th>Label</th>
		<td>sg-kubernetes</td>
		<td>default-k8s-runner-linux-x64</td>
		<td></td>
	</tr>
	<tr>
		<th>SG services</th>
		<td>〇</td>
		<td>〇</td>
		<td>Tested with Jira, GitHub, Artifactory-ha (*.com-"The Product".com)</td>
	</tr>
	<tr>
		<th>SS services</th>
		<td>X</td>
		<td>X</td>
		<td>Tested with Jira, GitHub, Artifactory-ha (*.tri-ad.tech)</td>
	</tr>
	<tr>
		<th>On-prem: QNX</th>
		<td>X</td>
		<td>X</td>
		<td>Reachable at qnx.tri-ad.tech:27000</td>
	</tr>
	<tr>
		<th>On-prem: zabbix01</th>
		<td>X</td>
		<td>X</td>
		<td>Reachable at URL: https://zabbix01.ts.tri-ad.global</td>
	</tr>
	</table>
</br>
<b>Connected runners</b>
</br>
<table>
	<tr>
		<th>wp-connected</th>
		<th>connected-runner</th>
		<th>Notes</th>
	</tr>
	<tr>
		<th>SG services</th>
		<td>〇</td>
		<td>〇</td>
		<td></td>
	</tr>
	<tr>
		<th>SS services</th>
		<td>〇</td>
		<td>〇</td>
		<td></td>
	</tr>
	<tr>
		<th>On-prem: QNX</th>
		<td>△</td>
		<td>△</td>
		<td>Reachable at IP: 172.24.4.67:27000</td>
	</tr>
	<tr>
		<th>On-prem: zabbix01</th>
		<td>△</td>
		<td>△</td>
		<td>Reachable at IP: 172.16.200.24:https</td>
	</tr>
	</table>
</br>
<b>Legend</b>:
<p>〇: DNS resolves hostname / reachable by hostname</p>
<p>△: DNS cannot resolve hostname / reachable by IP</p>
<p>X: DNS cannot resolve hostname / unreachable by IP</p>

<b>Note</b>: X-large, 3X-large, 4x-large runner, 3X-connected runners are coming soon.
</br>

<h3>Known Limitations</h3>

Refer to the <u>Known Limitations</u> for more details on the limitations with EMU.
</br>
</br>
