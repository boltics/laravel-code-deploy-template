# Laravel CodeDeploy Template
A template for deploying Laravel applications with AWS CodeDeploy across an autoscaling group.

## IAM Roles **Probably automatically created from CodeDeploy below. Don't do this!**
The following roles should be created **BEFORE** any launch configurations, autoscaling groups or CodeDeploy applications are created as they'll be needed during that creation process.

`code-deploy-role`

 - Requires the pre-baked policy **AWSCodeDeployRole**

`code-deploy-ec2-instance-role`

 - Requires the pre-baked policy **AmazonEC2FullAccess**
 - Requires a custom policy **code-deploy-ec2-permissions** with the following:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:Get*",
                "s3:List*"
            ],
            "Effect": "Allow",
            "Resource": "*"
        }
    ]
}
```

## Requirements
 - A compatible **Ubuntu 16.04** AMI (use the provided **provision.sh** script on a fresh instance and save an AMI image from that). **Use the default AMI directly. We can provision on build at a later time.**
 - A classic load balancer (preferably with sticky connections [5mins+]) **Application load balancer is ok**
 - An AWS launch configuration (using the aforementioned AMI) and an autoscaling group **create from CodeDeploy**
 - A CodeDeploy application and deployment group that references the aforementioned autoscaling group **create from CodeDeploy**
 - A S3 bucket for your application with two folders contained within: `bundles` and `env`

## Setup
 - Copy the `scripts` directory into the root of your Laravel application, as well as the `bundle.sh` and `appspec.yml` files **you should try to merge the provision.sh into this**
 - Replace the `XXX` placeholder inside of `bundle.sh` and `scripts/finish_install.sh` with your S3 bucket name **We probably don't need this**
 - Inside of the S3 bucket folder `env` place a `production.env` file that contains your production settings. This will be copied to each instance as a `.env` file during each deployment cycle **This is good**

## Usage **(We probably don't need this, because we are using aws pipelines and bitbucket to deploy on git push)**
 - Make sure `composer` and `npm` have been run and all dependencies are up to date and that the application works as expected locally and on staging
 - Use `bundle.sh` to automatically create a `tar.gz` archive of your Laravel application and upload it to the S3 bucket folder `bundles`
 - The `bundle.sh` will tar only the directories and files listed in `bundle.conf`
 - Create a new CodeDeploy revision deployment using S3 as the source and point the endpoint to the one provided by the `bundle.sh` script

## Provisioner overview
The provided `provision.sh` script used to generate the instance needed for the AMI does the following:
 - Installs **NGINX**, **PHP7.0-FPM**, **Composer**, **NPM**, the **CodeDeploy Agent**, the **AWS CLI** and a bunch of related/required dependencies
 - Configures NGINX to serve content out of the `/var/www/public` directory
 - Only allows `index.php` to be executed via `PHP7.0-FPM`
