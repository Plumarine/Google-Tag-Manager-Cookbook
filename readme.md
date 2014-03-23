Google Tag Manager Cookbook
===========================

Event Tracking
--------------

There are 2 methods for event tracking in Google Tag Manager.

1. Auto-Event Tracking
2. Using jQuery

There are some known limitations with auto-event tracking and they include the following

- Other non GTM event code blocking auto-event tracking
- No feature for non onclick events. eg. selected value from dropdown
- No ability to detect errors including if an element has been removed due to website changes.

For these reasons I recommend option 2 using jQuery.

Event Tracking Using jQuery
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