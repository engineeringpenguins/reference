# Active Directory Certificate Serivces  

Active Directory (AD) Certificate Services (CS) can be pretty rough to work with when things dont behave correctly. This article will run through everything you would need to be conversational (read: complain to sysadmins) about ADCS.

## Introduction and Background ('How to SSL good')  

We can't talk about any kind of Certificate Authority (CA) without highlighting operations the internet takes when using SSL. In ye olden times when a host accessed a remote file it would communicate without any encryption. People were not as mean in ye olden times and the internet was a much smaller place so your PC broadcasting potentially confidential information everywhere did not present the same concerns that it does today. Unfortunately, people are more mean now and so we have to be careful what unauthorized parties can see.  

Prior to encryption technologies like SSL, when a client connected to a server it would use a protocol like HTTP, FTP, Telnet, SMB1, SNMP1, etc. These protocols would communicate over the network (very simple analogy) by essentially sending text files between client and server. Every other machine on that network could observe these communications and read the text files being exchanged. It might not be a big deal if someone overhears your phone call with a coworker, but you probably wouldnt want someone to listen in on a phone call to your bank. One way to obfuscate the contents of your phone calls would be to speak in a foregin language that anyone listening doesnt know. Essentially that is what encryption does to data protected by SSL.  

![comparing encrypted vs unencrypted local traffic](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/adcs_lan.png)  

Now, when a client is connected to a server it might use a protocol like HTTPS, FTPS/VSFTP/SCP, SSH, SMB3, SNMP3, etc. These protocols communicate over the network (repeating simple analogy) by sending encrypted files written in a made up language that only the client and server know about. The 'made up language' in this case is the SSL certificate.  

SSL certificates provide the identity of a domain name (ex. google.com) and what authority is willing to validate this identity. You can view the SSL certificate of a website in your web browser (process varies, just google how to do this) or in a terminal: openssl s_client -connect github.com:443 -showcerts. Regardless of if you view the certificate in your browser, terminal, or the certificate manager in your OS, you will see the same content:  

Highlevel identity information about the certificate and its validity:  

- Who is this host (who is the certificate intended to be used for) and who issued the certificate  
- What encryption methods are used in this certificate  
- When the certificate was issued and when it will expire  
- Where the certificate reports the host exists geographically (sometimes left blank)  
- How this certificate is allowed to be used (unique fields and requirements that can be configured)  

The 'certificate chain' or 'certificate hierarchy' will provide a tree view of whatever authorities were involved in signing this certificate:  

- Also called 'the chain of trust', it shows the root CA and any intermediate CA's that signed this certificate  
- Your device will not trust the certificate unless it knows every member of the chain (the root and any intermediate authorities)  

![Example of github ssl certification](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/adcs_cert.png)  

## Preparing Active Directory for Certificate Services  

There are a ton of ways to setup different CA topologies but using ADCS is one of the easiest and most common methods. If you leverage AD a lot in your enviornment then using ADCS for your CA's could make sense. ADCS is fairly simple to set up and administer but as a result it is not very feature rich and tends to be very bare bones. Another thing to keep in mind is that the more things you integrate with AD the more vulnerabilities you are introducing into the core of your enviornment. It can be catastrophic if your AD root CA was compromised by a malicious party as they now own your entire AD enviornment. You can harden your position here in a few different ways but it is not uncommon to have your root CA be a non AD integrated machine that is powered off at all times except when undergoing maintenance and updates.  

Today I'm going to build an AD integrated root CA and not worry about setting up an entire CA toplogy.Setting up a non AD integrated CA is nearly identical to the process for an AD CA, the only exception being that the host machine is not joined to AD and there is no option during setup to integrate with AD.  

I'm using Windows Server 2022 for this example but anything previous Server releases will work. Start by using the Server Manager to add roles and features, 'Active Directory Certificate Services' should be the first role you can add so just check the box and click 'Next' through the following prompts. I personally like to enable 'Certificate Authority Web Enrollment' on one of the following screens so that domain admins can access the CA via internal website and request certificates without having to remote into the ADCS host.  

![Begin setup of ADCS](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/adcs_setup.png)  

Use appropriate level credentials to approve configuration of ADCS, check the box for ADCS (dont check other services we enabled yet), leave the radio button selected for 'Enterprise CA', select 'root CA', select 'create a new private key', leave the default at RSA 2048, leave the name as default (if you want to change it I would reccomend having '-CA' on the end of the name for organization), leave the validity period at 5 years, leave the default location for the certificate database, select configure to finish setup.

There are only a few places that you will likely go in your time using ADCS. If you left click on the name of your CA in the navigation column and then right click the name of the CA you will get a selection menu, hover over 'All tasks' with your mouse.  

- The most common thing you will do is 'submit a new request', this is where you would upload a CSR/REQ/DER file and the CA would provide you a signed certificate.  
- You may also select 'Backup CA ...' and export a list of all the certificates in your CA as well as the records for the CA itself.  
  - You could leverage GPO or Intune for this but the process wont be explained in this article.  
By selecting 'Pending requests' you would see any certificates that were submitted via AD (not manually submitted using bullet point above) or IIS.  
- Right click and approve or deny the certificates from this list.  
If you select 'Issued Certificates' you get a list of all certificates **this** CA has issued.
- Right click and revoke or export certificates for redeployment.  

![ADCS common operations](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/adcs_use.png)  

## Certificate Assignment & Enrollment  

Active Directory will not automatically tell AD joined PC's about the new root CA/Subordinate CA. For AD joined devices running Windows you can use Group Policy Objects to save certificates to the local 'trusted store'. You will need to get the public certificate (.cer) of each CA in the certificate chain and import it into AD.

Start by logging into your Root CA:

1. Open the Certificate Services MMC
2. Right click on the root CA in the navigation pane
3. Hover over 'All Tasks' and click 'Backup CA' and click 'Next'
4. Check the box for 'Private key and CA certificate'
5. Click the 'Browse' button and navigate to the Downloads directory
6. In the filepath to the left of browse, add a '\tmp\' after the '\Downloads'
7. Click 'OK' on the dialogue box asking if you want to make the \tmp\ directory
8. Use a good password on the following prompt to encrypt the file
9. Click 'Finish' to complete the export to a .PFX file

Log into a Domain Controller to continue:  

1. Copy the .PFX file from the previous step to the DC
2. Open the Group Policy MMC
3. Create a new GPO or edit an existing one as makes sense
4. Navigate to
   1. Computer Configuration
      1. Policies
         1. Windows Settings
            1. Security Settings
               1. Public Key Policies
5. Right click on the 'Trusted Root Certification Authorities' container/folder and click Import
6. Click 'Browse' and select the .PFX file from the previous step and click 'Next'
   1. you may need to change the filetype its looking for using the dropdown in the lower right
7. Put in the password you used to encrypt the file
   1. Do no check the box to make the key exportable
   2. Check the box to Include all extended properties
8. Click 'Next' and 'Finish'
9. Right click on the 'Intermediate Certification Authorities' container/folder and click Import
10. Repeat previous steps but log into the subordinate CA
    1. Do this for every CA in the certificate chain

## Terms (Buzzwords)  

- **Certificate** - File that encrypts communication or data (aka public key certificate)  
- **Encryption** - Proceess that results in altered data that can only be restored by knowing the algorithm and password used to change it  
- **RSA** - Mathematical algorithm that encrypts/decrypts data using a public/private key pair
- **Wildcard** - Way to include every variation of a domain with a single definition (ex. *.google.com)  
- **PKI** - Public Key Infrastructure, the system or framework you use to store/manage/approve/revoke/issue certificates  

### Certificate Authorities:  

- **CA** - Certificate Authority, entity that signs certificates  
- **Root CA** - Root Certificate Authority, the highest level of CA and the CA that authorizes other CAs to sign certificates  
- **Intermediate CA** - Intermediate Certificate Authority, a CA approved by the root CA to sign certificates on behalf of the root CA  
  - I call these proxy CA's despite that likely being a very incorrect term  
- **Sub CA** - Subordinate Certificate Authority, synonym for Intermediate CA  

### Protocols:  

- **SSL** - Secure Socket Layer, a protocol that encrypts data  
- **TLS** - Transport Layer Security, successor to SSL and is a protocol that encrypts data  
  - TLS is often refered to as SSL even though they are different protocols, dont get confused  
- **SCEP** - Simple Certificate Enrollment Protocol, way to request a certificate for non AD devices (ex. switches)
- **HTTP, FTP, Telnet, SMB1, SNMP1** - unencrypted communication protocols referenced in this article  
- **OCSP** - Online Certificate Status Protocol, used in tandem with CRLs to determine validity of an issued certificate  
- **HTTPS, FTPS/VSFTP/SCP, SSH, SMB3, SNMP3** - encrypted communication protocols referenced in this article  

### Keys:

- **Public Key** - used to encrypt data, cannot decrypt any encrypted data  
- **Private Key** - used to decrypt data, cannot encrypt any data  

### Filestypes:

- **PKCS7** - Public Key Certificate System #7, an all encompassing certificate that contains a public key and a private key  
- **PFX** - Personal Information Exchange, synonomous with PKCS7  
- **CER** - Certificate file (synonomous with DER or PEM files), binary format with no private key  
- **CSR** - Certificate Signing Request, file that contains a hosts public key and is submitted to a CA for signing  
- **REQ** - Certificate Request, file that contains a hosts public key and is submitted to a CA for signing  

### ADCS Stuff:

- **ADCS** - Active Directory Certificate Services, Certificate management in Active Directory  
- **CRL** - Certificate Revocation List, list of revoked certificates that all CA's in the PKI must be agree (sync) on  

## OpenSSL and CertReq/Util Troubleshooting

- Check if OSCP thinks example.pem is revoked  
  'openssl ocsp -issuer issuer_cert.pem -cert example.pem -text -url http://ocsp.responder.url'  
- Print a certificate file to the console  
  'openssl x509 -in certificate.crt -text -noout'  
- Print a certificate from a website to the console  
  'openssl s_client -connect example.com:443 -showcerts'  

- Submit a request via AD to a CA  
  'certreq -submit -attrib "CertificateTemplate:SubCA" C:/Users/example/Downloads/example.cer'  
- Check all CRL references for the local CA certificate  
  'certutil -verify -urlfetch myCACertificate.crt'  
- Flush CRL cache on a CA  
  'certutil -urlcache * delete'  
- Manually pull CRL from another CA  
  'certutil -url [CRL URL]'  
- Output CRL to console  
  'certutil -dump example.crl'  
- Verbose output of CRL download  
  'certutil -URLcache CRL http://example.example/certenroll/example.crl'  
- Add a new Root CA using cert file  
  'certutil -addstore "Root" newRootCA.crt'  

## MISC. Operations - not used or currently relevant  

Export Public Key of a CA:

1. Open the Certificate Services MMC
2. Right click on the root CA in the navigation pane and click 'properties'
3. The 'General' tab will open, click 'view certificate'
4. Click the details tab on the following menu
5. Select 'copy to file'
6. Use DER encoded binary and proceed by clicking 'Next'
7. Click 'Browse' and navigate to your target directory (if temporary I use Downloads)
8. Type your file name in the dialog box and click 'Save' followed by 'Next' and 'Finish'

## Concepts Summary

For a machine to trust a website using SSL the machine must 'trust' the certificate the website is using. At minimum there will be one Certificate Authority that signs a certificate (like a CEO signing approval on a document) but usually there are more than one Certificate Authorities signing a document (The CEO has authorized the CFO to sign a document with the CEO's authority). For a machine to trust that certificate it must trust every certificate authority listed in the certificate chain.  

## Improving this article

If there is a usecase for it, I will add more content to this article:

- Online enrollment via SCEP/NDES  
- Usecases for ADCS  
- Creating new SSL certificates
