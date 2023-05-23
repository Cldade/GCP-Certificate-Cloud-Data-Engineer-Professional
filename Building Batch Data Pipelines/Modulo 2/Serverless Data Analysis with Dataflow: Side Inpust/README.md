# Task 1. Preparation

## Assign the Dataflow Developer role

If the account does not have the Dataflow Developer role, follow the steps below to assign the required role.

1. On the Navigation menu, click IAM & Admin > IAM.

2. Select the default compute Service Account {project-number}-compute@developer.gserviceaccount.com.

3. Select the Edit option (the pencil on the far right).

4. Click Add Another Role.

5. Click inside the box for Select a Role. In the Type to filter selector, type and choose Dataflow Developer.

6. Click Save.

## Ensure that the Dataflow API is successfully enabled

To ensure access to the necessary API, restart the connection to the Dataflow API.

1. In the Cloud Console, enter Dataflow API in the top search bar. Click on the result for Dataflow API.

2. Click Manage.

3. Click Disable API.

If asked to confirm, click Disable.

4. Click Enable.

When the API has been enabled again, the page will show the option to disable.

## Open the SSH terminal and connect to the training VM

You will be running all code from a curated training VM.

1. In the Console, on the Navigation menu , click Compute Engine > VM instances.

2. Locate the line with the instance called training-vm.

3. On the far right, under Connect, click on SSH to open a terminal window.

4. In this lab, you will enter CLI commands on the training-vm.

## Download Code Repository

- Next you will download a code repository for use in this lab. In the training-vm SSH terminal enter the following:

`git clone https://github.com/GoogleCloudPlatform/training-data-analyst`

## Create a Cloud Storage bucket

Follow these instructions to create a bucket.

1. In the Console, on the Navigation menu, click Cloud overview.

2.Select and copy the Project ID.

For simplicity you will use the Qwiklabs Project ID, which is already globally unique, as the bucket name.

3. In the Console, on the Navigation menu, click Cloud Storage > Buckets.

4. Click + Create.

5. Specify the following, and leave the remaining settings as their defaults:

| Property         | Value (type value or select option as specified)|
|------------------|-------------------------------------------------|
| Name             | <your unique bucket name (Project ID)>          | 
| Location type    | Multi-Region                                    | 

6. Click Create.

7. If you get the Public access will be prevented prompt, select Enforce public access prevention on this bucket and click Confirm.

Record the name of your bucket. You will need it in subsequent tasks.

8. In the training-vm SSH terminal enter the following to create two environment variables. One named "BUCKET" and the other named "PROJECT". Verify that each exists with the echo command:

`BUCKET="<your unique bucket name (Project ID)>"
echo $BUCKET`

 `PROJECT="<your unique project name (Project ID)>"
echo $PROJECT`

# Task 2. Try using BigQuery query

1. In the console, on the Navigation menu (Navigation menu icon), click BigQuery.

2. If prompted click Done.

3. Click Compose a new query and type the following query:

`SELECT
  content
FROM
  `cloud-training-demos.github_repos.contents_java`
LIMIT
  10`

4. Click on Run.

What is being returned?

The BigQuery table fh-bigquery.github_extracts.contents_java_2016 contains the content (and some metadata) of all the Java files present in GitHub in 2016.

5. To find out how many Java files this table has, type the following query and click Run:

`SELECT
  COUNT(*)
FROM`
  `cloud-training-demos.github_repos.contents_java`

How many files are there in this dataset?

Is this a dataset you want to process locally or on the cloud?