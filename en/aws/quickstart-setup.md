# Setup

You'll need an Amazon Web Services account and credentials set up your development machine.

On *nix systems AWS Credentials are listed in:

```bash
~/.aws/credentials
```

Or on Windows systems:

```bash
C:\Users\USER_NAME\.aws\credentials
```

If that file doesn't exist, create it, and add something like the following:

```bash
[work]
aws_access_key_id=xxx
aws_secret_access_key=xxx

[personal]
aws_access_key_id=xxx
aws_secret_access_key=xxx
```

All arc npm run scripts require `AWS_PROFILE` and `AWS_REGION` environment variables set. Currently we reccomend putting them in your `npm run` scripts and *not* setting them globally on your system. This leaves room for the common scenario of people and/or organizations with multiple AWS accounts.

> Tip: Windows users will want to use [cross-env](https://www.npmjs.com/package/cross-env) for cross platform env vars; more info about setting up for Windows can found in [this issue thread](https://github.com/arc-repos/arc-docs/issues/19#issuecomment-321738242)

Having your personal AWS setup separated from the work one is just a suggestion! (You can call them anything.)

You can learn more about AWS creds from the source: [Amazon Configuration and Credential Files](http://docs.aws.amazon.com/cli/latest/userguide/cli-config-files.html).

## Next: [Install architect](/quickstart/install)
