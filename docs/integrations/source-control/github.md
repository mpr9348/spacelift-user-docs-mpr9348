---
description: >-
  Describes how to setup the GitHub VCS integration on Spacelift, as well as the
  features supported by the integration.
---

# GitHub

One of the things we're most proud of at Spacelift is the deep integration with everyone's favorite version control system - GitHub.

You can set up multiple Space-level and one default GitHub integration per account.

!!! warning
    There is a built-in integration set up automatically for users who select GitHub as their logic source. If you wish to utilize multiple GitHub accounts or organizations or connect Spacelift to your GitHub Enterprise instance, you will need to set up a custom GitHub integration via a [GitHub App](#setting-up-the-custom-application).

## Setting up the integration

{% if is_saas() %}

### Installing the Marketplace application (Preferred)

The easiest way to connect your GitHub account (personal or organization) is to install the [Spacelift application](https://github.com/marketplace/spacelift-io){: rel="nofollow"} from the GitHub Marketplace.

![](<../../assets/screenshots/CleanShot 2022-09-15 at 17.27.12.png>)

At the bottom of the page, select the GitHub account where you want to install the Spacelift application and then click on the _Install it for free_ button.

![](<../../assets/screenshots/CleanShot 2022-09-15 at 17.44.49.png>)

On the next page click on the _Complete order and begin installation_ button.

![](<../../assets/screenshots/CleanShot 2022-09-15 at 17.46.56.png>)

On the last screen, you will be able to select which repositories should be available to Spacelift, and review the required permissions. When ready, click on the _Install_ button, and voilà!

![](<../../assets/screenshots/CleanShot 2022-09-15 at 17.50.03.png>)

{% endif %}

### Setting up the custom application

In some cases, using the Spacelift application from the Marketplace is not an option, or you might already have it installed and want to link another GitHub account with your Spacelift account. For these advanced uses cases, you can use the custom Spacelift application.

#### Creating the custom application

In order to do so, navigate to the **Source code** page, click on the **Set up integration** button and choose GitHub.
You will be presented with two options:

![](<../../assets/screenshots/CleanShot 2022-09-16 at 09.38.30.png>)

=== "Wizard"

    The easiest and recommended way to install the custom Spacelift application in your GitHub account is by using the wizard.

    You will be asked a few questions, then you will be redirected to GitHub to create the application. Once it's done, you'll be redirected back to Spacelift to finish the integration.

    <p align="center">
        <img src="../../assets/screenshots/GitHub_wizard_final_step.png"/>
    </p>

    - **Integration name** - the friendly name of the integration. The name cannot be changed after the integration is created. That is because the Spacelift webhook endpoint are generated based on the integration name.
    - **Integration type** - either default or [Space](../../concepts/spaces/README.md)-specific. The default integration is available to **all** stacks and modules. There can be only one default integration per VCS provider. Space-level integrations however are only available to those stacks and modules that are in the same Space as the integration (or [inherit](../../concepts/spaces/access-control.md#inheritance) permissions through a parent Space). For example if your integration is in `ParentSpace` and your stack is in `ChildSpace` with inheritance enabled, you'll be able to attach the integration to that stack. Refer to the [Spaces documentation](../../concepts/spaces/access-control.md) to learn more about Space access controls and inheritance.
    - **Labels** - a set of labels to help you organize integrations.
    - **Description** - a markdown-formatted free-form text field that can be used to describe the integration.

=== "Manual setup"

    You can create the custom Spacelift application for GitHub manually.

    !!! warning
        This should be used as a last resort when the other methods can not be used as it is more tedious and error-prone.

    After selecting the option to enter your details manually, you should see the following form:

    <p align="center"><img src="../../assets/screenshots/CleanShot 2022-09-16 at 10.14.05.png" width="450"></p>

    Let's choose a friendly name (it can contain spaces) for your integration and choose the integration type.

    The integration type could be either **default** or **space-specific**. The default integration is available to **all** stacks and modules. There can be only one default integration per VCS provider. Space-level integrations however are only available to those stacks and modules that are in the same Space as the integration (or [inherit](../../concepts/spaces/access-control.md#inheritance) permissions through a parent Space). For example if your integration is in `ParentSpace` and your stack is in `ChildSpace` with inheritance enabled, you'll be able to attach the integration to that stack. Refer to the [Spaces documentation](../../concepts/spaces/access-control.md) to learn more about Space access controls and inheritance.

    Once the integration name and the type are chosen, a **webhook endpoint** and a **webhook secret** will be generated in the middle of the form. These two are enough to create the GitHub App, so let's do that now.

    Open GitHub, navigate to the _GitHub Apps_ page in the _Developer Settings_ for your account/organization, and click on _New GitHub App._

    You can either create the App in an individual user account or within an organization account:

    ![](<../../assets/screenshots/image (52).png>)

    Give your app a name and homepage URL (these are only used for informational purposes within GitHub):

    ![](<../../assets/screenshots/image (53).png>)

    Enter your Webhook URL and secret:

    ![](<../../assets/screenshots/image (54).png>)

    **Set the following Repository permissions:**

    | Permission      | Access       |
    | --------------- | ------------ |
    | Checks          | Read & write |
    | Commit statuses | Read & write |
    | Contents        | Read-only    |
    | Deployments     | Read & write |
    | Metadata        | Read-only    |
    | Pull requests   | Read & write |
    | Webhooks        | Read & write |

    **Set the following Organization permissions:**

    | Permission | Access    |
    | ---------- | --------- |
    | Members    | Read-only |

    **Subscribe to the following events:**

    - Organization
    - Pull request
    - Pull request review
    - Push
    - Repository

    Finally, choose whether you want to allow the App to be installed on any account or only on the account it is being created in and click on _Create GitHub App:_

    ![](<../../assets/screenshots/image (55).png>)

    Once your App has been created, make a note of the _App ID_ in the _About_ section:

    ![](<../../assets/screenshots/image (56).png>)

    Now scroll down to the _Private keys_ section of the page and click on _Generate a private key:_

    ![](<../../assets/screenshots/image (57).png>)

    This will download a file onto your machine containing the private key for your GitHub app. The file will be named `<app-name>.<date>.private-key.pem`, for example `spacelift.2021-05-11.private-key.pem`.

    Now that your GitHub App has been created, go back to the integration configuration screen in Spacelift, and enter your _API host URL_ (the URL to your GitHub server), _User facing host URL_, the _App ID_, and paste the contents of your private key file into the Private key box:

    !!! info
        If you are using `github.com` set your API host URL as: [https://api.github.com](https://api.github.com){: rel="nofollow"}.

    !!! info
        User facing host URL is the URL that will be displayed in the Spacelift UI. Typically, this is the same as the API host URL unless you are using [VCS Agents](../../concepts/vcs-agent-pools.md): in that case, the API host URL will look like `private://vcs-agent-pool-name`, but the User facing host URL can look more friendly (for example `https://vcs-agent-pool.mycompany.com/`) since it isn't actually being used by Spacelift.

    <p align="center"><img src="../../assets/screenshots/Screen Shot 2022-04-20 at 4.30.53 PM.png" width="450"></p>

    Click on the **Set up** button to save your integration settings.

Congratulations! You are almost done! 🎉

The last step is to install the application you just created so that Spacelift can interact with GitHub. This is what the next section is about.

#### Installing the custom application

Now that you've created a GitHub App and configured it in Spacelift, the last step is to install your App in one or more accounts or organizations you have access to. You can either use the shortcut link on the Spacelift UI, or manually navigating to it in GitHub.

### Via Spacelift UI

On the Source code page:

<p align="center">
  <img src="../../assets/screenshots/github_install_app.png" width="400"/>
</p>

#### Via GitHub UI

Find your App in the GitHub Apps page in your account settings, and click on the _Edit_ button next to it:

![](<../../assets/screenshots/image (58).png>)

Go to the _Install App_ section, and click on the _Install_ button next to the account your want Spacelift to access:

![](<../../assets/screenshots/image (59).png>)

Choose whether you want to allow Spacelift access to all the repositories in the account, or only certain ones:

![](<../../assets/screenshots/image (60).png>)

Congrats, you've just linked your GitHub account to Spacelift!

## Using GitHub with stacks and modules

If your Spacelift account is integrated with GitHub, the stack or module creation and editing forms will show a dropdown from which you can choose the VCS integration to use.

<p align="center">
  <img src="../../assets/screenshots/Screen Shot 2022-04-21 at 12.43.09 PM.png"/>
</p>

The rest of the process is exactly the same as with [creating a GitHub-backed stack](../../concepts/stack/creating-a-stack.md#integrate-vcs) or module, so we won't go into further details.

## Access controls

### Space-level integrations

{% if is_saas() %}
!!! hint
    This feature is only available to Enterprise plan. Please check out our [pricing page](https://spacelift.io/pricing){: rel="nofollow"} for more information.
{% endif %}

If you're using Space-level integrations, you can use the [Spaces](../../concepts/spaces/README.md) to control the access of your GitHub integrations. For example, if you have a Space called `Rideshare`, you can create a GitHub integration in that Space, and that can only be attached to those stacks and modules that are in the same Space (or [inherit](../../concepts/spaces/access-control.md#inheritance) permissions through a parent Space).

#### Integration details visibility

Only Space **admins** will be able to see the webhook URLs and secrets of Space-level integrations. Space **readers** will only be able to see the name, description, and labels of the integration.
The details of default integrations are only visible to **root** Space admins.

### Legacy method

You can reuse GitHub's native teams as well. If you're using GitHub as your identity provider (which is the default), upon login, Spacelift uses GitHub API to determine organization membership level and team membership within an organization and persists it in the session token which is valid for one hour. Based on that you can set up [login policies](../../concepts/policy/login-policy.md) to determine who can log in to your Spacelift account, and [stack access policies](../../concepts/policy/stack-access-policy.md) that can grant an appropriate level of access to individual [Stacks](../../concepts/stack/README.md).

!!! info
    The list of teams is empty for individual/private GitHub accounts.

## Notifications

### Commit status notifications

Commit status notifications are triggered for [_proposed_ runs](../../concepts/run/proposed.md) to provide feedback on the proposed changes to your stack - running a preview command (eg. `terraform plan` for Terraform) with the source code of a short-lived feature branch with the state and config of the stack that's pointing to another, long-lived branch. Here's what such a notification looks like:

...when the run is in progress ([initializing](../../concepts/run/README.md#initializing)):

![](../../assets/screenshots/Test_a_change_by_marcinwyszynski_·_Pull_Request__6_·_spacelift-io_marcinw-end-to-end.png)

...when it succeeds _without changes_:

![](<../../assets/screenshots/Test_a_change_by_marcinwyszynski_·_Pull_Request__6_·_spacelift-io_marcinw-end-to-end (2).png>)

...when it succeeds _with changes_:

![](<../../assets/screenshots/Test_a_change_by_marcinwyszynski_·_Pull_Request__6_·_spacelift-io_marcinw-end-to-end (1).png>)

...and when it fails:

![](<../../assets/screenshots/Test_a_change_by_marcinwyszynski_·_Pull_Request__6_·_spacelift-io_marcinw-end-to-end (3).png>)

In each case, clicking on the _Details_ link will take you to the GitHub check view showing more details about the run:

![](<../../assets/screenshots/Add_Azure_integration_variables_by_adamconnelly_·_Pull_Request__561_·_spacelift-io_infra (1).png>)

The Check view provides high-level information about the changes introduced by the push, including the list of changing resources, including cost data if [Infracost](../../vendors/terraform/infracost.md) is set up.

From this view you can also perform two types of Spacelift actions:

- **Preview** - execute a [proposed run](../../concepts/run/proposed.md) against the tested commit;
- **Deploy** - execute a tracked run against the tested commit;

#### PR (Pre-merge) Deployments

The _Deploy_ functionality has been introduced in response to customers used to the Atlantis approach, where the deployment happens from within a Pull Request itself rather than on merge, which we see as the default and most typical workflow.

If you want to prevent users from deploying directly from GitHub, you can add a simple [plan policy](../../concepts/policy/terraform-plan-policy.md) to that effect, based on the fact that the run trigger always indicates GitHub as the source (the exact format is `github/$username`).

```opa
package spacelift

deny["Do not deploy from GitHub"] {
  input.spacelift.run.type == "TRACKED"
  startswith(input.spacelift.run.triggered_by, "github/")
}
```

The effect is as follows:

![](../../assets/screenshots/Update_README_md_·_Private_worker_pool.png)

#### Using Spacelift checks to protect branches

You can use commit statuses to protect your branches tracked by Spacelift stacks by ensuring that _proposed_ runs succeed before merging their Pull Requests:

![](<../../assets/screenshots/New_branch_protection_rule (1).png>)

This is an important part of our proposed workflow - please refer to [this section](github.md#proposed-workflow) for more details.

##### Aggregated checks

{% if is_saas() %}
!!! info
    This feature is only available to Business plan and above. Please check out our [pricing page](https://spacelift.io/pricing){: rel="nofollow"} for more information.
{% endif %}

If you have multiple stacks tracking the same repository, you can enable the _Aggregate VCS checks_ feature in the integration's settings.
This will group all the checks from the same commit into a predefined set of checks, making it easier to see the overall status of the commit.

![](<../../assets/screenshots/aggregated-checks-github-settings.png>)

When the aggregated option is enabled, Spacelift will post the following checks:

- **spacelift/tracked** - groups all checks from tracked runs
- **spacelift/proposed** - groups all checks from proposed runs
- **spacelift/modules** - groups all checks from module runs

Here's how the summary looks like:

![](<../../assets/screenshots/aggregated-checks-github-summary.png>)

In each case, clicking on the _Details_ link will take you to the GitHub check view showing more details about stacks or modules included in the aggregated check:

![](<../../assets/screenshots/aggregated-checks-github-details.png>)

### Deployment status notifications

[Deployments](https://developer.github.com/v3/guides/delivering-deployments/){: rel="nofollow"} and their associated statuses are created by tracked runs to indicate that changes are being made to the Terraform state. A GitHub deployment is created and marked as _Pending_ when the [planning](../../concepts/run/proposed.md#planning) phase detects changes and a [tracked run](../../concepts/run/tracked.md) either transitions to [Unconfirmed](../../concepts/run/tracked.md#unconfirmed) state or automatically starts [applying](../../concepts/run/tracked.md#applying) the diff:

![](<../../assets/screenshots/Deployments_·_spacelift-io_marcinw-end-to-end (1).png>)

If the user does not like the proposed changes during the manual review and [discards](../../concepts/run/tracked.md#discarded) the [tracked run](../../concepts/run/tracked.md), its associated GitHub deployment is immediately marked as a _Failure_. Same happens when the user [confirms](../../concepts/run/tracked.md#confirmed) the [tracked run](../../concepts/run/tracked.md) but the [Applying](../../concepts/run/tracked.md#applying) phase fails:

![](<../../assets/screenshots/Deployments_·_spacelift-io_marcinw-end-to-end (2).png>)

If the [Applying](../../concepts/run/tracked.md#applying) phase succeeds (fingers crossed!), the deployment is marked as _Active_:

![](../../assets/screenshots/Deployments_·_spacelift-io_marcinw-end-to-end.png)

The whole deployment history broken down by stack can be accessed from your repo's _Environments_ section - a previously obscure feature that's recently getting more and more love from GitHub:

![](../../assets/screenshots/spacelift-io_infra__Infrastructure_definitions_for_Spacelift.png)

That's what it looks like for our test repo, with just a singe stack pointing at it:

![](<../../assets/screenshots/Deployments_·_spacelift-io_marcinw-end-to-end (4).png>)

GitHub deployment environment names are derived from their respective stack names. This can be customized by setting the `ghenv:` label on the stack. For example, if you have a stack named `Production` and you want to name the deployment environment `I love bacon`, you can set the `ghenv:I love bacon` label on the stack. You can also disable the creation of a GitHub deployments by setting the `ghenv:-` label on the stack.

!!! info
    The _Deployed_ links lead to their corresponding Spacelift [tracked runs](../../concepts/run/tracked.md).

## Pull Requests

In order to help you keep track of all the pending changes to your infrastructure, Spacelift also has a PRs tab that lists all the active Pull Request against your tracked branch. Each of the entries shows the current status of the change as determined by Spacelift, and a link to the most recent Run responsible for determining that status:

![](../../assets/screenshots/Pull_Requests_·_Spacelift_development.png)

Note that this view is read-only - you can't change a Pull Request through here, but clicking on the name will take you to GitHub where you can make changes.

Once a Pull Request is closed, whether with or merging or without merging, it disappears from this list.

## Proposed workflow

In this section, we'd like to propose a workflow that has worked for us and many other DevOps professionals working with infrastructure-as-code. Its simplest version is based on a single stack tracking a long-lived branch like _main_, and short-lived feature branches temporarily captured in Pull Requests. A more sophisticated version can involve multiple stacks and a process like [GitFlow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow){: rel="nofollow"}.

!!! tip
    These are mere suggestions and Spacelift will fit pretty much any Git workflow, but feel free to experiment and find what works best for you.

### Single stack version

Let's say you have a single stack called _Infra_. Let's have it track the default `master` branch in the repository called... `infra`. Let's say you want to introduce some changes - define an Amazon S3 bucket, for example. What we suggest is opening a short-lived feature branch, making your change there, and opening a Pull Request from that branch to `master`.

At this point, a proposed run is triggered by the push notification, and the result of running `terraform plan` with the new code but existing state and config is reported to the Pull Request. First, we should ensure that the Pull Request does not get merged to master without a successful run, so we'd protect the branch by **requiring a successful status check** from your stack.

Second, we can decide whether we just need a tick from Spacelift, or we'd rather **require a manual review**. We generally believe that more eyes is always better, but sometimes that's not practicable. Still, it's possible to protect the tracked branch in a way that requires manual Pull Request approval before merging.

We're almost there, but let's also consider a scenario where our coworkers are also busy modifying the same stack. One way of preventing snafus as a team and get meaningful feedback from Spacelift is to **require that branches are up to date before merging**. If the current feature branch is behind the PR target branch, it needs to be rebased, which triggers a fresh Spacelift run that will ultimately produce the newest and most relevant commit status.

### Multi-stack version

One frequent type of setup involves two similar or even identical environments - for example, _staging_ and _production_. One approach would be to have them in a single repository but in different directories, setting [`project_root`](../../concepts/configuration/runtime-configuration/README.md#project_root-setting) runtime configuration accordingly. This approach means changing the _staging_ directory a lot and using as much or as little duplication as necessary to keep things moving, and a lot of commits will necessarily be no-ops for the _production_ stack. This is a very flexible approach, and we generally like it, but it leaves Git history pretty messy and some people really don't like that.

If you're in that group, you can create two long-lived Git branches, each linked to a different stack - the default `staging` branch linked to the _staging_ stack, and a `production` branch linked to the _production_ stack. Most development thus occurs on the staging branch and once the code is perfected there over a few iterations, a Pull Request can be opened from the `staging` to `production` branch, incorporating all the changes. That's essentially how we've seen most teams implement [GitFlow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow){: rel="nofollow"}. This approach keeps the history of the `production` branch clear and allows plenty of experimentation in the `staging` branch.

With the above GitFlow-like setup, we propose protecting both `staging` and `production` branches in GitHub. To maximize flexibility, `staging` branch may require a green commit status from its associated stack but not necessarily a manual review. In the meantime, `production` branch should probably require both a manual approval and a green commit status from its associated stack.

## Webhook integrations

Below is the list of some of the GitHub webhooks we subscribe to with a brief explanation of what we do with those.

### Push events

Any time we receive a repository code push notification, we match it against Spacelift repositories and - if necessary - [create runs](../../concepts/run/README.md). We'll also _stop proposed runs_ that have been superseded by a newer commit on their branch.

### App installation creation

When the Spacelift GitHub app is installed on an account, we create a corresponding Spacelift account.

### Organization renamed

If a GitHub organization name is changed, we change the name of the corresponding account in Spacelift.

!!! warning
    This is only applicable for accounts that were created using GitHub originally.

### Pull Request events

Whenever a Pull Request is opened or reopened, we generate a record in our database to show it on the Stack's _PRs_ page. When it's closed, we delete that record. When it's synchronized (eg. new push) or renamed, we update the record accordingly. This way, what you see in Spacelift should be consistent with what you see in GitHub.

### Pull Request Review events

Whenever a review is added or dismissed from a Pull Request, we check whether a new run should be triggered based on any push policies attached to your stacks. This allows you to make decisions about whether or not to trigger runs based on the approval status of your Pull Request.

### Repository renamed

If a GitHub repository is renamed, we update its name in all the [stacks](../../concepts/stack/stack-settings.md#repository-and-branch) pointing to it.

## GitHub Action

You can use the [Setup Spacectl](https://github.com/marketplace/actions/setup-spacectl){: rel="nofollow"} GitHub Action to install our [spacectl](https://github.com/spacelift-io/spacectl){: rel="nofollow"} CLI tool to easily interact with Spacelift.

## Git checkout support

By default Spacelift uses the GitHub API to download a tarball containing the source code for your stack or module. We are introducing experimental support for downloading the code using a standard Git checkout. If you would like to enable this for your stacks/modules, there are currently two options available:

1. Add a label called `feature:enable_git_checkout` to each stack or module that you want to use Git checkout on. This allows you to test the new support without switching over all your stacks at once.
2. Contact our support team and ask us to enable the feature for all stacks/modules in your account.

## Unlinking GitHub and Spacelift

=== "Uninstalling the Marketplace application"

    If you wish to uninstall the Spacelift application you installed from the GitHub Marketplace, go to the GitHub account settings and select the *Applications* menu item.

    ![](<../../assets/screenshots/CleanShot 2022-09-15 at 17.54.57.png>)

    Click on the *Configure* button for the *spacelift.io* application.

    ![](<../../assets/screenshots/CleanShot 2022-09-15 at 17.57.41.png>)

    Finally, click on the *Uninstall* button.

    ![](<../../assets/screenshots/CleanShot 2022-09-15 at 17.58.05.png>)

=== "Uninstalling the custom application"

    Go to the _Developer settings_ of the GitHub account, then in the _GitHub Apps_ section click on the _Edit_ button for the Spacelift application.

    ![](<../../assets/screenshots/CleanShot 2022-09-16 at 10.28.11.png>)

    On the page for the Spacelift application, go to the *Advanced* section and click on the _Delete GitHub App_ button. Confirm by typing the name of the application and it is gone.

    ![](<../../assets/screenshots/CleanShot 2022-09-16 at 10.24.08.png>)

    You can now remove the integration via the *Delete* button on the *Source code* page:

    <p align="center"><img src="../../assets/screenshots/Gitlab_delete.png"/></p>

    !!! warning
        Please note that you can delete integrations **while stacks are still using them**. As a consequence, when a stack has a detached integration, it will no longer be able to receive webhooks from Github and you won't be able to trigger runs manually either.
        To fix it, you'll need to open the stack, go to the **Settings** tab and choose a new integration.
