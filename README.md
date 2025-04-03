# lab10

Aashika Uppala
4/3/2025
Part 1
2. # Assistant
import pandas as pd
import os

def get_unique_sites(file_path):
    # Load the CSV file into a DataFrame
    df = pd.read_csv(file_path)
    
    # Filter for the names of water quality measurement sites and location information
    sites = df[['MonitoringLocationName', 'LatitudeMeasure', 'LongitudeMeasure']].drop_duplicates()
    
    return sites

# Path to the CSV file
# Make sure to provide the correct path to your file
file_path = 'station.csv'  # You may need to provide the full path

# Check if the file exists before trying to read it
if not os.path.exists(file_path):
    print(f"File not found: {file_path}")
    print(f"Current working directory: {os.getcwd()}")
    # You might need to adjust the path or upload the file
else:
    # Get unique sites and their location information
    unique_sites = get_unique_sites(file_path)
    
    # Display the unique sites
    print(unique_sites)
    3. file:///C:/Users/Aashika/Downloads/site_map.html
    
  Part 2
  3. 
  import pandas as pd
import matplotlib.pyplot as plt

def plot_water_quality(file_path, characteristic_name):
    # Load the data
    df = pd.read_csv(file_path)

    # Filter the data for the desired water quality characteristic
    filtered_df = df[df['CharacteristicName'] == characteristic_name]

    # Convert the 'ActivityStartDate' to datetime format
    filtered_df['ActivityStartDate'] = pd.to_datetime(filtered_df['ActivityStartDate'])

    # Ensure 'ResultMeasureValue' is numeric and handle missing values
    filtered_df['ResultMeasureValue'] = pd.to_numeric(filtered_df['ResultMeasureValue'], errors='coerce')
    filtered_df = filtered_df.dropna(subset=['ResultMeasureValue'])

    # Plot the results
    plt.figure(figsize=(10, 6))
    
    for site in filtered_df['MonitoringLocationIdentifier'].unique():
        site_data = filtered_df[filtered_df['MonitoringLocationIdentifier'] == site]
        plt.plot(site_data['ActivityStartDate'], site_data['ResultMeasureValue'], label=site)

    plt.xlabel('Date')
    plt.ylabel(f'{characteristic_name} Value')
    plt.title(f'{characteristic_name} Over Time')
    plt.legend()
    plt.grid(True)
    plt.show()

# Example usage
file_path = 'narrowresult (1).csv'
characteristic_name = 'Dissolved oxygen (DO)'
plot_water_quality(file_path, characteristic_name)

App:
# Assistant
# First, install the required dependencies
import sys
!pip install altair==4.2.2  # Install a compatible version of altair
!pip install streamlit --upgrade  # Ensure streamlit is up to date
!pip install folium streamlit-folium  # Ensure these are installed

# Restart the kernel after installing packages, then run this code
import pandas as pd
import streamlit as st
import folium
from streamlit_folium import folium_static

# Load the data
station_data = pd.read_csv('station (1).csv')

# Streamlit app
st.title('Water Quality Measurement Sites')

# Create a map centered around the average latitude and longitude
map_center = [station_data['LatitudeMeasure'].mean(), station_data['LongitudeMeasure'].mean()]
site_map = folium.Map(location=map_center, zoom_start=6)
# Add markers for each site
for _, row in station_data.iterrows():
    folium.Marker(
        location=[row['LatitudeMeasure'], row['LongitudeMeasure']],
        popup=row['MonitoringLocationName']
    ).add_to(site_map)

# Display the map in Streamlit
folium_static(site_map)

# Filter data for display
st.subheader('Site Information')
selected_site = st.selectbox('Select a site:', station_data['MonitoringLocationName'].unique())
site_info = station_data[station_data['MonitoringLocationName'] == selected_site]
st.write(site_info)
