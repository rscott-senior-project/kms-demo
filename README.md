AWS Developer Associate Content Series : Encryption in AWS - Demo  

1.  [AWS Developer Associate Content Series](index.html)
2.  [AWS Developer Associate Content Series](AWS-Developer-Associate-Content-Series_33099.html)
3.  [Domains](Domains_589841.html)
4.  [Security](Security_589827.html)

AWS Developer Associate Content Series : Encryption in AWS - Demo
=================================================================

Created by Rob Scott, last modified on Jan 26, 2022

*   Client Side encryption with AWS KMS
*   Server side encryption (example in S3)
*   envelope encryption with AWS KMS data key
*   Type of Keys

Client Side Encryption with AWS Key Management Service
------------------------------------------------------

Client side encryption is done to guarantee that data is encrypted while entering AWS Services. KMS can secure encryption keys and help this process. The CLI will allow you to encrypt and decrypt plaintext before sending it over the wire to, say, another S3 resource.

Check to see if there are any existing keys available for use

`aws kms list-keys`

If there are no keys, go to the AWS Console and navigate to Key Management Service. There, generate a new key with as the admin user and make sure that your admin account is specified as an eligible user of the cryptographic operations.

{
    "Id": "key-consolepolicy-3",
    "Version": "2012-10-17",
    "Statement": \[
        {
            "Sid": "Enable IAM User Permissions",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::170976071768:root"
            },
            "Action": "kms:\*",
            "Resource": "\*"
        },
        {
            "Sid": "Allow access for Key Administrators",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::170976071768:user/rob-senior-project-admin"
            },
            "Action": \[
                "kms:Create\*",
                "kms:Describe\*",
                "kms:Enable\*",
                "kms:List\*",
                "kms:Put\*",
                "kms:Update\*",
                "kms:Revoke\*",
                "kms:Disable\*",
                "kms:Get\*",
                "kms:Delete\*",
                "kms:TagResource",
                "kms:UntagResource",
                "kms:ScheduleKeyDeletion",
                "kms:CancelKeyDeletion"
            \],
            "Resource": "\*"
        },
        {
            "Sid": "Allow use of the key",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::170976071768:user/rob-senior-project-admin"
            },
            "Action": \[
                "kms:Encrypt",
                "kms:Decrypt",
                "kms:ReEncrypt\*",
                "kms:GenerateDataKey\*",
                "kms:DescribeKey"
            \],
            "Resource": "\*"
        },
    \]
}

List all KMS keys again and make sure it is there now. That key ID will be used to encrypt and decrypt our plaintext.

`echo "secret database string" >> db-string.txt`

Lets pretend that we need to encrypt this database string before uploading it to S3. Use this CLI command to use the KMS key and put it back in its own encrypted file on our system.

`aws kms encrypt --key-id <KEY-ID> --plaintext fileb://db-string.txt --query "CiphertextBlob" > encrypted`

Breaking a few things down:

*   The `fileb://` is a flag for the CLI to take a file in as plaintext, rather than a string as an arg
    
*   `--query` selects the part of the CLI response we want, which is the ciphertext
    
*   `> encrypted` redirects our chosen output to a file, where our encrypted text is stored.
    

To decrypt the ciphertext, we will use the CLI again:

`aws kms decrypt --ciphertext-blob $(cat encrypted) --query Plaintext --output text | base64 --decode`

Notice how `decrypt` did not require the `key-id` to decrypt the ciphertext. Thats because the ciphertext blob has metadata attached to choose the appropriate key for decryption.

Envelope Encryption with Key Management Service
-----------------------------------------------

Envelope encryption is used to encrypt large amounts of data that would otherwise be hard to send to AWS KMS servers. This may be more efficient, but now our encryption key and encrypted data are in the same place (never good). As a solution, KMS’s implementation of envelope encryption will return two keys, a plaintext copy and encrypted copy. Encrypt the data and promptly dispose of the plaintext key. The encrypted key can reside with the data, and then be decrypted via KMS and used when the data needs to be decrypted. The plaintext key can be used to decrypt the ciphertext.

#### Procedure

Install the AWS Encryption CLI following the instruction the AWS docs. After its installed, we will use our existing KMS key to perform envelope encryption.

First, make another example file to operate on:

`echo "very large file" >> large-data.txt`

Next, the Encryption CLI will take the ARN of your existing key to encrypt a data key. This data key is part of the ciphertext generated from the encryption. When its ready to be decrypted, the encrypted data key is identified and decrypted to a plaintext key via KMS. This means that the CLI only needs the ARN of our existing key within KMS; the data key is generated and handled within the command.

To get the ARN of our KMS key, hit the following command:

`aws kms list-keys`

To encrypt the file, use the following command:

aws-encryption-cli --encrypt \\
--input large-data.txt \\
--wrapping-keys key=<KEY\_ARN> \\
--metadata-output ./metadata \\
--output encrypted-large-data

Now our data is encrypted within `encrypted-large-data`. The data key was never handled by use, but it was generated, encrypted our file as plaintext, and then encrypted using the `wrapping-keys` parameter.

To decrypt the data, we will use a similar decrypt command:

aws-encryption-cli --decrypt \\
--input encrypted-large-data \\
--wrapping-keys key=<KEY\_ARN> \\
--metadata-output ./metadata \\
--output decrypted-large-data.txt

Enabling Server-Side Encryption
-------------------------------

Server-side encryption can be used on several storage services in an AWS environment. For this example, we will create an S3 bucket and then do two things:

*   Restrict public access to the bucket; this is often standard unless the bucket is hosting a website
    
*   Configure server-side encryption on the bucket
    

The latter is the most important operation of this demonstration. Open up the console and navigate to the S3 page to verify that changes are being made through the CLI. Its important to get used to using the CLI because these procedures can be turned into scripts and favor automation throughout any given infrastructure.

Create the bucket using the CLI and verify that the bucket now exists in the console:

`aws s3api create-bucket --bucket rsp-sse-demo`

Navigate into the bucket permissions and block all public access to the bucket:

aws s3api put-public-access-block --bucket "rsp-sse-demo" \\ 
--public-access-block-configuration "BlockPublicAcls=true,IgnorePublicAcls=true,\\
BlockPublicPolicy=true,RestrictPublicBuckets=true"

Permissions should say that Block _all_ access in On now. Next, hit the following command and verify that server-side encryption is now enabled:

aws s3api put-bucket-encryption --bucket rsp-sse-demo \\
--server-side-encryption-configuration '{"Rules": \\
\[{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}\]}'

S3 is going to use SSE-S3, a form of server-side encryption. Lets break down the types of SSE within S3.

#### Type of S3 Server Side Encryption

1.  **SSE-S3**: The standard server-side encryption when first enabled. S3 will manage encryption keys and rotate as needed. The administrators and users of the bucket will not have to think about the keys being used to encrypt their data
    
2.  **SSE-KMS**: Uses a key stored in KMS to encrypt and decrypt S3 object. This can be configured through KMS to keep track of all keys being used in one place.
    
3.  **SSE-C:** Upload customer specific keys for cryptographic operations. This is normally used when vendors have compliance obligations to completely manage their keys.
    

Document generated by Confluence on Jan 31, 2022 19:25

[Atlassian](http://www.atlassian.com/)
