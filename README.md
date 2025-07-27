# Api-integration-and-visualization
import requests
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

def fetch_flight_data(api_key, dep_iata, arr_iata, date):
    url = "http://api.aviationstack.com/v1/flights"
    params = {
        'access_key': api_key,
        'dep_iata': dep_iata,
        'arr_iata': arr_iata,
        'limit': 100
    }

    try:
        response = requests.get(url, params=params)
        response.raise_for_status()
        data = response.json()
    except Exception as e:
        print("❌ API error:", e)
        return pd.DataFrame()

    flights = []
    for flight in data.get("data", []):
        try:
            flights.append({
                'Airline': flight['airline']['name'],
                'Flight Number': flight['flight']['iata'],
                'Departure Time': flight['departure']['scheduled'],
                'Arrival Time': flight['arrival']['scheduled'],
                'Terminal': flight['departure']['terminal'],
                'Gate': flight['departure']['gate'],
                'Status': flight['flight_status']
            })
        except:
            continue

    return pd.DataFrame(flights)

def visualize_flight_data(df):
    if df.empty:
        print("⚠️ No flight data available.")
        return

    print("\n📋 Sample Data:\n", df.head())

    sns.set(style="whitegrid")

    # 🔹 Bar Chart: Number of flights per airline
    plt.figure(figsize=(10, 6))
    airline_counts = df['Airline'].value_counts()
    sns.barplot(x=airline_counts.values, y=airline_counts.index, palette="mako")
    plt.title("Number of Scheduled Flights per Airline")
    plt.xlabel("Flight Count")
    plt.ylabel("Airline")
    plt.tight_layout()
    plt.show()

    # 🔹 Pie Chart: Distribution of flight statuses
    plt.figure(figsize=(7, 7))
    status_counts = df['Status'].value_counts()
    plt.pie(status_counts, labels=status_counts.index, autopct='%1.1f%%', startangle=140, colors=sns.color_palette('Set3'))
    plt.title("Flight Status Distribution")
    plt.axis('equal')
    plt.show()

def main():
    print("🛫 Aviationstack Flight Visualizer")

    api_key = input("🔑 Enter your Aviationstack API key: ").strip()
    dep_iata = input("🛫 Enter departure IATA code (e.g., DEL): ").strip().upper()
    arr_iata = input("🛬 Enter arrival IATA code (e.g., DXB): ").strip().upper()
    date = input("📅 (Optional) Enter date (YYYY-MM-DD): ").strip()

    print("\n📡 Fetching flight data...")
    df = fetch_flight_data(api_key, dep_iata, arr_iata, date)

    if df.empty:
        print("❌ No data found.")
    else:
        visualize_flight_data(df)

if __name__ == "__main__":
    main()

