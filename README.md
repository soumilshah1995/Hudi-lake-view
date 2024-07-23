# Hudi-lake-view
Hudi-lake-view
![Screenshot 2024-07-23 at 4 01 25 PM](https://github.com/user-attachments/assets/b3cd364a-1ff3-47e2-83f1-b41c6148b3d6)


# Steps 

## Step 1: Signup for LakeView
* https://cloud.onehouse.ai/lakeview/signup



## Step 2: Download the jar and upload it to S3
* https://drive.google.com/drive/folders/1ULWGZ9Tv7nY1GAit5H4euF_e8D6ajMb8

## Step 3: Create Glue job 
```
import sys
import json
import pyspark
import pyspark.sql.types as T
from awsglue.job import Job
from awsglue.transforms import *
from awsglue.context import GlueContext
from pyspark.context import SparkContext
from awsglue.utils import getResolvedOptions


def get_args():
    """
    Retrieve job parameters ref: https://docs.aws.amazon.com/glue/latest/dg/aws-glue-api-crawler-pyspark-extensions-get-resolved-options.html
    """
    args_list = [
        'API_KEY',
        'API_SECRET',
        'USER_ID',
        'REGION',
        'BASE_PATH_1',
        'PROJECT_ID',
        'DATABASE_NAME',
        'LAKE_NAME'
    ]
    return getResolvedOptions(sys.argv, args_list)


def create_extractor_input(args):
    """
    Construct the input to be passed to the metadata extractor.
    we pass the option -c as the config is being passed inline

    Args:
        args (dict): Dictionary of job parameters.

    Returns:
        str: Formatted configuration string.
    """
    return (
        "[\"-c\", \"{{version: V1, onehouseClientConfig: {{projectId: {project_id}, apiKey: {api_key}, apiSecret: {api_secret}, userId: {user_id}}}, fileSystemConfiguration: {{s3Config: {{region: {region}}}}}, metadataExtractorConfig: {{jobRunMode: ONCE, parserConfig: [{{lake: {lake_name}, databases: [{{name: {database}, basePaths: [{base_path_1}]}}]}}]}}}}\"]"
    ).format(
        project_id=args['PROJECT_ID'],
        api_key=args['API_KEY'],
        api_secret=args['API_SECRET'],
        user_id=args['USER_ID'],
        region=args['REGION'],
        lake_name=args['LAKE_NAME'],
        database=args['DATABASE_NAME'],
        base_path_1=args['BASE_PATH_1']
    )


def main():
    args = get_args()
    input_string = create_extractor_input(args)

    conf = pyspark.SparkConf()
    glueContext = GlueContext(SparkContext.getOrCreate(conf=conf))

    spark_session = glueContext.spark_session
    spark_session.udf.registerJavaFunction(
        name="metadata_extractor_wrapper",
        javaClassName="com.onehouse.GlueWrapperMain",
        returnType=T.StringType()
    )

    spark_session.sql(f"SELECT metadata_extractor_wrapper('{input_string}') as answer").show()


if __name__ == "__main__":
    main()

```

## Add Conf
![image](https://github.com/user-attachments/assets/15404a4f-2638-4cb4-8de1-dd355f3483e9)
![image](https://github.com/user-attachments/assets/a6d2a007-c69f-4f1a-a289-c7e9cb9ad93c)


Run the job 

![image](https://github.com/user-attachments/assets/d5739001-84b3-4ee2-9105-3689d23de8d0)


