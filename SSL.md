# **Deep Dive into SSL certificates**

## **Types of encryption**

&nbsp;

There are 2 types of encryption mechanisms used today.
 

1. Symmetric - [Click for more](https://sectigostore.com/blog/what-is-asymmetric-encryption-how-does-it-work/) 
 

> *Here the encryption and decryption are done using the same key.*

&nbsp;

2. Asymmetric - [Click for more](https://sectigostore.com/blog/what-is-asymmetric-encryption-how-does-it-work/) 


> *Here the encryption is done using a key and decryption is done using another key, hence asymmetric.*

&nbsp;

SSL encryption security works on asymmetric encryption,Asymmetric encryption works on two cryptographic keys, i.e., the public key and the private key. It is the basis of PKI (Public Key Infrastructure).

> *The private key is private to the one who is wanting to send data out to the public. The public key is shared with the world. Data encrypted using private key can only be decrypted using the public key and vice versa. This is how the distribution issue is avoided in the asymmetric encryption mechanisms.*


&nbsp;

## **The life cycle of SSL certificates**

&nbsp;

## Step 1: Create an RSA Private Key and CSR


&nbsp;

It is advised to issue a new private key each time you generate a CSR. Hence, the steps below instruct on how to generate both the private key and the CSR.

```
openssl req -new -newkey rsa:2048 -nodes -keyout your_domain.key -out your_domain.csr
```
> *NOTE: Make sure to replace **<your_domain>**  with the actual domain you’re generating a CSR for.*

&nbsp;

Here there will be several questions that we need to answer, like:

![image](https://raw.githubusercontent.com/jigarsoni17/imagesformd/main/seagage-csr-image.png)

&nbsp;

After all these questions are answered, we will get a .csr file & private key.

![image](https://raw.githubusercontent.com/jigarsoni17/imagesformd/main/seagage_com.csr-image.png)

#### Pic 2. A sample CSR file. Notice the CSR file is only containing the PUBLIC key and some metadata.

&nbsp;

## Step 2: Apply for a CSR(Certificate Signing Request)


&nbsp;

Now, Login into your certificate provider in our case its *https://www.namecheap.com/* .

![seagage](https://raw.githubusercontent.com/jigarsoni17/imagesformd/main/seagage-csr.gif)

--- 
&nbsp;

Now, Apply for the a CSR by following steps:
- Navigate ***SSL Certificates*** from the left sideof pannel.
- Find your Domain Name and click on ***ACTIVATE***.
- Now paste your ***CSR*** file content, in our case its the file that we created using openssl command (Pic-1), Click Next.
- Enter DCV Methos choose ***Add CNAME Record***, Click Next. 
- Enter Admin email address, Click Next.
- Verify details that you provide and click Submit.
- Now there is the hyper link ***Get a [CNAME](https://www.techtarget.com/searchwindowsserver/definition/canonical-name) record from this page (Edit methods)***, click on that link. 

&nbsp;

![namecheapsteps](https://raw.githubusercontent.com/jigarsoni17/imagesformd/main/seagage-steps.gif)

---
&nbsp;

After clicking that link you will see one option called ***EDIT METHODS***, under your domain list, click on it and you will get another button named ***Get Record***. Click on it and you will see one pop-up window like this:

![pop-up](https://raw.githubusercontent.com/jigarsoni17/imagesformd/main/step-9.png)

- Suppose Under ***Host*** option your record is created with value ***"_DAWHDWBNNFEINB.<your_domain>"***. 
- Now Copy the value before your_domain name and Add CNAME record into AWS Route 53 and copy the value of ***TARGET*** and add into AWS Route 53's target under the same hosted zone, click save and exit.
- After waiting for 4 to 5 minutes certificate is ready and there is option for download on namecheap, so download it.
- it inclueds total three files as shown in below image, one is Domain certificate file second one is ***CA bundle*** file and third one is .p7b file. 

&nbsp;


> ## *NOTE: If you dont know what is CA bundle file and what is fullchain certificate please [click here](https://www.ssldragon.com/blog/what-is-a-ca-bundle/)*.

&nbsp;

## Step 3: Validate Private key and Domain certificate with MD5


You could probably just try to install your new certificate and private key, reload your webserver config, and see if it works. But that’s not very convenient if you want to validate your private key and certificate beforehand.

So, how do you verify that a private key matches your certificate and that they’re valid?

&nbsp;

**Calculate MD5 hash of private key**
```
openssl rsa -noout -modulus -in /path/to/your/private.key  2> /dev/null | openssl md5
```
- Sample output: **(stdin)= 3a5a1682678d243b6b8337360b55ff10**

&nbsp;

**Calculate MD5 hash of certificate**

```
openssl x509 -noout -modulus -in /path/to/your/certificate.crt 2> /dev/null | openssl md5
```
- Sample output: **(stdin)= 3a5a1682678d243b6b8337360b55ff10**

&nbsp;


The MD5 hash from the private key and the certificate should be the exact same. If they’re not, the private key can not be used together with the certificate and something in the CSR process has probably gone wrong. This can mean a wrong CSR was used, a wrong private key was stored, … Up to you to find out. ;-)

&nbsp;

## Step 4: Importing certificates into AWS Certificate Manager


SSL/TLS certificates provided by AWS Certificate Manager (ACM), you can import certificates that you obtained outside of AWS. You might do this because you already have a certificate from a third-party certificate authority (CA), or because you have application-specific requirements that are not met by ACM issued certificates.

## Importing a certificate

Import (console)

The following example shows how to import a certificate using the AWS Management Console.

- Open the ACM console at https://console.aws.amazon.com/acm/home, If this is your first time using ACM, look for the AWS Certificate Manager heading and choose the Get started button under it.

- Choose Import a certificate.

- Do the following:

    - For Certificate body, paste the PEM-encoded certificate to import. It should begin with -----BEGIN CERTIFICATE----- and end with -----END CERTIFICATE-----.

    - For Certificate private key, paste the certificate's PEM-encoded, unencrypted private key. It should begin with -----BEGIN PRIVATE KEY----- and end with -----END PRIVATE KEY-----.

    - (Optional) For Certificate chain, paste the PEM-encoded certificate chain.

- Choose Review and import.

- On the Review and import page, check the displayed metadata about your certificate to ensure that it is what you intended. The fields include:

    - Domains — A list of fully qualified domain names (FQDN) authenticated by the certificate

    - Expires in — The number of days until the certificate expires

    - Public key info — The cryptographic algorithm used to generate the key pair

    - Signature algorithm — The cryptographic algorithm used to create the certificate's signature

    - Can be used with — A list of ACM integrated services that support the type of certificate you are importing

If everything is correct, choose Import, & its done Cheers.... !! 

[Click here for AWS Certificate Manager User guide... ](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html)
