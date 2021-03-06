---
id: AK-Central-Site-Support
title: Central Site Collection Support
---

### Central Site Collection

The following configuration allows for rapid re-use of Akumina tooling across many site collections using an 'install once' model. This allows for easy maintenance, upgrades and code deployments.  Deploy once and affect all sites configured to use it. 

![](https://akuminadownloads.blob.core.windows.net/wiki/AkuminaDev/Akumina-Central-Site-Collection-Architecture.jpg)

* **Central Site** - site collection where the following Akumina specific components reside - this is where 'developers' will primarily being deploying too
    * Widgets
    * Widget Views
    * Configuration settings (DigispaceConfigurationIDS_AK)
    * Lists that need to be shared across many sites - may be specific to your implementation
* **Delivery Site** - site collection that will be configured to point to the 'Central Site' - this is where you will be site specific lists and libraries - especially those that are not going to be shared across other sites.

Your enterprise environment can also contain MULTIPLE Central Site's - Possibly 1 site collection for where Akumina widgets are installed and 1 site collection where shared lists can reside.  

For example:
* **Central Site** (Akumina) - /sites/akumina-assets
    * Code / Widget deployment  
* **Central Site** (Client Specific) /sites/customer-shared-assets
    * Specific lists / libraries / data

> It is up to your information archecture on how many 'Central Sites' you want to use. Akumina recommends at least 1 central site for the Akumina bits as denoted above.

**Quick usecases for a Central/Cross site**
* Maybe you want to share a MENU across multiple sites, allowing a content author to update a menu item in 1 list and have it reflect on all sites sharing using that MENU
* You may have a GLOBAL ALERT List where content authors can add a new alert in one location and have it show up across all sites
* You want to have a GLOBAL FOOTER where you dont have to manage the content for the footer in each site
* You want to roll out a new menu DESIGN / HTML / CSS across all sites without duplicating code or deploying to each site


### Delivery Site Contents - Classic

#### Master page updates
The classic Sharepoint master page should contain the following references

* JS
    * /sites/**centralsite**/digitalworkplace.vendor.min.js
    * /sites/**centralsite**/digitalworkplace.vendor-nomin.js
    * /sites/**centralsite**/digitalworkplace.min.js
    * /sites/**localsite**/digitalworkplace.env.js (this will contain configuration items to communicate with the appropriate central site)
* CSS
    * /sites/**centralsite**/digitalworkplace.min.css


#### digitalworkplace.env.js updates
````js
var LoaderConfiguration = LoaderConfiguration || {};
if ((typeof LoaderConfiguration.Environment) === 'undefined') {
    //Add shipped steps to loader

    LoaderConfiguration.Environment = {
        Init: function (config) {
            // tenant config:
           Akumina.Digispace.ConfigurationContext.InterchangeURL = "<AppManagerUrl>";
           Akumina.Digispace.ConfigurationContext.InterchangeQueryKey = "<AppManagerQueryKey>";
        }
    }
}

var AdditionalSteps = AdditionalSteps || {};
if ((typeof AdditionalSteps.EnvSteps) === 'undefined') {
    AdditionalSteps.EnvSteps = {
        Init: function () {
            var steps = [];
            steps.push({ stepName: "Auto Clear Local Cache", additionalSteps: [{ name: "Custom SetConfig", callback: AdditionalSteps.EnvSteps.SetConfig }] });
            return steps;
        },
        SetConfig: function () { 
            //uniqueId from central site collection (see below)
            Akumina.Digispace.SiteContext.UniqueId = "<UniqueId>"; 
            //Configure where views are loaded from
            Akumina.Digispace.ConfigurationContext.TemplateURLPrefix = "<Central Site CollectionUrl>"; //can be CDN as well
            //Configure what site the widgets/widget instances come from
            Akumina.Digispace.ConfigurationContext.WidgetInstanceSiteUrl = "<Central Site CollectionUrl>";

            Akumina.Digispace.AppPart.Eventing.Publish('/loader/onexecuted/');
        }
    }
}

````

### Delivery Site Contents - Modern
Modern usage in this mode is very minimal - this mode is refered to as "SPA" - Single Page Application - this means there is only 1 driver page

* App
   * akumina-single-page-application-client-side-solution.spkg
* Site Pages
   * akumina.aspx 

You can think of the single page application spkg as the replacement to the env.js we managed in Classic - it also contains the 'VirtualPageWidget' instance which handles all of the future Akumina page rendering and minipulation

![](https://akuminadownloads.blob.core.windows.net/wiki/AkuminaDev/Modern-SPA.PNG)


#### Getting UniqueId

![](https://akuminadownloads.blob.core.windows.net/wiki/AkuminaDev/GlobalSettings-UniqueId.PNG)


### Widget Scope


Using the central site model, widgets can be either global or local. Where the widget is deployed dictates how broadly it can be used and the degree of local control over the widget instance that can be exercised.
A site collection will use its own widgets or else the central site widget instances; the instance IDs can be the same for both. In both cases the widget instances are in the Akumina Assets central site collection; the site id and web id values are different to denote where the widget applies.
A widget deployed to the Akumina Assets central site can be used within any site collection that uses the central site.	A widget deployed to a specific site collection can be used only in that site collection, as shown below.
 
![](https://akuminadownloads.blob.core.windows.net/wiki/AkuminaDev/WidgetScope-Central.png)
![](https://akuminadownloads.blob.core.windows.net/wiki/AkuminaDev/WidgetScope-SiteCollection.png)

Delivery site 2 will use the same instance as delivery site 1 – any change made will affect the widget on all sites.	Delivery site 2 can use an instance with the same ID as delivery site 1, however, it is its own instance, needing to be deployed and managed independently. A change to the properties of the delivery site 1 instance will affect only delivery site 1, likewise a change to the properties of the delivery site 2 instance will affect only delivery site 2, etc.

**NOTE**: The widget instances can be overridden when used within a virtual page as well, giving further local control under either model.



### Widget Support for Cross site collection data retrieval

There are many widgets which support a hidden property called 'sitecollectionurl' - the code of the widgets are written to understand this property.  If you donot see the property in your installation, you can simply add a new text property to the widget definition.

**Widgets that support sitecollectionurl**  
The plan is to have every widget support this feature in future releases - if a widget is missing this support it is easy to patch / hotfix - submit a request on our support form: <https://www.akumina.com/technical-support/>
* BannerWidget
* CollatedDepartmentNewsWidget
* ContentBlockWidget
* DepartmentListWidget
* DepartmentNewsWidget
* DepartmentNewsItemWidget
* DepartmentNewsListWidget
* EventDetailWidget
* GenericItemWidget
* GenericListWidget
* ImportantDatesWidget
* LatestMediaWidget
* MyFormsWidget
* QuickLinksWidget
* BlogDetailWidget
* BlogsWidget
* BreakingNewsDetailWidget
* FoundationJobVacanciesWidget
* MyAnnouncementsWidget
* NewsCommentWidget
* NewsDetailWidget
* RelatedNewsWidget
* SpecialAnnouncementsDetailWidget

**sitecollectionurl usage**  
> name: sitecollectionurl - type: text - value: empty

Widget Definition
![](https://akuminadownloads.blob.core.windows.net/wiki/AkuminaDev/SiteCollectionUrl-AppManager.PNG)

Current user experience for setting the site collection url
![](https://akuminadownloads.blob.core.windows.net/wiki/AkuminaDev/SiteCollectionUrl-WidgetManager.PNG)

> Coming soon - site selector when choosing a list - this site picker may exist in the list selector

![](https://akuminadownloads.blob.core.windows.net/wiki/AkuminaDev/SitePicker.png)

### Using GetList to query across site collections

The GetList method supports a request object allowing you to specify the site collection you want to query, this allows for cross site collection data retrieval.. 

Example to query Central Site:

````js

    var request = {};
    request.listName = 'FoundationTopNavigation_AK';
    request.selectFields = 'ID,Title';
    //set the site collection url here
    //if contenxtSiteUrl is NOT set, it will query the list in the current site that the widget is rendering in
    request.contextSiteUrl = 'https://tenant.sharepoint.com/sites/centralsite'
    var legacyQuery = true;
    new Akumina.Digispace.Data.DataFactory(legacyQuery).GetList(request).then(function(x) {
        var listEnumerator = x.response.listItems.getEnumerator();
        var itemArr = [];
        while(listEnumerator.moveNext()) {
             var listItem = listEnumerator.get_current();
             var id = listItem.get_item('ID');
            var title = listItem.get_item('Title');
            itemArr.push({'ID': id, 'Title': title});
        }
        console.log(itemArr);
    });
    
````

### Deployment scenarios and Package setup
With the additional site collections in use, your deployment methodology will change slightly as the information architecture has changed
Here is an example project setup

* Project for Widgets -  This project will deploy to /sites/akumina-assets
* Project for Global Lists / Assets - this project will deploy to /sites/customer-shared-assets
* Project for a Particular Site - this project will deploy to /sites/foundationsite

This allows for different deployment configurations and minimal impact depending on the scenario - IE, I have to deploy a widget change, I only deploy the Widget Project.  If I need to deploy a global update for an asset or list, I dont interfere with the other sites directly or mess around with Akumina bits.

|  | **akumina-assets** | **customer-shared-assets** | **delivery** |
| --- | --- | --- | --- |
| `Prerequisite`     | Core install | Core Content Types | Core Content Types or Foundation Delivery |
|  |  |  |  |
| **Site Deployer Steps** | **akumina-assets** | **customer-shared-assets** | **delivery** |
| `masterpage`      |  |  | x |
| `js`              | x |  |  |
| `css`             | x |  |  |
| `lists`           |  | x | x |
| `layouts`         |  |  | x |
| `pages`           |  |  | x |
| `virtualpages`    |  |  | x |
| `controls`        |  |  | x |
| `widgets`         | x |  |  |
| `contentfiles`    | \* |  |  |
| `imagefiles`      | x |  |  |
| `homepage`        |  |  | x |
| `fonts`           | \* |  |  |
| `updatelists`     |  | x | x |
| `addtermsets`     |  | \* | x |
| `cdnassets`       |  |  |  |
| `sleep`           |  |  |  |
| `exportlists`     |  |  |  |
| `uploadfiles`     |  | \* | x |
| `spusersearch`    |  |  |  |
| `webpartgallery`  |  |  | \* |
| `groups`          | \* | \* | \* |
| `siteproperties`  |  |  | x |
| `spusersearch`    |  |  |  |
| `virtuallayout`   |  |  | x |
| `updatepagecache` |  |  |  |

\* optional 

### Sample Site Packages for use with Site Deployer
We can easily share site deployer packages for use with rolling up minimal delivery environments for both classic and modern - the goal is to eventually have these as an option in the Site Creator Management App within App Manager which allows business users to deploy Akumina functionality to any site they wish..  Look for those in upcoming point releases.

* Classic Minimal Site Deployer package (for developers) - Coming soon
* Modern Minimal Site Deploery package (for developers) - Coming soon








