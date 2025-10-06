# CodeDeploy NGINX Sample

This repository contains a minimal application designed to plug into the Terraform pipeline from exercise 4:

1. CodePipeline detects pushes to this repository through CodeStar Connections.
2. CodeBuild packages the artifact using `buildspec.yml`.
3. CodeDeploy copies the content to EC2 instances that have the CodeDeploy agent installed and the `CodeDeploy=Blue` tag.

## Layout

```
.
├── appspec.yml            # CodeDeploy descriptor (hooks + copy to /usr/share/nginx/html)
├── buildspec.yml          # Packages appspec + scripts + static content
├── html/
│   └── index.html         # Landing page served by NGINX
└── scripts/               # Lifecycle hooks executed by CodeDeploy
    ├── install_dependencies
    ├── start_server
    └── stop_server
```

Update `html/index.html` to change the page served by NGINX. Each push to the monitored branch triggers the pipeline and CodeDeploy rolls out the new version to the target instances.

## Instance requirements

- CodeDeploy agent installed and running. Amazon Linux 2023 is the latest general-purpose Amazon Linux release, but it does **not** ship with the agent preinstalled, so keep it running via user data or SSM.
- Permission to execute `systemctl` as root (the scripts use `sudo`).
- NGINX is installed/enabled automatically during the `BeforeInstall` hook.

## Quick start

1. Authorize this repository in your CodeStar connection.
2. Update `terraform.tfvars` in the infrastructure project:
   ```hcl
   github_owner       = "your-github-user"
   github_repository  = "code-deploy"
   github_branch      = "main"
   codebuild_buildspec = null  # Use the buildspec committed in this repo
   ```
3. Run `terraform apply` to (re)create the pipeline.
4. Commit a change to `html/index.html` and push to test the deployment.

You now have an end-to-end pipeline that keeps NGINX in sync with every commit.
