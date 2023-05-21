# Task 1: Lift and shift

## Migrate existing Spark jobs to Cloud Dataproc

You will create a new Cloud Dataproc cluster and then run an imported Jupyter notebook that uses the cluster's default local Hadoop Distributed File system (HDFS) to store source data and then process that data just as you would on any Hadoop cluster using Spark. This demonstrates how many existing analytics workloads such as Jupyter notebooks containing Spark code require no changes when they are migrated to a Cloud Dataproc environment.

1. In the GCP Console, on the **Navigation menu**, in the **Analytics** section, click **Dataproc**.

2. Click **Create Cluster**.

3. Click **Create** for the item **Cluster on Compute Engine**.

4. Enter sparktodp for **Cluster Name**.

5. In the **Versioning** section, click **Change** and select **2.1 (Debian 11, Hadoop 3.3, Spark 3.3)**.

This version includes Python3, which is required for the sample code used in this lab.

6. Click **Select**.

7. In the **Components > Component gateway** section, select **Enable component gateway**.

8. Under **Optional components**, Select **Jupyter Notebook**.

9. Below **Set up cluster** from the list on the left side, click **Configure nodes** _(optional)_.

10. Under **Manager node** change **Series** to **E2** and **Machine Type** to **e2-standard-2** (2 vCPU, 8 GB memory).

11. Under **Worker nodes** change **Series** to **E2** and **Machine Type** to **e2-standard-2** (2 vCPU, 8 GB memory).

12. Click **Create**.

The cluster should start in a few minutes. **Please wait until the Cloud Dataproc Cluster is fully deployed to proceed to the next step**.

## Clone the source repository for the lab

In the Cloud Shell you clone the Git repository for the lab and copy the required notebook files to the Cloud Storage bucket used by Cloud Dataproc as the home directory for Jupyter notebooks.

1. To clone the Git repository for the lab enter the following command in Cloud Shell

`git -C ~ clone https://github.com/GoogleCloudPlatform/training-data-analyst`

2. To locate the default Cloud Storage bucket used by Cloud Dataproc enter the following command in Cloud Shell:

`!export DP_STORAGE="gs://$(gcloud dataproc clusters describe sparktodp --region=us-central1 --format=json | jq -r '.config.configBucket')"`

3. To copy the sample notebooks into the Jupyter working folder enter the following command in Cloud Shell:

`gcloud storage cp ~/training-data-analyst/quests/sparktobq/*.ipynb $DP_STORAGE/notebooks/jupyter`

## Login in to the Jupyter Notbook

As soon as the cluster has fully started up you can connect to the Web interfaces. Click the refresh button to check as it may be deployed fully by the time you reach this stage.

1. On the Dataproc Clusters page wait for the cluster to finish starting and then click the name of your cluster to open the **Cluster details** page.

2. Click **Web Interfaces**.

3. Click the **Jupyter** link to open a new Jupyter tab in your browser.

This opens the Jupyter home page. Here you can see the contents of the /notebooks/jupyter directory in Cloud Storage that now includes the sample Jupyter notebooks used in this lab.

4. Under the **Files** tab, click the **GCS** folder and then click **01_spark.ipynb** notebook to open it.

5. Click **Cell** and then **Run All** to run all of the cells in the notebook.

6. Page back up to the top of the notebook and follow as the notebook completes runs each cell and outputs the results below them.

You can now step down through the cells and examine the code as it is processed so that you can see what the notebook is doing. In particular pay attention to where the data is saved and processed from.

---

# Task 2: Separate compute and storage

## Modify Spark jobs to use Cloud Storage instead of HDFS

Taking this original 'Lift & Shift' sample notebook you will now create a copy that decouples the storage requirements for the job from the compute requirements. In this case, all you have to do is replace the Hadoop file system calls with Cloud Storage calls by replacing `hdfs://` storage references with `gs://` references in the code and adjusting folder names as necessary.

You start by using the cloud shell to place a copy of the source data in a new Cloud Storage bucket.

1. In the Cloud Shell create a new storage bucket for your source data:

`export PROJECT_ID=$(gcloud info --format='value(config.project)')
gcloud storage buckets create gs://$PROJECT_ID`

2. In the Cloud Shell copy the source data into the bucket:

`wget https://storage.googleapis.com/cloud-training/dataengineering/lab_assets/sparklab/kddcup.data_10_percent.gz
gcloud storage cp kddcup.data_10_percent.gz gs://$PROJECT_ID/`

Make sure that the last command completes and the file has been copied to your new storage bucket.

3. Switch back to the _Task1_ Jupyter Notebook tab in your browser.

4. Click **File** and then select **Make a Copy**.

5. When the copy opens, click the _Task1-Copy1_ title and rename it to _Task2_.

6. Open the Jupyter tab for _Task1_.

7. Click **File** and then **Save and checkpoint** to save the notebook.

8. Click **File** and then **Close and Halt** to shutdown the notebook.

- If you are prompted to confirm that you want to close the notebook click **Leave** or **Cancel**.

9. Switch back to the De-couple-storage Jupyter Notebook tab in your browser, if necessary.

You no longer need the cells that download and copy the data onto the cluster's internal HDFS file system so you will remove those first.

To delete a cell, you click in the cell to select it and then click the cut selected cells icon (the scissors) on the notebook toolbar.

10. Delete the first three code cells ( In [1], In [2], and In [3]) so that the notebook now starts with the section **Reading in Data**.

You will now change the code in the first cell ( still called In[4] unless you have rerun the notebook ) that defines the data file source location and reads in the source data. The cell currently contains the following code:

`from pyspark.sql import SparkSession, SQLContext, Row
spark = SparkSession.builder.appName("kdd").getOrCreate()
sc = spark.sparkContext
data_file = "hdfs:///kddcup.data_10_percent.gz"
raw_rdd = sc.textFile(data_file).cache()
raw_rdd.take(5)`

11. Replace the contents of cell In [4] with the following code. The only change here is create a variable to store a Cloud Storage bucket name and then to point the data_file to the bucket we used to store the source data on Cloud Storage:

`from pyspark.sql import SparkSession, SQLContext, Row
gcs_bucket='[Your-Bucket-Name]'
spark = SparkSession.builder.appName("kdd").getOrCreate()
sc = spark.sparkContext
data_file = "gs://"+gcs_bucket+"//kddcup.data_10_percent.gz"
raw_rdd = sc.textFile(data_file).cache()
raw_rdd.take(5)`

12. In the cell you just updated, replace the placeholder [Your-Bucket-Name] with the name of the storage bucket you created in the first step of this section. You created that bucket using the Project ID as the name, which you can copy here from the Qwiklabs lab login information panel on the left of this screen. Replace all of the placeholder text, including the brackets [].

13. Click **Cell** and then **Run All** to run all of the cells in the notebook.

You will see exactly the same output as you did when the file was loaded and run from internal cluster storage. Moving the source data files to Cloud Storage only requires that you repoint your storage source reference from `hdfs://` to `gs://`.

---

# Task 3: Deploy Spark jobs

## Optimize Spark jobs to run on Job specific clusters

You now create a standalone Python file, that can be deployed as a Cloud Dataproc Job, that will perform the same functions as this notebook. To do this you add magic commands to the Python cells in a copy of this notebook to write the cell contents out to a file. You will also add an input parameter handler to set the storage bucket location when the Python script is called to make the code more portable.

1. In the _Task2_ Jupyter Notebook menu, click **File** and select **Make a Copy**.

2. When the copy opens, click the _Task2_ and rename it to _Task3_.

3. Open the Jupyter tab for _Task2_.

3. Click **File** and then **Save and checkpoint** to save the notebook.

4. Click **File** and then **Close and Halt** to shutdown the notebook.

- If you are prompted to confirm that you want to close the notebook click **Leave** or **Cancel**.

6. Switch back to the _Task3_ Jupyter Notebook tab in your browser, if necessary.

7. Click the first cell at the top of the notebook.

8. Click **Insert** and select **Insert Cell Above**.

9. Paste the following library import and parameter handling code into this new first code cell:

`%%writefile spark_analysis.py
import matplotlib
matplotlib.use('agg')
import argparse
parser = argparse.ArgumentParser()
parser.add_argument("--bucket", help="bucket for input and output")
args = parser.parse_args()
BUCKET = args.bucket`

The `%%writefile spark_analysis.py` Jupyter magic command creates a new output file to contain your standalone python script. You will add a variation of this to the remaining cells to append the contents of each cell to the standalone script file.

This code also imports the matplotlib module and explicitly sets the default plotting backend via `matplotlib.use('agg')` so that the plotting code runs outside of a Jupyter notebook.

10. For the remaining cells insert `%%writefile -a spark_analysis.py` at the start of each Python code cell. These are the five cells labelled In [x].

`%%writefile -a spark_analysis.py`

11. Repeat this step, inserting %%writefile -a spark_analysis.py at the start of each code cell until you reach the end.

12. In the last cell, where the Pandas bar chart is plotted, remove the %matplotlib inline magic command.

13. Make sure you have selected the last code cell in the notebook then, in the menu bar, click Insert and select Insert Cell Below.

14. Paste the following code into the new cell:

`%%writefile -a spark_analysis.py
ax[0].get_figure().savefig('report.png');`

15. Add another new cell at the end of the notebook and paste in the following:

`%%writefile -a spark_analysis.py
import google.cloud.storage as gcs
bucket = gcs.Client().get_bucket(BUCKET)
for blob in bucket.list_blobs(prefix='sparktodp/'):
    blob.delete()
bucket.blob('sparktodp/report.png').upload_from_filename('report.png')`


16. Add a new cell at the end of the notebook and paste in the following:

`%%writefile -a spark_analysis.py
connections_by_protocol.write.format("csv").mode("overwrite").save(
    "gs://{}/sparktodp/connections_by_protocol".format(BUCKET))`

`%%writefile -a spark_analysis.py
connections_by_protocol.write.format("csv").mode("overwrite").save(
    "gs://{}/sparktodp/connections_by_protocol".format(BUCKET))`

## Test automation

You now test that the PySpark code runs successfully as a file by calling the local copy from inside the notebook, passing in a parameter to identify the storage bucket you created earlier that stores the input data for this job. The same bucket will be used to store the report data files produced by the script.

1. In the PySpark-analysis-file notebook add a new cell at the end of the notebook and paste in the following:

`BUCKET_list = !gcloud info --format='value(config.project)'
BUCKET=BUCKET_list[0]
print('Writing to {}'.format(BUCKET))
!/opt/conda/miniconda3/bin/python spark_analysis.py --bucket=$BUCKET``

This code assumes that you have followed the earlier instructions and created a Cloud Storage Bucket using your lab Project ID as the Storage Bucket name. If you used a different name modify this code to set the BUCKET variable to the name you used.

2. Add a new cell at the end of the notebook and paste in the following:

`!gcloud storage ls gs://$BUCKET/sparktodp/**`

This lists the script output files that have been saved to your Cloud Storage bucket.

3. To save a copy of the Python file to persistent storage, add a new cell and paste in the following:

`!gcloud storage cp spark_analysis.py gs://$BUCKET/sparktodp/spark_analysis.py`

4. Click Cell and then Run All to run all of the cells in the notebook.

