[![Build Status](https://secure.travis-ci.org/xipki/xipki.svg)](http://travis-ci.org/xipki/xipki)
[![GitHub release](https://img.shields.io/github/release/xipki/xipki.svg)](https://github.com/xipki/xipki/releases)
[![License](https://img.shields.io/badge/license-Apache%202-4EB1BA.svg)](https://www.apache.org/licenses/LICENSE-2.0.html)
[![Github forks](https://img.shields.io/github/forks/xipki/xipki.svg)](https://github.com/xipki/xipki/network)
[![Github stars](https://img.shields.io/github/stars/xipki/xipki.svg)](https://github.com/xipki/xipki/stargazers)

# XiPKI
XiPKI (e**X**tensible s**I**mple **P**ublic **K**ey **I**nfrastructure) is
a highly scalable and high-performance open source PKI (CA and OCSP responder).

## License
* The Apache Software License, Version 2.0

## Owner
Lijun Liao, [LinkedIn](https://www.linkedin.com/in/lijun-liao-644696b8)

## Support
Just create [issues](https://github.com/xipki/xipki/issues).

## Prerequisite
* OS: Linux, Windows, MacOS
* JRE / JDK 8 (build 162+), 9, 10, 11, 12, 13
* Database: DB2, MariaDB, MySQL, Oracle, PostgreSQL, H2, HSQLDB

## Tested PKCS#11 Devices
* [Softhsm v1 & v2](https://www.opendnssec.org/download/packages/),
* [Smartcard HSM EA+](http://www.smartcard-hsm.com/features.html#usbstick),
* Thales nCipher Connect, Thales nCipher Solo, Utimaco Se

## Get Started

### JAVA_HOME
  Set the environment variable `JAVA_HOME` to point to root directory of the to
  the JRE/JDK installation.

### CA Server and OCSP Responder

Download the binaries `ca-war-<version>.zip`, `ocsp-war-<version>.zip` and
`xipki-cli-<version>.tar.gz` from
[releases](https://github.com/xipki/xipki/releases).

Only if you want to use the development version, build it from source code as
follows.

- Get a copy of project code
  ```sh
  git clone https://github.com/xipki/xipki
  ```
- Build the project

  In folder `xipki`
  ```sh
  mvn clean install -DskipTests
  ```

  Then you will find the following binaries:
   - CA: `assembles/ca-war/target/ca-war-<version>.zip`
   - OCSP: `assembles/ocsp-war/target/ocsp-war-<version>.zip`
   - CLI (Command Line Interface): `assembles/xipki-cli/target/xipki-cli-<version>.tar.gz`

## Install CA Server

1. Unpack the binary `ca-war-<version>.zip` and install CA as described in the
   unpacked README file.

2. Adapt the database configurations ${CONTAINER_ROOT}/xipki/etc/ca/database/ca-db.properties.
   - If you use database other than MariaDB and MySQL, you need to overwrite the configuration templates
   from the sub folder.
   - If you use database other than MariaDB, MySQL and PostgreSQL, you need to get the JDBC driver and copy
     it to the container directory for external jars (e.g. `lib` in tomcat, and `lib/ext` in jetty).   

3. Create new databases configured in Step 2.

4. Initialize the databases configured in Step 2.

```
 ca-war-<version>/dbtool/bin/initdb.sh \
   --db-conf xipki/etc/ca/database/ca-db.properties \
   --db-schema xipki/sql/ca-init.xml
```

## Install OCSP Responder

Note that CA and OCSP can be installed in the same servlet container.

1. Unpack the binary `ocsp-war-<version>.zip` and install OCSP responder as described in the
   unpacked README file.

2. Adapt the database configuration ${CONTAINER_ROOT}/xipki/etc/ocsp/database/ca-db.properties.
   - If you use database other than MariaDB and MySQL, you need to overwrite the configuration templates
   from the sub folder.
   - If you use database other than MariaDB, MySQL and PostgreSQL, you need to get the JDBC drivers and copy
     it to the container directory for external jars (e.g. `lib` in tomcat, and `lib/ext` in jetty).   

## Install Command Line Interface

1. Unpack the binary `xipki-cli-<version>.tar.gz`
2. Adapt the CMP client configuration `xipki/cmpclient/cmpclient.json`

## Configure PKCS#11 device (optional)

   This step is only required if the real PKCS#11 device instead of the emulator
   is used.

  * Copy `xipki/security/example/pkcs11-hsm.json` to `xipki/security/pkcs11.json`, and adapt the PKCS#11 configuration.

## Configure how to handle SSL client certificate (optional)

  This step is only required if the CA is behind a reverse proxy apache httpd.

  * Add the java property org.xipki.reverseproxy.mode
    ```sh
    -Dorg.xipki.reverseproxy.mode=APACHE
    ```

  * configure the proxy to forward the headers via mod_proxy with the following
    configuration

    ```sh
    RequestHeader set SSL_CLIENT_VERIFY "%{SSL_CLIENT_VERIFY}s"
    RequestHeader set SSL_CLIENT_CERT "%{SSL_CLIENT_CERT}s"
    ```

    For more details please refer to

      * [Jetty/Howto/Configure mod proxy](https://wiki.eclipse.org/Jetty/Howto/Configure_mod_proxy)
      * [Jetty: Tricks to do client certificate authentications behind a reverse proxy](http://www.zeitoun.net/articles/client-certificate-x509-authentication-behind-reverse-proxy/start)
      * [Apache Module mod_ssl](http://httpd.apache.org/docs/2.2/mod/mod_ssl.html#envvars)

## Setup CA Server and OCSP Responder

1. Start the servlet container  
HSM devices of Thales, e.g. nCipher, can use Thales preload to manage the
PKCS#11 sessions. In this case, the servlet container should be started as follows
```sh
preload <start script>
```

2. Setup CA in CLI
   * Start CLI.
      `bin/karaf`
 
   * Setup CA
      * In case of using new keys and certificates, in CLI:  
        `source xipki/setup/cacert-none/setup-*.script`
         where * is place holder.

      * In case of using existing keys and certificates, in CLI:  
        `source xipki/setup/cacert-present/setup-*.script`
         where * is place holder.

   * Verify the installation, execute the command in CLI:  
     `ca-info myca1`

## Enroll/Revoke Certificate and Get CRL via Shell (optional)

- The following shell script demonstrates how to enroll and revoke certificates, and how to get the current CRL:
  `<CLI_ROOT>/xipki/client-script/rest.sh`

  Note that this script tells CA to generate real certificates. DO NOT use it in the production environment.

## Enroll/Revoke Certificate

* SCEP  
  Using any SCEP client. XiPKI provides also a SCEP client.

  The binary `xipki-cli-<version>`.tar.gz contains an example script in the folder xipki/client-script.
  It can be executed in the CLI as follows:  
  - `source xipki/client-script/scep-client.script`

* XiPKI CLI
  XiPKI CLI provides both the full-featured client and the lite version to enroll and
  revoke certificates via CMP.

  The binary `xipki-cli-<version>`.tar.gz contains an example script in the folder xipki/client-script.
  It can be executed in the CLI as follows:  
  - `source xipki/client-script/cmp-client.script`

* REST API  
  The shell script `xipki/client-script/rest.sh` of the `xipki-cli` demonstrates
  the use of REST API.

  The binary `xipki-cli-<version>`.tar.gz contains an example script in the folder xipki/client-script.
  It can be executed in the CLI as follows:  
  - `source xipki/client-script/rest-client.script`
 
CLI Commands
-----
Please refer to [commands.md](commands.md) for more details.

Features
-----
- CA (Certification Authority)
  - X.509 Certificate v3 (RFC 5280)
  - X.509 CRL v2 (RFC 5280)
  - EdDSA Certificates (RFC 8410, RFC 8032)
  - Diffie-Hellman Proof-of-Possession Algorithms (RFC 6955)
  - SCEP (draft-gutmann-scep-00, draft-nourse-scep-23)
  - EN 319 411 (eIDAS)
  - EN 319 412 (eIDAS)
  - Supported databases: DB2, MariaDB, MySQL, Oracle, PostgreSQL, H2, HSQLDB
  - Direct and indirect CRL
  - FullCRL and DeltaCRL
  - Customized extension to embed certificates in CRL
  - CMP (RFC 4210 and RFC 4211)
  - API to specify customized certificate profiles
  - Support of JSON-based certificate profile
  - API to specify customized publisher, e.g. for LDAP and OCSP responder
  - Support of publisher for OCSP responder
  - Public key types of certificates
    - RSA
    - EC
    - DSA
    - Ed25519, Ed448
    - X25519, X448
    - SM2
  - Signature algorithms of certificates
    - Ed25519, Ed448
    - SM3withSM2
    - SHA3-*withRSA: where * is 224, 256, 384 and 512
    - SHA3-*withRSAandMGF1: where * is 224, 256, 384 and 512
    - SHA3-*withECDSA: where * is 224, 256, 384 and 512
    - SHA3-*withDSA: where * is 224, 256, 384 and 512
    - SHA*withRSA: where * is 1, 224, 256, 384 and 512
    - SHA*withRSAandMGF1: where * is 1, 224, 256, 384 and 512
    - SHA*withECDSA: where * is 1, 224, 256, 384 and 512
    - SHA*withPlainECDSA: where * is 1, 224, 256, 384 and 512
    - SHA*withDSA: where * is 1, 224, 256, 384 and 512
 - Native support of X.509 extensions (other extensions can be supported by
   configuring it as blob)
    - AdditionalInformation (German national standard CommonPKI)
    - Admission (German national standard CommonPKI)
    - AuthorityInformationAccess (RFC 5280)
    - AuthorityKeyIdentifier (RFC 5280)
    - BasicConstraints (RFC 5280)
    - BiometricInfo (RFC 3739)
    - CertificatePolicies (RFC 5280)
    - CRLDistributionPoints (RFC 5280)
    - CT Precertificate SCTs (RFC 6962)
    - ExtendedKeyUsage (RFC 5280)
    - FreshestCRL (RFC 5280)
    - InhibitAnyPolicy (RFC 5280)
    - IssuerAltName (RFC 5280)
    - KeyUsage (RFC 5280)
    - NameConstraints (RFC 5280)
    - OcspNoCheck (RFC 6960)
    - PolicyConstrains (RFC 5280)
    - PolicyMappings (RFC 5280)
    - PrivateKeyUsagePeriod (RFC 5280)
    - QCStatements (RFC 3739, eIDAS standard EN 319 412)
    - Restriction (German national standard CommonPKI)
    - SMIMECapabilities (RFC 4262)
    - SubjectAltName (RFC 5280)
    - SubjectDirectoryAttributes (RFC 3739)
    - SubjectInfoAccess (RFC 5280)
    - SubjectKeyIdentifier (RFC 5280)
    - TLSFeature (RFC 7633)
    - ValidityModel (German national standard CommonPKI)
    - GM/T 0015 IdentityCode (个人身份标识码, Chinese Standard GM/T 0015-2012)
    - GM/T 0015 InsuranceNumber (个人社会保险号, Chinese Standard GM/T 0015-2012)
    - GM/T 0015 ICRegistrationNumber (企业工商注册号, Chinese Standard GM/T 0015-2012)
    - GM/T 0015 OrganizationCode (企业组织机构代码, Chinese Standard GM/T 0015-2012)
    - GM/T 0015 TaxationNumber (企业税号, Chinese Standard GM/T 0015-2012)

 - Management of multiple CAs in one software instance
 - Support of database cluster
 - Multiple software instances (all can be in active mode) for the same CA
 - Native support of management of CA via embedded OSGi commands
 - API to specify CA management, e.g. GUI
 - Database tool (export and import CA database) simplifies the switch of
   databases, upgrade of XiPKi and switch from other CA system to XiPKI CA
 - Client to enroll, revoke, unrevoke and remove certificates, to generate and
   download CRLs
 - All configuration of CA except those of databases is saved in database

- OCSP Responder
  - OCSP Responder (RFC 2560 and RFC 6960)
  - Support of Common PKI 2.0
  - Management of multiple certificate status sources
  - Support of certificate status source based on the database of XiPKI CA
  - Support of certificate status source based on the OCSP database published by XiPKI CA
  - Support of certificate status source CRL and DeltaCRL
  - Support of certificate status source published by EJBCA
  - API to support proprietary certificate sources
  - Support of both unsigned and signed OCSP requests
  - Multiple software instances (all can be in active mode) for the same OCSP
    signer and certificate status sources.
  - Supported databases: DB2, MariaDB, MySQL, Oracle, PostgreSQL, H2, HSQLDB
  - Database tool (export and import OCSP database) simplifies the switch of
    databases, upgrade of XiPKi and switch from other OCSP system to XiPKI OCSP.
  - Client to send OCSP request

- SCEP
  - Supported SCEP versions
    - draft-gutmann-scep-00
    - draft-nourse-scep-23

- Toolkit (for both PKCS#12 and PKCS#11 tokens)
  - Generating keypairs of RSA, EC and DSA in token
  - Deleting keypairs and certificates from token
  - Updating certificates in token
  - Generating CSR (PKCS#10 request)
  - Exporting certificate from token

- For both CA and OCSP Responder
  - Support of PKCS#12 and JKS keystore
  - Support of PKCS#11 devices, e.g. HSM
  - API to use customized key types, e.g. smartcard
  - High performance
  - Support of health check
  - Audit with syslog and slf4j

- For CA, OCSP Responder and Toolkit
  - API to resolve password
  - Support of PBE (password based encryption) password resolver
     - All passwords can be encrypted by the master password
  - Support of OBF (as in jetty) password resolver

Use OCSP with customized Certificate Status Source (OcspStore)
-----
  - See the example modules
    - `ocsp-store-example`: implementation of a customized OcspStore.
    - `ocsp-store-example-assembly`: assembly the binaries.
