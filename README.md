# S3 Sync

Command line tools to mirror your current directory in an AWS bucket.

## Set up

This CLI will look in your home directory for a file called ".aws.json"
that contains the JSON string of your AWS credentials.

```json
{
  "access_key_id": "YOUR_ACCESS_KEY_ID",
  "secret_access_key": "YOUR_SECRET_ACCESS_KEY"
}
```

To get these credentials, log into your [AWS S3 console][s3 console], click on your name in
the navigation menu and open "Security Credentials". See [here][credentials docs] for more details.

[s3 console]: https://console.aws.amazon.com/s3
[credentials docs]: http://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html

## Tasks

**s3:auth**

This command will prompt you for your AWS credentials and store them in your home
directory in a file called ".aws.json".

**s3:upload**

This uploads every file in the current directory and its subdirectories to an
AWS bucket. Any file beginning with "." is ignored. It syncs to arbitary levels
of nesting so be careful with symlinks that could cause an infinite loop.

The bucket will take the name of the current directory, replacing underscores
with dashes. Note that the bucket's name must conform with DNS requirements if
you plan on publishing it as a website.

*Caution: If a bucket with that name already exists and contains files with the
same names as the files in your directory, those files will be overriden without
warning.*

