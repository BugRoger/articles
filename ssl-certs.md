Howto Create a Working SSL-Certificate 
======================================

What you ultimately need is:

  1. A SSL certificate key
  2. A host specific SSL certificate
  3. Optionally, to validate client certificates for SSO, the SAP root certificate

The good news is that you can create all of this yourself. The process works like this:

  1. Create a certificate key
  2. Create a certificate signing request
  3. Submit your request in a self service web form -> You get your host specific certificate. Yay!
  4. Download the SAP root certificate
  5. Remove the passphrase from your key
  6. Install your certifcates 

Let's walk through these steps for a system which should be reachable by [https://expertondemand.wdf.sap.corp](https://expertondemand.wdf.sap.corp "https://expertondemand.wdf.sap.corp").


Create the Certificate Key
--------------------------

    d038720@wdfm00259937a:ssl-certs $ openssl genrsa -des3 -out expertondemand.key 1024
    Generating RSA private key, 1024 bit long modulus
    ..................................................................................++++++
    ......++++++
    e is 65537 (0x10001)
    Enter pass phrase for expertondemand.key:
    Verifying - Enter pass phrase for expertondemand.key:

You need to enter a passphrase. We will remove it again it a later stage. Generally it would be great to keep the passphrase. But who wants to type in the passphrase every time the server needs to be restarted?


Create a certificate signing request
------------------------------------

Next we need to create a certificate signing request. It is very important that the request already contains the fully qualified hostname of the system the certificate will be used for. Otherwise the browsers are going to display phishing warnings. Against common sense the FQDN hast to go into the `Common Name (eg, YOUR name)` field. 

    d038720@wdfm00259937a:ssl-certs $ openssl req -new -key expertondemand.key -out expertondemand.csr
    Enter pass phrase for expertondemand.key:
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [AU]:DE
    State or Province Name (full name) [Some-State]:
    Locality Name (eg, city) []:
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:SAP-AG
    Organizational Unit Name (eg, section) []:Global IT - Web & KM
    Common Name (eg, YOUR name) []:expertondemand.wdf.sap.corp
    Email Address []:michael02.schmidt@sap.com 

    Please enter the following 'extra' attributes
    to be sent with your certificate request
    A challenge password []:
    An optional company name []:

Note again that the fully qualified hostname needs to go into the `Common Name` field. This must be the hostname that will be visible for the browser. Aliases or CNames won't work.


Create the certificate using the security self service 
------------------------------------------------------

Go to: [SAPOnlineCA](https://security.wdf.sap.corp/TCS/cgi-bin/secuWPCA.pl "SAP Online CA - Certificate Signing Request")

Select `SAPNetCA` as CA. Also select `certify the cert. req.` for cmd.

Next past in the content of expertondemand.csr:

    -----BEGIN CERTIFICATE REQUEST-----
    MIIB4zCCAUwCAQAwgaIxCzAJBgNVBAYTAkRFMRMwEQYDVQQIEwpTb21lLVN0YXRl
    MQ8wDQYDVQQKEwZTQVAtQUcxHTAbBgNVBAsUFEdsb2JhbCBJVCAtIFdlYiAmIEtN
    MSQwIgYDVQQDExtleHBlcnRvbmRlbWFuZC53ZGYuc2FwLmNvcnAxKDAmBgkqhkiG
    9w0BCQEWGW1pY2hhZWwwMi5zY2htaWR0QHNhcC5jb20wgZ8wDQYJKoZIhvcNAQEB
    BQADgY0AMIGJAoGBANIPjtH3SKKkQ36xgE8O53MDM/iylSMXYsYMOtoNm//QcUf9
    h+kwu2LO9Wymca01ytu9k76g+9uL1h8tMJUlh5AUGS4GKYA/joFgIiO/vhBtjaF2
    EKUs1SvOB4cMjQVnaOgmqzec9Onb25wmUeiY7G8oubxYOsJKBPVs9i9VlPdNAgMB
    AAGgADANBgkqhkiG9w0BAQUFAAOBgQAR9bW0oksxQ65tJ6gtOGLArvgBVpCe/7s4
    WE81xXPwj2kGCt0ZhYsp+0ka+uoz/HnGh82x1WXECwsH3orNULr8algUbkDpmYE8
    rRTtkG0UvrKlxvzT9AY+b+JxAybGNCBlEVmRv3fyzH2ErXWbq1kZAHdTXJDZ4aM8
    UyqLlZ0rpQ==
    -----END CERTIFICATE REQUEST-----

The certificate will appear in the lower frame. Copy&Paste the following part:

    -----BEGIN CERTIFICATE-----
    MIIC6TCCAlKgAwIBAgIEAQA5PzANBgkqhkiG9w0BAQUFADBpMQswCQYDVQQGEwJE
    RTEPMA0GA1UEChMGU0FQLUFHMQ8wDQYDVQQLEwZTQVBOZXQxETAPBgNVBAMTCFNB
    UE5ldENBMSUwIwYJKoZIhvcNAQkBFhZtYWlrLm11ZWxsZXJAc2FwLWFnLmRlMB4X
    DTExMDQwNjE1MjYyMloXDTEzMDQwNTE1MjYyMlowYzELMAkGA1UEBhMCREUxDzAN
    BgNVBAoTBlNBUC1BRzEdMBsGA1UECxQUR2xvYmFsIElUIC0gV2ViICYgS00xJDAi
    BgNVBAMTG2V4cGVydG9uZGVtYW5kLndkZi5zYXAuY29ycDCBnzANBgkqhkiG9w0B
    AQEFAAOBjQAwgYkCgYEA0g+O0fdIoqRDfrGATw7ncwMz+LKVIxdixgw62g2b/9Bx
    R/2H6TC7Ys71bKZxrTXK272TvqD724vWHy0wlSWHkBQZLgYpgD+OgWAiI7++EG2N
    oXYQpSzVK84HhwyNBWdo6CarN5z06dvbnCZR6Jjsbyi5vFg6wkoE9Wz2L1WU900C
    AwEAAaOBozCBoDAJBgNVHRMEAjAAMCQGA1UdEQQdMBuBGUQwMzg3MjBAZXhjaGFu
    Z2Uuc2FwLmNvcnAwHQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMCMA4GA1Ud
    DwEB/wQEAwIE8DAdBgNVHQ4EFgQUkOHwgilbqa67bE9si5tH1oudKtswHwYDVR0j
    BBgwFoAUewwvmTxL4JMG59r3f+pykWI5oMkwDQYJKoZIhvcNAQEFBQADgYEAZJLG
    jJ8bkJ5DQC3h4g7ZsE+NpqhtYjxpI2pAkO5uaRl7xhEcu9grfzWnM8QKDuwulfRs
    3K8vgZKjg7/ZVEFHjNqLjxcXvGX4fwUUBNK9bTXGtvkRhxsaJ9LS/prryr1bcjT5
    IqW1DcHhn2oQakkRqrOXWZfnNMILkt8xokcMPOQ=
    -----END CERTIFICATE-----

Save it as `expertondemand.crt`


Download the SAP root certificate
---------------------------------

On [TCSRootCert](https://sapneth4.wdf.sap.corp/TCSRootCert "SAP Onlince CA - TCSRootCert") you will find SAP's root certificates. The one we are interessted in is the SSO CA Certificate.

Servers need this root certificate to verify the SAP-internal client certificates. Therefore, you have to import this certificate into your web servers, to authenticate SAP employees who use client certificates as a more secure alternative to simple authentication with User ID and password.

To make the certificate usable by we need to convert it into the pem format.

    d038720@wdfm00259937a:ssl-certs $ openssl x509 -in getCert.cer -inform DER -out sso_ca.pem
    d038720@wdfm00259937a:ssl-certs $ cat sso_ca.pem 
    -----BEGIN CERTIFICATE-----
    MIIB+jCCAWOgAwIBAgIEAQAAADANBgkqhkiG9w0BAQUFADAvMQswCQYDVQQGEwJE
    RTEPMA0GA1UEChMGU0FQLUFHMQ8wDQYDVQQDFAZTU09fQ0EwHhcNOTgwNTA0MTI1
    OTMzWhcNMjMwODMxMTIwMDAwWjAvMQswCQYDVQQGEwJERTEPMA0GA1UEChMGU0FQ
    LUFHMQ8wDQYDVQQDFAZTU09fQ0EwgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGB
    APy5r2Ns7e13CeTfbNgC+eQ2ZrQ3/KmuY7kzQrRcnMnm3oZ6eEYvBozjBQMiYtnw
    SMm9vhbINAM+6Tq2/8xaTFaSMCsN4HnXjC/emjhjrX2C6GZRpB0pxieajLWJ/8vj
    eRlmBsc7uoAxgXgVtkg7U1ihb2wpEoneoiP7cIBCXJoxAgMBAAGjIzAhMA8GA1Ud
    EwEB/wQFMAMBAf8wDgYDVR0PAQH/BAQDAgH2MA0GCSqGSIb3DQEBBQUAA4GBAHFp
    8soQSqAnOJ1AlfHjDdahyIzaeYtC+Sjjhm1EcJF9wR+VONnOpPDlsXtyMKHcrNe9
    roUSxq0fYeljalisxSIzWHMaVOXr1aXxOk0Z8XmG7Uud7LyTPwMp4YTof4d6xxTg
    vrEcgtBVIuzM+sVEp7RRM6Y+fL9u+69krtndZ8Ft
    -----END CERTIFICATE-----


Remove the passphrase from your key
-----------------------------------

    d038720@wdfm00259937a:ssl-certs $ mv expertondemand.key expertondemand.key.original
    d038720@wdfm00259937a:ssl-certs $ openssl rsa -in expertondemand.key.original -out expertondeman.key
    Enter pass phrase for expertondemand.key.original:
    writing RSA key
    d038720@wdfm00259937a:ssl-certs $ ll
    total 40
    drwxr-xr-x   7 d038720  SAP_ALL\domain users   238 Apr  6 18:37 .
    drwxr-xr-x  55 d038720  SAP_ALL\domain users  1870 Apr  6 17:20 ..
    -rw-r--r--   1 d038720  SAP_ALL\domain users   887 Apr  6 18:37 expertondemand.key
    -rw-r--r--   1 d038720  SAP_ALL\domain users   745 Apr  6 18:34 expertondemand.crt
    -rw-r--r--   1 d038720  SAP_ALL\domain users   733 Apr  6 17:24 expertondemand.csr
    -rw-r--r--   1 d038720  SAP_ALL\domain users   951 Apr  6 17:20 expertondemand.key.original
    -rw-r--r--@  1 d038720  SAP_ALL\domain users   510 Apr  6 18:32 getCert.cer
    -rw-r--r--   1 d038720  SAP_ALL\domain users   745 Apr  6 18:34 sso_ca.pem


Install your certifcates
------------------------

The final step missing is to install the certificates into the webserver. In this example we'll be using Nginx. 

Copy the following files to your webserver. Using Chef it would look like this:

    ["expertondemand.crt", "expertondemand.key", "sso_ca.pem"].each do |cert|
      cookbook_file "#{node[:nginx][:dir]}/certs/#{cert}" do
        source "certs/#{cert}"
        mode "0644"
        owner "root"
        group "root"
      end
    end

Certificates for different hosts can be stored in a cookbook like this. Chef's file specificity will take care to install the correct certificate for each host. The certificate key and the root certificate can be shared between hosts. 

    d038720@wdfm00259937a:expertondemand $ tree files
    files
    |-- default
    |   `-- certs
    |       |-- expertondemand.key
    |       `-- sso_ca.pem
    |-- expertondemand.wdf.sap.corp
    |   `-- certs
    |-- host-expertondemand-staging.wdf.sap.corp
    |   `-- certs
    |       `-- expertondemand.crt
    `-- host-expertondemand.wdf.sap.corp
        `-- certs
            `-- expertondemand.crt

To finalize the configuration you will have to create a configuration like this:

    ssl_certificate      <%= @node[:nginx][:dir] %>/certs/expertondemand.crt;
    ssl_certificate_key  <%= @node[:nginx][:dir] %>/certs/expertondemand.key;

    server {
      listen                 443 default ssl;
      server_name            <%= @node[:hostname] %>;
      ssl_client_certificate <%= @node[:nginx][:dir] %>/certs/sso_ca.pem;
      ssl_verify_client      optional;


      more_set_input_headers "SSL_CLIENT_S_DN: $ssl_client_s_dn"
                             "SSL_CLIENT_VERIFY: $ssl_client_verify";
    }

Tadaaa. You know have a working SSL configuration without Phishing warnings. And working SSO using client side SSL certificates. The username will be stored in the header variable `SSL_CLIENT_S_DN`. The variable `SSL_CLIENT_VERIFY` will contain the status of the verification: success or failure. 
