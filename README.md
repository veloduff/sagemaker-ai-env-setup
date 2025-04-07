# SageMaker Setup

This process details the steps needed to launch a SageMaker Domain and Studio, and uses an Amazon Built-In Algorithm to test the environment. Specifically, the Linear Learner with MNIST Notebook is used. For additional information about getting started with SageMaker, see the <a href="https://sagemaker-examples.readthedocs.io/en/latest/intro.html">Introduction to Amazon SageMaker</a>.

## 1. Create an S3 bucket and an EFS file system

Rather than having SageMaker create an S3 Bucket and an EFS file system for each domain,
create an S3 Bucket and EFS file system before and use this when setting up a domain. 
For example:

* S3 Bucket name: **myusername-sagemaker-01**
* EFS file system name: **sagemaker-efs-01**, make sure that the security groups used for the
  EFS file system are the same that are used for the SageMaker Domain.


## 2. Create IAM Policy for running JupyterLab Notebooks

This workshop uses the <a href="https://github.com/aws/amazon-sagemaker-examples/tree/0abd974c89f7601e66cf2665b0d26975648f8b5d/introduction_to_amazon_algorithms">Amazon SageMaker Examples - Introduction to Amazon Algorithms</a> GitHub repo. When running Notebooks, there will be several Service dependencies for which the default permissions for the SageMaker Execution IAM role (created in the next step) do not include. An additional IAM Policy will be created and then attached to the SageMaker Execution role when it is created.

* Go to the AWS Console -> IAM -> Access management -> **Policies**, select **Create policy**
* Specify permissions: select **JSON**
* Remove the default policy and paste in the policy below:

> [!WARNING]
> Replace ```[ACCOUNT-ID]``` with your account.

  ```json
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "VisualEditor0",
              "Effect": "Allow",
              "Action": "iam:PassRole",
              "Resource": "arn:aws:iam::[ACCOUNT-ID]:role/*"
          },
          {
              "Sid": "VisualEditor1",
              "Effect": "Allow",
              "Action": [
                  "logs:DescribeLogStreams",
                  "logs:GetLogEvents",
                  "sagemaker:AddTags",
                  "sagemaker:CreateEndpoint",
                  "sagemaker:CreateEndpointConfig",
                  "sagemaker:CreateHyperParameterTuningJob",
                  "sagemaker:CreateModel",
                  "sagemaker:CreateTrainingJob",
                  "sagemaker:DeleteEndpoint",
                  "sagemaker:DescribeEndpoint",
                  "sagemaker:DeleteEndpointConfig",
                  "sagemaker:InvokeEndpoint"
              ],
              "Resource": "*"
          }
      ]
  }
  ```
* For name: **sagemaker-custom-notebook-perms**
* Click **Create policy**


## 3. Increase quota for PoliciesPerRole

The default number of policies per role is 10. This should be increased to 20 before the next step.

> [!CAUTION]
> As IAM is global service, you have to be in **us-east-1** to adjust IAM quotas and limits!


1. Set region to **us-east-1** (N. Virginia)
1. Verify that the region is **us-east-1** (N. Virginia)
1. At the AWS Console, go to **Service Quotas** -> **AWS services** and search for "*identity and access*", and select **AWS Identity and Access Management (IAM)**
1. Search for "*policies per role*" and select **Managed policies per role**
1. Click the button **Request increase at account level** at the top right
1. For Increase quota value, set to **20** (max), and click **Request**

You should see an approval message "Quota increase request for Managed policies per role was Approved with the Value of 20."


## 4. Create a custom SageMaker Execution Role 

Before a SageMaker Domain is created, a custom SageMaker Execution Role should be 
created first.  This will be used for creating SageMaker domains and for running 
JupyterLab Notebooks. This also allows for the flexibility to use custom naming
conventions.

When you create a Role using **Role manager** , "SageMaker-" is automatically added 
as a prefix. For example if you use **Custom-Execution-Role-01** for the Role name, 
the resulting name will be **SageMaker-Custom-Execution-Role-01**.

* Go to the SageMaker console, and under **Admin configurations** -> **Role manager**, select **Create a role**:

  <img src="/_assets/images/sagemaker_setup/sm_demo-01.png" width="600px">

* **Step 1: Enter role information**

  * **Set up SageMaker role**:
    * Role name suffix: **Custom-Execution-Role-01** 
    * Description: **SageMaker custom execution role 01**
    * Select a persona for this role's permissions: **Custom role settings**
  
    <img src="/_assets/images/sagemaker_setup/sm_demo-02.png" width="600px">
  
  * **Network conditions**
    This step is optional, but you should be specifying custom VPCs and security group(s) for productions environments.<br>
    <img src="/_assets/images/sagemaker_setup/sm_demo-03.png" width="600px">
  
  * **Encryption conditions**
    * Enable **Encryption customization**
    * For the **Data** and **Volume** encryption keys, these can be AWS Managed (shown below) or customer managed keys.
  
    <img src="/_assets/images/sagemaker_setup/sm_demo-04.png" width="600px">
  
  Click **Next**
  
* **Step 2: Configure ML activities**

  For the ML activities, select the below options and input in the S3 Bucket created in Step 1:

  * **Configure new role**

    <img src="/_assets/images/sagemaker_setup/sm_demo-05.png" width="600px">

  * **Set the S3 Bucket name**

    <img src="/_assets/images/sagemaker_setup/sm_demo-06.png" width="600px">

  Click **Next**

* **Step 3: Add additional policies & tags**

  * **Add additional IAM policies to this role**: search for "*sagemaker-custom*", or the used for policy created in Step 2:<br>
    <img src="/_assets/images/sagemaker_setup/sm_demo-07.png" width="600px">
  * Optionally add tags

  Click **Next**

* **Review role**

  Click **Submit**

You should see a **Success** message, and a button for **Go to role**. Click on the **Go to role** button. If the button is not there, go in to IAM -> Roles and open the role from there.


## 5. Create SageMaker Domain

Now that the S3 Bucket and IAM Role (with a custom Policy) have been created, the SageMaker Domain can now be created.

Go to the SageMaker console, and under **Admin configurations** -> **Domains**, select **Create domain**:

* Set up SageMaker Domain: select **Set up for organizations**, click **Set up**<br>
  <img src="/_assets/images/sagemaker_setup/sm_demo-08.png" width="600px">

  * **Step 1: Set up Domain details & users**
    * Domain name: example **sagemaker-domain-01**
    * How do you want to access Studio?: **Login through IAM**
    * Who will use SageMaker?: **Do not add a user, this will be done in a step below** 
    <br> 
    <img src="/_assets/images/sagemaker_setup/sm_demo-09.png" width="600px"><br>
  
    Click **Next**
  
  * **Step 2: Configure roles and ML activities**
    * What ML activities will users perform?: **Use an existing role**, and choose the role created in Step 4:
    <br>
    <img src="/_assets/images/sagemaker_setup/sm_demo-10.png" width="600px"><br>

    Click **Next**

  * **Step 3: Configure Applications**<br>
    Accept the defaults.

    Click **Next**

  * **Step 4: Customize Studio UI**<br>
    Accept the defaults.

    Click **Next**

  * **Step 5: Set up network settings**<br>
    * How do you want to connect to other AWS services?: **Public internet access**
    * VPC: Select a VPC
    * Subnets: Select subnets
    * Security groups: Select security groups

    <img src="/_assets/images/sagemaker_setup/sm_demo-11.png" width="600px"><br>

    > [!NOTE] 
    > Note: The message "**Important: The selected VPC is not connected to the following services:**" can be ignored.

    Click **Next**

  * **Step 6: Configure storage**<br>
    Accept the defaults.

    Click **Next**

  * **Step7: Review and create**<br>
    
    Click **Submit**

* Wait for the **Status** to say:  <span style="color:green">**Ready**<span>


## 6. Create a user 

Before a studio can be launched, a user will need to be created.

Go in to the Domain that was just created, and select the **User profiles** tab. Select **Add user**:<br><br>
<img src="/_assets/images/sagemaker_setup/sm_demo-11a.png" width="600px"><br>

* **Step 1: General settings**
  * **User Profile:**
    * Name: **Enter a user name**
    * Execution role: **Enter the role that was created previously**

  <img src="/_assets/images/sagemaker_setup/sm_demo-11b.png" width="600px"><br>

* **Step 2: Configure Applications**<br>
  Accept the defaults.

  Click **Next** 

* **Step 3: Customized Studio UI**<br> 
  Accept the defaults.

  Click **Next** 

* **Step 4: Review and create**<br>
    
  Click **Submit**


## 7. Launch a Studio 

* Go in to the Domain that was just created, and select the **User profiles** tab. For the 
  user that was just created, click **Launch** -> **Studio**, and this
  will redirect you to the **SageMaker Studio** Application portal:<br>

  <img src="/_assets/images/sagemaker_setup/sm_demo-12.png" width="600px"><br>


## 8. Create a JupyterLab space

* In **SageMaker Studio** go to **Applications** on left, click on **JupyterLab**:<br>

  <img src="/_assets/images/sagemaker_setup/sm_demo-13.png" width="600px"><br>

* Click on **Create JupyterLab space**:<br>
  
  <img src="/_assets/images/sagemaker_setup/sm_demo-14.png" width="600px"><br>

* Provide a **Name** and select **Sharing**: 

  <img src="/_assets/images/sagemaker_setup/sm_demo-15.png" width="600px"><br>

  Click **Create space**

It will take a few seconds.

## 9. Run the JupyterLab space

* If not already, go in the JupyterLab space that was just created, you can optionally changed the **Instance**, 
  **SageMaker Distribution Image**, adjust the **Storage**, **Lifecycle**, or attach an **existing
  EFS file system** (directions for custom EFS file system are below).

* Select **Run space**, it could take several minutes to start the space.
  
  <img src="/_assets/images/sagemaker_setup/sm_demo-16.png" width="600px"><br>


## 10. Open the JupyterLab space and clone the Amazon SageMaker Examples repo

* Once the space is running, select **Open JupyterLab**:<br>
  
  <img src="/_assets/images/sagemaker_setup/sm_demo-17.png" width="600px"><br>

* In the JupyterLab space, select **Terminal**:<br>

  <img src="/_assets/images/sagemaker_setup/sm_demo-18.png" width="600px"><br>

* In the terminal, clone the repo:<br>

  ```sh
  git clone https://github.com/aws/amazon-sagemaker-examples.git
  ```

* After the clone, the repo will be available in the file browser on the left:

  <img src="/_assets/images/sagemaker_setup/sm_demo-19.png" width="600px"><br>


## 11. Launch the **Linear Learner** example Notebook

* In the file browser on the left, go in to:<br>

  ```sh
  amazon-sagemaker-examples/introduction_to_amazon_algorithms/linear_learner_mnist/
  ```

  Double click on the ```linear_learner_mnist.ipynb``` Notebook, and select the default kernel:<br>
  
  <img src="/_assets/images/sagemaker_setup/sm_demo-20.png" width="600px"><br>


## 12. Set the S3 Bucket

Rather than creating a bucket for each notebook (which is what the ```sagemaker.Session().default_bucket()``` call 
will do by default), use an existing bucket for all Notebooks. Change ```bucket``` from this:

```py
bucket = sess.default_bucket()
```
to this: 
```py
bucket = sess.default_bucket = 'my-sagemaker-bucket'
```

# Optional steps

## Delete SageMaker Domain and other resources created by SageMaker 

If you need to delete a SageMaker Domain, you will need go through a series of steps:

1. Delete all spaces and applications
1. Delete all users
1. Delete the domain
1. Delete EFS file system(s)
1. Delete Security Groups 
1. Delete IAM Roles (e.g., SageMaker-ExecutionRole-*)


## Optionally attach a custom EFS file system

1. The domain should already exist, this process updates an existing 
   domain's **default user settings**.
1. Create the EFS File System using the custom option to be able to set 
   each of the Subnets, and assign a Security Group that the SageMaker 
   instances will also have access to.
1. Using the ARN for the custom SageMaker Execution role created above, create a JSON file 
   (e.g., sagemaker-user-settings.json) with the EFS File System ID:
   ```json
   {
       "ExecutionRole": "arn:aws:iam::111222333444:role/service-role/SageMaker-role-01",
       "CustomFileSystemConfigs":
       [
           {
               "EFSFileSystemConfig":
               {
                   "FileSystemId": "fs-00001111aaaa22222",
                   "FileSystemPath": "/"
               }
           }
       ]
   }
   ```
1. Run the ```aws sagemaker update-domain``` command to attach the EFS file system
  
   ```sh
   $ aws sagemaker update-domain --domain-id d-aaaa00001111 --default-user-settings file://sagemaker-user-settings.json
   ```
   The output should be similar to this:
   
   ```json
   {
       "DomainArn": "arn:aws:sagemaker:us-west-2:111222333444:domain/d-aaaa1111bbbb",
   }
   ```
1. Update the **default user settings** to use the same **Security Group** as the EFS file system:
   
   ```sh
   $ aws sagemaker update-domain --domain-id d-aaaa1111bbbb --default-user-settings SecurityGroups="sg-0000aaaa1111bbbbb"
   ```
1. The EFS file system can be selected when launching a space. 


