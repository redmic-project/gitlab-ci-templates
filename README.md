# GitLab CI templates

Collection of CI/CD templates used and extended by REDMIC platform projects

## <a name="backward-compatibility"></a>Backward compatibility

Until [`v1.6.0`](https://gitlab.com/redmic-project/gitlab-ci-templates/-/releases/v1.6.0), templates were defined at project root. Now they are grouped into subdirectories, but there is a compatibility template in place for each one.

These compatibility templates only import current version of templates, adding some warnings about using deprecated version of templates.

Legacy projects using these deprecated templates should continue working ok, but is recommended to migrate template imports to updated versions.

This compatibility layer will be removed when `v2.0.0` is released. If you need to use old templates after this breaking change is done, fix your templates include to a previous version.

## Templates description

You can use templates available at this project at your own projects, importing them at your `.gitlab-ci.yml` file.

Each template contains one or more jobs, which you can use *as-is* or customize, overriding values or extending your own jobs from them.

Note that some templates have a base version, prefixed with an underscore (`_`). You can import them if this base version is more convenient to your project.

### Root level

All templates at root level (except `deprecation-warning.yml`) are deprecated, but linked to current version of templates. Check [Backward compatibility](#backward-compatibility) for further details.

### Building

#### Maven

Build an application with Maven, using [`redmic-project/docker/maven`](https://gitlab.com/redmic-project/docker/maven).

* **functional-unit.yml**: Build a functional-unit project. Include 3 stages: `build-parent`, `build-lib` and `build-service`.
* **library.yml**: Build a library project. Run at `build` stage.
* **microservice.yml**: Build a microservice project. Run at `build` stage.

### Deployment external service

Deploy ancillary services (predefined externally) to your main service.

* **backup-files.yml**: Deploy a files backup service. Defined at [`redmic-project/maintenance/backup-files`](https://gitlab.com/redmic-project/maintenance/backup-files). Run at `deploy-external-service` stage.
* **backup-postgresql.yml**: Deploy a PostgreSQL database backup service. Defined at [`redmic-project/postgres/backup-postgresql`](https://gitlab.com/redmic-project/postgres/backup-postgresql). Run at `deploy-external-service` stage.

### Deployment package

* **npm-deployment.yml**: Upload an already built NPM package to *GitLab packages* of your project. Run at `deploy` stage. Inherit from [GitLab CI templates](https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates).

### Deployment service

Deploy a service (based in Docker) in a remote environment, using [`redmic-project/docker/docker-deploy`](https://gitlab.com/redmic-project/docker/docker-deploy).

* **custom-image.yml**: Adapt jobs from `deployment-service/docker-deploy.yml` to use a custom Docker image, defined at your project and uploaded to *GitLab Docker registry* of your project.

* **docker-deploy.yml**: Deploy one or more services defined at your project (using compose files and related contents) to a remote environment of your choice. Run at `deploy` stage. Support multiple environments (like `dev` and `pro`).

* **maintenance.yml**: Relaunch already deployed services, in order to do maintenance tasks. You must trigger these jobs using *GitLab pipeline schedules*. Run at `maintenance` stage. Support multiple environments (like `dev` and `pro`).

#### Functional unit

* **docker-deploy.yml**: Same as `deployment-service/docker-deploy.yml`, but preconfigured to deploy services defined into a functional-unit, using always custom images (like `deployment-service/custom-image.yml` does). Run at `deploy` stage.

### Packaging Docker

Only for projects with Docker image definitions. Include Docker image building, tagging and pushing, and also Dockerfile linting and Docker image scanning.

* **docker-build.yml**: Build a Docker image defined at your project, using [`pedroetb-projects/docker-build`](https://gitlab.com/pedroetb-projects/docker-build). Imports jobs from `packaging-docker/dockerfile-linting.yml` and `scanning/container-scanning.yml` too. Include 3 stages: `pre-package`, `package` and `post-package`.
* **dockerfile-linting.yml**: Run syntax checks over Dockerfile at your project. Run at `pre-package` stage. Already included at `packaging-docker/docker-build.yml`.

### Scanning

* **container-scanning.yml**: Run vulnerability checks over your Docker image. Run at `post-package` stage. Already included at `packaging-docker/docker-build.yml`. Inherit from [GitLab CI templates](https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates).
* **dependency-scanning.yml**: Lookup dependencies at your code, finding vulnerabilities and licenses compliance status. Run at `test` stage. Inherit from [GitLab CI templates](https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates).
