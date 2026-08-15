This lab is about file encryption and setting up a file recovery agent. Before explaining the lab, Id like to explain
a few things.

HOW FILE ENCRYPTION WORKS:
When a user encrypts a file, the file gets encrypted by the FEK(file encryption key). This is a symmetrical key meaning
it is used to encrypt and decrypt. Now that FEK gets encrypted by the user's public key which can then be decrypted by their
private key. This concept is called public key infrastructure. 

HOW RECOVERY AGENT WORKS:
Adding a recovery agent basically means having another user to encrypt the FEK and decrypt the FEK. Meaning that when a 
user encrypts a file, the FEK not only gets encrypted by the user's public key, but also the recovery agent's public key.
In that way, if the user looses their private key, the recovery agent's private key can be used.

STEPS FOR SETTING UP RECOVERY AGENT:
1. In admin account, create the recovery agent by going into powershell and typing cipher /r:recover. This should create 2 files
2. The .cer file contains the certificate and public key (encryption)while .pfx contains certificate, public and private key (decryption)
3. Import the .pfx file to the admin's certificate store. This gives the admin the private key of the recovery agent allowing
   the admin to decrypt and see encrypted files from users
4. Import the .cer file into encrypting file system which is located in local security policy. This basically makes all local
   users have the recovery agent use its public key as well when encrypting their FEK.

