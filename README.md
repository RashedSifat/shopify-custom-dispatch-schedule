# shopify-custom-dispatch-schedule
Shopify product page custom order dispatch time 

This code contains the javascript and liquid code to create a customizable order dispatch day showcasing on the product page.
From theme customizer user can select the available weekdays and the upcomming custom weekends/holidays. there is a cutoff time if the dispatch will be sent in between the day.
For example, if the cutoff hour is 12 PM, the orders which are made before 12 PM will be dispatched today, otherwise it'll be dispatched in the follwoing day based on the week, weekend and holidays.

The following javascript code is needed to add to the product liquid page at the very above of the file.

````
<script>
  document.addEventListener("DOMContentLoaded", function() {
    function updateDeliveryDate() {
      // Office weekdays and holidays configuration from settings
      var officeDays = {
        "monday": {{ section.settings.monday | json }},
        "tuesday": {{ section.settings.tuesday | json }},
        "wednesday": {{ section.settings.wednesday | json }},
        "thursday": {{ section.settings.thursday | json }},
        "friday": {{ section.settings.friday | json }},
        "saturday": {{ section.settings.saturday | json }},
        "sunday": {{ section.settings.sunday | json }}
      };
      
      // Fetching and parsing holidays setting
      var holidaysRaw = "{{ section.settings.holidays | escape }}";
      var holidays = holidaysRaw ? holidaysRaw.split(',').map(function(date) {
        return date.trim();
      }) : [];

      // Function to check if a date is a holiday
      function isHoliday(date) {
        var day = ("0" + date.getDate()).slice(-2);
        var month = ("0" + (date.getMonth() + 1)).slice(-2);
        var year = date.getFullYear();
        var dateString = day + "/" + month + "/" + year;
        return holidays.indexOf(dateString) !== -1;
      }

      // Function to get the next available date
      function getNextAvailableDate(currentDate) {
        var nextDate = new Date(currentDate);
        while (true) {
          nextDate.setDate(nextDate.getDate() + 1);
          var dayName = nextDate.toLocaleDateString('en-GB', { weekday: 'long' }).toLowerCase();
          if (officeDays[dayName] && !isHoliday(nextDate)) {
            return nextDate;
          }
        }
      }

      // Function to calculate the delivery date
      function calculateDeliveryDate() {
        var currentDateTime = new Date();
        var orderCutoffHour = {{ section.settings.hour_estimated_offset }};
        console.log({{ section.settings.hour_estimated_offset }});
        var deliveryDate;
        var todayStatus = false; // Initialize todayStatus outside to ensure it's accessible to other parts of the function

        var dayName = currentDateTime.toLocaleDateString('en-GB', { weekday: 'long' }).toLowerCase();

        if (dayName in officeDays && officeDays[dayName]) {
          todayStatus = true;
        }

        // Check if current time is before the cutoff and today is a working day and not a holiday
        if (currentDateTime.getHours() < orderCutoffHour && todayStatus && !isHoliday(currentDateTime)) {
          return "TODAY";
        } else {
          deliveryDate = getNextAvailableDate(currentDateTime);
        }

        dayName = deliveryDate.toLocaleDateString('en-GB', { weekday: 'long' }).toLowerCase();

        // Ensure the delivery date is an office day and not a holiday
        while (!officeDays[dayName] || isHoliday(deliveryDate)) {
          deliveryDate = getNextAvailableDate(deliveryDate);
          dayName = deliveryDate.toLocaleDateString('en-GB', { weekday: 'long' }).toLowerCase();
        }

        return deliveryDate.toLocaleDateString('en-GB');
      }

      // Get the calculated delivery date message
      var deliveryMessage = calculateDeliveryDate();

      // Display the delivery date message
      var deliveryDateElement = document.getElementById('delivery-date');
      if (deliveryDateElement) {
        deliveryDateElement.innerText = deliveryMessage;
      }
    }

    // Initial call to update the delivery date
    setInterval(updateDeliveryDate, 1000);
      
  });
</script>
````

The variables for example {{ section.settings.monday }} is captured from the schema. The added schema for the customizer input is as followed in the same product page:

````
    {
      "type": "header",
      "content": "Working days"
    },
    {
      "type": "checkbox",
      "id": "sunday",
      "default": false,
      "label": "Sunday"
    },
    {
      "type": "checkbox",
      "id": "monday",
      "default": true,
      "label": "Monday"
    },
    {
      "type": "checkbox",
      "id": "tuesday",
      "default": true,
      "label": "Tuesday"
    },
    {
      "type": "checkbox",
      "id": "wednesday",
      "default": true,
      "label": "Wednesday"
    },
    {
      "type": "checkbox",
      "id": "thursday",
      "default": true,
      "label": "Thursday"
    },
    {
      "type": "checkbox",
      "id": "friday",
      "default": true,
      "label": "Friday"
    },
    {
      "type": "checkbox",
      "id": "saturday",
      "default": false,
      "label": "Saturday"
    },
    {
      "type": "textarea",
      "id": "holidays",
      "label": "Holidays (comma-separated, in DD/MM/YYYY format)"
    },
    {
      "type": "range",
      "id": "hour_estimated_offset",
      "label": "Delivery hour offset (24-hour based format)",
      "min":        1,
      "max":        24,
      "step":       1,
      "default":    14
    }
````

Now, the message about the dispatch time will be published in the product page as:

````
<p>WILL BE SENT <span id="delivery-date"></span></p>
````


If you still faceing any issues to use the codes, feel free to contact me at: <a href="mailto:me.sifat@live.com">me.sifat@live.com</a>
Or, Directly hire me: <a href="https://www.upwork.com/o/profiles/users/~01c3507b22db0d551a/">Hire on Upwork</a>

Happy Coding!
