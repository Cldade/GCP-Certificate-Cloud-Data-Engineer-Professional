# Task 1: Ensure that the Dataflow API is successfully enable

- Execute the following block of code in the Cloud Shell:

`gcloud services disable dataflow.googleapis.com --force
gcloud services enable dataflow.googleapis.com``

# Task 2: Preparation

## Open the SSH terminal and connect to the training VM

You will be running all code from a curated training VM.

1. In the console, on the **Navigation menu**, click **Compute Engine** > **VM instances**.

2. Locate the line with the instance called **training-vm**.

3. On the far right, under **Connect**, click on **SSH** to open a terminal window.

4. In this lab, you will enter CLI commands on the **training-vm**.

## Download code repository

- Download a code repository to use in this lab. In the training-vm SSH terminal enter the following:

`git clone https://github.com/GoogleCloudPlatform/training-data-analyst`

## Create a Cloud Storage bucket

Follow these instructions to create a bucket.

1. In the console, on the **Navigation menu**, click **Cloud overview**.

2. **Select and copy** the Project ID.

For simplicity use the Project ID found in the Lab details panel is already globally unique. Use it as the bucket name.

3. In the console, on the **Navigation menu**, click **Cloud Storage > Buckets**.

4. Click + **Create***.

5. Specify the following, and leave the remaining settings as their defaults:

| Property         | Value (type value or select option as specified)|
|------------------|-------------------------------------------------|
| Name             | <your unique bucket name (Project ID)>          | 
| Location type    | Multi-Region                                    | 

6. Click **Create**.

7. If you get the Public access will be prevented prompt, select Enforce public access prevention on this bucket and click **Confirm**.

Record the name of your bucket to use in subsequent tasks.

8. In the **training-vm** SSH terminal enter the following to create an environment variable named "BUCKET" and verify that it exists with the echo command:

`BUCKET="<your unique bucket name (Project ID)>"
echo $BUCKET`

You can use $BUCKET in terminal commands. And if you need to enter the bucket name <your-bucket> in a text field in the console, you can quickly retrieve the name with echo $BUCKET.

# Task 3: Pipeline filtering

The goal of this lab is to become familiar with the structure of a Dataflow project and learn how to execute a Dataflow pipeline.

1. Return to the *training-vm* SSH terminal and navigate to the directory `/training-data-analyst/courses/data_analysis/lab2/python` and view the file grep.py.

2. View the file with Nano. Do not make any changes to the code:

`cd ~/training-data-analyst/courses/data_analysis/lab2/python
nano grep.py`

3. Press CTRL+X to exit Nano.

Can you answer these questions about the file grep.py?

- What files are being read?

All files found in the path: "../javahelp/src/main/java/com/google/cloud/training/dataanalyst/javahelp/"

- What is the search term?

All files that ending with ".java"

- Where does the output go?

The output go in the path: "/tmp/output"

There are three transforms in the pipeline:

- What does the transform do?

GetJava: This is a user-defined label given to the first transformation in the pipeline. It represents reading lines from the input Java files.

The beam.io.ReadFromText function is used to read text from the input files specified by the input variable. In this case, the input variable contains a file pattern (../javahelp/src/main/java/com/google/cloud/training/dataanalyst/javahelp/*.java) that matches multiple Java files. The transformation emits individual lines from these files as the pipeline's initial data.

- What does the second transform do?

Grep: This is another user-defined label representing the FlatMap transformation that filters lines containing the specified search term.

The beam.FlatMap function applies the provided lambda function, lambda line: my_grep(line, searchTerm), to each input element (line). The lambda function, my_grep, checks if the search term is present in the line. If it is, the line is yielded as an output element from the transformation.

- What does it do with this input?

1. Read lines from Java files.

2. Filter lines containing the specifiv search term.

3. Write filteres lines to a text file.

- What does it write to its output?

The Apache Beam pipeline writes the filtered lines to its output, which is a text file. The specific content written to the output file depends on the lines that match the specified search term.

- What does the third transform do?

write: This is the final user-defined label representing the WriteToText transformation that writes the filtered lines to a text file.

The beam.io.WriteToText function writes the filtered lines received from the previous transformation to a text file. The output file path is specified by the output_prefix variable.

# Task 4: Execute the pipeline locally

1. In the *training-vm* SSH terminal, locally execute grep.py:

`python3 grep.py`

Note: Ignore the warning if any.

The output file will be output.txt. If the output is large enough, it will be sharded into separate parts with names like: output-00000-of-00001.

2. Locate the correct file by examining the file's time:

`ls -al /tmp`

3. Examine the output file(s).

4. You can replace "-*" below with the appropriate suffix:

`cat /tmp/output-*`

Does the output seem logical?

What has been saved in the 'outpur' file are all the imports it has found.

# Task 5: Execute the pipeline on the cloud

1. Copy some Java files to the cloud. In the training-vm SSH terminal, enter the following command:

`gcloud storage cp ../javahelp/src/main/java/com/google/cloud/training/dataanalyst/javahelp/*.java gs://$BUCKET/javahelp`

2. Using Nano, edit the Dataflow pipeline in grepc.py:

`nano grepc.py`

3. Replace PROJECT and BUCKET with your Project ID and Bucket name.

Example strings before you update:

`PROJECT='cloud-training-demos'
BUCKET='cloud-training-demos'`

Example strings after edit (use your values):

`PROJECT='qwiklabs-gcp-your-value'
BUCKET='qwiklabs-gcp-your-value'`

Save the file and close Nano by pressing the CTRL+X key, then type Y, and press Enter.

4. Submit the Dataflow job to the cloud:

`python3 grepc.py`

Because this is such a small job, running on the cloud will take significantly longer than running it locally (on the order of 7-10 minutes).

5. Return to the browser tab for the console.

6. On the Navigation menu, click Dataflow and click on your job to monitor progress.

7. Wait for the Job status to be Succeeded.

8. Examine the output in the Cloud Storage bucket.

9. On the Navigation menu, click Cloud Storage > Buckets and click on your bucket.

10. Click the javahelp directory.

This job generates the file output.txt. If the file is large enough, it will be sharded into multiple parts with names like: output-0000x-of-000y. You can identify the most recent file by name or by the Last modified field.

11. lick on the file to view it.

Alternatively, you can download the file via the training-vm SSH terminal and view it:

`gcloud storage cp gs://$BUCKET/javahelp/output* .
cat output*`