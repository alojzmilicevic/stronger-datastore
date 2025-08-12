# stronger-datastore

Infrastructure as code for the Exercises datastore using AWS CloudFormation.

## Layout

- `cloudformation/exercises.yaml` — DynamoDB table (PK: `id`)
- `cloudformation/iam-exercises-access.yaml` — IAM role + managed policy to access the table
- `cloudformation/main.yaml` — parent template that nests the above two
- `data/example-item.json` — example document to insert

## Deploy

1. Create (or choose) an S3 bucket to host nested templates, e.g.:

   ```sh
   aws s3 mb s3://$ACCOUNT-$REGION-exercises-cfn-artifacts --region $REGION
   ```

2. Upload the nested templates to the bucket under `cloudformation/`:

   ```sh
   aws s3 cp cloudformation/exercises.yaml s3://$BUCKET/cloudformation/exercises.yaml
   aws s3 cp cloudformation/iam-exercises-access.yaml s3://$BUCKET/cloudformation/iam-exercises-access.yaml
   ```

3. Deploy the parent stack:

   ```sh
   aws cloudformation deploy \
     --stack-name ExercisesRoot \
     --template-file cloudformation/main.yaml \
     --parameter-overrides ArtifactsBucketName=$BUCKET StackNamePrefix=Exercises \
     --capabilities CAPABILITY_NAMED_IAM
   ```

4. Insert an example item (replace table name if customized):

   ```sh
   aws dynamodb put-item \
     --table-name ExercisesTable \
     --item file://data/example-item.json \
     --return-consumed-capacity TOTAL
   ```

Notes:
- The table is schemaless beyond the partition key `id` (string).
- Billing mode defaults to `PAY_PER_REQUEST`; adjust via parameters.
- The IAM role is very open in `AssumeRolePolicy` for demo purposes. Tighten for real use.
