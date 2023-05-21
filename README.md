#python 

პროექტი შექმნილია API-ის გამოყენებით COVID-19-ის მონაცემების მისაღებად, ინფორმაციის დასამუშავებლად და ჩვენებაზე, შესანახად JSON ფაილში და SQLite მონაცემთა ბაზაში და უზრუნველყოს Windows-ის შეტყობინება გლობალური COVID-19-ის შეჯამებით.
ძირითადი ფუნქციის გაშვებით, პროექტი იღებს მონაცემებს, ინახავს მათ JSON ფაილში, აჩვენებს ინფორმაციას კონსოლზე, ინახავს მონაცემთა ბაზაში და ააქტიურებს შეტყობინებას. ეს საშუალებას აძლევს მომხმარებლებს ადვილად მიიღონ წვდომა და თვალყური ადევნონ COVID-19 მონაცემებს, ამასთან, უზრუნველყონ რეალურ დროში განახლებები შეტყობინებების საშუალებით.

import requests
import json
import sqlite3
from win10toast import ToastNotifier

def get_covid_data():
    url = "https://api.covid19api.com/summary"

    try:
        response = requests.get(url)
        
        if response.status_code == 200:
            data = response.json()
            return data
        else:
            print("Request failed with status code:", response.status_code)
    except requests.exceptions.RequestException as e:
        print("Error retrieving COVID-19 data:", e)

def store_json_data(data, file_path):
    with open(file_path, "w") as file:
        json.dump(data, file, indent=4)

def access_information(data):
    global_summary = data["Global"]
    print("Global COVID-19 Summary:")
    print("Total cases:", global_summary["TotalConfirmed"])
    print("Total deaths:", global_summary["TotalDeaths"])
    print("Total recovered:", global_summary["TotalRecovered"])
    print()
    
    countries = data["Countries"]
    print("COVID-19 Summary by Country:")
    for country in countries:
        print("Country:", country["Country"])
        print("Total cases:", country["TotalConfirmed"])
        print("Total deaths:", country["TotalDeaths"])
        print("Total recovered:", country["TotalRecovered"])
        print()

def store_in_database(data):
    conn = sqlite3.connect("covid_data.db")
    cursor = conn.cursor()

    cursor.execute("CREATE TABLE IF NOT EXISTS covid_data (country TEXT, total_cases INTEGER, total_deaths INTEGER, total_recovered INTEGER)")

    countries = data["Countries"]
    for country in countries:
        cursor.execute("INSERT INTO covid_data VALUES (?, ?, ?, ?)",
                       (country["Country"], country["TotalConfirmed"], country["TotalDeaths"], country["TotalRecovered"]))
    
    conn.commit()
    conn.close()
    print("COVID-19 data stored in the database.")

def show_notification(data):
    global_summary = data["Global"]
    toast = ToastNotifier()
    toast.show_toast("COVID-19 Summary", f"Total cases: {global_summary['TotalConfirmed']}\nTotal deaths: {global_summary['TotalDeaths']}\nTotal recovered: {global_summary['TotalRecovered']}",
                     duration=10)

def main():
    data = get_covid_data()

    if data:
        file_path = "covid_data.json"
        store_json_data(data, file_path)
        print("COVID-19 data stored in JSON file:", file_path)
        
        access_information(data)
        
        store_in_database(data)
        
        show_notification(data)
    else:
        print("No COVID-19 data available.")

if __name__ == "__main__":
    main()
