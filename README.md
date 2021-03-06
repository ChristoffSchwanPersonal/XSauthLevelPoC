[![Build Status](http://ci.computerservice-wolf.com/buildStatus/icon?job=CI_XSauthLevelPoC_master_build)](http://ci.computerservice-wolf.com/job/CI_XSauthLevelPoC_master_build)

# Native HANA PoC for anonymous, public and private XSODATA Service

This sample project should show how the same HANA database table can be exposed anonymously, authenticated (public) and restricted (private). The anonymous service must be implemented in XSJS to make use of Anonymous SQL connection (XSSQLCC). The public and private service is using XSODATA. The anonymous service restricts the number of fields. The public service is read only. The private service allows full CRUD operations.

There is also a frontend projects implemented in SAPUI5 which you can find here:

* [XSauthLevelPoCAdminUI](https://github.com/gregorwolf/XSauthLevelPoCAdminUI)
* [XSauthLevelPoCPublicUI](https://github.com/gregorwolf/XSauthLevelPoCPublicUI)

## Setup Guide

You must have developer authorization in your HANA System. To try this project just spin up your own HANA Multitennant Database Container (MDC) on the HANA Cloud Platform Trial (HCP). Open the SAP HANA Web-based Development Workbench and create the package:

    de/linuxdozent/gittest

After you've created the package right click on the gittest package and choose **Syncronize with GitHub**. Provide your GitHub credentials to allow the HANA system to read your GitHub repositories. As you can't specify a GitHub repository URL you have to clone the project so you have it in your repository list. Then coose the cloned repository and GitHub branch **master**. Click **Fetch** to retreive the content. After that step you have to activate the artifacts. Try a right click **activate all**. If that fails start with the BOOK.hdbdd file in the data package and work your way through.

To enable the anonymous access you have to assign your admin user the role **sap.hana.xs.admin.roles::SQLCCAdministrator**. Then you can access the XS Admin tool at **/sap/hana/xs/admin/index.html#/package/de.linuxdozent.gittest.anonymous/sqlcc/anonymous** and there the entry **anonymous.xssqlcc** should be visible. Click on **Edit** and tick the **Active** checkbox and **Save** the settings.

Now add the role **de.linuxdozent.gittest.roles::admin** to your development user and the role **de.linuxdozent.gittest.roles::public** to a new user. Don't use your S- or P-User as username if you want to use Application-to-Application Single Sign On (App2AppSSO) with automatic user generation. It will cause an user already exists error. 

### Sending E-Mails

With the following steps you can setup your HANA XS Engine to send E-Mails via Google's SMTP Server. Unfortunately this setup works only on on premise and SAP Cloud Platform Factory (Production) installation. I was not able to get an E-Mail sent on the HANA instances in the SCP Trial. The configuration basic can be found at [Use SMTP settings to send mail from a printer, scanner, or app](https://support.google.com/a/answer/176600?hl=en). 

I've used smtp.gmail.com which requires a SSL/TLS connection. To establish such a connection the certificate used to sign the endpoint certificate must be uploaded to the HANA Trust store. To find out which Certificate is uses to sign the endpoint get the the instructions from [Use G Suite certificates for secure transport (TLS)](https://support.google.com/a/answer/6180220?hl=en). I've issued this command:

```
openssl s_client -starttls smtp -connect smtp.gmail.com:587 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p'
```
and stored the resulting text in GMail.crt. After Uploading this certificate to the HANA Trust Manager in the newly created GMail trust store at **/sap/hana/xs/admin/#/trustManager** the issuer is displayed as **CN=Google Internet Authority G2, O=Google Inc, C=US**. The certificate for this ca can be downloaded at https://pki.google.com/. You have to import it also to the GMail trust store. Now you can switch to the SMTP Configurations at **/sap/hana/xs/admin/#/smtp** and maintain the following:

![SAP Cloud Platform Factory - HANA SMTP configuration](SCP-Factory-HANA-SMTP-Configuration.png?raw=true "SAP Cloud Platform Factory - HANA SMTP configuration")

I've used an App password created at https://myaccount.google.com/apppasswords for the password that has to be provided for Authentication.

## Test 

There is an automated test implemented using Jasmine. Read though the [setup guide](https://github.com/sapmentors/SITreg/blob/master/test/README.md) and do this stepps accordingly for this project.

You can also try the API via the path /de/linuxdozent/gittest/odata/service.xsodata using a tool like Postman. Then give /de/linuxdozent/gittest/odatapublic/service.xsodata a try.

With the public role you should be able to read all the details of the books. Also you should be able to create a customer and to read and update update the details. But always only for your own user.

You can use the [OData Explorer](http://scn.sap.com/community/developer-center/hana/blog/2014/12/02/sap-hana-sps-09-new-developer-features-new-xsodata-features) that is part of HANA since SPS 09 to test i.e. the admin service using the path: **/sap/hana/ide/editor/plugin/testtools/odataexplorer/index.html?appName=/de/linuxdozent/gittest/odata/service.xsodata/**.

## Application-to-Application Single Sign On (App2AppSSO)

If you want to use the HANA MDC XSODATA Service in a HCP HTML5 app and with App2AppSSO then follow the great Blog by Martin Raepple: [Principal Propagation between HTML5- or Java-based applications and SAP HANA XS on SAP HANA Cloud Platform](http://scn.sap.com/community/developer-center/cloud-platform/blog/2016/03/21/principal-propagation-between-html5-and-sap-hana-xs-on-sap-hana-cloud-platform).

## Build

Trigger build by another change.
