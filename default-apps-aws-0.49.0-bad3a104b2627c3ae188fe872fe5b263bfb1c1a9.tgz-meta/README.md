[![CircleCI](https://circleci.com/gh/giantswarm/default-apps-aws.svg?style=shield)](https://circleci.com/gh/giantswarm/default-apps-aws)

# default-apps-aws chart

Giant Swarm offers a default-apps-aws App including the apps pre-installed in all AWS workload clusters.

# Testing

We currently have two different pipelines to test both cluster creation and cluster upgrades. You can trigger these pipelines by writing these commands in a pull request comment or description:

- `/test create`: This will trigger the `create-cluster-capi-pure` pipeline.
- `/test upgrade`: This will trigger the `upgrade-cluster-capi-pure` pipeline.

After writing these comments, the pipelines are triggered in [Tekton](https://tekton.giantswarm.io/#/pipelineruns). Eventually, the GitHub checks `create` and `upgrade` for these pipelines should show up in the commit statuses.
It may happen that the status is never shown on GitHub UI. If you want to check the status or result of the pipelines you can check [Tekton](https://tekton.giantswarm.io/#/pipelineruns).
