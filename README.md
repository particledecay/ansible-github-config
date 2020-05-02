# ansible-github-config
An Ansible module for declarative configuration of GitHub teams and repositories

## Description
This module allows you to define a configuration for your GitHub

## Installation
In order to make use of this module, download the [github_config.py](https://raw.githubusercontent.com/particledecay/ansible-github-config/master/github_config.py) file to the `library` directory in the root of your Ansible project:
```
myproject/
├── ansible.cfg
├── inv/
├── library/
│   ├── github_config.py
├── playbooks/
├── roles/
```

## Usage
The module `github_config` takes the following options:

| Name    | Description     | Required? |
| ------- | --------------- | ------- |
| `token` | The GitHub token for API access | yes |
| `path` | The path to the [GitHub config](#Configuration) | one of 'path' or 'config' |
| `config` | The full [GitHub config](#Configuration) (vars) | one of 'path' or 'config' |
| `url` | URL to the GitHub API (default is 'https://github.com/api/v3') | no |

## Configuration
The config structure consists of a dict where the top-level keys are GitHub Org names, with supported subsections:

| Key | Description |
| --- | ----------- |
| `global` | settings that will apply to all repositories |
| `teams` | [definitions for your teams](#team-settings) |
| `repos` | [definitions for your repositories](#repository-settings) |

### Global settings
Note: All global settings can be overridden in individual [repo settings](#repository-settings).

| Key | Description |
| --- | ----------- |
| `access` | teams allowed to access all repositories by default |
| `allow_merge_commit` | (for PRs) add all commits from the head branch to the base branch with a merge commit |
| `allow_rebase_merge` | (for PRs) add all commits from the head branch onto the base branch individually |
| `allow_squash_merge` | (for PRs) combine all commits from the head branch into a single commit in the base branch |
| `automated_security_fixes` | enable automated security PRs for all repositories |
| `delete_branch_on_merge` | automatically delete head branches after a PR is merged |
| `has_issues` | enable GitHub Issues for all repositories |
| `has_projects` | enable GitHub Projects for all repositories |
| `has_wiki` | enable GitHub Wiki for all repositories |
| `private` | ensure all repositories are private |
| `vulnerability_alert` | receive alerts when a new vulnerability is found in one of your dependencies |
| `default_branch` | set the default branch for all repositories |
| `default_branch_protection` | section for protection options for the default branch ([config](#default-branch-protection)) |

#### Default branch protection
The following are settings defined within a `default_branch_protection` block:

| Key | Description |
| --- | ----------- |
| `required_status_checks` | list of required check names that must pass prior to merging a PR |
| `enforce_admins` | enforce protection restrictions for administrators as well |
| `required_pull_request_reviews` | section for required PR review definitions ([config](#required-pull-request-reviews)) |

#### Required pull request reviews
The following are settings defined within a `required_pull_request_reviews` block:

| Key | Description |
| --- | ----------- |
| `dismiss_stale_reviews` | new reviewable commits pushed to the PR will dismiss existing approvals |
| `require_code_owner_reviews` | require approval from designated code owners if a CODEOWNERS file exists |
| `required_approving_review_count` | number of required approvals for PRs |

### Team settings

| Key | Description | Required? |
| --- | ----------- | -------- |
| `id` | the team's seven-digit GitHub ID (see [Obtaining Team ID](#obtaining-team-id)) | yes |
| `name` | the full display name of the team (determines the URL slug) | no |
| `description` | team description text displayed on the team page | no |
| `members` | a list of valid GitHub usernames | no |

#### Obtaining Team ID
GitHub does not currently provide an easy way to determine a team's ID number. However, you can grab any team's ID using a simple curl command and an API token (you need one to use this Ansible module anyway):

```sh
curl -H "Authorization: token MY_GITHUB_TOKEN" https://api.github.com/orgs/MyOrg/teams | jq -r '.[] | select(.name == "My Team Name") | .id'
```

... or using the team's slug instead:

```sh
curl -H "Authorization: token MY_GITHUB_TOKEN" https://api.github.com/orgs/MyOrg/teams | jq -r '.[] | select(.slug == "my-team-name") | .id'
```

### Repository settings

| Key | Description | Required? |
| --- | ----------- | --------- |
| `name` | the name of the repository | yes |
| `description` | the repository description | no |

Additionally, any one of the [global settings](#global-settings) can be set per-repository.

## Full example

```yaml
MyOrg:
  global:
    access:
      - team: 1234567  # Designated approval team
        permission: admin
      - team: 2323232
        permission: push
      - team: 9876543
        permission: pull
    allow_merge_commit: False
    allow_rebase_merge: False
    allow_squash_merge: True  # we only do squash merges
    automated_security_fixes: True
    delete_branch_on_merge: True
    has_issues: True
    has_projects: False
    has_wiki: False
    private: True
    vulnerability_alert: True
    default_branch: master  # typical
    default_branch_protection:
      required_status_checks:
        - WIP
        - my_github_action
      enforce_admins: True
      required_pull_request_reviews:
        dismiss_stale_reviews: True
        require_code_owner_reviews: True
        required_approving_review_count: 1

  teams:
    - id: 1234567
      name: Designated Approvers
      description: Team leads designated for approving code reviews
      members:
        - particledecay
        - davegrohl
        - carlsagan
        - harambe
    - id: 2323232
      name: Developers
      description: All developers at the company
      members:
        - not-going
        - to-write
        - an-entire
        - companys-worth
        - of-usernames
    - id: 9876543
      name: Contractors
      description: Third-party contractors
      members:
        - lexluthor
        - drdoom
        - redskull

  repos:
    - name: galaxy-github-config
      description: Ansible Galaxy role for MyOrg's GitHub config
      access:  # don't want any other teams having access to this repo
        - team: 1234567
          permission: admin
    - name: company-backend
      description: Backend code for MyOrg's app
    - name: company-web
      description: Web frontend to MyOrg's app
    - name: company-mobile
      description: Mobile app by MyOrg
```
