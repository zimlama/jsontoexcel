<div>

# **jsontoexcel wit aws lambda**

<table class="properties">
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<tbody>
<tr class="property-row property-row-checkbox">
<th><span class="icon property-icon"></span>
<div data-testid="/icons/checkmark-square_gray.svg"
style="width:14px;height:14px;flex-shrink:0;transform:scale(1.2);mask:url(/icons/checkmark-square_gray.svg?mode=light) no-repeat center;-webkit-mask:url(/icons/checkmark-square_gray.svg?mode=light) no-repeat center;background-color:rgba(71, 70, 68, 0.6);fill:rgba(71, 70, 68, 0.6)">
&#10;</div>
Estado</th>
<td><div class="checkbox checkbox-off">
&#10;</div></td>
</tr>
<tr class="property-row property-row-status">
<th><span class="icon property-icon"></span>
<div data-testid="/icons/burst_gray.svg"
style="width:14px;height:14px;flex-shrink:0;transform:scale(1.2);mask:url(/icons/burst_gray.svg?mode=light) no-repeat center;-webkit-mask:url(/icons/burst_gray.svg?mode=light) no-repeat center;background-color:rgba(71, 70, 68, 0.6);fill:rgba(71, 70, 68, 0.6)">
&#10;</div>
Status</th>
<td><span class="status-value"></span>
<div class="status-dot">
&#10;</div>
Not started</td>
</tr>
<tr class="property-row property-row-multi_select">
<th><span class="icon property-icon"></span>
<div data-testid="/icons/list_gray.svg"
style="width:14px;height:14px;flex-shrink:0;transform:scale(1.2);mask:url(/icons/list_gray.svg?mode=light) no-repeat center;-webkit-mask:url(/icons/list_gray.svg?mode=light) no-repeat center;background-color:rgba(71, 70, 68, 0.6);fill:rgba(71, 70, 68, 0.6)">
&#10;</div>
Tags</th>
<td></td>
</tr>
</tbody>
</table>

</div>

<div class="page-body">

# **json to excel**

### Create repository

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

------------------------------------------------------------------------

# How to Create a Python Lambda Function with Libraries

Here’s an example of an AWS Lambda function in Python that takes a table
(for example, sent as JSON from an event) and converts it into an Excel
(.xlsx) file, storing it in an Amazon S3 bucket.

✅ **Requirements**:

- Python 3.8 or higher

<!-- -->

- Libraries: pandas, openpyxl, boto3

<!-- -->

- Permissions: The Lambda must have permissions to write to S3

📁 **Expected event structure**:

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

🐍 **Lambda Code**

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

🛠 **How to Deploy**:

- Upload your dependencies (pandas, openpyxl) as a layer or inside the
  ZIP package.

<!-- -->

- Assign the proper permissions to the Lambda role
  (e.g., `s3:PutObject`).

<!-- -->

- Test the function with an event like the one shown above.

### Create bucket:

Select `Create bucket`

<figure id="25a91892-6feb-8038-ab3c-c7b1a1bf4281" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_8.42.52_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_8.42.52_AM.png"
style="width:2142px" /></a>
</figure>

Create `Bucket name`

<figure id="25a91892-6feb-805c-b656-d8b34b1beb2d" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_8.44.53_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_8.44.53_AM.png"
style="width:2150px" /></a>
</figure>

Select `Create bucket`

<figure id="25a91892-6feb-8016-bcfa-ef4c6d93ccf1" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_8.45.53_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_8.45.53_AM.png"
style="width:2560px" /></a>
</figure>

Bucket created

<figure id="25a91892-6feb-8045-9e77-c9935176d857" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_8.47.46_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_8.47.46_AM.png"
style="width:2134px" /></a>
</figure>

------------------------------------------------------------------------

## ✅ Option 1: Using Docker to Create an AWS Lambda Layer

With Docker you can create an AWS Lambda Layer that contains the pandas
and openpyxl libraries, so you can reuse it in multiple Lambda functions
without repackaging everything each time.

### Why use Layers?

- Avoid uploading pandas, openpyxl every time you change your code.

<!-- -->

- Keep the Lambda package lighter.

<!-- -->

- Share the layer across multiple functions.

### Create Lambda:

Select `Create a function`

<figure id="25a91892-6feb-8079-bcf5-c7f88c4caea1" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_8.53.29_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_8.53.29_AM.png"
style="width:2154px" /></a>
</figure>

Create `Function Name` and select language `Runtime`

<figure id="25a91892-6feb-80fe-aad1-ce636b8b1b98" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_8.55.06_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_8.55.06_AM.png"
style="width:2138px" /></a>
</figure>

Select `Architecture` and seleect `Create function`

<figure id="25a91892-6feb-8010-8bac-d9060175543f" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.17.22_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.17.22_AM.png"
style="width:2160px" /></a>
</figure>

<figure id="25a91892-6feb-80f5-b80a-f4f31cb547ee" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.16.00_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.16.00_AM.png"
style="width:2560px" /></a>
</figure>

------------------------------------------------------------------------

🐳 **Steps to Create a Layer Using Docker (Python 3.8)**

📁 **1. Create folder structure**

``` code
mkdir -p lambda_layer/python
cd lambda_layer/python
```

AWS expects dependencies inside a subdirectory named `python`.

🐳 **2. Install pandas and openpyxl using Docker for Python 3.8**

From the `lambda_layer` directory, run:

``` code
docker run -v "$PWD/python":/var/task lambci/lambda:build-python3.8 pip install pandas openpyxl -t /var/task
```

<figure id="25a91892-6feb-809d-b738-f62cf17360cc" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.24.19_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.24.19_AM.png"
style="width:2058px" /></a>
</figure>

This installs the libraries inside `lambda_layer/python`, 100%
compatible with AWS Lambda Python 3.8.

📦 **3. Create the ZIP for the layer**

``` code
zip -r lambda_layer_pandas_openpyxl.zip python
```

<figure id="25a91892-6feb-8096-a106-cb37fda3a250" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.30.23_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.30.23_AM.png"
style="width:709.9739990234375px" /></a>
</figure>

☁️ **4. Upload the ZIP as a Lambda Layer**

- Go to AWS Lambda → **`Layers`**. select `Create a Layer`
  <figure id="25a91892-6feb-80d4-b8c6-e7313614cdfd" class="image">
  <a
  href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.40.08_AM.png"><img
  src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.40.08_AM.png"
  style="width:2170px" /></a>
  </figure>

<!-- -->

- Click **`Create layer`**.

<!-- -->

- Assign a name, select Python 3.8, and
  upload `lambda_layer_pandas_openpyxl.zip`.
  <figure id="25a91892-6feb-80c5-820f-f3712b7cbf7f" class="image">
  <a
  href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.42.05_AM.png"><img
  src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.42.05_AM.png"
  style="width:2160px" /></a>
  </figure>

<!-- -->

- Select `Compatible architectures` and `Compatible runtimes`
  <figure id="25a91892-6feb-805f-a5c5-c4401e8032c5" class="image">
  <a
  href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.43.32_AM.png"><img
  src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.43.32_AM.png"
  style="width:2160px" /></a>
  </figure>

<!-- -->

- `Create` the layer.
  <figure id="25a91892-6feb-80f8-a16b-d9af1d258a83" class="image">
  <a
  href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.48.00_AM.png"><img
  src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.48.00_AM.png"
  style="width:2160px" /></a>
  </figure>

🧩 **5. Add the Layer to your Lambda function**

- Go to your Lambda function.
  <figure id="25a91892-6feb-80e8-98b4-dc91991eb171" class="image">
  <a
  href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.49.58_AM.png"><img
  src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.49.58_AM.png"
  style="width:2162px" /></a>
  </figure>

<!-- -->

- Under **Layers**, click **`Add a layer`**.

<!-- -->

- Select the layer you just created. **`Custom layers`** **and**
  **`Version`**
  <figure id="25a91892-6feb-801b-b12b-f1137731264f" class="image">
  <a
  href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.51.42_AM.png"><img
  src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.51.42_AM.png"
  style="width:2158px" /></a>
  </figure>

<!-- -->

- Save with `Add`.
  <figure id="25a91892-6feb-80e0-a9d8-d9b223090e87" class="image">
  <a
  href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.54.00_AM.png"><img
  src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.54.00_AM.png"
  style="width:2224px" /></a>
  </figure>

📜 **Then in your Lambda code** you can directly paste de code en
lambda_function.py use:

<figure id="25a91892-6feb-80e0-b521-c2541ba212e3" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.59.39_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_9.59.39_AM.png"
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

<figure id="25a91892-6feb-80b6-904e-e1cdabf9fe27" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.02.01_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.02.01_AM.png"
style="width:2074px" /></a>
</figure>

------------------------------------------------------------------------

👉 To add the permission in S3 bucket select `Configuration` and
`Permission` , in `Role name` click in `jsontoexcel-role-odqxkhah`

<figure id="25a91892-6feb-80c1-86e0-f4cd0f3a2e0d" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.06.08_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.06.08_AM.png"
style="width:2218px" /></a>
</figure>

In the new link in `Permission` select `Add permissions` with de option
`Attach policies`

<figure id="25a91892-6feb-80f9-82e5-ecd3c7b1c1c2" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.10.30_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.10.30_AM.png"
style="width:2560px" /></a>
</figure>

In `Other permissions policies` write in the search S3 and select de
option `AmazonS3FullAccess` (For the purpose of this practice only, a
restrictive usage policy should be created for security reasons.)

<figure id="25a91892-6feb-809c-b8b3-f51261ce09b2" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.15.56_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.15.56_AM.png"
style="width:2560px" /></a>
</figure>

Click en `Add permissions`

<figure id="25a91892-6feb-8035-8bf8-eed8926e6b17" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.19.30_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.19.30_AM.png"
style="width:2560px" /></a>
</figure>

## To test the lambda function

Go to lambda select the function **`jsontoexcel`, click in** **`test`**
**and** **`Test event action`** **create** **`Event name`**

<figure id="25a91892-6feb-8059-896a-d22cbf9f3771" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.25.45_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.25.45_AM.png"
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

<figure id="25a91892-6feb-8024-9fff-f3847a1250bd" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.29.04_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.29.04_AM.png"
style="width:2222px" /></a>
</figure>

And select `Save`

<figure id="25a91892-6feb-80b8-914c-c274e9033553" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.30.49_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.30.49_AM.png"
style="width:2224px" /></a>
</figure>

Make a test with click in Test

<figure id="25a91892-6feb-8063-a724-e02c4027a992" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.33.04_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.33.04_AM.png"
style="width:2264px" /></a>
</figure>

To verify go to `S3` and select the bucket `lambdajsontoexcel`

<figure id="25a91892-6feb-8031-b651-d71316345b0c" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.35.12_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.35.12_AM.png"
style="width:709.9739990234375px" /></a>
</figure>

To donwload slect the file `jsontoexcel.xlsx` and click in `Donwload`

<figure id="25a91892-6feb-80d3-a2cd-fa9aef3d603a" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.36.45_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.36.45_AM.png"
style="width:2560px" /></a>
</figure>

<figure id="25a91892-6feb-80e3-ada8-dffdb520b57d" class="image">
<a
href="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.38.16_AM.png"><img
src="jsontoexcel%20wit%20aws%20lambda%2025a918926feb808ab56ef7e71904cdff/Screenshot_2025-08-25_at_10.38.16_AM.png"
style="width:325.9895935058594px" /></a>
</figure>

</div>

<span class="sans" style="font-size:14px;padding-top:2em"></span>
