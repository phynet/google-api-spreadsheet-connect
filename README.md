# How to use Google API to connect with Google Sheets
Imagine that you want to retrieve information from a goole sheet, in order to process any work in a terminal, CI, etc. The job is easy, but at the same time Google docs are messy. I've followed these steps. As always for future me, which will forget how to do it, just read this.

## Go to Google Cloud Platform
1.- https://console.cloud.google.com/

2.- **Create a google cloud project**

Read this informaiton: https://developers.google.com/workspace/guides/create-project

**Important** ⚠️

You need to be in a organization. I haven't tried doing it witout it (since I need it for work) 

<img width="455" alt="image" src="https://user-images.githubusercontent.com/724536/167264201-24378f8a-dcff-4a65-b72e-06d2d416200a.png">

3.- **Enable Google Workspace APIs picking **Library**

https://developers.google.com/workspace/guides/enable-apis

<img width="432" alt="image" src="https://user-images.githubusercontent.com/724536/167264448-17083425-a0ff-4f3f-8090-794d121afcb1.png">

3.1.- **Select Google Sheet and Enable it**

<img width="1001" alt="image" src="https://user-images.githubusercontent.com/724536/167264473-608e2c60-69c1-42b4-9c1a-12b36df16fc8.png">

4.- **Create a Service Account**

https://developers.google.com/workspace/guides/create-credentials#create_a_service_account

5.- **Create the credentials for the service account:**

This will give you a set of credentials that you will later use to connect Sheets in your script. Download it and name it. Keep it close since you will add it to the script next. 

https://developers.google.com/workspace/guides/create-credentials#create_credentials_for_a_service_account

6.- **Pick the language of your script.**

I choosed Ruby, but there are several.

7.- **Since I picked Ruby I needed to install some gems:**

     sudo gem install google-cloud-storage
     sudo gem install googleauth
     
 I'd also review the libraries that I needed here: https://cloud.google.com/storage/docs/reference/libraries
 
 8.- **Persist key path**
 
 For this test I needed to setup the App credentials path so it was persisted in my current session
 
    export GOOGLE_APPLICATION_CREDENTIALS="KEY_PATH"
    
Check more information about this here: https://cloud.google.com/storage/docs/reference/libraries#setting_up_authentication

9.- **Use the script that appears on Google's Doc.**

BUT!! I was trying to use the code that appears on this page: https://developers.google.com/sheets/api/quickstart/ruby  and I was always having troubles with the connection.

      Expected top level property 'installed' or 'web' to be present
      
This was the first stone in the way I've found out it was a thing of authorization

https://stackoverflow.com/questions/57262961/missing-top-level-element-error-in-google-apis-in-ruby

So at the end I used a script that I found in stackoverflow (of course) it was almost the same, except for the authorize function. 

      require "google/apis/sheets_v4"
      require "googleauth"

      #CREDENTIALS_PATH = "google_service_account_credentials_path/google_service_account_credentials.json"
      CREDENTIALS_PATH = "credentials.json"
      SCOPE = Google::Apis::SheetsV4::AUTH_SPREADSHEETS_READONLY

      # Author & Authen with Google by Google Service Account Credentials
        authorizer = Google::Auth::ServiceAccountCredentials.make_creds(
        json_key_io: File.open(CREDENTIALS_PATH),
        scope: SCOPE)

        authorizer.fetch_access_token!

      # Initialize the API
        service = Google::Apis::SheetsV4::SheetsService.new
        service.authorization = authorizer

      # Some demo code to prove it run
      # Prints the names and majors of students in a sample spreadsheet:
      # https://docs.google.com/spreadsheets/d/1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms/edit
      spreadsheet_id = "1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms"
      range = "Class Data!A2:E"
      response = service.get_spreadsheet_values spreadsheet_id, range
      puts "Name, Major:"
      puts "No data found." if response.values.empty?
      response.values.each do |row|
        # Print columns A and E, which correspond to indices 0 and 4.
        puts "#{row[0]}, #{row[4]}"
      end

In here the original post: https://stackoverflow.com/questions/71677531/erro-to-connect-google-sheets-api


10.- **And last but not least! setup the permission inside Google Sheet.**

Search in your `credentials.json` the `client_email` which you will need to share with as if it was any other person accessing your sheet.

I've found this information in Github https://github.com/googleapis/google-api-ruby-client/issues/461#issuecomment-243328234

If you run this script (which uses the sheet I've pasted bellow) you will see something like:

<img width="111" alt="image" src="https://user-images.githubusercontent.com/724536/167265291-ebe75efe-8fd5-4530-a4f7-dcdb1bfe9501.png">


## Some Test Sheet
https://docs.google.com/spreadsheets/d/1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms/edit


# Integrate it to Bitrise

What I wanted to do was to select a single value from the sheet and set this value as an ENV in bitrise. The value was the nick of someone in slack and the idea was that for a release workflow I should pick the owner of the release through the sheet and send it as a message to slack.

Something like: "The release has started and the owner is 'sofia.swidarowicz' "

## How?

It's a simple step, although I needed a couple of hours to figure out the correct way of apps to use.

1.- Go to your workflow and use `Generic File Storage`, add `Ruby Script` and `Slack message`

<img width="371" alt="image" src="https://user-images.githubusercontent.com/724536/172587797-a869207c-46e3-4823-ad23-aadca372b496.png">

2.- You will need to upload the `credentials.json` file that I mentioned before and upload it to the Code Signing tab. Add it as a GENERIC FILE STORAGE.

<img width="336" alt="image" src="https://user-images.githubusercontent.com/724536/172589084-c1f493fb-6868-471e-9829-d6b7fc60deee.png">

3.- Modify the Ruby Script. You will need to import several gems

``` 
gem 'googleauth'
gem 'google-cloud-storage' (this is optional)
gem 'google-apis-sheets_v4'
```

This is the Ruby script, I've modified some value for privacy:

Note: read here how to refer the ENV for the generic file storage: https://www.bitrise.io/integrations/steps/generic-file-storage 


```
require "google/apis/sheets_v4"
require "googleauth"

puts "Starting..."
puts "$GENERIC_FILE_STORAGE #{ENV['GENERIC_FILE_STORAGE']}"

CREDENTIALS_PATH = "#{ENV['GENERIC_FILE_STORAGE']}/credentials.json"


SCOPE = ::Google::Apis::SheetsV4::AUTH_SPREADSHEETS_READONLY

# Author & Authen with Google by Google Service Account Credentials
authorizer = Google::Auth::ServiceAccountCredentials.make_creds(
  json_key_io: File.open(CREDENTIALS_PATH),
  scope: SCOPE)

authorizer.fetch_access_token!

# Initialize the API
service = Google::Apis::SheetsV4::SheetsService.new
service.authorization = authorizer

# Example Spread sheet
# https://docs.google.com/spreadsheets/d/1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms/edit
spreadsheet_id = "1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgvE2upms"

range = "F11"
response = service.get_spreadsheet_values spreadsheet_id, range
puts "Next Release Master Facilitator:"
puts "No data found." if response.values.empty?
response.values.each do |row|
  # Print column which correspond to indices 0
  RELEASE_MASTER = row[0]
  puts "Next Release Master: #{RELEASE_MASTER}"

  system( "envman add --key RELEASE_MASTER_NAME --value '@#{RELEASE_MASTER}'" )

end
```

- **RELEASE_MASTER_NAME** is the ENV VAR that is used in the flow in the `Env Var section`
- In order to override the ENVAR with a new value, we need to use envman https://github.com/bitrise-io/envman/ using this code:

     `system( "envman add --key RELEASE_MASTER_NAME --value '@#{RELEASE_MASTER}'" )`

4.- Use the ENV VAR an add it to the message of slack 

     `My message with the name of the owner: $RELEASE_MASTER_NAME`


And that's it :D

