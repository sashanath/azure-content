<properties linkid="dev-java-how-to-use-acs-java" urldisplayname="Access Control" headerexpose="" pagetitle="How to Use the Access Control Service from Eclipse" metakeywords="Azure Access Control Service Java" footerexpose="" metadescription="" umbraconavihide="0" disquscomments="1"></properties>

# How to Authenticate Web Users with Windows Azure Access Control Service Using Eclipse

This guide will show you how to use the Windows Azure Access Control Service (ACS) within the Windows Azure Plugin for Eclipse with Java (by Microsoft Open Technologies). For more information on ACS, see the [Next steps](#next_steps) section.

<div class="dev-callout"> 
<b>Note</b> 
<p>The Windows Azure Access Services Control Filter (by Microsoft Open Technologies) is a community technology preview. As pre-release software, it is not formally supported by Microsoft Open Technologies, Inc. nor Microsoft.</p> 
</div>


## Table of Contents

-   [What is ACS?][]
-   [Concepts][]
-   [Prerequisites][]
-   [Create an ACS namespace][]
-   [Add identity providers][]
-   [Add a relying party application][]
-   [Create rules][]
-   [Upload a certificate to your ACS namespace][]
-   [Review the Application Integration Page][]
-   [Create a Java web application][]
-   [Add the ACS Filter library to your application][]
-   [Deploy to the compute emulator][]
-   [Deploy to Windows Azure][]
-   [Next steps][]

## <a name="what-is">What is ACS?</a>

Most developers are not identity experts and generally do not want to
spend time developing authentication and authorization mechanisms for
their applications and services. ACS is a Windows Azure service that
provides an easy way of authenticating users who need to access your web
applications and services without having to factor complex
authentication logic into your code.

The following features are available in ACS:

-   Integration with Windows Identity Foundation (WIF).
-   Support for popular web identity providers (IPs) including Windows
    Live ID, Google, Yahoo!, and Facebook.
-   Support for Active Directory Federation Services (AD FS) 2.0.
-   An Open Data Protocol (OData)-based management service that provides
    programmatic access to ACS settings.
-   A Management Portal that allows administrative access to the ACS
    settings.

For more information about ACS, see [Access Control Service 2.0][].

## <a name="concepts">Concepts</a>

Windows Azure ACS is built on the principals of claims-based identity -
a consistent approach to creating authentication mechanisms for
applications running on-premises or in the cloud. Claims-based identity
provides a common way for applications and services to acquire the
identity information they need about users inside their organization, in
other organizations, and on the Internet.

To complete the tasks in this guide, you should understand the following
concepts:

**Client** - In the context of this how-to guide, this is a browser that
is attempting to gain access to your web application.

**Relying party (RP) application** - An RP application is a web site or
service that outsources authentication to one external authority. In
identity jargon, we say that the RP trusts that authority. This guide
explains how to configure your application to trust ACS.

**Token** - A token is a collection of security data that is usually
issued upon successful authentication of a user. It contains a set of *claims*, attributes of the authenticated user. A claim can represent a
user's name, an identifier for a role a user belongs to, a user's age,
and so on. A token is usually digitally signed, which means it can
always be sourced back to its issuer, and its content cannot be tampered
with. A user gains access to a RP application by presenting a valid
token issued by an authority that the RP application trusts.

**Identity Provider (IP)** - An IP is an authority that authenticates
user identities and issues security tokens. The actual work of issuing
tokens is implemented though a special service called Security Token
Service (STS). Typical examples of IPs include Windows Live ID,
Facebook, business user repositories (like Active Directory), and so on.
When ACS is configured to trust an IP, the system will accept and
validate tokens issued by that IP. ACS can trust multiple IPs at once,
which means that when your application trusts ACS, you can instantly
offer your application to all the authenticated users from all the IPs
that ACS trusts on your behalf.

**Federation Provider (FP)** - IPs have direct knowledge of users,
and authenticate them using their credentials and issue claims about what
they know about them. A Federation Provider (FP) is a different kind of
authority: rather than authenticating users directly, it acts as an
intermediary and brokers authentication between one RP and one or more
IPs. Both IPs and FPs issue security tokens, hence they both use
Security Token Services (STS). ACS is one FP.

**ACS Rule Engine** - The logic used to transform incoming tokens from
trusted IPs to tokens meant to be consumed by the RP is codified in form
of simple claims transformation rules. ACS features a rule engine that
takes care of applying whatever transformation logic you specified for
your RP.

**ACS Namespace** - A namespace is a top level partition of ACS that you
use for organizing your settings. A namespace holds a list of IPs you
trust, the RP applications you want to serve, the rules that you expect
the rule engine to process incoming tokens with, and so on. A namespace
exposes various endpoints that will be used by the application and the
developer to get ACS to perform its function.

The following figure shows how ACS authentication works with a web
application:

![ACS flow diagram][acs_flow]

1.  The client (in this case a browser) requests a page from the RP.
2.  Since the request is not yet authenticated, the RP redirects the
    user to the authority that it trusts, which is ACS. The ACS presents
    the user with the choice of IPs that were specified for this RP. The
    user selects the appropriate IP.
3.  The client browses to the IP's authentication page, and prompts the
    user to log on.
4.  After the client is authenticated (for example, the identity
    credentials are entered), the IP issues a security token.
5.  After issuing a security token, the IP redirects the client to ACS
    and the client sends the security token issued by the IP to ACS.
6.  ACS validates the security token issued by the IP, inputs the
    identity claims in this token into the ACS rules engine, calculates
    the output identity claims, and issues a new security token that
    contains these output claims.
7.  ACS redirects the client to the RP. The client sends the new
    security token issued by ACS to the RP. The RP validates the
    signature on the security token issued by ACS, validates the claims
    in this token, and returns the page that was originally requested.

## <a name="pre">Prerequisites</a>

To complete the tasks in this guide, you will need the following:

- A Java Developer Kit (JDK), v 1.6 or later.
- Eclipse IDE for Java EE Developers, Helios or later. This can be downloaded from <http://www.eclipse.org/downloads/>. 
- A distribution of a Java-based web server or application server, such as Apache Tomcat, GlassFish, JBoss Application Server, or Jetty.
- A Windows Azure subscription, which can be acquired from <http://www.microsoft.com/windowsazure/offers/>.
- The Windows Azure Plugin for Eclipse with Java (by Microsoft Open Technologies) – June 2012 CTP. For more information, see [Installing the Windows Azure Plugin for Eclipse with Java (by Microsoft Open Technologies)](http://msdn.microsoft.com/en-us/library/windowsazure/hh690946.aspx).
- An X.509 certificate to use with your application. You will need this certificate in both public certificate (.cer) and Personal Information Exchange (.PFX) format. (If you want to create this certificate through the Eclipse plugin, you can open an existing Windows Azure deployment project, right-click the project name in Eclipse's Project Explorer, click **Properties**, expand **Windows Azure**, click **Remote Access**, and then click **New** to create a new X.509 certificate, which you can then save to your local file system for use later in this tutorial).
- Familiarity with the Windows Azure compute emulator and deployment techniques discussed at [Creating a Hello World Application for Windows Azure in Eclipse](http://msdn.microsoft.com/en-us/library/windowsazure/hh690944.aspx).

## <a name="create-namespace">Create an ACS Namespace</a>

To begin using Access Control Service (ACS) in Windows Azure, you must
create an ACS namespace. The namespace provides a unique scope for
addressing ACS resources from within your application.

1.  Log into the [Windows Azure Management Portal][].

    ![Windows Azure Management Portal page][portal_home_image]

2.  In the lower left navigation pane of the Management Portal, click **Service Bus, Access Control & Caching**.

    ![Management Portal Service Bus, Access Control, and Caching section][portal_sb_acs_caching]

3.  In the upper left navigation pane of the Management Portal, click **Access Control**, and then click **New**.

    ![New Access Control Service namespace][new_acs_namespace]

4.  In **Create a new Service Namespace**, enter a namespace, and then to
    make sure that it is unique, click **Check Availability**.

    ![Create a new service namespace dialog][new_acs_namespace_dialog]

5.  If it is available, select the country or region in which to use ACS
    (for the best performance, use the same country/region in which you
    are deploying your application), and then click **Create
    Namespace**.

The namespace appears in the Management Portal and takes a few minutes
to activate. Wait until the status is **Active** before moving on to add
IPs to your namespace.

## <a name="add-IP">Add identity providers</a>

In this task, you add IPs to use with your RP application for
authentication. For demonstration purposes, this task shows how to add
Windows Live as an IP, but you could use any of the IPs listed in the ACS
Management Portal.

1.  In the upper left navigation pane of Windows Azure Management
    Portal, click **Access Control**, select the ACS namespace that you
    want to configure, and then click **Access Control Service**.  
    The ACS Management Portal appears. 
    ![ACS Management Portal][acs_home_page]
2.  In the left navigation pane of the ACS Management Portal, click **Identity providers**.
3.  Windows Live ID is enabled by default, and cannot be deleted. For purposes of this tutorial, only Windows Live ID is used.
    This screen, however, is where you could add other IPs, by clicking the **Add** button.

Windows Live ID is now enabled as an IP for your ACS namespace. Next, you
specify your Java web application (to be created later) as an RP.

## <a name="add-RP">Add a relying party application</a>

In this task, you configure ACS to recognize your Java web
application as a valid RP application.

1.  On the ACS Management Portal, click **Relying party applications**.
2.  On the **Relying Party Applications** page, click **Add**.
3.  On the **Add Relying Party Application** page, do the following:
    1.  In **Name**, type the name of the RP. For purposes of this tutorial, type **Azure Web
        App**.
    2.  In **Mode**, select **Enter settings manually**.
    3.  In **Realm**, type the URI to which the security token issued by ACS
        applies. For this task, type **http://localhost:8080/**.
        ![Relying party realm for use in compute emulator][relying_party_realm_emulator]
    4.  In **Return URL,** type the URL to which ACS returns the security
        token. For this task, type **http://localhost:8080/MyACSHelloWorld/index.jsp**
        ![Relying party return URL for use in compute emulator][relying_party_return_url_emulator]
    5.  Accept the default values in the rest of the fields.

4.  Click **Save**.

You have now successfully configured your Java web application when it is run in the Windows Azure compute emulator (at
http://localhost:8080/) to be an RP in your ACS namespace. Next, create
the rules that ACS uses to process claims for the RP.

## <a name="create-rules">Create rules</a>

In this task, you define the rules that drive how claims are passed from
IPs to your RP. For the purpose of this guide, we will simply configure
ACS to copy the input claim types and values directly in the output
token, without filtering or modifying them.

1.  On the ACS Management Portal main page, click **Rule groups**.
2.  On the **Rule Groups** page, click **Default Rule Group for Azure Web App**.
3.  On the **Edit Rule Group** page, click **Generate**.
4.  On the **Generate Rules: Default Rule Group for Azure Web App** page, ensure Windows
    Live ID is checked and then click **Generate**.
5.  On the **Edit Rule Group** page, click **Save**.

## <a name="upload-certificate">Upload a certificate to your ACS namespace</a>

In this task, you upload a .PFX certificate that will be used to sign token requests created by your ACS namespace.

1.  On the ACS Management Portal main page, click **Certificates and keys**.
2.  On the **Certificates and Keys** page, click **Add** above **Token Signing**.
3.  On the **Add Token-Signing Certificate or Key** page:
    1. In the **Used for** section, click **Relying Party Application** and select **Azure Web App** (which you previously set as the name of your relying party application).
    2. In the **Type** section, select **X.509 Certificate**.
    3. In the **Certificate** section, click the browse button and navigate to the X.509 certificate file that you want to use. This will be a .PFX file. Select the file, click **Open**,  and then enter the certificate password in the **Password** text box.
    4. Ensure that **Make Primary** is checked. Your **Add Token-Signing Certificate or Key** page should look similar to the following.
        ![Add token-signing certificate][add_token_signing_cert]
    5. Click **Save** to save your settings and close the **Add Token-Signing Certificate or Key** page.

Next, review the information in the Application Integration page and
copy the URI that you will need to configure your Java web
application to use ACS.

## <a name="review-app-int">Review the Application Integration page</a>

You can find all the information and the code necessary to configure
your Java web application (the RP application) to work with ACS on
the Application Integration page of the ACS Management Portal. You will
need this information when configuring your Java web application for
federated authentication.

1.  On the ACS Management Portal, click **Application integration**.  
2.  In the **Application Integration** page, click **Login Pages**.
3.  In the **Login Page Integration** page, click **Azure Web App**.

In the **Login Page Integration: Azure Web App** page, the URL listed in **Option 1: Link to an ACS -hosted login page** will be used in your Java web application. You will need this value when you add the Windows Azure Access Control Services Filter library to your Java application.

## <a name="create-java-app">Create a Java web application</a>
1. Within Eclipse, at the menu click **File**, click **New**, and then click **Dynamic Web Project**. (If you don't see **Dynamic Web Project** listed as an available project after clicking **File**, **New**, then do the following: click **File**, click **New**, click **Project**, expand **Web**, click **Dynamic Web Project**, and click **Next**.) For purposes of this tutorial, name the project **MyACSHelloWorld**. (Ensure you use this name, subsequent steps in this tutorial expect your WAR file to be named MyACSHelloWorld). Your screen will appear similar to the following:

    ![Create a Hello World project for ACS exampple][create_acs_hello_world]

    Click **Finish**.
2. Within Eclipse's Project Explorer view, expand **MyACSHelloWorld**. Right-click **WebContent**, click **New**, and then click **JSP File**.
3. In the **New JSP File** dialog, name the file **index.jsp**. Keep the parent folder as MyACSHelloWorld/WebContent, as shown in the following:

    ![Add a JSP file for ACS example][add_jsp_file_acs]

    Click **Next**.

4. In the **Select JSP Template** dialog, select **New JSP File (html)** and click **Finish**.
5. When the index.jsp file opens in Eclipse, add in text to display **Hello ACS World!** within the existing `<body>` element. Your updated `<body>` content should appear as the following:

        <body>
          <b><% out.println("Hello ACS World!"); %></b>
        </body>
    
    Save index.jsp.
  
## <a name="add_acs_filter_library">Add the ACS Filter library to your application</a>

1. In Eclipse's Project Explorer, right-click **MyACSHelloWorld**, click **Build Path**, and then click **Configure Build Path**.
2. In the **Java Build Path** dialog, click the **Libraries** tab.
3. Click **Add Library**.
4. Click **Windows Azure Access Control Services Filter (by MS Open Tech)** and then click **Next**. The **Windows Azure Access Control Services Filter** dialog is displayed.  (The **Location** field may have a different path, depending on where you installed Eclipse, and the version number could be different, depending on software updates.)
    ![Add ACS Filter library][add_acs_filter_lib]
5. Using a browser opened to the **Login Page Integration** page of the Management Portal, copy the URL listed in the **Option 1: Link to an ACS-hosted login page** field and paste it into the **ACS Authentication Endpoint** field of the Eclipse dialog.
6. Using a browser opened to the **Edit Relying Party Application** page of the Management Portal, copy the URL listed in the **Realm** field and paste it into the **Relying Party Realm** field of the Eclipse dialog.
7. Within the **Security** section of the Eclipse dialog, click **Browse**, navigate to the certificate you want to use, select it, and click **Open**.
8. [Optional] Keep **Require HTTPS connections** checked. If you set this option, you'll need to access your application using the HTTPS proptocol. If you don't want to require HTTPS connections, uncheck this option.
9. For a deployment to the compute emulator, your **Windows Azure ACS Filter** settings will look similar to the following.
    ![Windows Azure ACS Filter settings for a deployment to the compute emulator][add_acs_filter_lib_emulator]
10. Click **Finish**.
10. Click **Yes** when presented with with a dialog box stating that a web.xml file will be created.
11. Click **OK** to close the **Java Build Path** folder.

## <a name="deploy_compute_emulator">Deploy to the compute emulator</a>

1. In Eclipse's Project Explorer, right-click **MyACSHelloWorld**, click **Windows Azure**, and then click **Package for Windows Azure**.
2. For **Project name**, type **MyAzureACSProject** and click **Next**.
3. Select a JDK and application server. (These steps are covered in detail in the [Creating a Hello World Application for Windows Azure in Eclipse](http://msdn.microsoft.com/en-us/library/windowsazure/hh690944.aspx) tutorial). Your screen will look similar to the following:
    ![New Windows Azure deployment project][new_azure_deployment]
4. Click **Finish**.
5. Click the **Run in Windows Azure Emulator** button.
6. After your Java web application starts in the compute emulator, close all instances of your browser (so that any current browser sessions do not interfere with your ACS login test).
7. Run your application by opening <http://localhost:8080/MyACSHelloWorld/> in your browser (or <https://localhost:8080/MyACSHelloWorld/> if you checked **Require HTTPS connections**). You should be prompted for a Windows Live ID login, then you should be taken to the return URL specified for your relying party application.
99.  When you have finished viewing your application, click the **Reset Windows Azure Emulator** button.

## <a name="deploy_azure">Deploy to Windows Azure</a>

To deploy to Windows Azure, you'll need to change the relying party realm and return URL for your ACS namespace, and you'll need to package your certificate with the deployment. (You didn't need to package the certificate for the compute emulator, because the compute emulator had access to the local path for your certificate.)

1. Within the Windows Azure Management Portal, in the **Edit Relying Party Application**, modify **Realm** to be the URL of your deployed site. Replace **example** with the DNS name you specified for your deployment.
    ![Relying party realm for use in production][relying_party_realm_production]
2. Modify **Relying Party Return URL** to be the URL of your application. Replace **example** with the DNS name you specified for your deployment.
    ![Relying party return URL for use in production][relying_party_return_url_production]
3. Click **Save** to save your updated replying party realm and return URL changes.
4. Keep the **Login Page Integration** page open in your browser, you'll need to copy from it shortly.
5. In Eclipse's Project Explorer, right-click **MyACSHelloWorld**, click **Build Path**, and then click **Configure Build Path**.
6. Click **Libraries**, click **Windows Azure Access Control Services Filter**, and then click **Edit**.
7. Using a browser opened to the **Login Page Integration** page of the Management Portal, copy the URL listed in the **Option 1: Link to an ACS-hosted login page** field and paste it into the **ACS Authentication Endpoint** field of the Eclipse dialog.
8. Using a browser opened to the **Edit Relying Party Application** page of the Management Portal, copy the URL listed in the **Realm** field and paste it into the **Relying Party Realm** field of the Eclipse dialog.
9. Within the **Security** section of the Eclipse dialog, type **${env.JAVA_HOME}/mycert.cer**.
10. [Optional] Keep **Require HTTPS connections** checked. If you set this option, you'll need to access your application using the HTTPS proptocol. If you don't want to require HTTPS connections, uncheck this option.
11. For a deployment to Windows Azure, your Windows Azure ACS Filter settings will look similar to the following.
    ![Windows Azure ACS Filter settings for a production deployment][add_acs_filter_lib_production]
12. Click **Finish** to close the **Edit Library** dialog.
13. Click **OK** to close the **Properties for MyACSHelloWorld** dialog.
14. In Eclipse, right-click **MyAzureACSProject**, right-click **WorkerRole1**, expand **Windows Azure Role**, and click **Components**.
15. Click **Add**.
16. With the **Add Component** dialog:
    1. In the **Import** section:
        1. Use the **File** button to navigate to the certificate you want to use. 
        2. For **Method**, select **copy**.
    2. For **As Name**, click on the text box and accept the default name.
    3. In the **Import** section:
        1. For **Method**, select **copy**.
        2. For **To directory**, type **%JAVA_HOME%**.
    4. Your **Add Component** dialog should look similar to the following.
        ![Add certificate component][add_cert_component]
    5. Click **OK**.
17. In Eclipse, click the **Publish to Windows Azure Cloud** button. Respond to the prompts, similar as done in the **To deploy your application to Windows Azure** section of the [Creating a Hello World Application for Windows Azure in Eclipse](http://msdn.microsoft.com/en-us/library/windowsazure/hh690944.aspx) topic. 

After your web application has been deployed, close any open browser sessions, run your web application, and you should be prompted to sign in with Windows Live ID credentials, followed by being sent to the return URL of your relying party application.

When you are done using your ACS Hello World application, remember to delete the deployment (you can learn how to delete a deployment in the [Creating a Hello World Application for Windows Azure in Eclipse](http://msdn.microsoft.com/en-us/library/windowsazure/hh690944.aspx) topic).


## <a name="next_steps">Next steps</a>

To further explore ACS's functionality and to experiment with more sophisticated scenarios, see [Access Control Service 2.0][].

[What is ACS?]: #what-is
[Concepts]: #concepts
[Prerequisites]: #pre
[Create a Java web application]: #create-java-app
[Create an ACS Namespace]: #create-namespace
[Add Identity Providers]: #add-IP
[Add a Relying Party Application]: #add-RP
[Create Rules]: #create-rules
[Upload a certificate to your ACS namespace]: #upload-certificate
[Review the Application Integration Page]: #review-app-int
[Configure Trust between ACS and Your ASP.NET Web Application]: #config-trust
[Add the ACS Filter library to your application]: #add_acs_filter_library
[Deploy to the compute emulator]: #deploy_compute_emulator
[Deploy to Windows Azure]: #deploy_azure
[Next steps]: #next_steps
[Access Control Service 2.0]: http://go.microsoft.com/fwlink/?LinkID=212360
[Windows Identity Foundation]: http://www.microsoft.com/download/en/details.aspx?id=17331
[Windows Identity Foundation SDK]: http://www.microsoft.com/download/en/details.aspx?id=4451
[Windows Azure Management Portal]: http://windows.azure.com
[acs_flow]: ../Media/ACSFlow.png
[portal_home_image]: ../Media/PortalHomeImage.png
[portal_sb_acs_caching]: ../Media/PortalSBACSCaching.png
[new_acs_namespace]: ../Media/NewACSNamespace.png
[new_acs_namespace_dialog]: ../Media/NewACSNamespaceDialog.png
[acs_home_page]: ../Media/ACSHomePage.png

<!-- Eclipse-specific -->
[add_acs_filter_lib]: ../Media/AddACSFilterLibrary.png
[add_acs_filter_lib_emulator]: ../Media/AddACSFilterLibraryEmulator.png
[add_acs_filter_lib_production]: ../Media/AddACSFilterLibraryProduction.png
[new_azure_deployment]: ../Media/NewAzureDeployment.png
[relying_party_realm_emulator]: ../Media/RelyingPartyRealmEmulator.png
[relying_party_return_url_emulator]: ../Media\RelyingPartyReturnURLEmulator.png
[relying_party_realm_production]: ../Media/RelyingPartyRealmProduction.png
[relying_party_return_url_production]: ../Media/RelyingPartyReturnURLProduction.png
[add_cert_component]: ../Media/AddCertificateComponent.png
[add_jsp_file_acs]: ../Media/AddJSPFileACS.png
[create_acs_hello_world]: ../Media/CreateACSHelloWorld.png
[add_token_signing_cert]: ../Media/AddTokenSigningCertificate.png