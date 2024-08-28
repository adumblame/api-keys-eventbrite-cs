# api-keys-eventbrite-cs
to pull eventbrite data for yre


function fetchEventbriteAttendeesForMultipleEvents() {
  var oauthToken = 'OAuth'; // Replace with your OAuth token do not lose this
  var eventIds = [
    'EventID', // Replace with your specific Eventbrite event IDs to add more events add (event_id_n+1) (event_id_8) and so on
    'EventID',
    'EventID',
    'EventID',
    'EventID',
    'EventID',
  ];
  
  var spreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  
  // Loop through each event ID
  for (var j = 0; j < eventIds.length; j++) {
    var eventId = eventIds[j];
    var url = 'https://www.eventbriteapi.com/v3/events/' + eventId + '/attendees/';
    
    var params = {
      'method': 'GET',
      'headers': {
        'Authorization': 'Bearer ' + oauthToken
      },
      'muteHttpExceptions': true
    };
    
    var response = UrlFetchApp.fetch(url, params);
    var result = JSON.parse(response.getContentText());

    if (response.getResponseCode() != 200) {
      Logger.log('Error for event ' + eventId + ': ' + result.error_description);
      continue;
    }
    
    var attendees = result.attendees;
    if (attendees.length === 0) {
      Logger.log('No attendees found for event ' + eventId);
      continue;
    }
    
    // Create or select a sheet for the event
    var sheetName = 'YRE Event' + (j + 1);
    var sheet = spreadsheet.getSheetByName(sheetName);
    if (sheet == null) {
      sheet = spreadsheet.insertSheet(sheetName);
    } else {
      sheet.clear(); // Clear any existing content
    }
    
    // Set up headers
    sheet.getRange(1, 1).setValue('First Name');
    sheet.getRange(1, 2).setValue('Last Name');
    sheet.getRange(1, 3).setValue('Email Address');
    sheet.getRange(1, 4).setValue('Phone Number');
    
    // Insert attendees into the sheet
    for (var i = 0; i < attendees.length; i++) {
      var attendee = attendees[i];
      
      sheet.getRange(i + 2, 1).setValue(attendee.profile.first_name);
      sheet.getRange(i + 2, 2).setValue(attendee.profile.last_name);
      sheet.getRange(i + 2, 3).setValue(attendee.profile.email);
      sheet.getRange(i + 2, 4).setValue(attendee.profile.cell_phone || 'N/A');
    }
    
    Logger.log('Fetched ' + attendees.length + ' attendees for event ' + eventId);
  }
}

