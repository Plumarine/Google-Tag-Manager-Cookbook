Google Tag Manager Cookbook
===========================

Learning Devtools, JavaScript & jQuery
----------------------------

Google Tag Manager can be extremely useful even if you do not know any javascript but I recommend you go through the following as it will give you the skills to get maximum use out of Google Tag Manager. 

#### Chrome Dev Tools

- Network tab - https://developers.google.com/chrome-developer-tools/docs/network
- Console tab - https://developers.google.com/chrome-developer-tools/docs/console
- Tips & Tricks - https://developers.google.com/chrome-developer-tools/docs/tips-and-tricks
- http://code.tutsplus.com/articles/chrome-dev-tools-networking-and-the-console--net-28167

#### Firefox Dev Tools

- http://code.tutsplus.com/tutorials/debugging-with-the-firefox-devtools--net-36999

#### JavaScript

- http://jsforcats.com/
- https://github.com/airbnb/javascript
- http://jqfundamentals.com/chapter/javascript-basics

#### jQuery

- http://jqfundamentals.com/chapter/jquery-basics


StackOverflow & GTM G+
----------------------

If you ever need help with javascript, jQuery or Google Tag Manager then the best resources are

- stackoverflow.com
- GTM Google+ Community - https://plus.google.com/communities/104865292981489764063

Naming Best Practise
--------------------

### Tags

Begin with|Tag Vendor|Tag Description|Pages
----------|----------|---------------|-----
Tag       | UA       | Page View     | All pages
Tag       | UA       | Event         | All pages
Auto-event module| GTM | Click Listener | All pages
jQuery-event | jQuery | onclick download button | product page


Event Tracking
--------------

There are 2 methods for event tracking in Google Tag Manager.

1. Auto-Event Tracking
2. Using jQuery

There are some known limitations with auto-event tracking and they include the following

- Other non GTM event code blocking auto-event tracking
- Cannot detect right click or cmd+click (mac) or ctrl+click(windows)
- If you create an event that passes in a lot of custom dimensions you will need to add these over and over again
- No feature for non onclick events. eg. selected value from dropdown
- No ability to detect errors including if an element has been removed due to website changes. (solved with option 2 using jQuery)
- If the link contains `src="onclick=function(){}"` auto-event tracking will not work (http://www.conversionworks.co.uk/blog/2013/10/08/google-tag-manager-auto-event-tracking-deeper-dive/)
- If you have elements within elements that can be clicked on GTM will use the child element which may result in the need for multiple rules to select all the child elements. eg see code below.

```html
<li>
  <a href="/family-center/family-filter?category=cards">
    <h3>Cards</h3>
    <div class="img-bg">
    </div>
    <div class="img-holder">
      <img src="/~/media/creative-center/images/category-images/family-center/cards.ashx" alt="Cards" width="271" height="229">
    </div>
  </a>
</li>
```



For these reasons if you feel you will need those I recommend option 2 using jQuery.

Option 1: Auto-Event Tracking
---------------------------

Option 2: Event Tracking Using jQuery
---------------------------

### Event Tracking with built in test

The following code is an example of how you can bind an event to an element but it also includes a test first to see if that element exists on the page. This is a very useful way to easily detect if any of your events stop working due to website changes you may be unaware of.

```html
<script>
var elementSelector = '#btnStoreSearc';

if( $(elementSelector).length === 0 ){

  console.log( 'missing' );
  
  dataLayer.push({
    'event' : 'event tracking error',
    'eventCategory' : 'event tracking error',
    'eventLabel' : elementSelector,
    'eventAction' : 'onclick element missing',
    'eventNonInteraction' : true,
  })

} else {

  $(elementSelector).on('click',function(){
  
    console.log( 'found' );
    console.log( $( this ) );
  
    dataLayer.push({
      'event' : 'event tracking',
      'eventCategory' : 'event category',
      'eventLabel' : 'event label',
      'eventAction' : 'event action',
      'eventNonInteraction' : true,
    })
  
  })

}
</script>
```

### Event Tracking delay default behaviour

You may need to delay the behaviour of the event such as form submit so there is enough time for the tag to execute.

In the example below you can prevent the form from submitting when the button is clicked for 1 second.

```html
<script>
$( '#cccontent_0_pagecontent_0_PersonaliseButton' ).on( "click", function( event ) {
  
  // prevent the form from submitting
  event.preventDefault();
  
  dataLayer.push({
      'event' : 'event tracking',
      'eventCategory' : 'event category',
      'eventLabel' : 'event label',
      'eventAction' : 'event action',
      'eventNonInteraction' : true,
    })
  
  // submit the form after 1 second (1000 milliseconds)
  setTimeout(function(){
  $('form').submit();
  },1000)
});
</script>
```

### Event tracking and capturing child element information

```html
<script>
$('.categories-container li').on('click', function(event) {

event.preventDefault();

console.log('click');
console.dir( $(this)[0] );
// select element within this
// code via http://stackoverflow.com/questions/306583/this-selector-and-children?lq=1
console.log( $('h3',this)[0] );
console.log( $('h3',this)[0].innerText );
console.dir( event.target );
console.log( event.type );

})
</script>
```

Advanced
--------

### Page Type Path Tracking

```js
// include indexOf polyfill
// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/indexOf

var pageTypePathArr = readCookie('pageTypePath').split(' > ');

var pageTypeArr = pageTypePathArr;
if (pageTypeArr.indexOf(dataLayer[0].pageType < 0)){
pageTypeArr.push(dataLayer[0].pageType);
}
var pageTypePath = pageTypeArr.sort().join(' > ')

document.cookie = 'pageTypePath='+pageTypePath
//"pageTypePath=landing > product"

// pass in as custom dimension session level

var pageTypePathArr = readCookie('pageTypePath').split(' > ');

createCookie('pageTypeArr','blah > blah > blah', 180, '.brother.eu')

function createCookie(name,value,days,domain) {
  if (days) {
    var date = new Date();
    date.setTime(date.getTime()+(days*24*60*60*1000));
    var expires = "; expires="+date.toGMTString();
  }
  else var expires = "";
  document.cookie = name+"="+value+expires+"; path=/; domain="+ domain;
}

function readCookie(name) {
  var nameEQ = name + "=";
  var ca = document.cookie.split(';');
  for(var i=0;i < ca.length;i++) {
    var c = ca[i];
    while (c.charAt(0)==' ') c = c.substring(1,c.length);
    if (c.indexOf(nameEQ) == 0) return c.substring(nameEQ.length,c.length);
  }
  return null;
}

function eraseCookie(name,domain) {
  createCookie(name,"",-1,domain);
}
```

Detect User Page Navigation Type Browsing Behaviour
-------------------------

The following code detects if a user clicked through to a URL or if they pressed the back button or came through an external link. This is used for promo banner tracking. It cannot detect forward clicks

```js
function(){
window.pageNavigationFuncRun = window.pageNavigationFuncRun || 0;
window.pageNavigationType = window.pageNavigationType || '';

if (window.pageNavigationFuncRun === 1){

  //console.log('pageNavigationFuncRun already completed');
  //console.log(window.pageNavigationType)
  return window.pageNavigationType;

} else {

  var previousPageCookieValue = readCookie('currentPage')

  if (previousPageCookieValue){

    createCookie('previousPage', previousPageCookieValue, 180, '.brother.eu')

  }

  createCookie('currentPage', window.location.href, 180, '.brother.eu')

  if (parseURLhostname(document.referrer) !== location.host){

    //console.log('external click through')
    pageNavigationType = 'external click through';

  } else if (previousPageCookieValue === document.referrer){

    //console.log('internal click through');
    pageNavigationType = 'internal click through';

  } else if (previousPageCookieValue === readCookie('currentPage')){

    //console.log('page refresh');
    pageNavigationType = 'page refresh';

  } else {

    //console.log('back button');
    pageNavigationType = 'back button';

  }

// declare page navigation value has run
window.pageNavigationFuncRun = 1;
//console.log(window.pageNavigationType)
return window.pageNavigationType;

}

function createCookie(name,value,days,domain) {
  if (days) {
    var date = new Date();
    date.setTime(date.getTime()+(days*24*60*60*1000));
    var expires = "; expires="+date.toGMTString();
  }
  else var expires = "";
  document.cookie = name+"="+value+expires+"; path=/; domain="+ domain;
}

function readCookie(name) {
  var nameEQ = name + "=";
  var ca = document.cookie.split(';');
  for(var i=0;i < ca.length;i++) {
    var c = ca[i];
    while (c.charAt(0)==' ') c = c.substring(1,c.length);
    if (c.indexOf(nameEQ) == 0) return c.substring(nameEQ.length,c.length);
  }
  return null;
}

function eraseCookie(name,domain) {
  createCookie(name,"",-1,domain);
}

function parseURLhostname(url) {
    var a=document.createElement('a');
    a.href=url;
    return a.hostname;
}
}
```

Detect Right or CMD clicks
--------------------------

GTM will track onclick events but what about CMD clicks?

What about right click open in new tab?  

What about right click open in new window? 

```js
$('a').on('click', function(e) {
  if(e.metaKey && e.shiftKey) {
    console.log(".a - Command+Shift+Click", e);
  } else if (e.metaKey) {
    console.log(".a - Command+Click", e);
  } else {
    console.log(".a - Other click", e);
  }
});
```

### Possible Solutions:


#### A. Delay click by 1 second and use event callback.
**CONS:**
- Will be a delay before a user is sent to the next page affecting the user experience.


#### B. Pass data on the next page.  
**CONS:**  
- You will lose all the dataLayer information about the previous page which you may want to pass as custom dimensions.
- URLs may start to look messy.

1. Pass data in the URL as #hashtag and send click on next page  

2. Save data in a cookie on mousedown and then send event data on next page  

YouTube Video Tracking
----------------------

http://www.cardinalpath.com/youtube-video-tracking-with-gtm-and-ua-a-step-by-step-guide/
