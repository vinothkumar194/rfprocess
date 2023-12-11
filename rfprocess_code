import streamlit as st
import pandas as pd
import re
from datetime import datetime


# Function to check if a year is a leap year
def is_leap_year(year):
    return year % 4 == 0 and (year % 100 != 0 or year % 400 == 0)


# Function to extract year from filename
def extract_year_from_filename(filename):
    match = re.search(r'\d{4}', filename)
    return int(match.group()) if match else None


# Main processing function
def process_data(uploaded_file, year):
    # Read the file
    data = pd.read_excel(uploaded_file)

    # Data preprocessing
    if 'temp' in data.columns:
        data.drop(['temp'], axis=1, inplace=True)

    data['Year'].fillna(year, inplace=True)
    data['Mth'].fillna(1, inplace=True)
    data['Day'].fillna(1, inplace=True)
    data['Year'] = data['Year'].astype(int).astype(str)
    data['Mth'] = data['Mth'].astype(int).astype(str)
    data['Day'] = data['Day'].astype(int).astype(str)
    data['Date'] = pd.to_datetime(data[['Year', 'Mth', 'Day']].agg('-'.join, axis=1))
    data.drop(['Year', 'Mth', 'Day'], axis=1, inplace=True)

    # Handling missing data
    places = data['Place'].unique()
    missing_data = pd.DataFrame()
    total_missing_days = 0

    for place in places:
        place_data = data[data['Place'] == place]
        full_range = pd.date_range(start=f'{year}-01-01', end=f'{year}-12-31')
        missing_dates = full_range.difference(place_data['Date'])
        total_missing_days += len(missing_dates)
        missing_rows = [{'Date': date, 'Lng': place_data['Lng'].iloc[0], 'Lat': place_data['Lat'].iloc[0], 'RF': -99.9,
                         'Place': place} for date in missing_dates]
        missing_data = pd.concat([missing_data, pd.DataFrame(missing_rows)], ignore_index=True)

    # Combining data
    combined_data = pd.concat([data, missing_data]).drop_duplicates(subset=['Place', 'Date']).sort_values(
        by=['Place', 'Date'])

    # Further manipulation of combined data
    df_from_dict = combined_data.pivot_table(index='Date', columns='Place', values='RF', aggfunc='mean')

    # Percentage of Missing Data
    total_days = len(places) * (366 if is_leap_year(year) else 365)
    percentage_of_missing = (total_missing_days / total_days) * 100

    return combined_data, missing_data, df_from_dict, percentage_of_missing


def main():
    st.title("Rainfall Data Processing App")

    uploaded_file = st.file_uploader("Upload an Excel file", type="xlsx")

    if uploaded_file is not None:
        year = extract_year_from_filename(uploaded_file.name)
        if year is None:
            st.error("Year not found in file name.")
        else:
            combined_data, missing_data, output_data, percentage_of_missing = process_data(uploaded_file, year)

            # Display the percentage of missing data
            st.markdown(f"<h2 style='color: red;'>Percentage of Missing Data: {percentage_of_missing:.2f}%</h2>",
                        unsafe_allow_html=True)

            # Show and download Combined Data
            st.subheader("Combined Data")
            st.dataframe(combined_data)
            st.download_button("Download Combined Data", combined_data.to_csv().encode('utf-8'),
                               f"combined_data_{year}.csv", "text/csv")

            # Show and download Missing Data
            st.subheader("Missing Data")
            st.dataframe(missing_data)
            st.download_button("Download Missing Data", missing_data.to_csv().encode('utf-8'),
                               f"missing_data_{year}.csv", "text/csv")

            # Show and download Output Data
            st.subheader("Output Data")
            st.dataframe(output_data)
            st.download_button("Download Output Data", output_data.to_csv().encode('utf-8'), f"output_data_{year}.csv",
                               "text/csv")


if __name__ == "__main__":
    main()

# streamlit run RF_web_app2.py
