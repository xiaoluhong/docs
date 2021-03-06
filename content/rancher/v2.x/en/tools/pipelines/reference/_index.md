---
title: Pipeline Variable Reference
weight: 8000
---

For your convenience, the following variables are available for your pipeline configuration scripts. During pipeline executions, these variables are replaced by metadata. You can reference them in the form of `${VAR_NAME}`.

Variable Name           | Description
------------------------|------------------------------------------------------------
`CICD_GIT_REPO_NAME`      | Repository name (Github organization omitted).
`CICD_GIT_URL`            | URL of the Git repository.
`CICD_GIT_COMMIT`         | Git commit ID being executed.
`CICD_GIT_BRANCH`         | Git branch of this event.
`CICD_GIT_REF`            | Git reference specification of this event.
`CICD_GIT_TAG`            | Git tag name, set on tag event.
`CICD_EVENT`              | Event that triggered the build (`push`, `pull_request` or `tag`).
`CICD_PIPELINE_ID`        | Rancher ID for the pipeline.
`CICD_EXECUTION_SEQUENCE` | Build number of the pipeline.
`CICD_EXECUTION_ID`       | Combination of `{CICD_PIPELINE_ID}-{CICD_EXECUTION_SEQUENCE}`.
`CICD_REGISTRY`           | Address for the Docker registry for the previous publish image step, available in the Kubernetes manifest file of a `Deploy YAML` step.
`CICD_IMAGE`              | Name of the image built from the previous publish image step, available in the Kubernetes manifest file of a `Deploy YAML` step. It does not contain the image tag.<br/><br/> [Example](https://github.com/rancher/pipeline-example-go/blob/master/deployment.yml)

## Full `.rancher-pipeline.yml` Example

```yaml
# example
stages:
  - name: Build something
    # Conditions for stages
    when:
      branch: master
      event: [ push, pull_request ]
    # Multiple steps run concurrently
    steps:
    - runScriptConfig:
        image: busybox
        shellScript: echo ${FIRST_KEY} && echo ${ALIAS_ENV}
      # Set environment variables in container for the step
      env:
        FIRST_KEY: VALUE
        SECOND_KEY: VALUE2
      # Set environment variables from project secrets
      envFrom:
      - sourceName: my-secret
        sourceKey: secret-key
        targetKey: ALIAS_ENV
    - runScriptConfig:
        image: busybox
        shellScript: date -R
      # Conditions for steps
      when:
        branch: [ master, dev ]
        event: push
  - name: Publish my image
    steps:
    - publishImageConfig:
        dockerfilePath: ./Dockerfile
        buildContext: .
        tag: rancher/rancher:v2.0.0
        # Optionally push to remote registry
        pushRemote: true
        registry: reg.example.com
  - name: Deploy some workloads
    steps:
    - applyYamlConfig:
        path: ./deployment.yml
# branch conditions for the pipeline
branch:
  include: [ master, feature/*]
  exclude: [ dev ]

```
