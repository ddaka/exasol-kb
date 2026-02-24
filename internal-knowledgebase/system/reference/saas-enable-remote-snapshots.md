---
tool_name: cos
doc_type: reference
category: system
title: "SaaS - Enable Remote Snapshots"
summary: "Example:"
---
# SaaS - Enable Remote Snapshots

1. Create remote bucket in the offline account:

    Example:

        Local bucket name: `test-s3bucket-pj8bq9vf91ly`
        Remote bucket name: `test-s3bucket-pj8bq9vf91ly-offline`

2. Get instance role arn of the stack and create the policy for the new bucket like this:
    ```
    {
    	"Version": "2012-10-17",
    	"Statement": [
    		{
    			"Sid": "backupsync",
    			"Effect": "Allow",
    			"Principal": {
    				"AWS": "arn:aws:iam::533884745449:role/test-SaaSDBRole-1CHQH8HVNMDOX"
    			},
    			"Action": "s3:*",
    			"Resource": [
    				"arn:aws:s3:::test-s3bucket-pj8bq9vf91ly-offline",
    				"arn:aws:s3:::test-s3bucket-pj8bq9vf91ly-offline/*"
    			]
    		},
    	    {
    			"Sid": "rootaccess",
    			"Effect": "Allow",
    			"Principal": {
    				"AWS": "arn:aws:iam::533884745449:root"
    			},
    			"Action": "s3:*",
    			"Resource": [
    				"arn:aws:s3:::test-s3bucket-pj8bq9vf91ly-offline",
    				"arn:aws:s3:::test-s3bucket-pj8bq9vf91ly-offline/*"
    			]
    		}
    	]
    }
    ```
3. Stop sbfs process running on master plus comment crontab jobs for snapshot and backup under the /etc/crontab.d/exasnapshotbackups
4. Stop DB to prevent data corruption during:
    ```
    dwad_client stop Exasol
    ```
5. Copy volumes 3 & 4 data to the new remote bucket `backup` folder (just in case something bad happens)
    ```
    aws s3 sync --profile dev s3://test-s3bucket-pj8bq9vf91ly/volumes/3 s3://test-s3bucket-pj8bq9vf91ly-offline/backup/3
    aws s3 sync --profile dev s3://test-s3bucket-pj8bq9vf91ly/volumes/4 s3://test-s3bucket-pj8bq9vf91ly-offline/backup/4
    ```
6. Remove volumes 3 and 4 from the instance:
    ```
    csvol -d -v 3
    csvol -d -v 4
    ```
7. Add volumes 3 and 4 to instance now pointing to the remote backup we created above:
    ```
    export REMOTE_BUCKET=test-s3bucket-pj8bq9vf91ly-offline
    ```

    ```
    csvol --create --bucket "$REMOTE_BUCKET" --set-shared false --permissions rwx------ --bucket-type S3 --region eu-central-1 --volume 3
    csvol --set-name --name DataVolume2Sync -v 3
    csvol -O -u 500 -g 500 -v 3
    csvol --create --bucket "$REMOTE_BUCKET" --set-shared false --permissions rwx------ --bucket-type S3 --region eu-central-1 --volume 4
    csvol --set-name --name SnapshotVolumeSync -v 4
    csvol -O -u 500 -g 500 -v 4
    sbfs remote-format 4 -metadata-volume 0 -data-volume 3
    ```
8. Push snapshots to the remote bucket:
    ```
    sbfs sync-all 2 4 -next-expiring-first
    ```
9. Verify remote bucket data integrity:
    ```
    sbfs verify-dataintegrity 4
    ```
10. Restart the DB:
    ```
    dwad_client start-wait Exasol
    ```
11. Reactivate crontab jobs
