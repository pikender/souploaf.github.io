---
layout: post
title: Integrating Omniture Analytics
---

- Create Report Suite
  - Admin -> Report Suites -> Create New
- Add Traffic Variables
  - Admin -> Report Suites or Traffic Management -> Edit Settings -> Traffic -> Traffic Variables -> Add New
  - Note the serial number for variables
  - From App, you will send prop1, prop2 etc and its meaning will make sense here once you see mapping
- Add Custom Events
  - Admin -> Report Suites or Traffic Management -> Edit Settings -> Conversion -> Success Events -> Add New
  - It gives you event vs name mapping
  - From App, you will send event and its meaning will make sense here once you see mapping
- Get AppMeasurement.js and VisitorAPI.js
  - Admin -> Code Manager -> Javascript (new)
- Update VisitorAPI.js

```javascript
var visitor = new Visitor("123@AdobeOrg");
visitor.trackingServer = "metrics.piks.com"; // same as s.trackingServer
visitor.trackingServerSecure = "securemetric.piks.com"; //same as s.trackingServerSecure
```
- Update AppMeasurement.js

```javascript
//initialize AppMeasurement
var s=s_gi(s_account)
/******** VISITOR ID SERVICE CONFIG - REQUIRES VisitorAPI.js ********/
s.visitor=Visitor.getInstance("123@AdobeOrg")
/************************** CONFIG SECTION **************************/
/* You may add or alter any code config here. */
/* Link Tracking Config */
s.trackDownloadLinks=true
s.tracksternalLinks=true
s.trackInlineStats=true
s.linkDownloadFileTypes="exe,zip,wav,mp3,mov,mpg,avi,wmv,pdf,doc,docx,xls,xlsx,ppt,pptx"
s.linkInternalFilters="javascript:" //optional: add your internal domain
here
s.linkLeaveQueryString=false
s.linkTrackVars="None"
s.linkTrackEvents="None"

/* uncomment below to use doPlugins */
 s.usePlugins=true
 /*function s_doPlugins(s) {
 //use implementation plug-ins that are defined below
 // in this section. For example, if you copied the append
 // list plug-in code below, you could call:
 // s.events=s.apl(s.events,"event1",",",1);
 s.campaign=s.getQueryParam('source');

 }
 source.doPlugins=s_doPlugins */

 /* WARNING: Changing any of the below variables will cause drastic
 changes to how your visitor data is collected.  Changes should only be
 made when instructed to do so by your account manager.*/
 s.trackingServer="metrics.piks.com"
 s.trackingServerSecure="securemetrics.piks.com"
```
- http://stevenbenner.com/2010/03/custom-link-click-tracking-using-omniture/
- Create omniture\_events.js

```javascript
const Omniture = {
  getBaseTrackData: function(data) {
    var _this = this;
    return {
      events: undefined,
      variables: {
        prop1: window.location.href,
        prop2: window.location.host,
        prop3: data.locale
				// other props
      },
      eventName: 'ChangeIt'
    }
  },
  trackPageView: function(data) {
    var s = s_gi(s_account);
    s.pageUrl = window.location.origin + window.location.pathname;
    for(var k in data) {
      s[k] = data[k];
    }
    s.t();
  },
  // Wrapper function for native Omniture SiteCatalyst link tracking (s.tl())
  trackLink: function (referenceObject, trackingData, targetReportSuite, linkType = 'o') {
    if (typeof referenceObject === 'undefined' || typeof trackingData === 'undefined') {
      console.log("Early Exit");
      // If you're missing your settings object or any variables/events to fire, you fail
      return;
    }
    var linkTrackVarsArray = [],
      // Get values out of the configuration object
      tEvents = trackingData.events, // String || undefined
      tVars = trackingData.variables, // Object || undefined
      tEventName = trackingData.eventName, // String || undefined
      // Allow specification of a target reporting suite, to allow testing
      // of new events in Dev prior to rollout in Production
      reportSuite = targetReportSuite || s_account,
      s = s_gi(reportSuite);

    s.linkTrackEvents = 'None';
    s.linkTrackVars = 'None';

    if (tEvents && typeof tEvents === 'string') {
      linkTrackVarsArray.push('events');
      s.linkTrackVars = linkTrackVarsArray.join(',');
      s.linkTrackEvents = tEvents;
      s.events = tEvents;
    }
    if (tVars && typeof tVars === 'object') {
      for (var key in tVars) {
        if (tVars.hasOwnProperty(key)) {
          // tVars should be in format { eVar1: 'value', prop5: 'value' }
          linkTrackVarsArray.push(key);
          s.linkTrackVars = linkTrackVarsArray.join(',');
          s[key] = tVars[key]; // Assign the value
        }
      }
    }
    // Fire event
    s.tl(referenceObject, linkType, tEventName);

    // Resetting them here ensures that next event starts clean
    // For e.g. Exit links might pick last event track data
    s.linkTrackEvents = 'None';
    s.linkTrackVars = 'None';
  }
}

export default Omniture;

```
- Use it where tracking needed

```javascript
import Omniture from '../lib/omniture_events';

Omniture.trackChangeLocale(getState());
```

## Verify in Omniture

- Reports -> Site Metrics -> Custom Events -> Custom Events 1-20 -> Event Name (Change Locale)
- Reports -> Site Metrics -> Page Views
- Reports -> Custom Traffic -> Custom Traffic 1-20 -> Var Name (Locale)
