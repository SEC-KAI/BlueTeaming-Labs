In this lab, I practiced using AWS Identity and Access Management (IAM), creating users and groups, assigning permissions through policies, and working with Amazon S3.

## IAM User and Group

I created an IAM user and added the user to a group with the **AdministratorAccess** policy.

This helped me understand that permissions can be assigned to a group instead of configuring every user individually.

## IAM Policies

I explored how IAM policies control what users are allowed to do in AWS.

Policies can define:

* Which AWS service can be accessed
* What actions are allowed or denied
* Which resources the permissions apply to
* Optional conditions

For example, an S3 policy can allow a user to read files while preventing them from deleting them.

## Amazon S3

I also created an Amazon S3 bucket.

S3 is AWS's storage service, while a bucket is the container used to store files. The files stored inside the bucket are called objects.

Examples of objects include:

* Documents
* Images
* Backups
* Log files

## S3 Object Ownership

While creating the bucket, I kept **ACLs disabled**, which is the recommended option.

With this setting, the AWS account that owns the bucket also owns the objects stored inside it. Access is mainly controlled through IAM policies and bucket policies instead of individual ACL permissions.

## Network Security

I also learned about AWS network security controls.

* **Security Groups** use allow rules to control traffic.
* **Network ACLs (NACLs)** can use both allow and deny rules.

I practiced creating a NACL rule that denied inbound traffic on **port 22 (SSH)**.

## What I Learned

This lab helped me understand how AWS uses IAM users, groups, and policies to control access. I also learned how S3 buckets store objects and how AWS uses Security Groups and NACLs to control network traffic.
