---
sidebar_position: 2
---

# Deploy projects

The `Monaco` tool can deploy a configuration or a set of configurations in the form of projects. 
A project is a folder containing files that define the configurations to be deployed to an environment or a group of environments. 
This is done by passing the `--project` flag (or `-p` for short).

## Running the tool

Below you find a few samples on how to run the tool to deploy your configurations:

```shell
# Deploy all projects in the current folder to all environments
monaco -e=environments.yaml

# Deploy projects-root-folder/project and any projects in projects-root-folder it depends on to all environments
monaco -e=environments.yaml -p="project" projects-root-folder

# Deploy projects-root-folder/projectA, projectB and dependencies to all environments
monaco -e=environments.yaml -p="projectA, projectB" projects-root-folder

# Deploy all projects in the current folder to the "dev" environment defined in environments.yaml
monaco -e=environments.yaml -se dev
```

If `project` contains additional sub-projects, all projects are deployed recursively. If `project` depends on different projects under the same root,
those are also deployed.

Multiple projects can be specified by `-p="projectA, projectB, projectC/subproject"`.

To deploy the configuration, `Monaco` needs a valid API Token(s) for each environment.
These are defined as `environment variables`; you can define the name of that env var in the environments file that is specified as an argument to the `-e` option.

To deploy to a specific environment within an `environments.yaml` file, use the `-specific-environment` or `-se` flag:

```shell
monaco -e=environments.yaml -se=my-environment -p="my-environment" cluster
```
