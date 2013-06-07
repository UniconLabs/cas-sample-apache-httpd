cas-sample-apache-httpd
=======================

Example CAS configuration with Apache httpd web server


Installing Apache CAS Client Binary (mod-auth-cas)
-------------------------------------------------

Note: The instructions below assume a simply binary install. 
The `mod_auth_cas` client for Apache is available via the EPEL respository. To install the client, the EPEL repository must first be enabled in the list of packages. EPEL for RHEL6 can be installed via the following package on the local system:

```
http://ftp.jaist.ac.jp/pub/Linux/Fedora/epel/6/i386/repoview/epel-release.html
```

Details about EPEL itself is available here:
```
http://fedoraproject.org/wiki/EPEL
```

Once you have EPEL enabled, you can install the CAS apache client via the following command:
```
yum install --enablerepo=epel mod_auth_cas
```

The binary module will be installed inside apache's `modules` directory and a sample configuration file will be available at the `conf.d` directory. 

**NOTE:** The binary version of the cas client does have issues in supporting SAML validation requests to the CAS server. If you wish to consume attributes from the CAS server, you do need to compile the client from source, as instructions below explain. 

Compiling Apache CAS Client (mod-auth-cas)
------------------------------------------

To compile the client from source, the following dependencies and tools are required:

1. C compiler, such as `gcc`
2. [Apache extension build tool](http://httpd.apache.org/docs/2.2/programs/apxs.html)
3. The `devel` package of `openssl`
4. The `devel` package of `libcurl`
5. The `devel` package of `libpcre`

To obtain the source, you may issue a `wget` command to the following url: [https://github.com/Jasig/mod_auth_cas/archive/master.zip](https://github.com/Jasig/mod_auth_cas/archive/master.zip)

Unzip the above package and execute the following command:
```
./configure && make && sudo make install
```
If the build complains about apxs not found, you may specify the location of apxs via the following option:
```
./configure --with-apxs=/usr/sbin/apxs && make && sudo make install
```

Once the build is complete, you may copy the `mod_auth_cas.so` module from the `/var/www/mod_auth_cas/src/.libs` directory over to apache's `modules` directory.

mod-auth-cas Apache Configuration
---------------------------------

Make sure the CAS client configuration file is named `zzz_auth_cas.conf`. Doing so allows all other module dependencies to be loaded first, such as ssl, upon which the CAS client might and does depend. 

A snippet of the client configuration follows below:
```
#
# mod_auth_cas is an Apache 2.0/2.2 compliant module that supports the
# CASv1 and CASv2 protocols
#

LoadModule auth_cas_module modules/mod_auth_cas.so
CASVersion 2
CASCookiePath /var/www/cas/
CASLoginURL https://sso.cas.edu/cas/login
CASValidateURL https://sso.cas.edu/cas/samlValidate
CASValidateSAML On
CASCertificatePath /etc/pki/tls/certs/sso.cas.edu.crt
CASIdleTimeout 600
CASCookieDomain cas.edu
CASDebug On
CASValidateServer Off

<Location /TeamMemberPortal>
    AuthType CAS
    AuthName "Authentication via CAS"
    require valid-user
    CASAuthNHeader CAS_force_password_change
    CASAuthNHeader CAS_first_name
    CASAuthNHeader CAS_birth_date
    CASAuthNHeader CAS_email
    CASAuthNHeader CAS_work_unit
    CASAuthNHeader CAS_last_name
    CASScope /
</Location>
```

**Note**: Instead of adding all of the attributes with CASAuthNHeader in front the following alternative configuration would pull in all attributes for the authenticated user as well:

```
<Location /TeamMemberPortal>
    AuthType CAS
    AuthName "Authentication via CAS"
    require valid-user
    CASAuthNHeader CAS-User
    CASScope /
</Location>

```

Important notes about the CAS client configuration
--------------------------------------------------

* Ensure that all CAS protocol endpoints such as `samlValidate`, and `serviceValidate` are properly configured in the ssl configuration. `samlValidate` does have the ability to consume and return CAS attributes back to the application via the SAML 1.1 protocol while `serviceValidate` merely is able to consume the CAS protocol and return the authenticated userid at this point.
* Ensure that the cookie directory configured for the CAS client has sufficient permissions such that `httpd` would be able read and write to it. 


