# Moving collections data in S3
A set of GitHub actions to move data periodically from one S3 location (`S3_SRC_BUCKET`) to another (`S3_DST_BUCKET`)

This set of GitHub Actions take advantage of the [`configure-aws-credentials` action](https://github.com/marketplace/actions/configure-aws-credentials-action-for-github-actions) to set up the [AWS cli tool](https://aws.amazon.com/cli/). Once we have this tool set up we can simply call `aws s3 mv <SOURCE> <DESTINATION>` to move our files. 

## 2 GitHub Actions
We have two actions:
1. [`small-collections.yml`](./small-collections.yml)
2. [`large-collections.yml`](./large-collections.yml)

`small-collections.yml` takes care of collections that do not produce hundreds of thousands of `insert`s and/or `update`s daily. These collections _could_ be moved every few days or weekly. At this moment, we are executing this action daily like `large-collections.yml`

`large-collections.yml` takes care of the collections that produce hundreds of thousands of records daily. The move these records "by the hour" instead of moving the whole day in one command. 

To illustrate:

We move small collections by running `aws s3 mv <SOURCE>/2022/10/03/ <DESTINATION>/2022/10/03/ --recursive` 

This will move all the files contained in `<SOURCE>/2022/10/03/` to `<DESTINATION>/2022/10/03/`. In the case of "small collections" the number of records does not exceed one hundred thousand often. 

For large collections we take advantage of the fact that we can run multiple jobs in parallel to move the entire day "per hour" simultaneously. 

We move them by running `aws s3 mv <SOURCE>/2022/10/03/${{ matrix.hour}}/ <DESTINATION>/2022/10/03/${{ matrix.hour }}/ --recursive` and we set up a [strategy matrix](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstrategy) as such:

```yaml
...
strategy:
  matrix:
    hour: ["00", "01", "02", ..., "23"]
...
```

Often, some of these hour directories will contain over one hundred thousand records each. 

## Limits
GitHub Actions can produce a maximum of 256 jobs. At the moment, we have 2 collections in `large-collections.yml`, two operation types, and 24 hours meaning we produce (2 * 2 * 24) 96 jobs. 

## Requirements
4 environment variables are required for this Action to run:
1. AWS_ACCESS_KEY_ID
2. AWS_SECRET_ACCESS_KEY
3. S3_SRC_BUCKET (s3 source path)
4. S3_DST_BUCKET (s3 destination path)
