Release process 📦
==================================

Our backend has a micro-service architecture. 
To help improve the release process we are using [Semantic Release](https://semantic-release.gitbook.io/semantic-release/) in our commits.

To automate finding the next version, we use [go-semrel-gitlab](https://juhani.gitlab.io/go-semrel-gitlab/) in our pipelines.


## Merge requests

Every merge request should run a pipeline to build, test and lint those changes, and only when everything is ✅ the merge can be done. 

The stages for the merge requests are:

![Merge request](./assets/merge_request.png)

- `BuildMR` - Build the project
- `LintMR` - Run lint checks
- `TestsMR` - Run tests 


## After merge

After the merge, a new pipeline should run and effectively release the new build.

The stages are: 

![Merge request](./assets/merged.png)

- `Version` - Find the new version for the build. As we do not deploy automatically to **production**, we use a build number for the releases. Usually we use [GitHub flow](https://guides.github.com/introduction/flow/), so the build number could be the number of commits in the `master` branch, since it represents the number of changes.

- `Build` - Build the project and produce the binary and it should the defined as an artifact.

- `Docker` - Create the docker image and upload it to the GitLab Registry, to the `builds` folder and tagged with the version defined in the `Version` job.

- `TagBuild` - Create a tag, with the format `builds/[number]`, and push it to the remote. Also, this will trigger a new job, `DeployDevelopment`, that automatically will deploy to `development` environment.

- `TagVersion` - Manual step that tags a version, based on `Semantic Release`, this will trigger a new pipeline that will deploy this build to `production` environment.

- `DeployDevelopment` - Deploys the tagged Docker image to the `delopment` environment.

- `DeployProduction` - Manual trigger that deploys the tagged Docker image to the `production` environment.


# Template

Two files were created to help setting up this on Gitlab-CI 
- [master-gitlab-ci.yml](./templates/gitlab-ci/master-gitlab-ci.yml): master file with configurations and jobs that should be common to most of the projects
- [example-gitlab-ci.yml](./templates/gitlab-ci/example-gitlab-ci.yml): file that should be copied to your repository and updated where it makes sence 