# digital analytics fundamentals
Customers are more in control in their purchasing decisions than ever. Information about products, prices and options is readily available on the Internet. Since customers can approach your product from a number of different avenues, you need to focus on the customer, not on the channels.

Patterns of consumer behavior have become more complex, but also more measurable, thus increasing demand for analytics.

Google Analytics is:

 - a huge database that can store any data you want, as long as you structure it in a way it can understand. It organizes data into interactions/hits and can accommodate offline behavior.
 - an API that you use to store this data, and some Javascript code that allows you to interact with the API
 - set of configuration tools to pre-process the data. (And an important note: this processing is irreversible!)
 - a set of principles for organizing that data (eg, classifying sources of traffic on the dimensions of source and medium)
 - a set of reports / ways of looking at the data
 - a reporting API for getting the processed data out

## business objectives
common business objectives online and their outcomes:
 - ecommerce (sales)
 - lead generation (contacts)
 - content publishing (number of ads shown)
 - support/info (successful and quick dissemination of info)
 - branding (brand exposure and engagement with the brand)

An objective is fully met = **macro conversion**. A **micro conversion** is a consumer action that is considered a step towards the main objective. [Examples of micro conversions](https://support.google.com/analytics/answer/2665210)

**Attribution** means assigning credit to a conversion. You say that you got this amount of money (or another metric) from that marketing action (e.g., a particular channel). Assists are all the marketing activities before the last one before the desired outcome is reached. A common attribution is the last click attribution, meaning that the last marketing activity gets all the credit. [Other attribution models](https://support.google.com/analytics/answer/1665189)

## measurement plan
[link](https://www.kaushik.net/avinash/digital-marketing-and-measurement-model/)
 - define business objectives and strategies to achieve it (sales, drive store visits, blog visits and engagement...)
 - define KPIs (key metrics) and segmentation
 - implementation plan (query string parameters, server redirects, AJAX events...). It's not clear to me whether Analytics enables you to track all of these or whether you have to implement it on your code

# analysis

## collecting data
Google analytics is capable of tracking... everything? Websites, gaming consoles, IoT devices (!)...

Website tracking: you implement a piece of Javascript code on each page on your website (you should put it inside the head tags). It collects data from your website (eg, URL), from the user's browser or from the referring website and sends that data to a central location for processing. You can also create custom Javascript modules to collect additional data and send it to Google.

On mobile, you use native code instead of Javascript, you track activities not pageviews, and you can store the data locally for when the device isn't connected to the Internet.

In both cases, it is crucial to define what constitutes a user interaction, also called a hit. Each datapoint will refer to such an interaction: e.g,. page view, purchase, taking a photo, ...

## organization of data

 - create separate Analytics accounts for different businesses
 - define individual assets inside accounts (e.g., website, mobile app, ...)
 - rollup reporting: combining two or more assets in one report. You do this by [creating another property and sending data to it](https://analytics.googleblog.com/2009/09/advanced-structure-your-account-with.html)
 - define views for each properties. You use these to pre-process your data. Since data is lost after processing, it is advisable to have a raw, unfiltered view in addition to other views. You can also use these to restrict access to data on your account. 

## filters
Common applications

 - exclude IPs from your employees
 - set all page URLs to lowercase
 - restrict data to a geographic location

Filter order matters.

## variables
[reference](https://developers.google.com/analytics/devguides/reporting/core/dimsmets)

Dimensions are qualitative variables. They are valid for a particular level: users, sessions or interactions. Common dimensions
 - geography
 - traffic source
 - page title

Metrics are quantitative vars, behaviors. Common metrics
 - nr of visitors (user level)
 - avg nr of pages visited per session
 - conversion rates (session level)

A session is defined as a period of continuous activity on the website, until the session timeout expired. Timeout is commonly set to 30 mins but you can change it.

What is activity? **Page views** (automatically implemented) and **events** like watching video (require customization). The definition of activity, and most obviously the time metrics, is dependent on what you define as an interaction.

Bounce: a session with only one interaction, ie. landing on a page. Be careful when implementing new events, the bounce statistics will probably change since more interactions are recorded. But you can exclude an event from counting in bounce statistics.

When implementing events bear in mind that there is a steep limit on how many of them you can send: one per second after the first ten.

[more info on event tracking](https://support.google.com/analytics/answer/1033068)

## reporting
Use the Google dashboard or retrieve data using the API. Each query to the API needs to have View ID, start-end dates, metrics and dimensions

### main report
Time series graph. Options

 - set active time range, set granularity (day, week, month)
 - comparison date range
 - attach comments to dates (private or public)
 - change metric displayed, compare two metrics

A table below.

 - aggregate data on metrics across a dimension or a combination of two dimensions
 - adjust the sets of metrics that appear, you have metric groups (eg, "site usage")
 - filter on the primary dimension or a metric
 - change views (i.e., plot data)
    - percentage view = piechart
    - performance = bar graph
    - comparison view = bar graph with negative values for below average
    - pivot view = both rows and columns have dimensions (i.e., cross-table)
 - plot rows: select rows that you want to plot on the time graph (ie, dimension values you want to plot)

Save the settings as shortcuts

Segment the data using prepared or custom segments. Interesting: you can segment on sequences of user interactions (eg, started by looking at shirts, then shoes).

The process Google Analytics uses to retrieve data from large, complex data sets faster is called report sampling. This means that they do not use all sessions to calculate the table in order to speed up the data retrieval. You can set the size of the sample in config, but to get the whole sample you need to get a Premium account which costs 150 000$. Yes.

### audience reports
demographics
 - locations is a heatmap of the world. Also has a table (I guess each report has one)
 - languages

behavior
 - new vs returning
 - frequency: how often in a certain timeframe) vs recency (how long since last visit
 - engagement: how much time they spend (= visit duration), how many pages they view (= page depth)

technology
 - browser & OS
 - network

mobile
 - which device

custom dimensions, eg if you gave your users a form

### acquisition reports
How people find your website or app. Break by source, medium, keyword or other dimensions. Get the landing page as a dimension.

reports available

 - overview
 - channels (rolling up sources/mediums)
 - campaigns

### adwords reports
Goal is the same as with acquisition reports, to see why users go to your site. But it's specific to AdWords and has more granular data.

hierarchy: campaign -> ad groups -> keywords

metrics

 - visits
 - impressions, nr of ads displayed
 - clicks, should be lower than visits since users can re-visit your site another way (how do they track this, by cookies?). If you have fewer visits than clicks, could be that you did not install the js code on each landing page, could that users stopped the page from loading before analytics code was run or could be that users do not have js, images(?) and cookies enabled.
 - CTR (click through rate), ie conversion through clicks
 - RPC revenue per click, if you have ecommerce enabled or goal values
 - ROI return on investment, % you earned for what you paid. 0% means you earned as much as you paid.

dimensions

 - Bid adjustments helps you adjust the bids on adwords based on the campaign success rates
 - Keywords report analyze ... keywords
 - Matched search queries tell you which Google queries displayed which adword keywords for your users
 - time of the day
 - destination URLs report tells you which landing pages performed best. The ad distribution network shows a breakdown on google/search partners/content
 - placement: automatic vs managed placements
 - keyword position: which position did your keyword take (1st, 2nd...)

### behavior reports
What do look at, order of movement, on-site search engine

 - all pages: data on which pages are most popular and have the best metrics (eg, bounce rate)
 - content drilldown: pages by directory structure
 - landing pages and exit pages
 - events
 - site search
 - behavior flows (page-to-page). You can also include events.

### goal flow report
A funnel for goals you set up.

### ecommerce report
Need to enable ecommerce and add the code to the application

 - product performance: sales per product
 - sales performance: sales per date
 - transactions performance: basket analysis

Conversion is calculated in the wrong way: nr of transactions / nr of visits. This means if you have one user making 3 transactions, conversion = 300%.

enhanced ecommerce has

 - product list performance, I guess that's something like a list of the products you sell. It sounds like something analogous to shelves in a physical store.
 - product performance: sales, but also other shopping behavior data like clicks to see product detail, times it showed in a product list, times it was removed from the cart, cart-to-detail (% of times shoppers that viewed a product put it in the cart), buy-to-detail (% that bought)
 - shopping behavior, a funnel with these steps: arrive at page, product view, add to cart, checkout, buy
 - checkout behavior, funnel similar to shopping behavior: sign in, shipping, billing address, payment info, buying. You can create segments of users who dropped off. 

### multi-channel funnel reports

 - assisted conversions: analyze sequences of visits that lead to a transaction or a goal, across different sessions. Set the "lookback window" (ie, length of the period that you take into account), default is 30 days. These reports show the statistics involving assisted vs direct conversions (eg, ratio btw the two)
 - top conversion paths: shows unique paths users take to convert
 - time lag: how long btw first and last interaction before the conversion, ie length of sales cycle
 - path length: nr of channel interactions in the conversion paths

Assisted/Last Click or Direct Conversions and First/Last Click or Direct Conversions ratios summarize a channel’s overall role. A value close to 0 indicates that a channel completed more sales and conversions than it assisted. A value close to 1 indicates that the channel equally assisted and completed sales and conversions. The more this value exceeds 1, the more the channel assisted sales and conversions.

### attribution reports
How to assign value to steps in conversion paths. Check attribution models (here)[https://support.google.com/analytics/answer/1665189?hl=en]. Different models allow you to explore which role a given marketing channel should take (eg, drive awareness vs close sales) and to re-budget your marketing.

## goals / conversions
You can set up these types of goals

 - destination, if the user viewed a page or a screen). This means your page should have a meaningful URL scheme. You can also create funnels and monitor which path the users take to get to the goal.
 - engagement: duration or pages per visit
 - event

You can also set up tracking of ecommerce transactions. If you do, it's best not to track these with goals. Goals are tracked once per session, but there can be multiple transations.

## segmentation
Looking at data over different categories.

 - time
 - geography
 - devices (mobile, PC, ...)
 - marketing channels (email, Twitter, Facebook, ...)
 - repeat customers vs first-time customers

default channels

 - Direct
 - Organic Search
 - Referral
 - Email
 - Paid Search
 - Other Advertising
 - Social
 - Display

### segmentation by source and medium
Source: what was the referring site? Direct might mean users typed the name of the site in the URL box or clicked on a bookmark, but it also aggregates many other non-identified sources ([link](https://megalytic.com/blog/understanding-direct-traffic-in-google-analytics)).

Medium means how they got there. Three mediums are tracked automatically.

 - organic / unpaid search result
 - referral is from any other site other than search sites
 - none means typing the URL or clicking on a bookmark

According to Google, every referral to a website has a medium. Possible medium include: “organic” (unpaid search), “cpc” (cost per click, i.e. paid search), “referral” (referral), “email” (the name of a custom medium you have created), “none” (direct traffic has a medium of “none”).

To get more data you need to use link tags. You can overwrite the default source and medium tags. And you can add additional data: campaign, term (paid keyword campaigns), content (differentiate versions of an ad). 

Adwords populates tags automatically. CPC means cost-per-click, ie paid search.

Channels allow you to organize your traffic sources and mediums into groups. Several channels are predefined, like email, organic, social...

## context
External, looking at industry benchmark data. Are your results due to a general trend?

Internal, comparing to past data.

# google analytics platform principles
four parts: collection, configuration, processing, reporting

## data model
Main analysis units: users, sessions and interactions

## collection
Main tracking code for pageviews: On a website you put the code inside the head tags so that a pageview hit is collected even if the whole webpage doesn't load.

analytics.js code uses a first-party cookie (meaning set by the domain you are on, [link](https://www.quora.com/What-is-the-difference-between-first-and-third-party-cookies)) to track a user across sessions ([link](https://developers.google.com/analytics/devguides/collection/analyticsjs/cookie-usage)). It collects data from the browser and the website and sends that info to Analytics, which counts as a pageview.

You get the referrer info from the HTTP request for the webpage itself. More info [here](https://developers.google.com/analytics/resources/concepts/gaConceptsTrackingOverview#howAnalyticsGetsData)

On mobile devices, differentiating users is easier since when installing the app you get an ID. Reinstalling the app will overwrite this ID. Also it stores data locally and sends it later (they call this "dispatching", wow a new name for everything). Default is 30mins Android, 2mins iOS.

Using the Measurement protocol you can use GA on any device that is not directly supported.

## processing and configuration
steps in processing

 - organize data into users and sessions
 - import data from other systems: adwords, adsense, Google webmaster tools or other
 - modify data using the config
 - aggregate

users and sessions

 - users are defined by the unique ID sent in each hit. Clearing cookies or uninstalling an app resets this. This is specific to a device (I guess specific to a browser too). You can override this ID and therefore associate user interactions across multiple devices. I guess you can do this, eg, when you have a login on your site.
 - session is defined by the session timeout

importing data

 - account linking: link other Google products
 - data import: 
     + join other data to the existing tables (they call it dimension widening). Eg, the URL could be the key. Manual (csv...) or API. This means adding columns, specifically dimensions.
     + cost data import: show costs for non google campaigns (other search engines, mail campaigns). This means adding rows. I wonder why you couldn't use tagged URLs for this.

configuration

 - filtering
 - goal setting
 - grouping
     + channel group (eg, yahoo/bing/google is paid search)
     + content group (eg, grouping product pages together)

# tips for passing the test

## regex
Your company has 15 different IP addresses:
184.275.34.1 184.275.34.2 184.275.34.3 ... 184.275.34.15

`^184\.275\.34\.([1-9]|1[0-5])$`