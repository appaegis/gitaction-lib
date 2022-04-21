## Common Gitaction Library

### What is in here?
In this repo, you will find a set of reusable gitaction workflows and composite git actions. When building the gitaction workflow for each repo, we tried to make build common reusable workflows so that each individual repo can avoid to reinvent the wheel.

While the first time this readme is written, there are 5 workflows being created:

#### global-export.yaml
One of the limitation with gitaction when we first start to use gitaction for the CI workflow is that gitaction doesn't really support global variable across the jobs. That makes the unification of reusable workflow difficult. To address this problem, we create a dummy workflow that returns all the global variables. And then we make this workflow as dependency of other workflows, so that this workflow always run before another workflows, in that case, other workflows can import the output of this workflow as their input parameter

The global variables:
* ${{ needs.global-var.outputs.project }}, this is the current repository name
* ${{ jobs.export_job.outputs.region }}, this is the AWS region

##### coverage-state.yaml
This workflow calls github API to insert coverage check as one of the PR status check, if a git action workflow requires to verify the new code change doesn't reduce overall unit test coverage, then this workflow should be used. This workflow should be the first workflow after global variable exporter.

#### codecheckout.yaml
This workflow checkout current repo with golang-common. Of course, if the current repo, the golang-common will not be re-checkouted.

#### build-and-test.yaml
This workflow will run make build and make test against the current repo

#### build.yaml
This workflow will run make build without make test

#### coverage-check.yaml
This workflow verifies if the new unit test coverage is reduced and report failure or success based on the verification result

### Composite Gitaction
To support service container for integration tests. We create a composite job for build and test. The composite action can be found in build-and-test directory

### How to include a workflow
A workflow can use the "uses" statement, below is an example to invoke global-exporter.yaml:

```
jobs:
  global-var:
    uses: appaegis/gitaction-lib/.github/workflows/global-export.yaml@main
    with:
      repository: ${{ github.repository }}
```

In this example, you define a job, and the first step of job is to execute global-var by use workflow i n appaegis/gitaction-lib/.github/workflows/global-export.yaml@main. The string behind of @ defines the branch of the workflow to use

### How to include a composite git action
An example to find composite can be found in datalake repo, here is the copy the code segment:

```
    steps:
      - id: build-and-test
        uses: appaegis/gitaction-lib/build-and-test@composite-action
        with:

```
