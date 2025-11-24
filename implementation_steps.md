
This guide provides the step-by-step implementation for deploying a modern, serverless, three-tier application on AWS
* **Frontend (Presentation Tier):** AWS Amplify
* **Backend (Application Tier):** AWS Lambda (Python)
* **API Layer:** Amazon API Gateway
* **Database (Data Tier):** Amazon DynamoDB (NoSQL)


---

## 1. Deploy the Frontend using AWS Amplify

The frontend of the application is a web application deployed via AWS Amplify.

1.  **Navigate to Amplify:** Go to the AWS Console and search for **AWS Amplify**.
2.  **Start Deployment:** Click on **Deploy an app** and then select **GitHub** (or your chosen repository provider).
3.  **Connect Repository:** Authenticate with your GitHub account and select the project repository (e.g., `cloud-serverless-project`) and the branch you wish to deploy.
4.  **Specify Code Path:** Check the option to "Choose an existing repository and branch for your app's frontend" and specify the subdirectory for the frontend code, which is **`frontend`**.
5.  **Deploy:** Click **Next** and then **Save and deploy**.
    * Amplify will automatically build and deploy the web application, providing a public domain URL once complete.

---

## 2. Configure the DynamoDB Database

Next, you will create the NoSQL table that will store the student details.

1.  **Navigate to DynamoDB:** Go to the AWS Console and search for **DynamoDB**.
2.  **Create Table:** Click **Create table**.
3.  **Table Details:**
    * **Table name:** `Student-Details`
    * **Partition key:** `ID` (select the String data type)
4.  **Finalize:** Leave all other settings as default and click **Create table**.

---

## 3. Set Up Lambda Backend Functions and IAM Role

You will create two Lambda functions for the application's API logic (GET and POST) and grant them permission to interact with DynamoDB.

### A. Create `add-student-function` (POST)

1.  **Create Lambda Function:** Go to **Lambda** and click **Create function**.
2.  **Configuration:**
    * **Function name:** `add-student-function`
    * **Runtime:** **Python 3.10**
    * **Permissions:** Select **Create a new role with basic Lambda permissions**.
3.  **Deploy Code:** Copy the Python code for the post method (found in the repository's `backend/post` folder) into the Lambda function's code editor, replacing the default code. Click **Deploy**.

### B. Add DynamoDB Permissions

The Lambda function needs permission to write data to the DynamoDB table.

1.  **Access IAM Role:** In the `add-student-function` configuration, go to **Configuration** > **Permissions** and click on the **Role name** to open the IAM console.
2.  **Create Inline Policy:** Click **Add permissions** > **Create inline policy**.
3.  **Paste Policy:** Switch to the **JSON** tab and paste the required DynamoDB policy (from the repository's `DynamoDB_policy` file). Crucially, ensure you update the ARN in the JSON to match your AWS account ID and table name.
4.  **Save:** Name the policy (e.g., `DynamoDB-permission`) and click **Create policy**.

### C. Create `get-student-function` (GET)

1.  **Create Lambda Function:** Go back to Lambda and click **Create function**.
2.  **Configuration:**
    * **Function name:** `get-student-function`
    * **Runtime:** **Python 3.10**
    * **Permissions:** Select **Use an existing role** and choose the role created for the `add-student-function` (which now has DynamoDB permissions).
3.  **Deploy Code:** Copy the Python code for the get method (from the repository's `backend/get` folder) into the function's editor and click **Deploy**.

---

## 4. Configure API Gateway

The API Gateway acts as the public-facing entry point for your Lambda functions.

1.  **Create REST API:** Go to **API Gateway** and click **Create API**. Select **REST API** and click **Build**.
    * **API Name:** `Student-API` (or similar).
2.  **Create Resources:**
    * Click **Actions** > **Create Resource**. Set the **Resource Path** to **`add-student`** and click **Create Resource**.
    * Click **Actions** > **Create Resource** again. Set the **Resource Path** to **`get-student`** and click **Create Resource**.
3.  **Create POST Method for `/add-student`:**
    * Select the `/add-student` resource. Click **Actions** > **Create Method**.
    * Select **POST**.
    * Set **Integration type** to **Lambda Function**.
    * For **Lambda Function**, select **`add-student-function`**. Click **Create Method**.
4.  **Create GET Method for `/get-student`:**
    * Select the `/get-student` resource. Click **Actions** > **Create Method**.
    * Select **GET**.
    * Set **Integration type** to **Lambda Function**.
    * For **Lambda Function**, select **`get-student-function`**. Click **Create Method**.
5.  **Deploy API:**
    * Click **Actions** > **Deploy API**.
    * Select **[New Stage]** and set the **Stage Name** (e.g., `student-api`).
    * Click **Deploy**. Note the **Invoke URL** displayed (this is your API base URL).

---

## 5. Connect Frontend and Resolve CORS

The final step is to link the Amplify frontend with the API Gateway backend.

1.  **Update Frontend Endpoints:**
    * In your local copy of the repository's frontend code (e.g., `app.js`), replace the placeholder API endpoint variables with the **Invoke URL** from API Gateway.
    * The full endpoint URLs should be:
        * GET Endpoint: `[Invoke URL]/get-student`
        * POST Endpoint: `[Invoke URL]/add-student`
    * **Commit and push** these changes to your GitHub repository. Amplify will automatically detect the changes and redeploy the frontend.
2.  **Enable CORS on API Gateway:** To prevent cross-origin resource sharing (CORS) errors, you must enable it on the API Gateway.
    * In API Gateway, select the `/add-student` resource. Click **Actions** > **Enable CORS**.
    * In the configuration:
        * Set **Access-Control-Allow-Origin** to the **exact domain URL of your deployed Amplify app** (e.g., `https://main.xxxxxxxx.amplifyapp.com`) **without a trailing slash**.
        * Ensure **POST** is selected in **Access-Control-Allow-Methods**.
        * Click **Save**.
    * Repeat the same **Enable CORS** process for the `/get-student` resource, setting the same **Access-Control-Allow-Origin** domain and ensuring **GET** is selected.
3.  **Re-deploy API:** After enabling CORS, you must deploy the API again for the changes to take effect.
    * Click **Actions** > **Deploy API** and select your existing stage (e.g., `student-api`). Click **Deploy**.

The application is now fully functional. You can access the Amplify app URL to view existing data and add new records, validating the end-to-end serverless implementation.


## 🗑️ Clean Up Process (Resource Deletion)

After testing your application, it's crucial to delete all created AWS resources to stop incurring ongoing costs. Follow these steps in order, as some resources depend on others.

---

### Step 1: Delete the API Gateway

1.  Go to **API Gateway** in the AWS Console.
2.  Select the **`Student-API`** REST API.
3.  From the **Actions** dropdown, choose **Delete API**. Confirm the deletion.

---

### Step 2: Delete Lambda Functions and CloudWatch Logs

1.  **Delete Functions:** Go to **AWS Lambda**.
2.  Select both **`add-student-function`** and **`get-student-function`**.
3.  From the **Actions** dropdown, choose **Delete**. Confirm the deletion.
4.  **Delete Log Groups:** Go to **CloudWatch** > **Log Groups**.
5.  Search for the log groups associated with your Lambda functions (names typically start with `/aws/lambda/`).
6.  Select the log groups for both functions and choose **Actions** > **Delete log group**. Confirm the deletion.

---

### Step 3: Delete the DynamoDB Table

1.  **Delete Table:** Go to **DynamoDB** in the AWS Console.
2.  Select the **`student-details`** table.
3.  Click the **Delete** button and confirm the deletion.

---

### Step 4: Delete the IAM Role and Policy

Since the Lambda functions are deleted, you can remove the role they used.

1.  **Delete Role:** Go to **IAM** > **Roles**.
2.  Search for the Lambda execution role (the one created automatically or the one you manually configured).
3.  Select the role and click **Delete role**. Ensure any inline policies (like the `DynamoDB-permission`) are removed with the role.

---

### Step 5: Delete the AWS Amplify App

1.  **Delete App:** Go to **AWS Amplify**.
2.  Select your deployed application (the one connected to your GitHub repository).
3.  In the top right corner, click **Actions** > **Delete app**. Confirm the deletion. This will remove all frontend hosting resources.
