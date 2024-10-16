import requests
import json
from datetime import datetime

# Function to get the access token
def get_access_token():
    url = 'https://account-public-service-prod.ol.epicgames.com/account/api/oauth/token'
    headers = {
        'Content': 'application/x-www-form-urlencoded',
        'User-Agent': 'EOS-SDK/1.16.3020-34326581 (Windows/10.0.19041.3636.64bit) Fortnite/++Fortnite+Release-30.20-CL-34597766',
        'Content-Type': 'application/x-www-form-urlencoded',
        'Authorization': 'Basic ZWM2ODRiOGM2ODdmNDc5ZmFkZWEzY2IyYWQ4M2Y1YzY6ZTFmMzFjMjExZjI4NDEzMTg2MjYyZDM3YTEzZmM4NGQ='
    }
    data = {
        'grant_type': 'client_credentials',
        'deployment_id': '62a9473a2dca46b29ccf17577fcf42d7'
    }
    response = requests.post(url, headers=headers, data=data)
    response.raise_for_status()
    return response.json()['access_token']

# Function to get the JSON data from Fortnite API
def get_json_data(access_token):
    url = 'https://fngw-mcp-gc-livefn.ol.epicgames.com/fortnite/api/game/v2/world/info'
    headers = {
        'User-Agent': 'Fortnite/++Fortnite+Release-30.20-CL-34597766 Windows/10.0.19045.1.256.64bit',
        'Authorization': f'Bearer {access_token}'
    }
    response = requests.get(url, headers=headers)
    response.raise_for_status()
    return response.json()

# Function to update the tileType in the same object as the target string
def update_tile_type(data):
    target_string = '/STW_Zones/World/ZoneThemes/Outposts/BP_ZT_TheOutpost_PvE_01.BP_ZT_TheOutpost_PvE_01_C'
    updated = False

    if isinstance(data, dict):
        for key, value in data.items():
            if isinstance(value, (dict, list)):
                updated = update_tile_type(value) or updated
            if key == "zoneTheme" and value == target_string:
                if "tileType" in data and data["tileType"] == "Outpost":
                    data["tileType"] = "AlwaysActive"
                    updated = True

    elif isinstance(data, list):
        for item in data:
            if isinstance(item, (dict, list)):
                updated = update_tile_type(item) or updated

    return updated

# Function to send the updated JSON file to a Discord webhook
def send_to_discord_webhook(file_path, webhook_url, message):
    with open(file_path, 'rb') as file:
        response = requests.post(
            webhook_url,
            files={'file': (file_path, file)},
            data={'content': message}
        )
        response.raise_for_status()
        print("File sent to Discord webhook successfully.")

# Main function to execute the steps
def main():
    try:
        # Get the access token
        access_token = get_access_token()
        print("Access token obtained successfully.")

        # Get the JSON data from Fortnite API
        data = get_json_data(access_token)
        print("JSON data retrieved successfully.")

        # Update the tileType in the JSON data
        updated = update_tile_type(data)

        if updated:
            # Get today's date in dd-mm-yyyy format
            today_date = datetime.now().strftime('%d-%m-%Y')
            file_path = f'{today_date}.json'
            message = f'{today_date}.'

            # Save the updated JSON data to a file with ensure_ascii=False to maintain Unicode characters
            with open(file_path, 'w', encoding='utf-8') as file:
                json.dump(data, file, indent=4, ensure_ascii=False)
            print("The JSON data has been updated and saved successfully.")

            # Send the updated file to the Discord webhook
            webhook_url = 'WH_URL_HERE'
            send_to_discord_webhook(file_path, webhook_url, message)
        else:
            print("Target string not found in the JSON data or no update required.")

    except requests.HTTPError as http_err:
        print(f"HTTP error occurred: {http_err}")
        print(f"Response: {http_err.response.content}")
    except Exception as err:
        print(f"An error occurred: {err}")

# Run the main function
if __name__ == '__main__':
    main()
