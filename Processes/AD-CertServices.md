# Active Directory Certificate Serivces  

Active Directory (AD) Certificate Services (CS) can be pretty rough to work with when things dont behave correctly. This article should run through everything we would need to be conversational (read: complain to sysadmins) about ADCS.

## Introduction and Background ('How to SSL good')  

We cant talk about any kind of Certificate Authority (CA) without highlighting the operations the internet takes when using SSL. In ye olden times when a host (read: your PC) accessed a remote file (read: a website or shared document) it would communicate without any encryption. People wernt as mean in ye olden times and the internet was a much smaller place so your PC broadcasting potentially confidential information everywhere did not present the same concerns that it does today. Unfortunately, people are more mean now and so we have to be careful what unauthorized parties can see.  

Prior to encryption technologies like SSL, when a client connected to a server it would use a protocol like HTTP, FTP, Telnet, SMB1, SNMP1, etc. These protocols would communicate over the network (very simple analogy) by essentially sending text files between client and server. Every other machine on that network could observe these communications and read the text files being exchanged. It might not be a big deal if someone overhears your phone call with a coworker, but you probably wouldnt want someone to listen in on a phone call to your bank. One way to obfuscate the contents of your phone calls would be to speak in a foregin language that anyone listening doesnt know. Essentially that is what encryption does to data protected by SSL.  

![comparing encrypted vs unencrypted local traffic](https://github.com/engineeringpenguins/reference/tree/main/linked-images/adcs_lan.png)  

Now, when a client is connected to a server it might use a protocol like HTTPS, FTPS/VSFTP/SCP, SSH, SMB3, SNMP3, etc. These protocols communicate over the network (repeating simple analogy) by sending (encrypted) text files written in a made up language that only the client and server know about. The 'made up language' in this case is the SSL certificate.  

SSL certificates provide the identity of a domain name (ex. google.com) and what authority is willing to validate this identity. You can view the SSL certificate of a website in your web browser (process varies, just google how to do this) or in a terminal: openssl s_client -connect github.com:443 -showcerts. Regardless of if you view the certificate in your browser, terminal, or the certificate manager in your OS, you will see the same content:  
Highlevel identity information about the certificate and its validity  

- Who is this host (who is the certificate intended to be used for) and who issued the certificate  
- What encryption methods are used in this certificate  
- When the certificate was issued and when it will expire  
- Where the certificate reports the host exists geographically (sometimes left blank)  
- How this certificate is allowed to be used (unique fields and requirements that can be configured)  

The 'certificate chain' or 'certificate hierarchy' will provide a tree view of whatever authorities were involved in signing this certificate

- Also called 'the chain of trust', it shows the root CA and any intermediate CA's that signed this certificate  
- Your device will not trust the certificate unless it knows every member of the chain (the root and any intermediate authorities)  

![Example of github ssl certification](https://github.com/engineeringpenguins/reference/tree/main/linked-images/adcs_cert.png)  

## Preparing Active Directory for Certificate Services  

There are a ton of ways to setup different CA topologies but using ADCS is one of the easiest and most common methods. If you leverage AD a lot in your enviornment then using ADCS for your CA's could make sense. ADCS is fairly simple to set up and administer but as a result it is not very feature rich and tends to be very bare bones. Another thing to keep in mind is that the more things you integrate with AD the more vulnerabilities you are introducing into the core of your enviornment. It can be catastrophic if your AD root CA was compromised by a malicious party as they now own your entire AD enviornment. You can harden your position here in a few different ways but it is not uncommon to have your root CA be a non AD integrated machine that is powered off at all times except when undergoing maintenance and updates.  

Today I'm going to build an AD integrated root CA and not worry about setting up an entire CA toplogy.Setting up a non AD integrated CA is nearly identical to the process for an AD CA, the only exception being that the host machine is not joined to AD and there is no option during setup to integrate with AD.  

I'm using Windows Server 2022 for this example but anything previous Server releases will work. Start by using the Server Manager to add roles and features, 'Active Directory Certificate Services' should be the first role you can add so just check the box and click 'Next' through the following prompts. I personally like to enable 'Certificate Authority Web Enrollment' on one of the following screens so that domain admins can access the CA via internal website and request certificates without having to remote into the ADCS host.  

![Begin setup of ADCS](https://github.com/engineeringpenguins/reference/tree/main/linked-images/adcs_setup.png)  

Use appropriate level credentials to approve configuration of ADCS, check the boxes for whatever services you added and want to setup now, leave the radio button selected for 'Enterprise CA', select 'root CA', select 'create a new private key', leave the default at RSA 2048, leave the name as default (if you want to change it I would reccomend having '-CA' on the end of the name for organization), leave the validity period at 5 years, leave the default location for the certificate database, select configure to finish setup.

There are only a few places that you will likely go in your time using ADCS. If you left click on the name of your CA in the navigation column and then right click the name of the CA you will get a selection menu, hover over 'All tasks' with your mouse.  

- The most common thing you will do is 'submit a new request', this is where you would upload a CSR/REQ/DER file and the CA would provide you a signed certificate.  
- You may also select 'Backup CA ...' and export a list of all the certificates in your CA as well as the records for the CA itself. You would certainly want to make sure that the CA certificate itself is trusted by any host that will be using or viewing certificates.  
  - You could leverage GPO or Intune for this but the process wont be explained in this article.  
By selecting 'Pending requests' you would see any certificates that were submitted via AD (not manually submitted using bullet point above) or IIS.  
- Right click and approve or deny the certificates from this list.  
If you select 'Issued Certificates' you get a list of all certificates **this** CA has issued.
- Right click and revoke or export certificates for redeployment.  

![ADCS common operations](https://github.com/engineeringpenguins/reference/tree/main/linked-images/adcs_use.png)  

## Terms (Buzzwords)  

Certificate - File that encrypts communication or data (aka public key certificate)  
PKI - Public Key Infrastructure, the system or framework you use to store/manage/approve/revoke/issue certificates  
Encryption - Proceess that results in altered data that can only be restored by knowing the algorithm and password used to change it  
RSA - Mathematical algorithm that encrypts/decrypts data using a public/private key pair
Wildcard - Way to include every variation of a domain with a single definition (ex. *.google.com)  

### Certificate Authorities:  

- CA - Certificate Authority, entity that signs certificates  
- Root CA - Root Certificate Authority, the highest level of CA and the CA that authorizes other CAs to sign certificates  
- Intermediate CA - Intermediate Certificate Authority, a CA approved by the root CA to sign certificates on behalf of the root CA  
  - I call these proxy CA's despite that likely being a very incorrect term  
- Sub CA - Subordinate Certificate Authority, synonym for Intermediate CA  

### Protocols:  

- SSL - Secure Socket Layer, a protocol that encrypts data  
- TLS - Transport Layer Security, successor to SSL and is a protocol that encrypts data  
  - TLS is often refered to as SSL even though they are different protocols, dont get confused  
- HTTP, FTP, Telnet, SMB1, SNMP1 - unencrypted communication protocols referenced in this article  
- HTTPS, FTPS/VSFTP/SCP, SSH, SMB3, SNMP3 - encrypted communication protocols referenced in this article  

### Keys:

- Public Key - used to encrypt data, cannot decrypt any encrypted data  
- Private Key - used to decrypt data, cannot encrypt any data  

### Filestypes:

- PKCS7 - Public Key Certificate System #7, an all encompassing certificate that contains a public key and a private key  
- PFX - Personal Information Exchange, synonomous with PKCS7  
- CER - Certificate file (synonomous with DER or PEM files), binary format with no private key  
- CSR - Certificate Signing Request, file that contains a hosts public key and is submitted to a CA for signing  
- REQ - Certificate Request, file that contains a hosts public key and is submitted to a CA for signing  

### ADCS Stuff:

- ADCS - Active Directory Certificate Services, Certificate management in Active Directory  
- CRL - Certificate Revocation List, list of revoked certificates that all CA's in the PKI must be agree (sync) on  
- OCSP - Online Certificate Status Protocol, used in tandem with CRLs to determine validity of an issued certificate  

## OpenSSL and CertReq/Util Troubleshooting

- Check if OSCP thinks example.pem is revoked - 'openssl ocsp -issuer issuer_cert.pem -cert example.pem -text -url http://ocsp.responder.url'  
- Print a certificate file to the console - 'openssl x509 -in certificate.crt -text -noout'  
- Print a certificate from a website to the console - 'openssl s_client -connect example.com:443 -showcerts'  

- Submit a request via AD to a CA - 'certreq -submit -attrib "CertificateTemplate:SubCA" C:/Users/example/Downloads/example.cer'  
- Check all CRL references for the local CA certificate - 'certutil -verify -urlfetch myCACertificate.crt'  
- Flush CRL cache on a CA - 'certutil -urlcache * delete'  
- Manually pull CRL from another CA - 'certutil -url [CRL URL]'  
- Output CRL to console - 'certutil -dump example.crl'  
- Verbose output of CRL download - 'certutil -URLcache CRL http://example.example/certenroll/example.crl'  
- Add a new Root CA using cert file - 'certutil -addstore "Root" newRootCA.crt'  

## References

I don't have any external references or resources to provide. Its fairly intuitive so lab it up and click around.  

## Concepts Summary

For a machine to trust a website using SSL the machine must 'trust' the certificate the website is using. At minimum there will be one Certificate Authority that signs a certificate (like a CEO signing approval on a document) but usually there are more than one Certificate Authorities signing a document (The CEO has authorized the CFO to sign a document with the CEO's authority). For a machine to trust that certificate it must trust every certificate authority listed in the certificate chain.  
