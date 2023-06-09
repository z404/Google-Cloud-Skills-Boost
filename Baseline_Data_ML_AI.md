# Perform Foundational Data, ML, and AI Tasks in Google Cloud: Challenge Lab


### [GSP323](https://www.cloudskillsboost.google/focuses/11044?locale=en&parent=catalog)

![](https://cdn.qwiklabs.com/GMOHykaqmlTHiqEeQXTySaMXYPHeIvaqa2qHEzw6Occ%3D)


## Challenge scenario

As a junior data engineer in Jooli Inc. and recently trained with Google Cloud and a number of data services you have been asked to demonstrate your newly learned skills. The team has asked you to complete the following tasks.

You are expected to have the skills and knowledge for these tasks so don’t expect step-by-step guides.


## Task 1: Run a simple Dataflow job

1. Create a BigQuery dataset called `BIGQUERY_DATASET_NAME`.

    - In the **Cloud Console**, click the **Navigation menu** > **BigQuery**. Then click **Done**.
    - Click on the three dots next to your project name under **Explorer** section, then click **Create dataset**.
    - Input `BIGQUERY_DATASET_NAME` as your Dataset ID, then click **Create dataset**.

2. Create a BigQuery table.

    - Run the following command to view the schema:

        ```
        gsutil cp gs://cloud-training/gsp323/lab.schema .

        cat lab.schema
        ```

    - Click on the three dots next to created dataset and select **Open**. Then select **Create Table**.
    - Set the following values, leave all other values at their defaults:
    
        | Property | Value (type value or select option as specified) |
        | --- | --- |
        | Create table from | Google Cloud Storage |
        | Select file from GCS bucket | `gs://cloud-training/gsp323/lab.csv` |
        | Table name | customer |
        | Schema | *Enable* edit as text, then *copy* the schema |

    - Click **Create Table**.

3. Create a Cloud Storage Bucket called `CLOUD_STORAGE_BUCKET_NAME`.

    - In the **Cloud Console**, click the **Navigation menu** > **Cloud Storage** > **Browser**.
    - Click **Create Bucket**. 
    - Input `CLOUD_STORAGE_BUCKET_NAME`, then click **Create**.

4. Create a Dataflow Job.

    - In the **Cloud Console**, click the **Navigation menu** > **Dataflow** > **Jobs**.
    - Click **Create job from template**.
    - Enter **Job name** for your Cloud Dataflow job.
    - Under **Dataflow template**, select the **Text Files on Cloud Storage to BigQuery** template under **Process Data in Bulk (batch)**.
    - Set the following values, leave all other values at their defaults:

        | Property | Value (type value or select option as specified) |
        | --- | --- |
        | JavaScript UDF path in Cloud Storage | `gs://cloud-training/gsp323/lab.js` |
        | JSON path | `gs://cloud-training/gsp323/lab.schema` |
        | JavaScript UDF name | transform |
        | BigQuery output table | `OUTPUT_TABLE_NAME` |
        | Cloud Storage input path | `gs://cloud-training/gsp323/lab.csv` |
        | Temporary BigQuery directory | `TEMPORARY_BIGQUERY_DIRECTORY` |
        | Temporary location | `TEMPORARY_LOCATION` |

    - Click **Run job**.


## Task 2: Run a simple Dataproc job

1. Create a Cluster.

    - In the **Cloud Console**, click the **Navigation menu** > **Dataproc** > **Cluster**.
    - Click **Create cluster**.
    - Leave all settings by default, then click **Create**.

2. Submit a Job.

    - Select the created cluster. In the **Cluster details**, select **VM Instance** tab. Then click the SSH button.
    - Run the following command:

        ```
        hdfs dfs -cp gs://cloud-training/gsp323/data.txt /data.txt
        ```

    - Exit the SSH.
    - In the **Cluster details**, click **Submit job**.
    - Configure the following settings:

        | Property | Value (type value or select option as specified) |
        | --- | --- |
        | Region | `REGION` |
        | Job type | Spark |
        | Main class or jar | org.apache.spark.examples.SparkPageRank |
        | Jar files | file:///usr/lib/spark/examples/jars/spark-examples.jar |
        | Arguments | /data.txt |
        | Max restarts per hour | 1 |

    - Click **Submit**.


## Task 3: Run a simple Dataprep job

1. Initialize Cloud Dataprep.

    - In the **Cloud Console**, click the **Navigation menu** > **Dataprep**.
    - Sign in to Cloud Dataprep by Trifacta.

2. Import datasets.

    - In the left menu pane, select **Cloud Storage**, then click on the pencil to edit the file path.
    - Type `gs://cloud-training/gsp323/runs.csv` in the **Choose a file or folder path** text box, then click **Go**.
    - Dataset is listed in the right pane. Click **Continue**.

3. Prep the `run.csv` file.

    `gs://cloud-training/gsp323/runs.csv` structure:

    | runid | userid | labid | lab_title | start | end | time | score | state |
    | --- | --- | --- | --- | --- | --- | --- | --- | --- |
    | 5556 | 545 | 122 | Lab 122 | 2020-04-09 16:18:19 | 2020-04-09 17:10:11 | 3112 | 61.25 | SUCCESS |
    | 5557 | 116 | 165 | Lab 165 | 2020-04-09 16:44:45 | 2020-04-09 18:13:58 | 5353 | 60.5 | SUCCESS |
    | 5558 | 969 | 31 | Lab 31 | 2020-04-09 17:59:01 | 2020-04-09 18:02:09 | 188 | 0 | FAILURE |

    Perform the following transforms to ensure the data is in the right state:

    - Label columns with the names above.
    - Remove all rows with the state of **FAILURE**.
    - Remove all rows with 0 or 0.0 as a score (Use the regex pattern `/(^0$|^0\.0$)/`).

    Click **Run**. In the **Run job**, select **Dataflow** as your Running Environment. Then click **Run**.

## Task 4: AI

Complete one of the following tasks below.

1. Use Google Cloud Speech API to analyze the audio file `gs://cloud-training/gsp323/task4.flac`. Once you have analyzed the file you can upload the resulting analysis to `CLOUD_SPEECH_LOCATION`.

2. Use the Cloud Natural Language API to analyze the sentence from text about Odin. The text you need to analyze is "Old Norse texts portray Odin as one-eyed and long-bearded, frequently wielding a spear named Gungnir and wearing a cloak and a broad hat." Once you have analyzed the text you can upload the resulting analysis to `CLOUD_NATURAL_LANGUAGE_LOCATION`.

    - Create a new service account to access the Natural Language API.

        ```
        gcloud iam service-accounts create my-natlang-sa \
            --display-name "my natural language service account"
        ```

    - Create credentials to log in as your new service account and save it as a JSON file "~/key.json".

        ```
        gcloud iam service-accounts keys create ~/key.json \
            --iam-account my-natlang-sa@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com
        ```

    - Set to the full path of the credentials JSON file you created into environtment variable.

        ```
        export GOOGLE_APPLICATION_CREDENTIALS="/home/$USER/key.json"
        ```

    - Allow gcloud to use service account credentials to make requests.

        ```
        gcloud auth activate-service-account my-natlang-sa@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com --key-file=$GOOGLE_APPLICATION_CREDENTIALS
        ```

    - Make an Entity Analysis Request.

        ```
        gcloud ml language analyze-entities --content="Old Norse texts portray Odin as one-eyed and long-bearded, frequently wielding a spear named Gungnir and wearing a cloak and a broad hat." > result.json
        ```

    - Obtains access credentials for your user account via a web-based authorization flow.

        ```
        gcloud auth login
        ```

    - Upload the resulting analysis to `CLOUD_NATURAL_LANGUAGE_LOCATION`.

        ```
        gsutil cp result.json [CLOUD_NATURAL_LANGUAGE_LOCATION]
        ```

3. Use Google Video Intelligence and detect all text on the video `gs://spls/gsp154/video/train.mp4`. Once you have completed the processing of the video, pipe the output into a file and upload to `VIDEO_INTELLIGENCE_LOCATION`. Ensure the progress of the operation is complete and the service account you're uploading the output with has the Storage Object Admin role.
