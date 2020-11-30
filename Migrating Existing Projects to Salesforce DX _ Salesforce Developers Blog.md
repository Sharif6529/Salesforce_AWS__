Salesforce Developers Blog

Migrating Existing Projects to Salesforce DX {.h1heading .postheading}
============================================

Are you ready to move to [Salesforce
DX](https://developer.salesforce.com/platform/dx), but your source code
is currently in a Developer Edition (DE) or Sandbox org? In this blog
post, I describe the easy steps to convert existing source code to a
Salesforce DX project.

I recently converted the latest version of the
[DreamHouse](http://www.dreamhouseapp.io/) sample application to a
Salesforce DX project, and I thought it would be helpful to document the
process so you can use the same steps to convert your own projects to
Salesforce DX. The end result for the the DreamHouse app is available in
[this repository](https://github.com/dreamhouseapp/dreamhouse-sfdx), but
the steps below are applicable to any project.

Step 1: Convert your project to Salesforce DX {.h2heading .postheading}
---------------------------------------------

1.  Create an unmanaged package in your existing org, and include all
    the assets you want to move to your Salesforce DX project. You don’t
    need to upload the package, and you don’t have to worry about the
    75% code coverage for this purpose.
2.  If you haven’t already done so, authenticate with your hub org. Type
    the following command, then login with your hub org credentials and
    accept to provide access to Salesforce DX:
    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)
      --- -----------------------------------------------------------------
      1   `sfdx force:auth:web:login -d -a myhuborg`{.VisualForce .plain}
      --- -----------------------------------------------------------------

3.  Create a new Salesforce DX project:
    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)
      ------------------------------------ ------------------------------------
      1                                    `sfdx force:project:create -n myproj
      2                                    ect`{.VisualForce
                                           .plain}
                                           `cd myproject`{.VisualForce .plain}
      ------------------------------------ ------------------------------------

4.  Authenticate with your developer edition org. Type the following
    command, then authenticate with your developer edition org
    credentials and accept to provide access to Salesforce DX:
    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)
      --- --------------------------------------------------------------
      1   `sfdx force:auth:web:login -a mydevorg`{.VisualForce .plain}
      --- --------------------------------------------------------------

5.  Export the unmanaged package metadata in a temporary directory. Type
    the following commands in the root folder of your Salesforce DX
    project:

    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)

      ------------------------------------ ------------------------------------
      1                                    `mkdir temp`{.VisualForce .plain}
      2                                    `sfdx force:mdapi:retrieve -s -r ./t
                                           emp -u mydevorg -p mypackage`{.Visua
                                           lForce
                                           .plain}
      ------------------------------------ ------------------------------------

    “mypackage” is the name of the unmanaged package you created in step
    1. Afer executing this command, a file named **unpackaged.zip** is
    created in the **temp** directory.

6.  Unzip **unpackaged.zip** using a zip file management utility or from
    the command line. For example, on a Mac, double-click unpackaged.zip
    in Finder or type the following command on the command line:
    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)
      --- ---------------------------------------------------------------
      1   `unzip ./temp/unpackaged.zip -d ./temp/`{.VisualForce .plain}
      --- ---------------------------------------------------------------

7.  Convert the source code to the Salesforce DX project structure:

    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)

      --- -----------------------------------------------------------
      1   `sfdx force:mdapi:convert -r ./temp`{.VisualForce .plain}
      --- -----------------------------------------------------------

    You can delete the **temp** directory at this point.

8.  Create a scratch org:

    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)

      --- -----------------------------------------------------------------------------------------------------
      1   `sfdx force:org:create -s -f config/project-scratch-def.json -a  myscratchorg`{.VisualForce .plain}
      --- -----------------------------------------------------------------------------------------------------

    If you get an **expired access/refresh token** message, authenticate
    with your hub org again (step 2 above).

9.  Push the code to your scratch org:
    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)
      --- -----------------------------------------------
      1   `sfdx force:source:push`{.VisualForce .plain}
      --- -----------------------------------------------

10. Open the scratch org:
    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)
      --- --------------------------------------------
      1   `sfdx force:org:open`{.VisualForce .plain}
      --- --------------------------------------------

11. Create a permission set to provide access to your project’s assets:

    -   In **Setup**, type **permission** in the quick find box, click
        the **Permission Sets** link, and click **New**
    -   Provide a name for your permission set and click **Save**
    -   Add the required permissions
    -   Click **Manage Assignments** and assign the permission set to
        the default user (User, User)

    For example, here are the permissions I added for the DreamHouse
    app:

    -   **Assigned Apps**: I added the DreamHouse app
    -   **Object Settings**: I added all available permissions (Tab
        settings, Object Permissions, and Field permissions) for all the
        custom objects used in DreamHouse: Broker\_\_c, Property\_\_c,
        Favorite\_\_c and Bot\_Command\_\_c.
    -   **Apex Class Access**: I added all the classes
    -   **Visualforce Page Access**: I added all the Visualforce pages
    -   In the Find Settings box, I typed the name of all the tabs used
        in DreamHouse (HeatMap, HeatMap demo, Einstein Vision, and
        Command Center), and I enabled access (checked Visible).

12. Pull the permission set
    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)
      --- -----------------------------------------------
      1   `sfdx force:source:pull`{.VisualForce .plain}
      --- -----------------------------------------------

Step 2: Transfer sample data (Optional) {.h2heading .postheading}
---------------------------------------

The Salesforce DX CLI also makes it easy to export sample data from your
developer org and import it in your scratch orgs. It even preserves
master-details relationships between records. As an example, here is how
I did it for the DreamHouse app:

To export the data from my DE org, I ran the following command from the
root folder of my Salesforce DX project:

[?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)

  ------------------------------------ ------------------------------------
  1                                    `sfdx force:data:tree:export -q "\`{
  2                                    .VisualForce
  3                                    .plain}
  4                                    `    `{.VisualForce
  5                                    .spaces}`SELECT`{.VisualForce
  6                                    .apexKey} `Id, \`{.VisualForce
  7                                    .plain}
  8                                    `           `{.VisualForce
  9                                    .spaces}`Name, \`{.VisualForce
  10                                   .plain}
  11                                   `           `{.VisualForce
  12                                   .spaces}`Title__c`{.VisualForce
  13                                   .apexObjects}`, \`{.VisualForce
  14                                   .plain}
  15                                   `           `{.VisualForce
  16                                   .spaces}`Phone__c`{.VisualForce
  17                                   .apexObjects}`, \`{.VisualForce
  18                                   .plain}
  19                                   `           `{.VisualForce
  20                                   .spaces}`Mobile_Phone__c`{.VisualFor
  21                                   ce
  22                                   .apexObjects}`, \`{.VisualForce
  23                                   .plain}
  24                                   `           `{.VisualForce
  25                                   .spaces}`Email__c`{.VisualForce
                                       .apexObjects}`, `{.VisualForce
                                       .plain}`Picture__c`{.VisualForce
                                       .apexObjects}`, \`{.VisualForce
                                       .plain}
                                       `           `{.VisualForce
                                       .spaces}`(`{.VisualForce
                                       .plain}`SELECT`{.VisualForce
                                       .apexKey} `Id, \`{.VisualForce
                                       .plain}
                                       `                   `{.VisualForce
                                       .spaces}`Name, \`{.VisualForce
                                       .plain}
                                       `                   `{.VisualForce
                                       .spaces}`Address__c`{.VisualForce
                                       .apexObjects}`, \`{.VisualForce
                                       .plain}
                                       `                   `{.VisualForce
                                       .spaces}`City__c`{.VisualForce
                                       .apexObjects}`, \`{.VisualForce
                                       .plain}
                                       `                   `{.VisualForce
                                       .spaces}`State__c`{.VisualForce
                                       .apexObjects}`, \`{.VisualForce
                                       .plain}
                                       `                   `{.VisualForce
                                       .spaces}`Zip__c`{.VisualForce
                                       .apexObjects}`, \`{.VisualForce
                                       .plain}
                                       `                   `{.VisualForce
                                       .spaces}`Price__c`{.VisualForce
                                       .apexObjects}`, \`{.VisualForce
                                       .plain}
                                       `                   `{.VisualForce
                                       .spaces}`Title__c`{.VisualForce
                                       .apexObjects}`, \`{.VisualForce
                                       .plain}
                                       `                   `{.VisualForce
                                       .spaces}`Beds__c`{.VisualForce
                                       .apexObjects}`, \`{.VisualForce
                                       .plain}
                                       `                   `{.VisualForce
                                       .spaces}`Baths__c`{.VisualForce
                                       .apexObjects}`, \`{.VisualForce
                                       .plain}
                                       `                   `{.VisualForce
                                       .spaces}`Location__Longitude__s, \`{
                                       .VisualForce
                                       .plain}
                                       `                   `{.VisualForce
                                       .spaces}`Location__Latitude__s, \`{.
                                       VisualForce
                                       .plain}
                                       `                   `{.VisualForce
                                       .spaces}`Picture__c`{.VisualForce
                                       .apexObjects}`, \`{.VisualForce
                                       .plain}
                                       `                   `{.VisualForce
                                       .spaces}`Thumbnail__c`{.VisualForce
                                       .apexObjects}`, \`{.VisualForce
                                       .plain}
                                       `                   `{.VisualForce
                                       .spaces}`Description__c`{.VisualForc
                                       e
                                       .apexObjects} `\`{.VisualForce
                                       .plain}
                                       `            `{.VisualForce
                                       .spaces}`FROM`{.VisualForce
                                       .apexKey}
                                       `Properties__r) \`{.VisualForce
                                       .plain}
                                       `    `{.VisualForce
                                       .spaces}`FROM`{.VisualForce
                                       .apexKey} `Broker__c`{.VisualForce
                                       .apexObjects}`" \`{.VisualForce
                                       .plain}
                                       `    `{.VisualForce
                                       .spaces}`-u mydevorg --outputdir ./d
                                       ata --plan`{.VisualForce
                                       .plain}
  ------------------------------------ ------------------------------------

After executing this command, the sample data files (Broker\_\_cs.json,
Property\_\_cs.json, and Broker\_\_c-Property\_\_c-plan.json) are
available in the **data** subfolder of my project which means that they
will be part of the project when I put it under version control (step 3
below).

To import the data in my scratch org, I run the following command:

[?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)

  ------------------------------------ ------------------------------------
  1                                    `sfdx force:data:tree:`{.VisualForce
  2                                    .plain}`import`{.VisualForce
                                       .apexKey}
                                       `-u myscratchorg \`{.VisualForce
                                       .plain}
                                       `    `{.VisualForce
                                       .spaces}`--plan ./data/`{.VisualForc
                                       e
                                       .plain}`Broker__c-Property__c`{.Visu
                                       alForce
                                       .apexObjects}`-plan.json`{.VisualFor
                                       ce
                                       .plain}
  ------------------------------------ ------------------------------------

Step 3: Put your project under version control {.h2heading .postheading}
----------------------------------------------

We are done converting the project and transferring data. It’s time to
put the project under version control. For example, using Github:

[?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)

  ------------------------------------ ------------------------------------
  1                                    `git init`{.VisualForce .plain}
  2                                    `git add . -A`{.VisualForce .plain}
  3                                    `git commit -m `{.VisualForce
  4                                    .plain}`'first commit'`{.VisualForce
  5                                    .attrString}
                                       `git remote add origin https:`{.Visu
                                       alForce
                                       .plain}`//github.com/yourname/your-r
                                       epo.git`{.VisualForce
                                       .apexComments}
                                       `git push origin master`{.VisualForc
                                       e
                                       .plain}
  ------------------------------------ ------------------------------------

That’s it. It’s now easier than ever for other developers to work with
you on the project. Using the migrated DreamHouse project as an example,
all they have to do is:

1.  Clone the repository. For example:
    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)
      ------------------------------------ ------------------------------------
      1                                    `git clone https:`{.VisualForce
      2                                    .plain}`//github.com/dreamhouseapp/d
                                           reamhouse-sfdx`{.VisualForce
                                           .apexComments}
                                           `cd dreamhouse-sfdx`{.VisualForce
                                           .plain}
      ------------------------------------ ------------------------------------

2.  Create their own scratch org:
    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)
      --- ----------------------------------------------------------------------------------------------------
      1   `sfdx force:org:create -s -f config/project-scratch-def.json -a myscratchorg`{.VisualForce .plain}
      --- ----------------------------------------------------------------------------------------------------

3.  Push the app to their scratch org:
    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)
      --- -----------------------------------------------
      1   `sfdx force:source:push`{.VisualForce .plain}
      --- -----------------------------------------------

4.  Assign the permission set to the default user. For example:
    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)
      --- ---------------------------------------------------------------------
      1   `sfdx force:user:permset:assign -n dreamhouse`{.VisualForce .plain}
      --- ---------------------------------------------------------------------

5.  Import data in the scratch org. For example:
    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)
      ------------------------------------ ------------------------------------
      1                                    `sfdx force:data:tree:`{.VisualForce
      2                                    .plain}`import`{.VisualForce
                                           .apexKey}
                                           `-u myscratchorg \`{.VisualForce
                                           .plain}
                                           `    `{.VisualForce
                                           .spaces}`--plan ./data/`{.VisualForc
                                           e
                                           .plain}`Broker__c-Property__c`{.Visu
                                           alForce
                                           .apexObjects}`-plan.json`{.VisualFor
                                           ce
                                           .plain}
      ------------------------------------ ------------------------------------

6.  Open the scratch org:
    [?](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)
      --- --------------------------------------------
      1   `sfdx force:org:open`{.VisualForce .plain}
      --- --------------------------------------------

Summary {.h2heading .postheading}
-------

Salesforce DX enables modern development workflows on the Salesforce
platform. It makes team development, version control, and continuous
integration easier than ever. Migrate your existing projects to
Salesforce DX today using the easy steps described in this post and
watch the pull requests come in!

Resources {.h2heading .postheading}
---------

-   Trailhead Module: [App Development with Salesforce
    DX](https://trailhead.salesforce.com/modules/sfdx_app_dev)
-   [Salesforce DX
    microsite](https://developer.salesforce.com/platform/dx)

Published

By [Christophe
Coenraets](https://developer.salesforce.com/blogs/author/ccoenraets)

July 31, 2017

Topics:

[Architecture](https://developer.salesforce.com/blogs/category/architecture)\
[Developer
Experience](https://developer.salesforce.com/blogs/category/developer-experience)

[salesforce
dx](https://developer.salesforce.com/blogs/tag/salesforce-dx)

[](https://developer.salesforce.com/blogs/feed)

Subscribe to RSS

![](./Migrating%20Existing%20Projects%20to%20Salesforce%20DX%20_%20Salesforce%20Developers%20Blog_files/RSSIcon.svg)

Subscribe to email

![](./Migrating%20Existing%20Projects%20to%20Salesforce%20DX%20_%20Salesforce%20Developers%20Blog_files/mail-icon-white.svg)

[](https://www.facebook.com/sharer/sharer.php?u=&t= "Share on Facebook")

Share on Facebook

![](./Migrating%20Existing%20Projects%20to%20Salesforce%20DX%20_%20Salesforce%20Developers%20Blog_files/facebook-white.svg)
[](https://twitter.com/share?ref_src=twsrc%5Etfw)

Share on Twitter

![](./Migrating%20Existing%20Projects%20to%20Salesforce%20DX%20_%20Salesforce%20Developers%20Blog_files/twitter-white.svg)

[](http://www.linkedin.com/shareArticle?mini=true&url=&title=&summary=&source=)

Share on Linkedin

![](./Migrating%20Existing%20Projects%20to%20Salesforce%20DX%20_%20Salesforce%20Developers%20Blog_files/linkedin-white.svg)

You may also find interesting {.slds-size--1-of-1 .slds-text-heading--large .yarpp-header}
-----------------------------

[](https://developer.salesforce.com/blogs/2017/10/salesforce-dx-is-now-generally-available.html)

Salesforce DX is now Generally Available! {.cardtitle}
-----------------------------------------

October 17, 2017

By Wade Wegner

[](https://developer.salesforce.com/blogs/developer-relations/2017/07/northern-trail-outfitters-new-sample-application-lightning-components-platform-events-salesforce-dx.html)

Northern Trail Outfitters: New Sample Application with Lightning Components, Platform Events, and Salesforce DX {.cardtitle}
---------------------------------------------------------------------------------------------------------------

July 13, 2017

By Christophe Coenraets

[](https://developer.salesforce.com/blogs/2017/10/site-switching-apex-callouts.html)

Site Switching and Apex Callouts {.cardtitle}
--------------------------------

October 12, 2017

By Aaron Slettehaugh

[](https://developer.salesforce.com/blogs/2017/10/salesforce-dx-cli-plugin-update.html)

New Signature Validation in Salesforce CLI Plugins {.cardtitle}
--------------------------------------------------

October 20, 2017

By Dave Carroll

Like our posts?\
Subscribe to our blog!

Get notified when we publish new updates.

![](./Migrating%20Existing%20Projects%20to%20Salesforce%20DX%20_%20Salesforce%20Developers%20Blog_files/cody-salesforce-animated.gif)

Like our posts? Subscribe to our blog!

Get notified when we publish new updates.

\*First Name

\*Last Name

\*Email

\*Country

Loading

Select your Country

Blog subscription preferences

Send me new posts immediately

Send me a bi-weekly digest of all posts

Loading

Subscribe

Thanks for subscribing. You'll be among the first to learn about
Salesforce developer best practices and product news.\
A confirmation has been sent to the email address provided.

Feedback

Developer Site Feedback

Documentation

-   [All Documentation](https://developer.salesforce.com/docs/)
-   [Apex](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/)
-   [APIs](https://developer.salesforce.com/docs/?filter_text=api&service=All%20Services)
-   [Lightning Aura
    Components](https://developer.salesforce.com/docs/atlas.en-us.lightning.meta/lightning/)
-   [Lightning Component
    Library](https://developer.salesforce.com/docs/component-library/)

Developer Centers

-   [Commerce Cloud](https://developer.commercecloud.com/)
-   [Community
    Cloud](https://developer.salesforce.com/developer-centers/community-cloud/)
-   [Einstein
    Analytics](https://developer.salesforce.com/developer-centers/einstein-analytics/)
-   [Einstein
    Platform](https://developer.salesforce.com/developer-centers/ein-platform/)
-   [Embedded Service SDK for
    Mobile](https://developer.salesforce.com/developer-centers/service-cloud/sdk)
-   [Heroku Developer Center](https://devcenter.heroku.com/)
-   [Identity](https://developer.salesforce.com/developer-centers/identity/)
-   [Integration and
    APIs](https://developer.salesforce.com/developer-centers/integration-apis/)
-   [Lightning
    Apps](https://developer.salesforce.com/developer-centers/lightning-apps/)
-   [Lightning
    Flow](https://developer.salesforce.com/developer-centers/lightning-flow/)
-   [Marketing
    Cloud](https://developer.salesforce.com/developer-centers/marketing-cloud/)
-   [Mobile Developer
    Center](https://developer.salesforce.com/developer-centers/mobile/)
-   [Mulesoft Developer Center](https://developer.mulesoft.com/)
-   [Pardot Developer Center](http://developer.pardot.com/)
-   [Quip Developer Center](https://quip.com/dev/)
-   [Salesforce
    DX](https://developer.salesforce.com/developer-centers/developer-experience/)
-   [Security](https://developer.salesforce.com/developer-centers/security/)
-   [Service
    Cloud](https://developer.salesforce.com/developer-centers/service-cloud/)

Learn

-   [Blog](https://developer.salesforce.com/blogs/)
-   [Certification](https://trailhead.salesforce.com/credentials/developeroverview)
-   [Events and Webinars](https://developer.salesforce.com/calendar)
-   [Podcast](https://developer.salesforce.com/podcast)
-   [Sample Gallery](https://trailhead.salesforce.com/sample-gallery)
-   [Tools](https://developer.salesforce.com/tools)
-   [Trailhead](https://trailhead.salesforce.com/)
-   [Video Gallery](https://developer.salesforce.com/tv)

Community

-   [Developer Groups](https://trailblazercommunitygroups.com/)
-   [Forums](https://developer.salesforce.com/forums/)
-   [Partners](http://go.appexchange.com/partnerprogram)
-   [Success
    Stories](https://trailhead.salesforce.com/trailblazers?#role-dev)

![](./Migrating%20Existing%20Projects%20to%20Salesforce%20DX%20_%20Salesforce%20Developers%20Blog_files/Newsletter-Icon.svg)

Subscribe to our monthly newsletter

Your email adress:

go

[![](./Migrating%20Existing%20Projects%20to%20Salesforce%20DX%20_%20Salesforce%20Developers%20Blog_files/facebook.png)](https://www.facebook.com/salesforcedevs)
[![](./Migrating%20Existing%20Projects%20to%20Salesforce%20DX%20_%20Salesforce%20Developers%20Blog_files/twitter.png)](https://twitter.com/#!/salesforcedevs)
[![](./Migrating%20Existing%20Projects%20to%20Salesforce%20DX%20_%20Salesforce%20Developers%20Blog_files/linkedIn.png)](https://www.linkedin.com/groups/Developer-Force-Forcecom-Community-3774731)
[![](./Migrating%20Existing%20Projects%20to%20Salesforce%20DX%20_%20Salesforce%20Developers%20Blog_files/youTube.png)](https://www.youtube.com/user/DeveloperForce)

© Copyright 2000-2020 salesforce.com, inc. All rights reserved. Various
trademarks held by their respective owners. \
 Salesforce.com, inc. Salesforce Tower, 415 Mission Street, 3rd Floor,
San Francisco, CA 94105, United States

-   [Privacy Statement](https://www.salesforce.com/company/privacy.jsp)
-   [Security
    Statement](https://www.salesforce.com/company/security.jsp)
-   [Terms of Use](https://trailblazer.me/terms)
-   Cookie Preferences
-   [![Feedback](./Migrating%20Existing%20Projects%20to%20Salesforce%20DX%20_%20Salesforce%20Developers%20Blog_files/opinionlab-orange.gif)
    Feedback](javascript:void(0);)
-   [About Us](http://www.salesforce.com/company/)
-   [Language: English](javascript:void(0);)
-   Choose a Language
    -   English
    -   日本語

Cookie Consent Manager {#pc-title}
----------------------

### General Information {#privacy-text}

### General Information {#pc-privacy-header}

We use three kinds of cookies on our websites: required, functional, and
advertising. You can choose to opt out of functional and advertising
cookies. Click on the different cookie categories to find out more about
each category and to change the default settings. [Privacy
Statement](https://www.salesforce.com/company/privacy/full_privacy/)

-   ### Required Cookies

    ### Required Cookies {.category-header}

    Always Active

    Required cookies are necessary for basic website functionality. Some
    examples include: session cookies needed to transmit the website,
    authentication cookies, and security cookies.

    -   ### First Party Cookies {.first-party-cookie-header}

        -   heroku-session-affinity
        -   apex\_\_dfc\_locale
        -   \_session\_id
        -   BrowserId
        -   pctrk
        -   useSiteUrlRewriter
        -   OptanonConsent
        -   sccGuestUID
        -   dfc\_site\_production\_session
        -   PHPSESSID

    [Cookies
    Details‎](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)

-   ### Functional Cookies

    ### Functional Cookies {.category-header}

    Functional Cookies

    Functional cookies enhance functions, performance, and services on
    the website. Some examples include: cookies used to analyze site
    traffic, cookies used for market research, and cookies used to
    display advertising that is not directed to a particular individual.

    -   ### First Party Cookies {.first-party-cookie-header}

        -   optimizelyPendingLogEvents
        -   \_gat
        -   \_\_atuvc
        -   optimizelyEndUserId
        -   \_gat\_UA-45076517-1
        -   \_\_atuvs
        -   \_gat\_UA-134722592-1
        -   \_ga
        -   optimizelySegments
        -   \_gid
        -   optimizelyBuckets

    [Cookies
    Details‎](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)

-   ### Advertising Cookies

    ### Advertising Cookies {.category-header}

    Advertising Cookies

    Advertising cookies track activity across websites in order to
    understand a viewer’s interests, and direct them specific marketing.
    Some examples include: cookies used for remarketing, or
    interest-based advertising.

    [Cookies
    Details‎](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)

[](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)

Back Button

### Advertising Cookies {#vendors-list-title}

[](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)

Select All Vendors

Select All Hosts

Select All

-   REPLACE-WITH-DYANMIC-HOST-ID

    ### 33Across {.host-title}

    #### 33Across {.host-description}

    [View Third Party
    Cookies](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)

    -   1020, Inc. dba Placecast

-   REPLACE-WITH-DYANMIC-VENDOR-ID

    ### 33Across {.vendor-title}

    3 Purposes

    [View Privacy
    Notice](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)

    Consent Purposes

    Location Based Ads

    Consent Allowed

    Legitimate Interest Purposes

    Personalize

    [](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)

    Require Opt-Out

    Features

    Location Based Ads

[](https://developer.salesforce.com/blogs/developer-relations/2017/07/migrating-existing-projects-salesforce-dx.html#)

Clear Fliters

Information storage and access

Apply

Accept All Cookies

Save Settings

Powered By OneTrust

Link to OneTrust Website Powered by

\_\_PRESENT
