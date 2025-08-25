<div>

# ConvertingJSONtoExcel

</div>

<div class="page-body">

# 📘 Practical Guide: Converting JSON to Excel with AWS Lambda (Python)

This guide explains how to create an AWS Lambda function in Python that
converts a JSON table into an Excel (`.xlsx`) file and saves it to an
Amazon S3 bucket. It also covers how to package dependencies
using **Lambda Layers** with Docker, and how to configure permissions
and test the function.

------------------------------------------------------------------------

## ✅ Requirements

- **Python:** Version 3.8 or higher

<!-- -->

- **Libraries:** `pandas`, `openpyxl`, `boto3`

<!-- -->

- **Permissions:** The Lambda execution role must have permissions to
  write to S3 (e.g., `s3:PutObject`)

------------------------------------------------------------------------

## 📂 Expected Event Structure

Your Lambda function will expect an event payload in the following
format:

``` code
{
  "bucket": "lambdajsontoexcel",
  "key": "jsontoexcel.xlsx",
  "table": [
    {"name": "John", "age": 30},
    {"name": "Anna", "age": 25}
  ]
}
```

- **bucket** → Name of the S3 bucket

<!-- -->

- **key** → Name of the output Excel file

<!-- -->

- **table** → JSON array of objects to be converted into Excel

------------------------------------------------------------------------

## 🐍 Lambda Function Code

``` code
import json
import boto3
import pandas as pd
from io import BytesIO

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Extract input parameters
    table_data = event.get("table")
    bucket = event.get("bucket")
    key = event.get("key", "output.xlsx")

    # Validate input
    if not table_data or not bucket:
        return {
            "statusCode": 400,
            "body": json.dumps("Missing parameters: 'table' or 'bucket'")
        }

    # Convert JSON to Pandas DataFrame
    df = pd.DataFrame(table_data)

    # Write DataFrame to Excel in memory
    with BytesIO() as output:
        with pd.ExcelWriter(output, engine="openpyxl") as writer:
            df.to_excel(writer, index=False)
        output.seek(0)

        # Upload Excel file to S3
        s3.put_object(Bucket=bucket, Key=key, Body=output.getvalue())

    return {
        "statusCode": 200,
        "body": json.dumps(f"File saved at s3://{bucket}/{key}")
    }
```

------------------------------------------------------------------------

## ⚙️ Deployment Options

### 0. Create repository

``` code
git clone https://github.com/zimlama/jsontoexcel.git
cd jsontoexcel
touch README.md
echo "# jsontoexcel" >> README.md
git add README.md
git commit -m "first commit"
git branch -M main
git push -u origin main
```

### 0.1 Create bucket

Select `Create bucket`

<figure id="25a91892-6feb-80ab-b4fc-ce01f17c79a9" class="image">
<a
href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_8.42.52_AM.png"><img
src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_8.42.52_AM.png"
style="width:2142px" /></a>
</figure>

Create `Bucket name`

<figure id="25a91892-6feb-80ca-bbfe-eb095f84d37f" class="image">
<a
href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_8.44.53_AM.png"><img
src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_8.44.53_AM.png"
style="width:2150px" /></a>
</figure>

Select `Create bucket`

<figure id="25a91892-6feb-808b-aab2-f8da376dc15e" class="image">
<a
href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_8.45.53_AM.png"><img
src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_8.45.53_AM.png"
style="width:2560px" /></a>
</figure>

Bucket created

<figure id="25a91892-6feb-807b-9d51-fc30eddc05cd" class="image">
<a
href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_8.47.46_AM.png"><img
src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_8.47.46_AM.png"
style="width:2134px" /></a>
</figure>

### 1. Package Dependencies with Docker (Recommended)

Because AWS Lambda has limited built-in libraries, you need to
include **pandas** and **openpyxl**. Instead of uploading them every
time, you can build a **Lambda Layer** with Docker.

### Step 1: Create the folder structure

``` code
mkdir -p lambda_layer/python
cd lambda_layer/python
```

AWS requires dependencies to be stored in a subdirectory
called `python`.

### Step 2: Install dependencies inside Docker (Python 3.8)

``` code
docker run -v "$PWD":/var/task lambci/lambda:build-python3.8 \
    pip install pandas openpyxl -t /var/task/python
```

<figure id="25a91892-6feb-801c-b2cf-ea3e7dc65084" class="image">
<a
href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.24.19_AM.png"><img
src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.24.19_AM.png"
style="width:2058px" /></a>
</figure>

This ensures libraries are compiled for the same environment as AWS
Lambda.

### Step 3: Create a ZIP file for the layer

``` code
cd ..
zip -r lambda_layer_pandas_openpyxl.zip python
```

<figure id="25a91892-6feb-8083-9c8a-e3f859bf3c89" class="image">
<a
href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.30.23_AM.png"><img
src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.30.23_AM.png"
style="width:709.9739990234375px" /></a>
</figure>

### Step 4: Upload the ZIP as a Lambda Layer

1.  Go to **AWS Lambda → Layers**

<!-- -->

2.  Click **Create Layer**
    - Go to AWS Lambda → **`Layers`**. select `Create a Layer`
      <figure id="25a91892-6feb-80fc-9df1-e8237f4fcaad" class="image">
      <a
      href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.40.08_AM.png"><img
      src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.40.08_AM.png"
      style="width:2170px" /></a>
      </figure>

    <!-- -->

    - Click **`Create layer`**.

    <!-- -->

    - Assign a name, select Python 3.8, and
      upload `lambda_layer_pandas_openpyxl.zip`.
      <figure id="25a91892-6feb-8016-bb35-c627fa65bec9" class="image">
      <a
      href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.42.05_AM.png"><img
      src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.42.05_AM.png"
      style="width:2160px" /></a>
      </figure>

    <!-- -->

    - Select `Compatible architectures` and `Compatible runtimes`
      <figure id="25a91892-6feb-80b3-9eb0-f6c150ddb97b" class="image">
      <a
      href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.43.32_AM.png"><img
      src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.43.32_AM.png"
      style="width:2160px" /></a>
      </figure>

    <!-- -->

    - `Create` the layer.
      <figure id="25a91892-6feb-8064-a04c-fb0163681910" class="image">
      <a
      href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.48.00_AM.png"><img
      src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.48.00_AM.png"
      style="width:2160px" /></a>
      </figure>

<!-- -->

3.  Provide a name, select **Python 3.8**, and upload the ZIP file

<!-- -->

4.  Assign compatible architectures/runtimes

<!-- -->

5.  Create the layer

### Step 5: Attach the Layer to your Function

1.  Open your Lambda function
    - Go to your Lambda function.
      <figure id="25a91892-6feb-8027-8e01-df88897ba66f" class="image">
      <a
      href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.49.58_AM.png"><img
      src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.49.58_AM.png"
      style="width:2162px" /></a>
      </figure>

    <!-- -->

    - Under **Layers**, click **`Add a layer`**.

    <!-- -->

    - Select the layer you just created. **`Custom layers`** **and**
      **`Version`**
      <figure id="25a91892-6feb-807c-ac3c-dfcb874c8d22" class="image">
      <a
      href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.51.42_AM.png"><img
      src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.51.42_AM.png"
      style="width:2158px" /></a>
      </figure>

    <!-- -->

    - Save with `Add`.
      <figure id="25a91892-6feb-80e4-8930-c8894bda47a1" class="image">
      <a
      href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.54.00_AM.png"><img
      src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.54.00_AM.png"
      style="width:2224px" /></a>
      </figure>

<!-- -->

2.  Under **Layers**, click **Add a layer**

<!-- -->

3.  Choose your custom layer

<!-- -->

4.  Save

------------------------------------------------------------------------

### 2. Create the Lambda Function

1.  Navigate to **AWS Lambda → Create Function**

    Select `Create a function`

    <figure id="25a91892-6feb-80c9-af2b-f619a5dde0b0" class="image">
    <a
    href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_8.53.29_AM.png"><img
    src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_8.53.29_AM.png"
    style="width:2154px" /></a>
    </figure>

    Create `Function Name` and select language `Runtime`

    <figure id="25a91892-6feb-8044-bc12-e1e439ed6b64" class="image">
    <a
    href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_8.55.06_AM.png"><img
    src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_8.55.06_AM.png"
    style="width:2138px" /></a>
    </figure>

    Select `Architecture` and seleect `Create function`

    <figure id="25a91892-6feb-80f2-8d21-ef61092795a5" class="image">
    <a
    href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.17.22_AM.png"><img
    src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.17.22_AM.png"
    style="width:2160px" /></a>
    </figure>

    <figure id="25a91892-6feb-80d0-9def-c32789179758" class="image">
    <a
    href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.16.00_AM.png"><img
    src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.16.00_AM.png"
    style="width:2560px" /></a>
    </figure>

<!-- -->

2.  Choose a function name (e.g., `jsontoexcel`)

<!-- -->

3.  Select **Python 3.8** as the runtime

<!-- -->

4.  Select the architecture (x86_64 or arm64)

<!-- -->

5.  Click **Create Function**

<!-- -->

6.  Paste the Lambda code into `lambda_function.py`

    <figure id="25a91892-6feb-8080-ba79-fe54633c6e19" class="image">
    <a
    href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.59.39_AM.png"><img
    src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_9.59.39_AM.png"
    style="width:2224px" /></a>
    </figure>

    ``` code
    import json
    import boto3
    import pandas as pd
    from io import BytesIO

    s3 = boto3.client('s3')

    def lambda_handler(event, context):
        # Get data from event
        table_data = event.get("table")
        bucket = event.get("bucket")
        key = event.get("key", "output.xlsx")

        if not table_data or not bucket:
            return {
                "statusCode": 400,
                "body": json.dumps("Missing parameters: 'table' or 'bucket'")
            }

        # Convert to DataFrame
        df = pd.DataFrame(table_data)

        # Save to Excel in memory
        with BytesIO() as output:
            with pd.ExcelWriter(output, engine='openpyxl') as writer:
                df.to_excel(writer, index=False)
            output.seek(0)

            # Upload to S3
            s3.put_object(Bucket=bucket, Key=key, Body=output.getvalue())

        return {
            "statusCode": 200,
            "body": json.dumps(f"File saved at s3://{bucket}/{key}")
        }
    ```

    With the code paste `Deploy`

    <figure id="25a91892-6feb-8059-afa3-d3d6b5b847c2" class="image">
    <a
    href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.02.01_AM.png"><img
    src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.02.01_AM.png"
    style="width:2074px" /></a>
    </figure>

------------------------------------------------------------------------

## 🔒 Configure Permissions

1.  Open the Lambda function

<!-- -->

2.  Go to **Configuration → Permissions**

<!-- -->

3.  Click the execution role name (e.g., `jsontoexcel-role-xxxxxx`)

<!-- -->

4.  Add the required S3 policy:

    - For practice, you can attach **AmazonS3FullAccess**

    <!-- -->

    - ⚠️ For production, create a **restrictive custom policy** that
      only allows writing to the specific bucket

    👉 To add the permission in S3 bucket select `Configuration` and
    `Permission` , in `Role name` click in `jsontoexcel-role-odqxkhah`

    <figure id="25a91892-6feb-8090-86c9-cfebb2faad94" class="image">
    <a
    href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.06.08_AM.png"><img
    src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.06.08_AM.png"
    style="width:2218px" /></a>
    </figure>

    In the new link in `Permission` select `Add permissions` with de
    option `Attach policies`

    <figure id="25a91892-6feb-804e-9855-c5c314e899d0" class="image">
    <a
    href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.10.30_AM.png"><img
    src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.10.30_AM.png"
    style="width:2560px" /></a>
    </figure>

    In `Other permissions policies` write in the search S3 and select de
    option `AmazonS3FullAccess` (For the purpose of this practice only,
    a restrictive usage policy should be created for security reasons.)

    <figure id="25a91892-6feb-8073-a458-e03ea2c30b3d" class="image">
    <a
    href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.15.56_AM.png"><img
    src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.15.56_AM.png"
    style="width:2560px" /></a>
    </figure>

    Click en `Add permissions`

    <figure id="25a91892-6feb-80ff-bc44-e8859fafd8c1" class="image">
    <a
    href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.19.30_AM.png"><img
    src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.19.30_AM.png"
    style="width:2560px" /></a>
    </figure>

------------------------------------------------------------------------

## 🧪 Testing the Lambda

1.  Go to your Lambda function

<!-- -->

2.  Click **Test → Configure test event**

<!-- -->

3.  Use the JSON event payload shown earlier

<!-- -->

4.  Save and click **Test**

<!-- -->

5.  Check the **S3 bucket** for the generated `jsontoexcel.xlsx` file

<!-- -->

6.  Download it to verify the contents

Go to lambda select the function **`jsontoexcel`, click in** **`test`**
**and** **`Test event action`** **create** **`Event name`**

<figure id="25a91892-6feb-8060-a881-c35ed7b768bb" class="image">
<a
href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.25.45_AM.png"><img
src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.25.45_AM.png"
style="width:2560px" /></a>
</figure>

In `Event JSON` paste the code

``` code
{
  "bucket": "lambdajsontoexcel",
  "key": "jsontoexcel.xlsx",
  "table": [
    {"name": "John", "age": 30},
    {"name": "Anna", "age": 25}
  ]
}
```

<figure id="25a91892-6feb-8084-b55f-ddc61ddaf7aa" class="image">
<a
href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.29.04_AM.png"><img
src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.29.04_AM.png"
style="width:2222px" /></a>
</figure>

And select `Save`

<figure id="25a91892-6feb-808e-91c4-dac9514d952d" class="image">
<a
href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.30.49_AM.png"><img
src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.30.49_AM.png"
style="width:2224px" /></a>
</figure>

Make a test with click in Test

<figure id="25a91892-6feb-8047-9cef-d4308dfe2ec3" class="image">
<a
href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.33.04_AM.png"><img
src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.33.04_AM.png"
style="width:2264px" /></a>
</figure>

To verify go to `S3` and select the bucket `lambdajsontoexcel`

<figure id="25a91892-6feb-809e-99ba-dee0a12147b2" class="image">
<a
href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.35.12_AM.png"><img
src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.35.12_AM.png"
style="width:709.9739990234375px" /></a>
</figure>

To donwload slect the file `jsontoexcel.xlsx` and click in `Donwload`

<figure id="25a91892-6feb-80fd-993f-d0fbf340288b" class="image">
<a
href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.36.45_AM.png"><img
src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.36.45_AM.png"
style="width:2560px" /></a>
</figure>

<figure id="25a91892-6feb-800e-9048-d0263376eb35" class="image">
<a
href="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.38.16_AM.png"><img
src="ConvertingJSONtoExcel%2025a918926feb80aca0d8ff7e5175d3a0/Screenshot_2025-08-25_at_10.38.16_AM.png"
style="width:325.9895935058594px" /></a>
</figure>

------------------------------------------------------------------------

## 🎯 Benefits of Using Layers

- **Reusable**: One layer can be shared across multiple Lambda functions

<!-- -->

- **Lightweight**: Keeps your Lambda deployment package smaller

<!-- -->

- **Efficient**: No need to re-upload dependencies every time you change
  code

------------------------------------------------------------------------

✅ With this setup, you now have a fully working **serverless
function** that takes JSON input, converts it into Excel, and saves it
to S3 — all automated with AWS Lambda.

------------------------------------------------------------------------

</div>

<span class="sans" style="font-size:14px;padding-top:2em"></span>
