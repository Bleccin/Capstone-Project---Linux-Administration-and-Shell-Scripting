# Capstone-Project---Linux-Administration-and-Shell-Scripting
Project Scenario:
CloudOps Solution is a growing company that recently adopted AWS to manage its cloud infrastructure. As the company scales, theyhave decided to automate the process of managing AWS Identity and Access Management (IAM) resources. This includes the creation of users, user groups and assignment of permissions for new hires, especially their Devops team.
Purpose:
Extend Your current shell script "aws_cloud_manager.sh" script by creating more functions for creating IAM Users; IAM Group; Attach Administrative policy to Group; Assign Users to Group.

SOLUTION:
Script Enhancement:
1. I opened the existing "aws_cloud_manager.sh" to include perform the required tasks.

Define IAM User Names Array:
2. I defined an array of five IAM usernames.

Create IAM Users:
3. I created a function called "create_iam_users" to create new IAM users.
4. I defined a Loop to go through the array and create IAM users for all the usernames on the list.

Create IAM Group:
5. I created a function called "create_iam_group" for the creating an IAM group named "Admin".

Attach Administrative policy to Group:
6. I created a function named "attach_admin_policy_to_group" to attach administrative policy to the group.

Assign users to the admin Group:
7. I created a function named "assign_users_to_group" to assign all the created users to the admin group.
8. I created a for loop to loop through the array of users I created earlier and add them to the admin group.

Finally I called all the functions I created at the end of the script.

Here is he final code:

#!/bin/bash

# Environment variables
ENVIRONMENT=$1

check_num_of_args() \{
# Checking the number of arguments
if [ "$#" -ne 0 ]; then
    echo "Usage: $0 <environment>"
    exit 1
fi
\}

activate_infra_environment() \{
# Acting based on the argument value
if [ "$ENVIRONMENT" == "local" ]; then
  echo "Running script for Local Environment..."
elif [ "$ENVIRONMENT" == "testing" ]; then
  echo "Running script for Testing Environment..."
elif [ "$ENVIRONMENT" == "production" ]; then
  echo "Running script for Production Environment..."
else
  echo "Invalid environment specified. Please use 'local', 'testing', or 'production'."
  exit 2
fi
\}

# Function to check if AWS CLI is installed
check_aws_cli() \{
    if ! command -v aws &> /dev/null; then
        echo "AWS CLI is not installed. Please install it before proceeding."
        return 1
    fi
\}

# Function to check if AWS profile is set
check_aws_profile() \{
    if [ -z "$AWS_PROFILE" ]; then
        echo "AWS profile environment variable is not set."
        return 1
    fi
\}

# Function to create EC2 Instances
create_ec2_instances() \{

    # Specify the parameters for the EC2 instances
    instance_type="t2.micro"
    ami_id="ami-0cd59ecaf368e5ccf"
    count=2  # Number of instances to create
    region="eu-west-2" # Region to create cloud resources

    # Create the EC2 instances
    aws ec2 run-instances \
        --image-id "$ami_id" \
        --instance-type "$instance_type" \
        --count $count
        --key-name MyKeyPair

    # Check if the EC2 instances were created successfully
    if [ $? -eq 0 ]; then
        echo "EC2 instances created successfully."
    else
        echo "Failed to create EC2 instances."
    fi
\}

# Function to create S3 buckets for different departments
create_s3_buckets() \{
    # Define a company name as prefix
    company="datawise"
    # Array of department names
    departments=("Marketing" "Sales" "HR" "Operations" "Media")

    # Loop through the array and create S3 buckets for each department
    for department in "$\{departments[@]\}"; do
        bucket_name="$\{company\}-$\{department\}-Data-Bucket"
        # Create S3 bucket using AWS CLI
        aws s3api create-bucket --bucket "$bucket_name" --region your-region
        if [ $? -eq 0 ]; then
            echo "S3 bucket '$bucket_name' created successfully."
        else
            echo "Failed to create S3 bucket '$bucket_name'."
        fi

    done
\}

#Array of user names
    usernames=("Arinola" "Adedapo" "Iyiola" "Blessing" "Ire")


#Function to create IAM Users
create_iam_users() \{

    #Loop through the array and create IAM users for all the names on the list

    for username in "$\{usernames[@]\}"; do
            echo "creating IAM user: $username"
            aws iam create-user --user-name "$username"
 done
\}

#Function to create IAM User Group
 create_iam_group() \{
 echo "creating IAM group: admin"
 aws iam create-group --group-name admin
 \}

 #Function to attach administrative policy to the group
 attach_admin_policy_to_group() \{
  echo "Attaching Administrator Access policy to the admin group"
  aws iam attach-group-policy --group-name admin --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
\}

#Function to assign users to the admin group
assign_users_to_group() \{
  for username in "$\{usernames[@]\}"; do
    echo "Adding user $username to admin group"
    aws iam add-user-to-group --user-name "$username" --group-name admin
  done
\}


# Execute the functions

check_num_of_args
activate_infra_environment
check_aws_cli
check_aws_profile
create_ec2_instances
create_s3_buckets
create_iam_users
create_iam_group
attach_admin_policy_to_group
assign_users_to_group

echo "Script completed successfully."

