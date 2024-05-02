# ProSmartSystem

In this repository I will show you how to use the API of all ProSmartSystem (Computherm) like BBoil, PS Gate, PS Fire, PS Alarm, PS Water, PS Plug

First I recommend to try it first with logged in at https://api.prosmartsystem.com/ site.
Log in with /api/auth/login to get your token then log in with your code with the Authorize button.

For doing it with bash script, we need to get the token to control the device. Remember for that, the token will be only valid for 3600 seconds.

```
#!/bin/bash

# Make the curl request and save the response
response=$(curl -X 'POST' \
  'https://api.prosmartsystem.com/api/auth/login' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
  "email": "EMAIL_ADDRESS",
  "password": "YOUR_PASSWORD"
}')

# Extract the access token using jq
access_token=$(echo $response | jq -r '.access_token')

# Save the access token to token.txt file
echo "$access_token" > token.txt
```

Then, receive what devices belongs to your profile with /api/devices GET request.

In my case, I have a BBoil thermostat, and I wished to turn it to Scheduled program and turn it off via command line, you can reach it with this:

```
access_token=$(cat token.txt)
curl -X 'POST' \
  'https://api.prosmartsystem.com/api/devices/YOURDEVICEID/cmd' \
  -H 'accept: */*' \
  -H "Authorization: Bearer $access_token" \
  -H 'Content-Type: application/json' \
  -d '    {
       "mode": "SCHEDULE" #OR IT TURNED OFF WITH "OFF" COMMAND
    }'

```

This command is needed for me to forecast the today weather, and turn it the thermostat off, if the weather will be good enough to not turn it on for no reason.

```
#!/bin/bash

# OpenWeatherMap API key
API_KEY="YOUR_API_KEY" # Register it free at https://home.openweathermap.org/users/sign_up

# City name and country code
CITY="YOUR_CITY"

# Get today's date in YYYY-MM-DD format
TODAY=$(date +"%Y-%m-%d")

# Fetch weather data from OpenWeatherMap API
weather_data=$(curl -s "http://api.openweathermap.org/data/2.5/forecast?q=$CITY&appid=$API_KEY")

# Extract maximum temperature for today
max_temp=$(echo "$weather_data" | jq --arg TODAY "$TODAY" '.list[] | select(.dt_txt | startswith($TODAY)) | .main.temp_max' | sort -nr | head -n 1)

# Convert temperature from Kelvin to Celsius
max_temp_celsius=$(echo "scale=2; $max_temp - 273.15" | bc)

# Check if current temperature is greater than 20 degrees Celsius
if (( $(echo "$max_temp_celsius > 20" | bc -l) )); then
bash stop.sh # YOUR SCRIPT PATH
else
bash start.sh # YOUR SCRIPT PATH
fi
```

Your can do it other stup whatever you want learning to request the datas from API, those are short examples only in my use-case.
