# MEDOLINE
MEDOLINE: A real-time platform connecting patients, doctors, and pharmacies for seamless prescriptions.
from flask import Flask, render_template
import requests
from geopy.geocoders import Nominatim  # Ensure geopy is installed
streamlit==1.40.2
geopy==2.3.0
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/pratiksha259311/MEDOLINE.git
git push -u origin main
app = Flask(__name__)

# Function to get latitude and longitude from an address
def get_location(address):
    geolocator = Nominatim(user_agent="medoline_app")  # Use Nominatim correctly
    location = geolocator.geocode(address)
    if location:
        return location.latitude, location.longitude
    else:
        return None, None

# Function to find nearby pharmacies using Google Places API
def find_nearby_pharmacies(lat, lon, api_key, medication_name):
    google_api_url = f"https://maps.googleapis.com/maps/api/place/nearbysearch/json?location={lat},{lon}&radius=5000&type=pharmacy&keyword={medication_name}&key={api_key}"
    response = requests.get(google_api_url)
    if response.status_code == 200:
        places_data = response.json()
        return places_data['results']
    else:
        return None

# Mock function to simulate checking medication availability at pharmacies
def medication_in_stock(pharmacy_name):
    if pharmacy_name == "Shop A":  # Assume Shop A has Olsertan 40
        return True
    return False

# Function to check medication availability at pharmacies
def check_medication_availability(pharmacies):
    available_pharmacies = []
    for pharmacy in pharmacies:
        if medication_in_stock(pharmacy['name']):
            available_pharmacies.append(pharmacy)
    return available_pharmacies

# Route to display the patient's prescription and available pharmacies
@app.route('/prescription')
def show_prescription():
    address = "123 Patient St, City, Country"  # Use the patient's address here
    lat, lon = get_location(address)
    if lat and lon:
        api_key = "YOUR_GOOGLE_API_KEY"  # Replace with your actual Google API key
        medication_name = "Olsertan 40"  # Ensure it's in quotes, as a string
        pharmacies = find_nearby_pharmacies(lat, lon, api_key, medication_name)
        if pharmacies:
            available_pharmacies = check_medication_availability(pharmacies)
            return render_template('prescription.html', available_pharmacies=available_pharmacies)
        else:
            return "No pharmacies found nearby."
    else:
        return "Location not found."

if __name__ == '__main__':
    app.run(debug=True)

