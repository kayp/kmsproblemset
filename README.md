# kmsproblemset
Creating and Deploying KMS Keys in Client Accounts from AWS Admin account 


This repo was created in response to a questionnaire on creating AWS-KMS keys in one account and assign appropriate roles/permissions to multiple accounts t use the keys. To make the code modular and compact  nested templates were created. An additional template was also used to make keys for admin account only.


Stack sets were used to grant users permission to used in various accounts appropriate KMS permissions.

The following code samples are available in this repo:


(i) The master template that calls all the stacks


(ii) The nested template that takes a parameter and creates keys for the user.


(iii) One template only for host.

(iv) One template for granting appropriate permissions to users (not admin or root) i other accounts. This process assues the appropriate trust relationship
between admin key accounand client AWS accounts was created.

<b>A use case of KMS is presented below</b>
<p>
 KMS can be used to make improve security. For example AccountD may transfer a file to a user in AccountB with key permissions. The user B with permissions to use the key can decrypt the file. The same file hands even if acquired by an unauthorized entity would not be readable without appropriate permissions.
Good examples are available in AWS cli section 
https://docs.aws.amazon.com/cli/latest/reference/kms/encrypt.html
As an example account D can encrypt the file as follows.
<br> <b><br>
aws kms encrypt --key-id alias/AccountB --plaintext fileb://ExamplePlaintextFile --output text --query CiphertextBlob | base64 --decode > ExampleEncryptedFile
</b><br><br>
The file can be shared with AccountB or any other account included in the keys "KeyPolicy" in multiple ways such as ftp, placed in a shared drive or kept in S3. The file can be opened up via cli as follows-
<br> <b><br>
aws kms decrypt --ciphertext-blob fileb://ExampleEncryptedFile --output text --query Plaintext | base64 --decode > ExamplePlaintextFile
</b><br><br>

While decryption, no key-id alias is needed as the key is decrypted based on the key policy created in cloud formation.

The file can be encrypted or decrypted in S3 without CLI for those with console access. The file is decrypted in S3 based on the key policy. 
</p>   

<h4> Bringing own keys to AWS KMS </h4>
<p>
Customer Keys can be imported via AWS CLI or console. A secure token is generated to wrap the keys during import process. Once key is imported it can be accessed via acccounts like regular CMK. <br>

<br>
The following are pros and cons of AWS-KMS CMK versus BYOK.<br>
<b>
<br>AWS-KMS CMK advantages:</b> <br><br>
(i) Key management is easier.<br>
(ii) Keys are Highly Available and durable<br> 
(iii) AWS rotates key on its own schedule (~1 year) <br>
(iv) Region specific keys makes region based compliance easier (eg EMEA vs FDA).<br>
(v) While rotating keys, AWS automatically manages decryption/encryption with new keys<br>
<b><br>AWS-KMS CMK disadvantages: </b><br><br>

(i) Key rotation is on AWS schedule. Legal or business requirements may require more frequent changes.
<br>
(ii) AWS KMS manages symmteric keys. There might be a business or legal requirement for asymmetric keys. In general one will depend on AWS's cryptography rather than own organization's need. 
<br>
(iii) Keys are region specific. This can create complications due to cloud formation coding style (my answer to problem 2 also has this issue --   
{"Ref":"AWS::Region"} vs "us-east-1" or elsewhere). This can lead to unexpected errors.<br>
(iv) The data cannot be stored and decrypted outside AWS.<br>
<p>
