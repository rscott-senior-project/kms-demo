<!DOCTYPE html>
<html>
    <head>
        <title>AWS Developer Associate Content Series : Encryption in AWS - Demo</title>
        <link rel="stylesheet" href="styles/site.css" type="text/css" />
        <META http-equiv="Content-Type" content="text/html; charset=UTF-8">
    </head>

    <body class="theme-default aui-theme-default">
        <div id="page">
            <div id="main" class="aui-page-panel">
                <div id="main-header">
                    <div id="breadcrumb-section">
                        <ol id="breadcrumbs">
                            <li class="first">
                                <span><a href="index.html">AWS Developer Associate Content Series</a></span>
                            </li>
                                                    <li>
                                <span><a href="AWS-Developer-Associate-Content-Series_33099.html">AWS Developer Associate Content Series</a></span>
                            </li>
                                                    <li>
                                <span><a href="Domains_589841.html">Domains</a></span>
                            </li>
                                                    <li>
                                <span><a href="Security_589827.html">Security</a></span>
                            </li>
                                                </ol>
                    </div>
                    <h1 id="title-heading" class="pagetitle">
                                                <span id="title-text">
                            AWS Developer Associate Content Series : Encryption in AWS - Demo
                        </span>
                    </h1>
                </div>

                <div id="content" class="view">
                    <div class="page-metadata">
                        
        
    
        
    
        
        
            Created by <span class='author'> Rob Scott</span>, last modified on Jan 26, 2022
                        </div>
                    <div id="main-content" class="wiki-content group">
                    <ul class="inline-task-list" data-inline-tasks-content-id="1835027"><li class="checked" data-inline-task-id="9"><span class="placeholder-inline-tasks">Client Side encryption with AWS KMS</span></li><li class="checked" data-inline-task-id="10"><span class="placeholder-inline-tasks">Server side encryption (example in S3)</span></li><li class="checked" data-inline-task-id="11"><span class="placeholder-inline-tasks">envelope encryption with AWS KMS data key</span></li><li class="checked" data-inline-task-id="13"><span class="placeholder-inline-tasks">Type of Keys</span></li></ul><p /><h2 id="EncryptioninAWS-Demo-ClientSideEncryptionwithAWSKeyManagementService">Client Side Encryption with AWS Key Management Service</h2><p>Client side encryption is done to guarantee that data is encrypted while entering AWS Services. KMS can secure encryption keys and help this process. The CLI will allow you to encrypt and decrypt plaintext before sending it over the wire to, say, another S3 resource.</p><p>Check to see if there are any existing keys available for use</p><p><code>aws kms list-keys</code></p><p>If there are no keys, go to the AWS Console and navigate to Key Management Service. There, generate a new key with as the admin user and make sure that your admin account is specified as an eligible user of the cryptographic operations. </p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">{
    &quot;Id&quot;: &quot;key-consolepolicy-3&quot;,
    &quot;Version&quot;: &quot;2012-10-17&quot;,
    &quot;Statement&quot;: [
        {
            &quot;Sid&quot;: &quot;Enable IAM User Permissions&quot;,
            &quot;Effect&quot;: &quot;Allow&quot;,
            &quot;Principal&quot;: {
                &quot;AWS&quot;: &quot;arn:aws:iam::170976071768:root&quot;
            },
            &quot;Action&quot;: &quot;kms:*&quot;,
            &quot;Resource&quot;: &quot;*&quot;
        },
        {
            &quot;Sid&quot;: &quot;Allow access for Key Administrators&quot;,
            &quot;Effect&quot;: &quot;Allow&quot;,
            &quot;Principal&quot;: {
                &quot;AWS&quot;: &quot;arn:aws:iam::170976071768:user/rob-senior-project-admin&quot;
            },
            &quot;Action&quot;: [
                &quot;kms:Create*&quot;,
                &quot;kms:Describe*&quot;,
                &quot;kms:Enable*&quot;,
                &quot;kms:List*&quot;,
                &quot;kms:Put*&quot;,
                &quot;kms:Update*&quot;,
                &quot;kms:Revoke*&quot;,
                &quot;kms:Disable*&quot;,
                &quot;kms:Get*&quot;,
                &quot;kms:Delete*&quot;,
                &quot;kms:TagResource&quot;,
                &quot;kms:UntagResource&quot;,
                &quot;kms:ScheduleKeyDeletion&quot;,
                &quot;kms:CancelKeyDeletion&quot;
            ],
            &quot;Resource&quot;: &quot;*&quot;
        },
        {
            &quot;Sid&quot;: &quot;Allow use of the key&quot;,
            &quot;Effect&quot;: &quot;Allow&quot;,
            &quot;Principal&quot;: {
                &quot;AWS&quot;: &quot;arn:aws:iam::170976071768:user/rob-senior-project-admin&quot;
            },
            &quot;Action&quot;: [
                &quot;kms:Encrypt&quot;,
                &quot;kms:Decrypt&quot;,
                &quot;kms:ReEncrypt*&quot;,
                &quot;kms:GenerateDataKey*&quot;,
                &quot;kms:DescribeKey&quot;
            ],
            &quot;Resource&quot;: &quot;*&quot;
        },
    ]
}</pre>
</div></div><p>List all KMS keys again and make sure it is there now. That key ID will be used to encrypt and decrypt our plaintext.</p><p><code>echo &quot;secret database string&quot; &gt;&gt; db-string.txt</code></p><p>Lets pretend that we need to encrypt this database string before uploading it to S3. Use this CLI command to use the KMS key and put it back in its own encrypted file on our system.</p><p><code>aws kms encrypt --key-id &lt;KEY-ID&gt; --plaintext fileb://db-string.txt --query &quot;CiphertextBlob&quot; &gt; encrypted</code></p><p>Breaking a few things down:</p><ul><li><p>The <code>fileb://</code> is a flag for the CLI to take a file in as plaintext, rather than a string as an arg</p></li><li><p><code>--query</code> selects the part of the CLI response we want, which is the ciphertext</p></li><li><p><code>&gt; encrypted</code> redirects our chosen output to a file, where our encrypted text is stored.</p></li></ul><p>To decrypt the ciphertext, we will use the CLI again:</p><p><code>aws kms decrypt --ciphertext-blob $(cat encrypted) --query Plaintext --output text | base64 --decode</code></p><p>Notice how <code>decrypt</code> did not require the <code>key-id</code> to decrypt the ciphertext. Thats because the ciphertext blob has metadata attached to choose the appropriate key for decryption.</p><h2 id="EncryptioninAWS-Demo-EnvelopeEncryptionwithKeyManagementService">Envelope Encryption with Key Management Service</h2><p>Envelope encryption is used to encrypt large amounts of data that would otherwise be hard to send to AWS KMS servers. This may be more efficient, but now our encryption key and encrypted data are in the same place (never good). As a solution, KMS’s implementation of envelope encryption will return two keys, a plaintext copy and encrypted copy. Encrypt the data and promptly dispose of the plaintext key. The encrypted key can reside with the data, and then be decrypted via KMS and used when the data needs to be decrypted. The plaintext key can be used to decrypt the ciphertext.</p><h4 id="EncryptioninAWS-Demo-Procedure">Procedure</h4><p>Install the AWS Encryption CLI following the instruction the AWS docs. After its installed, we will use our existing KMS key to perform envelope encryption.</p><p>First, make another example file to operate on:</p><p><code>echo &quot;very large file&quot; &gt;&gt; large-data.txt</code></p><p>Next, the Encryption CLI will take the ARN of your existing key to encrypt a data key. This data key is part of the ciphertext generated from the encryption. When its ready to be decrypted, the encrypted data key is identified and decrypted to a plaintext key via KMS. This means that the CLI only needs the ARN of our existing key within KMS; the data key is generated and handled within the command.</p><p>To get the ARN of our KMS key, hit the following command:</p><p><code>aws kms list-keys</code></p><p>To encrypt the file, use the following command:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">aws-encryption-cli --encrypt \
--input large-data.txt \
--wrapping-keys key=&lt;KEY_ARN&gt; \
--metadata-output ./metadata \
--output encrypted-large-data</pre>
</div></div><p>Now our data is encrypted within <code>encrypted-large-data</code>. The data key was never handled by use, but it was generated, encrypted our file as plaintext, and then encrypted using the <code>wrapping-keys</code> parameter.</p><p>To decrypt the data, we will use a similar decrypt command:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">aws-encryption-cli --decrypt \
--input encrypted-large-data \
--wrapping-keys key=&lt;KEY_ARN&gt; \
--metadata-output ./metadata \
--output decrypted-large-data.txt</pre>
</div></div><h2 id="EncryptioninAWS-Demo-EnablingServer-SideEncryption">Enabling Server-Side Encryption</h2><p>Server-side encryption can be used on several storage services in an AWS environment. For this example, we will create an S3 bucket and then do two things:</p><ul><li><p>Restrict public access to the bucket; this is often standard unless the bucket is hosting a website</p></li><li><p>Configure server-side encryption on the bucket</p></li></ul><p>The latter is the most important operation of this demonstration. Open up the console and navigate to the S3 page to verify that changes are being made through the CLI. Its important to get used to using the CLI because these procedures can be turned into scripts and favor automation throughout any given infrastructure.</p><p>Create the bucket using the CLI and verify that the bucket now exists in the console:</p><p><code>aws s3api create-bucket --bucket rsp-sse-demo</code></p><p>Navigate into the bucket permissions and block all public access to the bucket:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">aws s3api put-public-access-block --bucket &quot;rsp-sse-demo&quot; \ 
--public-access-block-configuration &quot;BlockPublicAcls=true,IgnorePublicAcls=true,\
BlockPublicPolicy=true,RestrictPublicBuckets=true&quot;</pre>
</div></div><p>Permissions should say that Block <em>all</em> access in On now. Next, hit the following command and verify that server-side encryption is now enabled:</p><div class="code panel pdl" style="border-width: 1px;"><div class="codeContent panelContent pdl">
<pre class="syntaxhighlighter-pre" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence">aws s3api put-bucket-encryption --bucket rsp-sse-demo \
--server-side-encryption-configuration &#39;{&quot;Rules&quot;: \
[{&quot;ApplyServerSideEncryptionByDefault&quot;: {&quot;SSEAlgorithm&quot;: &quot;AES256&quot;}}]}&#39;</pre>
</div></div><p>S3 is going to use SSE-S3, a form of server-side encryption. Lets break down the types of SSE within S3.</p><h4 id="EncryptioninAWS-Demo-TypeofS3ServerSideEncryption">Type of S3 Server Side Encryption</h4><ol><li><p><strong>SSE-S3</strong>: The standard server-side encryption when first enabled. S3 will manage encryption keys and rotate as needed. The administrators and users of the bucket will not have to think about the keys being used to encrypt their data</p></li><li><p><strong>SSE-KMS</strong>: Uses a key stored in KMS to encrypt and decrypt S3 object. This can be configured through KMS to keep track of all keys being used in one place.</p></li><li><p><strong>SSE-C: </strong>Upload customer specific keys for cryptographic operations. This is normally used when vendors have compliance obligations to completely manage their keys.</p></li></ol><p /><p />
                    </div>

                    
                                                      
                </div>             </div> 
            <div id="footer" role="contentinfo">
                <section class="footer-body">
                    <p>Document generated by Confluence on Jan 31, 2022 19:25</p>
                    <div id="footer-logo"><a href="http://www.atlassian.com/">Atlassian</a></div>
                </section>
            </div>
        </div>     </body>
</html>
