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
import streamlit as st
import pandas as pd
import folium
from streamlit_folium import folium_static
import matplotlib.pyplot as plt

st.title("Water Quality Monitoring Dashboard")

# --- Upload CSV files ---
st.sidebar.header("Upload CSV Files")
result_file = st.sidebar.file_uploader("Upload result_data CSV", type="csv")
station_file = st.sidebar.file_uploader("Upload station_data CSV", type="csv")

# --- Only proceed if both files are uploaded ---
if result_file is not None and station_file is not None:
    result_data = pd.read_csv(result_file)
    station_data = pd.read_csv(station_file)

    # Convert dates
    result_data['ActivityStartDate'] = pd.to_datetime(result_data['ActivityStartDate'])

    # --- Sidebar filters ---
    st.sidebar.header("Filter Options")
    contaminant_list = result_data['CharacteristicName'].unique()
    contaminant = st.sidebar.selectbox("Select a contaminant:", contaminant_list)

    filtered_data = result_data[result_data['CharacteristicName'] == contaminant]
    min_val = float(filtered_data['ResultMeasureValue'].min())
    max_val = float(filtered_data['ResultMeasureValue'].max())

    min_value, max_value = st.sidebar.slider("Select value range:", min_val, max_val, (min_val, max_val))

    start_date, end_date = st.sidebar.date_input(
        "Select date range:",
        [filtered_data['ActivityStartDate'].min(), filtered_data['ActivityStartDate'].max()]
    )

    # Filter data
    filtered_result_data = filtered_data[
        (filtered_data['ResultMeasureValue'] >= min_value) &
        (filtered_data['ResultMeasureValue'] <= max_value) &
        (filtered_data['ActivityStartDate'] >= pd.to_datetime(start_date)) &
        (filtered_data['ActivityStartDate'] <= pd.to_datetime(end_date))
    ]

    if filtered_result_data.empty:
        st.warning("No data matches your filters.")
    else:
        # --- Map ---
        map_center = [station_data['LatitudeMeasure'].mean(), station_data['LongitudeMeasure'].mean()]
        site_map = folium.Map(location=map_center, zoom_start=6)

        for _, row in filtered_result_data.iterrows():
            site_info = station_data[station_data['MonitoringLocationIdentifier'] == row['MonitoringLocationIdentifier']]
            if not site_info.empty:
                popup_text = f"{row['MonitoringLocationName']}<br>{contaminant}: {row['ResultMeasureValue']}"
                folium.Marker(
                    location=[site_info.iloc[0]['LatitudeMeasure'], site_info.iloc[0]['LongitudeMeasure']],
                    popup=popup_text
                ).add_to(site_map)

        st.subheader("Monitoring Locations Map")
        folium_static(site_map)

        # --- Plot ---
        st.subheader(f"{contaminant} Trend Over Time")
        plt.figure(figsize=(10, 6))
        for site in filtered_result_data['MonitoringLocationIdentifier'].unique():
            site_data = filtered_result_data[filtered_result_data['MonitoringLocationIdentifier'] == site]
            plt.plot(site_data['ActivityStartDate'], site_data['ResultMeasureValue'], label=site)

        plt.xlabel('Date')
        plt.ylabel(f'{contaminant} Value')
        plt.title(f'{contaminant} Over Time')
        plt.legend()
        plt.grid(True)
        st.pyplot(plt.gcf())

else:
    st.info("Please upload both `result_data` and `station_data` CSV files to get started.")
