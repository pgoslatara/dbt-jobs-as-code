As jobs are defined in YAML files, we can use [YAML anchors](https://support.atlassian.com/bitbucket-cloud/docs/yaml-anchors/) to avoid duplicating the same values in multiple jobs.

For example, we can define a common `settings` section in an anchor and then reuse it in all our jobs:

```yaml title="jobs.yml"
anchors:
    schedule: &every_hour # using a non-job as an anchor
    cron: "0 * * * *"
    date:
        cron: "0 * * * *"
        type: "custom_cron"
    time:
        hours: null
        interval: 1
        type: every_hour
    ids_prod_env: &ids_prod_env
    account_id: 43791
    environment_id: 134459
    project_id: 176941

jobs:
    job1: &main_job # using parameters of a job as the anchor
    <<: *ids_prod_env
    dbt_version: null
    deactivated: false
    deferring_job_definition_id: null
    deferring_environment_id: null
    execute_steps:
    - "dbt run --select model1+"
    - "dbt run --select model2+"
    - "dbt compile"
    execution:
        timeout_seconds: 0
    generate_docs: false
    generate_sources: true
    name: "My Job 1 with a new name"
    run_generate_sources: true
    schedule:
        cron: "0 */2 * * *"
        date:
        cron: "0 */2 * * *"
        type: "custom_cron"
        time:
        type: "every_hour"
        interval: 1
    settings:
        target_name: production
        threads: 4
    state: 1
    triggers:
        git_provider_webhook: false
        github_webhook: false
        schedule: true
    custom_environment_variables: []
    job2:
    <<: *main_job # << means that we take all the values from the first job but we then overwrite them
    deferring_job_definition_id: null
    deferring_environment_id: null
    execute_steps:
    - dbt run-operation clone_all_production_schemas
    - dbt compile
    generate_docs: true
    generate_sources: true # what does it do??
    name: CI/CD run
    run_generate_sources: true
    schedule: *every_hour # * means that we take the value as-is
    settings:
        target_name: TEST
        threads: 4
    triggers:
        git_provider_webhook: false
        github_webhook: true # this job runs from webhooks
        schedule: false # this doesn't run on a schedule
    custom_environment_variables:
        - DBT_ENV1: My val
        - DBT_ENV2: My val2
```

We can use anchors when using glob patterns for the config and variables files, but the anchors won't be "shared" across files.
