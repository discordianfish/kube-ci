# Github CI with K8S and Argo.

This is a reimplementation of argo-ci written in Go,
and focused purely on GitHub integration.

# Why?
argo-ci doesn't work well with private github repositories at this time. The
underlying functionality is fairly simple. The assumption here is that we can
implement a minimal equivalent with full github app support, and ssh key
management, in a relatively few lines of code.

# What's here
The existing code is function, though quality needs improving. The existing
application:

If a given commit has a workflow file, location in ```.kube-ci/ci.yaml```,
kube-ci:
- Creates a Check Run when a Check Suite is requested.  
- Creates the a workflow based on the provided file. 
- Reports parsing errors of the user supplied workflow back the Github UI.
- Feeds back the status of Workflow pods to the Github UI
- Links to the argo-ui for the created workflow
- Populates `file:N:M: message` from logs as Annotations in GitHub check runs.

## TODO
- allow configuration of workflow creation options, e.g. namespace, TTL etc.
- allow filtering by branch
- allow different workflows for different branches.
- better metrics
- example deployment assets

## Maybe
- Help with createing per repo/branch temporary storage for cacheing.
- updates to slack

# Deploying

- Pick a URL for the webhook.The Kube-CI github WebHook needs to be reachable
  publicly by GitHub. e.g. at ``` https://kube-ci.example.com/webhooks/github```

- Create a GitHub application
  - Set a webhook secret, keep a note of it.
  - Note the AppID,  
  - Generate a private key. Download it somewhere.
  - Install the App on one of more accounts, you will need to take a note
    of the Installation ID (this shows up in the URL of the settings for the
    installation e.g.
    ```https://github.com/organizations/YOURORG/settings/installations/12345```
  - It need Read permission to repos and repo contents, and read/write to
    Checks.

- Deploy kube-ci
  - You should already have argo deployed. Some things currently assume a
    namespace of "argo" exists for deploying the workflows to.
  - I've only tested this in cluster, but the flags are there for providing
    a kube-config if you want one.
  - You may need to add RBAC rules along the lines of those in `rbac.yml`
  - Setup a service/ingress for the webhook.
  - Create the config files. You can use a secret to pass in the keys file and
    the webhook secret. The webhook secret should be on it's own in a file
    (beware of trailing new lines, I don't trim the secret at the moment).
    The private keys need to be in a file with the installation IDs:
    ``` 
      - ID: 12345
        Key: "Your Private Key Here"
    ```
   - Pass the Application ID on the command line.



