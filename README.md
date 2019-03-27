# AWS RDS - connecting using Authentication Token

this for to connect AWS-RDS Postgres instance with auth token (instead of password)

Step 1:

1. create the AWS-RDS Postgres instance
  
    a. make sure connect from Public accessibility is set to true, if you are connecting beyond vpc (generally yes for testing)
    b. make sure you enable IAM db authentication to true

2. Create a EC2 instance 

   Connect to AWS-RDS instance (either through psql if your EC2 already up and running or via pgadmin client)
   
   if psql,
   
    a. install psql in EC2 using "sudo amazon-linux-extras install postgresql10" command
    
    b. Once installed, login to psql using "psql -h xxx.xxxx.rds.amazonaws.com  -p 5432 -d dbname -U masterusername"
    
3. Create db user in postgres which will be used to login

    a. create user xxxxtestuser with LOGIN;
    
    b. GRANT rds_iam TO xxxxtestuser
    
    c. exit the psql with \q
    
4. create the IAM role 

    a. add the AmazonRDSFullAccess to new role
    
    b. add inline policy
    
    {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "rds-db:connect"
            ],
            "Resource": [
                "arn:aws:rds-db:**region**:**aws-account-id**:dbuser:**db-instanceid**/xxxxtestuser"
            ]
        }
    ]
}

    **region** is region name (ex. us-east-1)
    
    **aws-account-id** is your aws account id
    
    **db-instanceid** is your database instance id (can be found under configuration tab of your RDS instance)
    
    xxxxtestuser is user we created above

5. Assign this role to EC2, select EC2 server, assign IAM role, and assign newly created role

6. in EC2 instance, create .sh file

RDSHOST=xxxx.xxxx.xxxx.rds.amazonaws.com
USERNAME=xxxxtestuser
#FILENAME=$1
DBNAME=dbname
export PGPASSWORD="$( aws rds generate-db-auth-token --hostname $RDSHOST --port 5432 --username $USERNAME --region regioname )"
psql "host=$RDSHOST dbname=$DBNAME user=$USERNAME"
#psql "host=$RDSHOST dbname=$DBNAME user=$USERNAME" -f "$FILENAME"

7. run the script file sh scriptfile.sh

8. if you want to run this with sql file, use the $filename option in script
