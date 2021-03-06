
---

copyright:
 years: 2015, 2016

---

{:new_window: target="_blank"}
# Configuring credentials for APNs
{: #create-push-credentials-apns}
Last updated: 17 September 2016
{: .last-updated}

Apple Push Notification Service (APNs) allows application developers to send remote notifications from the {{site.data.keyword.mobilepushshort}} service instance on Bluemix (the provider) to iOS devices and applications. Messages are sent to a target application on the device. 

Obtain and configure your APNs credentials. The APNs certificates are securely managed by {{site.data.keyword.mobilepushshort}} service and used to connect to APNs server as a provider.

<!-- 1. Obtain an [Apple Developers](https://developer.apple.com/) account.-->

<!--2. [Register an App ID](#create-push-credentials-apns-register)
3. [Create a development and distribution APNs SSL certificate](#create-push-credentials-apns-ssl)
4. [Create a development provisioning profile](#create-push-credentials-dev-profile)
5. [Create a store distribution provisioning profile](#create-push-credentials-apns-distribute_profile)
6. [Creating .p12 push certificate file for Bluemix push](#create-p12-push-certificate-file-for-Bluemix-push)
7. [Set up APNs on the Push Dashboard](#create-push-credentials-apns-dashboard)
-->


##Registering an App ID
{: #create-push-credentials-apns-register}


The App ID (the bundle identifier) is a unique identifier that identifies a specific application. Each application requires an App ID. Services like the {{site.data.keyword.mobilepushshort}} service are configured to the App ID.



1. Make sure you have an [Apple Developers](https://developer.apple.com/) account.
2. Go to the [Apple Developer](https://developer.apple.com) portal, click **Member Center**, and select **Certificates, Identifiers & Profiles**.
3. Go to the **Registering App IDs** section in the [Apple Developer Library](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html#//apple_ref/doc/uid/TP40012582-CH30-SW991), and follow the instructions to register the App ID.

	When you register an App ID, select the following options:
	* Push Notifications

		![App Services](images/appID_appservices_enablepush.jpg)

	* Explicit ID Suffix

		![Explicit ID](images/appID_bundleID.jpg)

<!--4. Create a development and distribution APNs SSL certificate.-->

##Create a development and distribution APNs SSL certificate
{: #create-push-credentials-apns-ssl}

Before you can obtain an APNs certificate, you must first generate a certificate signing request (CSR) and submit it to Apple, the certificate authority (CA). The CSR contains information that identifies your company and your public and private key that you use to sign for your Apple push notifications. Then, generate the SSL certificate on the iOS Developer Portal. The certificate, along with its public and private key, is stored in Keychain Access.

<!-- ###Before you begin -->
<!-- {: before-you-begin-certificate} -->

<!--[Register an App ID](#create-push-credentials-apns-register)-->

You can use APNs in two modes: 

* Sandbox mode for development and testing.
* Production mode when distributing applications through the App Store (or other enterprise distribution mechanisms).

You must obtain separate certificates for your development and distribution environments. The certificates are associated with an App ID for the app that is the recipient of remote notifications. For production, you can create up to two certificates. Bluemix uses the certificates to establish an SSL connection with APNs.

<!-- Create a development and distribution SSL certificate. -->

1. Go to the [Apple Developer](https://developer.apple.com) website, click **Member Center**, and select **Certificates, Identifiers & Profiles**.
2. In the **Identifiers** area, click **App IDs**.
3. From your list of App IDs, select your <!--newly created--> App ID, then select **Settings**.
4. In the **Push Notifications** area, create a Development SSL certificate and then a Production SSL certificate.

	![Push Notification SSL certificates](images/certificate_createssl.jpg)

5. When the **About Creating a Certificate Signing Request (CSR) screen** displays, start the **Keychain Access** application on your Mac to create a Certificate Signing Request (CSR).
6. From the menu, select **Keychain Access > Certificate Assistant > Request a Certificate From a Certificate Authority…** 
7. In **Certificate Information**, enter the email address that is associated with your App Developer account and a common name. Give a meaningful name that helps you identify whether it is a certificate for development (sandbox) or distribution (production); for example, _sandbox-apns-certificate_ or _production-apns-certificate_.
8. Select **Save to disk** to download the `.certSigningRequest` file to your desktop, then click **Continue**.
9. In the **Save As** menu option, name the `.certSigningRequest` file and click **Save**.
10. Click **Done**. You now have a CSR.
11. Return to the **About Creating a Certificate Siging Request (CSR)** window and click **Continue**. 
12. From the **Generate** screen, click **Choose File ...** and select the CSR file that you saved on your desktop. Then, click **Generate**.
	![Generate certificate](images/generate_certificate.jpg)
13. When your certificate is ready, click **Done**.
14. On the **Push Notifications** screen, click **Download** to download your certificate, then click **Done**. 
	![Download certificate](images/certificate_download.jpg)
15. On your Mac, go to **Keychain Access > My Certificates**, and locate your newly installed certificate. Double-click the certificate to install it into the Keychain Access.
16. Select the certificate and private key, and then select **Export** to convert the certificate into the personal information exchange format (`.p12` format).
	![Export certificate and keys](images/keychain_export_key.jpg)
17. In the **Save As** field, provide the certificate a meaningful name. For example, `sandbox_apns.p12_certifcate` or `production_apns.p12`, then click **Save**.
	![Export certificate and keys](images/certificate_p12v2.jpg)
18. In the **Enter a password** field, enter a password to protect the exported items, then click **OK**. You can use this password to configure your APNs settings on the Push dashboard.
	![Export certificate and keys](images/export_p12.jpg)
19. The **Key Access.app** prompts you to export your key from the **Keychain** screen. Enter your administrative password for your Mac to allow your system to export these items, and then select the **Always Allow** option. A `.p12` certificate is generated on your desktop.


##Creating a development provisioning profile
{: #create-push-credentials-dev-profile}

The provisioning profile works with the App ID to determine which devices can install and run your app and which services your app can access. For each App ID, you create two provisioning profiles: one for development and the other for distribution. Xcode uses the development provisioning profile to determine which developers are allowed to build the application and which devices are allowed to be tested on the application.

###Before you begin
{: before-you-begin-provisioning-file}

Make sure you have registered an App ID, enabled it for Push Notification Service, and configured it to use a development and production APNs SSL certificate.

Create a development provisioning profile, as follows:

1. Go to the [Apple Developer](https://developer.apple.com) portal, click **Member Center**, and select **Certificates, Identifiers & Profiles**.
2. Go to the [Mac Developer Library ](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html#//apple_ref/doc/uid/TP40012582-CH30-SW62site) , scroll to the **Creating Development Provisioning Profiles** section, and follow the instructions to create a development profile.
**Note**: When you configure a development provision profile, select the following options:
	* **iOS App Development**
	* **For iOS and watchOS apps**



##Creating a store distribution provisioning profile
{: #create-push-credentials-apns-distribute_profile}

Use the store provisioning profile to submit your app for distribution to the App Store.

1. Go to the [Apple Developer](https://developer.apple.com) portal, click **Member Center**, and select **Certificates, Identifiers & Profiles**.
2. Double-click the downloaded provisioning profile to install it into Xcode.

##Creating .p12 push certificate file for Bluemix push
{: #create-p12-push-certificate-file-for-Bluemix-push}

To create a valid `.p12` push certificate file, use the APNs `.p12` certificate, along with the development/distribution profile certificate. 
Complete the steps. You can choose to pick up either the development or distribution certificate:

	{: screen}  
	developer_identity.cer - Development profile downloaded from Apple 
	apns.p12 - APNS .p12 exported from the keychain
	openssl x509 -in developer_identity.cer -inform DER -out
	developer_identity.pem -outform PEM
	openssl pkcs12 -nocerts -in apns.p12 -out apns.pem
	openssl pkcs12 -export -inkey apns.pem -in developer_identity.pem -out 	bluemixPush_dev.p12
	
	
	
Store the `bluemixPush_dev.p12` file on your desktop.

##Setting up APNs on the {{site.data.keyword.mobilepushshort}} Dashboard
{: #create-push-credentials-apns-dashboard}

To use the {{site.data.keyword.mobilepushshort}} service to send notifications, upload the SSL certificates that are required for Apple Push Notification Service (APNs). You can also use the REST API to upload an APNs certificate.


###Before you begin
{: before-you-begin-dashboard}


<!-- Get your development and production APNs SSL certificate and the password associated with each type of certificate. For information, see Creating and configuring push credentials for APNs.-->

The certificates that are needed for APNs are `.p12` certificates, which contain the private key and SSL certificates that are required to build and publish your application. You must generate the certificates from the Member Center of the Apple Developer website (for which a valid Apple Developer account is required). Separate certificates are required for the development environment (sandbox) and the production (distribution) environment.

**Note**: After the `.cer` file is in your key chain access, export it to your computer to create a `.p12` certificate.

For more information about using the APNs, see [iOS Developer Library: Local and Push Notification Programming Guide](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ProvisioningDevelopment.html#//apple_ref/doc/uid/TP40008194-CH104-SW4).

Set up APNs on the Push dashboard, as follows:

1. Open your backend application in the Bluemix dashboard and then click the **IBM Push Notifications** service to open the Push dashboard.

	The Push dashboard displays:
	![Set push notifications dashboard](images/wizard.jpg)
2. On the **Configuration** tab, go to the **Apple Push Certificate** section,  select **Sandbox** (development) or **Production** (distribution) and then upload the `p.12` certificate that you created using the previous [step](#create-p12-push-certificate-file-for-Bluemix-push).
	![Set push notifications configure tab](images/credential_screen.jpg)
3. In the **Password** field, enter the password that is associated with the `.p12` certificate file, then click **Save**.

After the certificates are uploaded successfully with a valid password, you can start sending notifications.
