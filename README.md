# On-Page A/B Testing Guide using Google Tag Manager with dataLayer

## Last update (Sun 22 Sep 2024 11:29 AM)

## 1. Set up Google Tag Manager
- Create a GTM account if you don't have one
- Add the GTM container code to your website

## 2. Plan Your Variants
- Decide which elements on your page you want to test
- Prepare the alternative versions of these elements

## 3. Set up Custom JavaScript Variable
- In GTM, go to Variables > New > Custom JavaScript
- Name it "AB Test Variant"
- Use this code:

```javascript
function() {
  var cookieName = "ab_test_variant";
  var variant = Math.random() < 0.5 ? "A" : "B";
  
  // Check if cookie exists
  if (document.cookie.indexOf(cookieName) === -1) {
    // Set cookie
    document.cookie = cookieName + "=" + variant + "; path=/; max-age=2592000"; // 30 days
  } else {
    // Read existing cookie
    var cookies = document.cookie.split(';');
    for (var i = 0; i < cookies.length; i++) {
      var cookie = cookies[i].trim();
      if (cookie.indexOf(cookieName + "=") === 0) {
        variant = cookie.substring(cookieName.length + 1);
        break;
      }
    }
  }
  
  return variant;
}
```

## 4. Create a Trigger
- Go to Triggers > New > Page View
- Name it "AB Test Pages"
- Set the trigger to fire on the pages you're testing

## 5. Create a Custom HTML Tag
- Go to Tags > New > Custom HTML
- Name it "AB Test Modifications"
- Use this code structure:

```html
<script>
  var variant = {{AB Test Variant}};
  
  // Push variant information to dataLayer
  dataLayer.push({
    'event': 'abTestImpression',
    'abTestName': 'Your Test Name',
    'abTestVariant': variant
  });

  if (variant === "B") {
    // Modify page elements for variant B
    // Example:
    // document.querySelector('#header h1').textContent = 'Welcome to our New Design!';
    // document.querySelector('#cta-button').style.backgroundColor = 'green';

    // You can push additional data specific to variant B if needed
    dataLayer.push({
      'abTestBSpecificData': 'Some B variant specific value'
    });
  }
</script>
```

## 6. Set up Tag Firing
- In the tag settings, choose the "AB Test Pages" trigger

## 7. Create Google Analytics 4 Events
- Go to Tags > New > Google Analytics: GA4 Event
- Name it "AB Test Event"
- Configure the tag:
  - Select your GA4 Configuration Tag
  - Event Name: ab_test_impression
  - Event Parameters:
    - test_name: Your Test Name
    - variant: {{AB Test Variant}}
- Set the trigger to fire on a Custom Event named "abTestImpression"

## 8. Create Custom Dimensions in Google Analytics 4
- In your GA4 property, go to Configure > Custom definitions
- Create two custom dimensions:
  - Name: AB Test Name, Scope: Event, Event parameter: test_name
  - Name: AB Test Variant, Scope: Event, Event parameter: variant

## 9. Set up Conversion Tracking
- Create a new tag for tracking conversions (e.g., form submissions, purchases)
- Include the A/B test information in the conversion event:

```html
<script>
  dataLayer.push({
    'event': 'conversion',
    'conversionType': 'form_submission',
    'abTestName': 'Your Test Name',
    'abTestVariant': {{AB Test Variant}}
  });
</script>
```

## 10. Preview and Publish
- Use GTM's preview mode to test your setup
- Verify that the dataLayer events are firing correctly
- Once confirmed working, publish your container

## 11. Analyze Results
- In Google Analytics 4, create a new exploration
- Use the AB Test Name and AB Test Variant dimensions to segment your data
- Compare metrics like page views, event counts, and conversions between variants

Remember to run your test for a statistically significant period and sample size before drawing conclusions.
