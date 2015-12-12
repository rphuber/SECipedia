Need to revisit for exam: Active Directory Integration


IAM
    Strategies
        Outside access
            Create 2 security groups.  1 for public access/1 for private.


    To initially set up everything in the main IAM console, you need to make sure that all of the security "checkmarks" are finished for the Security Status dialog

    Allows you to manage users and their level of access to the AWS Console
        Granular level of control: User/Group/Roles
            Group: A collection of users under one set of permissions
                Ex: We set up a Group Called developers and give them a policy name of "power user access", which is like a permissions template.
                    See more about this template below.
                    Policy templates
                        Admin: Full access to AWS services and resources
                        Power User Access: Admin but no user/group management
                        Read Only
                        You can only assign 2 policys to a role
                        * We don't have to use a template

                    We then create a user and assign them the group "developers".  
            Roles: Similar to a group, but you can assign both users and AWS resources (like EC2).  This allows you to centralize your account management so you don't store credentials on EC2, etc.  (This is a best practice)
                The Role ARN is the role's unique name in aws
                There's AWS Service Roles, Role for Cross-Account Access (accessing via other AWS accounts), Role for Identity Provider Access (logging in with facebook,etc)
                Ex: For example we can give our EC2 instances an "Amazon EC2" role which allows EC2 Instances to call AWS services on our behalf.
                    Then for a given role, we can attach up to 2 policies.  We will attach "AmazonS3FullAccess" as a policy.
                Now, any Ec2 instance that we utilize we can assign it the aforementioned role that we created

                Can't be assigned to an EC2 instance after that instance has been created. You cannot change them either.  Although, you can change the permissions of a role that has already been assigned to an Ec2 instance.
    Centalised control of your AWS account
    Integrates with existing active directory account allow for single sign on
    Multifactor Authentication
    Allows for temporary access for users/devices and services
    Can provision a custom password rotation policy

    Don't sign in with the root account for main activity

    When you create a user, it will only give you the Access Key ID (username) and Secret Access Key (password) once
        This pair will allow for access to the AWS CLI, API calls to the AWS portal, and using SDKs, etc.

    To give access to the AWS console, you create that in IAM as well


    Scenario:
        Through AWS CLI, access your S3 bucket while authenticating through IAM.  In other words, don't store your key pair on EC2.
            Create an s3 role in IAM
            Launch an EC2 and select that role
            ssh into box with key/pair in EC2 setup
         
