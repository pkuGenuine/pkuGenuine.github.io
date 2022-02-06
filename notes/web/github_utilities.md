# Github Utitlities

## Github Pages

## Github Actions

GitHub Actions is a continuous integration and continuous delivery (CI/CD) platform that allows you to automate your build, test, and deployment pipeline. You can create workflows that build and test every pull request to your repository, or deploy merged pull requests to production.

### The Components of Github Actions
You can configure a GitHub Actions workflow to be triggered when an event occurs in your repository, such as a pull request being opened or an issue being created. Your workflow contains one or more jobs which can run in sequential order or in parallel. Each job will run inside its own virtual machine runner, or inside a container, and has one or more steps that either run a script that you define or run an action, which is a reusable extension that can simplify your workflow.

> Summary
> 
> All components: workflow, event, job, step, action, runner.

![avatar](https://docs.github.com/assets/cb-25628/images/help/images/overview-actions-simple.png)

1. A workflow is a configurable automated process that will run one or more jobs. Workflows are defined by a YAML file checked in to your repository and will run when triggered by an event.
2. An event is a specific activity in a repository that triggers a workflow run. For example, activity can originate from GitHub when someone creates a pull request.
3. A job is a set of steps in a workflow that execute on the same runner. 
4. Each step is either a shell script that will be executed, or an action that will be run. Steps are executed in order and are dependent on each other. 
5. An action is a custom application for the GitHub Actions platform that performs a complex but frequently repeated task.
6. A runner is a server that runs your workflows when they're triggered. Each runner can run a single job at a time.

### Self-hosted Runner



## Github Apps

GitHub Apps also provides ways to build automation and workflow tools. Unlike Github actions, which are designed to be ephemeral and usually executed on Github's virtual machines, GitHub Apps are applications that need to be hosted somewhere and works great when persistent data is needed.

GitHub Apps can be installed directly on organizations and user accounts, and granted access to specific repositories. They come with built-in webhooks and narrow, specific [permissions](). When you set up your GitHub App, you can select the repositories you want it to access. For example, you can set up an app called MyGitHub that writes issues in the octocat repository and only the octocat repository. To install a GitHub App, you must be an organization owner or have admin permissions in a repository.

> Aside: Webhook & Github API
> 
> Webhook is also called reverse API. You subscribe some event, for example, PR on github, and your server will be accessed by with specific API. 
> 
> In your app's handler, you may want to access Github's API to manipulate your repo. Github provides almost all kinds of API to interact with. Search codes, emit issues, and even delete repos.
> There are two stable versions of the GitHub API: the [REST API](https://docs.github.com/en/rest) and the GraphQL API. 

To create an app, visit this [setting](https://github.com/settings/apps) page.