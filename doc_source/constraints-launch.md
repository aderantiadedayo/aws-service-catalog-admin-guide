# AWS Service Catalog Launch Constraints<a name="constraints-launch"></a>

A launch constraint specifies the AWS Identity and Access Management \(IAM\) role that AWS Service Catalog assumes when an end user launches a product\. An IAM role is a collection of permissions that an IAM user or AWS service can assume temporarily to use AWS services\. For an introductory example, see [Step 6: Add a Launch Constraint to Assign an IAM Role](getstarted-launchconstraint.md)\.

Launch constraints apply to products in the portfolio \(product\-portfolio association\)\. Launch constraints do not apply at the portfolio level or to a product across all portfolios\. To associate a launch constraint with all products in a portfolio, you must apply the launch constraint to each product individually\.

Without a launch constraint, end users must launch and manage products using their own IAM credentials\. To do so, they must have permissions for AWS CloudFormation, AWS services that the products use, and AWS Service Catalog\. By using a launch role, you can instead limit the end users' permissions to the minimum they require for that product\. For more information about end user permissions, see [Identity and Access Management in AWS Service Catalog](controlling_access.md)\.

To create and assign IAM roles, you must have the following IAM administrative permissions:
+ `iam:CreateRole`
+ `iam:PutRolePolicy`
+ `iam:PassRole`
+ `iam:Get*`
+ `iam:List*`

## Configuring a Launch Role<a name="constraints-launch-role"></a>

The IAM role that you assign to a product as a launch constraint must have permissions to use the following:
+ AWS CloudFormation
+ Services in the AWS CloudFormation template for the product
+ Read access to the AWS CloudFormation template in Amazon S3

The IAM role also must have a trust relationship with AWS Service Catalog, which you assign by selecting **AWS Service Catalog** as the role type in the following procedure\. The trust relationship allows AWS Service Catalog to assume the role during the launch process to create resources\.

**Note**  
The `servicecatalog:ProvisionProduct`, `servicecatalog:TerminateProduct`, and `servicecatalog:UpdateProduct` permissions cannot be assigned in a launch role\. You must use IAM roles, as shown in the inline policy steps in the section [Grant Permissions to Amazon Service Catalog End Users\.](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/getstarted-iamenduser.html)

**To create a launch role**

1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

1. Choose **Roles**\.

1. Choose **Create New Role**\.

1. Enter a role name and choose **Next Step**\.

1. Under **AWS Service Roles** next to **AWS Service Catalog**, choose **Select**\.

   Under **AWS Service Roles** next to **AWS Service Catalog**, choose **Select**\.

1. On the **Attach Policy** page, Choose **Next Step**\.

1. To create the role, choose **Create Role**\. 

**To attach a policy to the new role**

1. Choose the role that you created to view the role details page\.

1. Choose the **Permissions** tab, and expand the **Inline Policies** section\. Then, choose **click here**\.

1. Choose **Custom Policy**, and then choose **Select**\. 

1. Enter a name for the policy, and then paste the following into the **Policy Document** editor: 

   ```
     "Statement":[
         {
            "Effect":"Allow",
            "Action":[
               "s3:GetObject"
            ],
            "Resource":"*",
            "Condition":{
               "StringEquals":{
                  "s3:ExistingObjectTag/servicecatalog:provisioning":"true"
               }
            }
         }
      ]
   }
   ```
**Note**  
When you configure a launch role for a launch constraint, you must use this string: `"s3:ExistingObjectTag/servicecatalog:provisioning":"true"`\. 

1. Add a line to the policy for each additional service the product uses\. For example, to add permission for Amazon Relational Database Service \(Amazon RDS\), enter a comma at the end of the last line in the `Action` list, and then add the following line: 

   ```
   "rds:*"
   ```

1. Choose **Apply Policy**\.

## Applying a Launch Constraint<a name="constraints-launch-constraint"></a>

After you configure the launch role, assign the role to the product as a launch constraint\. This action tells AWS Service Catalog to assume the role when an end user launches the product\. 

**To assign the role to a product**

1. Open the AWS Service Catalog console at [https://console\.aws\.amazon\.com/servicecatalog/](https://console.aws.amazon.com/servicecatalog/)\.

1. Choose the portfolio that contains the product\.

1. Choose the **Constraints** tab and choose **Create constraint**\.

1. Choose the product from **Product** and choose **Launch** under **Constraint type**\. Choose **Continue**\.

1. In the **Launch constraint** section, you can select an IAM role from your account and enter an IAM role ARN, or enter the role name\.

   If you specify the role name and if an account uses the launch constraint, the account uses that name for the IAM role\. This approach allows launch\-role constraints to be account\-agnostic so you can create fewer resources per shared account\. 
**Note**  
The given role name must exist in the account that created the launch constraint and the account of the user who launches a product with this launch constraint\. 

1. After specifying the IAM role, choose **Create**\.

## Verifing the Launch Constraint<a name="constraints-launch-test"></a>

To verify AWS Service Catalog uses the role to launch the product and successfully provisions the product, launch the product from the AWS Service Catalog console\. To test a constraint prior to releasing it to users, create a test portfolio that contains the same products and test the constraints with that portfolio\.

**To launch the product**

1. In the menu for the AWS Service Catalog console, choose **Service Catalog**, **End user**\.

1. Choose the product to open the **Product details** page\. In the **Launch options** table, verify the Amazon Resource Name \(ARN\) of the role appears\.

1. Choose **Launch product**\.

1. Proceed through the launch steps, filling in any required information\.

1. Verify that the product starts successfully\.