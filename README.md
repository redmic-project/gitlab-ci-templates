# GitLab CI templates

Collection of CI/CD templates used and extended by REDMIC platform projects

[TOC]

## Templates description

You can use templates available at this project into your own projects, importing them at your `.gitlab-ci.yml` file.

Each template contains one or more *GitLab CI Jobs*, which you can use *as-is* or customize, overriding values or extending your own jobs from them.

Note that some templates have a base definition version, prefixed with an underscore (`_`). You can import them if this definition is more convenient to your project.

## Stages

*GitLab CI Jobs* from these templates run at a specific **stage** of your project *GitLab CI Pipelines*. Check description of each template for more info.

All jobs contained at these templates are prepared to run at any of [GitLab CI default pipeline stages](https://docs.gitlab.com/ci/yaml/#stages):

1. `.pre`
1. `build`
1. `test`
1. `deploy`
1. `.post`

If your project needs different stages or stage reordering, you may define these stages (honoring default stages, ordered as you need) at your project's `.gitlab-ci.yml`, or jobs at your custom stages will not run. Jobs stage value can be overwritten too.

## Templates usage

Templates are included into `.gitlab-ci.yml` like this:

```yaml
include:
  - project: 'redmic-project/gitlab-ci-templates'
    ref: main
    file: '/test/auto.yml'

...
```

## Available templates

Templates are located at different directories, attending their stage and purpose.

### Build

Template files located into `build/` directory defines jobs which run at `build` stage.

#### Maven

Build an application with Maven, using [`redmic-project/docker/maven`](https://gitlab.com/redmic-project/docker/maven).

* **build/maven/mvnw.yml**: Build a project using Maven Wrapper (**MVNW**). Requires using `maven >= v3.9.0` and using a project with `mvnw` available.
* *[legacy]* **build/maven/library.yml**: Build a library project.
* *[legacy]* **build/maven/microservice.yml**: Build a microservice project.
* *[legacy]* **build/maven/functional-unit.yml**: Build a functional unit project.

#### Docker

Only for projects with Docker image definitions. Include Docker image building, tagging and pushing (and other container-related jobs).

* **build/docker/docker-build.yml**: Build a Docker image defined at your project, using [`pedroetb-projects/docker-build`](https://gitlab.com/pedroetb-projects/docker-build). Imports jobs from `build/docker/dockerfile-linting.yml` and `test/container-scanning.yml` too, so includes jobs at 2 different stages: `build` and `test`.
* **build/docker/dockerfile-linting.yml**: Run syntax checks over Dockerfile at your project. Already included at `build/docker/docker-build.yml`.

### Test

Template files located into `test/` directory defines jobs which run at `test` stage.

Perform some scanning jobs over project resources. All these jobs inherit from [GitLab CI templates](https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates), adding specific configuration details.

* **test/auto.yml**: Include recommended scanning jobs.
* **test/code-quality.yml**: Identifies maintainability issues before they become technical debt.
* **test/sast.yml**: Static application security testing (SAST) discovers vulnerabilities in your source code before they reach production.
* **test/secret-detection.yml**: Minimize the risk of exposing your secrets.
* **test/dependency-scanning.yml**: Identify security vulnerabilities in your application's dependencies.
* **test/container-scanning.yml**: Run vulnerability checks over your Docker image. Already included at `build/docker/docker-build.yml`, but not by `test/auto.yml`.

### Deploy

Template files located into `deploy/` directory defines jobs which run at `deploy` stage.

#### Package deployment

* **deploy/package/npm.yml**: Upload an already built NPM package to *GitLab packages* of your project. Inherit from [GitLab CI templates](https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates).

#### Service deployment

Deploy a service (based in Docker) in a remote environment, using [`redmic-project/docker/docker-deploy`](https://gitlab.com/redmic-project/docker/docker-deploy).

* **deploy/service/docker-deploy.yml**: Deploy one or more services defined at your project (using compose files and related contents) to a remote environment. Usage of custom Docker images is supported when they are pulled from your project's *GitLab Docker registry*.
* **deploy/service/fu-docker-deploy.yml**: Same as `deploy/service/docker-deploy.yml`, but preconfigured to deploy services defined into a functional unit.

#### E2E

Perform some testing jobs over services deployments.

* **deploy/e2e/playwright.yml**: Run E2E (end to end) tests defined at your project with [Playwright](https://playwright.dev/). Include different jobs for branch, tag and schedule pipelines.
* **deploy/e2e/playwright-parallel.yml**: Same as `deploy/e2e/playwright.yml`, but running in parallel jobs (`2` by default, you can increase it overwritting `parallel` property). Use sharding to split tests between available parallel jobs.

#### DAST (Dynamic Application Security Testing)

Run DAST scans against deployed services using [OWASP ZAP](https://www.zaproxy.org/) with the [Automation Framework](https://www.zaproxy.org/docs/automate/automation-framework/).

* **deploy/dast/zap.yml**: Run OWASP ZAP scans against a deployed application.

Each project can provide its own ZAP Automation Framework configuration at `.zap/zap.yaml` in the project root. If this file is not present, a default baseline scan (spider + passive scan) is used automatically.

**Variables:**

| Variable | Description | Default |
| --- | --- | --- |
| `DAST_TARGET_URL` | URL of the deployed application to scan (required) | *(empty)* |
| `ZAP_CONFIG_FILE` | Path to the project ZAP config file | `.zap/zap.yaml` |

**Reference configuration:**

A reference default ZAP config is available below, which can be used as a starting point for custom project configurations.

``` yaml
env:
  contexts:
    - name: "Default Context"
      urls:
        - "${DAST_TARGET_URL}"
      includePaths:
        - "${DAST_TARGET_URL}.*"
  parameters:
    failOnError: true
    failOnWarning: false
    progressToStdout: true

jobs:
  - type: spider
    parameters:
      context: "Default Context"
      maxDuration: 5
      maxDepth: 5
      maxChildren: 10

  - type: spiderAjax
    parameters:
      context: "Default Context"
      maxDuration: 5
      maxCrawlDepth: 5
      numberOfBrowsers: 2

  - type: passiveScan-wait
    parameters:
      maxDuration: 10

  - type: report
    parameters:
      template: "traditional-html-plus"
      reportDir: "${CI_PROJECT_DIR}/zap-reports"
      reportFile: "zap-report"
      reportTitle: "ZAP DAST Report"
      reportDescription: "Automated DAST scan report generated by OWASP ZAP"
    risks:
      - high
      - medium
      - low
      - info

  - type: report
    parameters:
      template: "traditional-json-plus"
      reportDir: "${CI_PROJECT_DIR}/zap-reports"
      reportFile: "zap-report"
      reportTitle: "ZAP DAST Report"
      reportDescription: "Automated DAST scan report generated by OWASP ZAP"
    risks:
      - high
      - medium
      - low
      - info
```
