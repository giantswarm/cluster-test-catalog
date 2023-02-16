[![CircleCI](https://circleci.com/gh/giantswarm/default-apps-gcp.svg?style=shield)](https://circleci.com/gh/giantswarm/default-apps-gcp)

# default-apps-gcp chart

Giant Swarm offers a default-apps-gcp App including the apps pre-installed in all GCP workload clusters.

# Testing

We currently have two different pipelines to test both cluster creation and cluster upgrades. You can trigger these pipelines by writing these commands in a pull request comment or description
- `/test create` : this will trigger the `create-cluster-capi-pure` pipeline.
- `/test upgrade` : this will trigger the `upgrade-cluster-capi-pure` pipeline.

After writing these comments, the pipelines are triggered in [tekton](https://tekton.giantswarm.io/#/pipelineruns). Eventually, the Github checks `create` and `upgrade` for these pipelines should show up in the commit statuses.
It may happen that the status is never shown on Github UI. If you want to check the status or result of the pipelines you can check [tekton](https://tekton.giantswarm.io/#/pipelineruns).
