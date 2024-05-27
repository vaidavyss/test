import re
import json
import requests
from datetime import datetime, timedelta

url = "https://www.ober-haus.lt/"
response = requests.get(url)
html_content = response.text

# Regular expression to extract the relevant data
data_regex = re.compile(r'<ul class="date-\d{2}-\d{4}">(.*?)</ul>', re.DOTALL)
value_regex = re.compile(r'<li class="(.*?)">(.*?)</li>')

# Parsing the HTML content
parsed_data = []
for ul in data_regex.findall(html_content):
    values = dict(value_regex.findall(ul))
    parsed_data.append(values)

# Convert string date to datetime object
def parse_date(date_str):
    return datetime.strptime(date_str, "%m/%Y")

# Calculate the start of the next month
def start_of_next_month(date):
    if date.month == 12:
        return datetime(date.year + 1, 1, 1)
    else:
        return datetime(date.year, date.month + 1, 1)

# Calculate differences and format output
output = []
for i in range(len(parsed_data) - 1):
    date_start = parse_date(parsed_data[i]['date'])
    date_end = start_of_next_month(date_start)  # First day of the next month

    # Example for 'all_cities' - modify as needed for other cities
    price_change = ((float(parsed_data[i + 1]['all_cities']) / float(parsed_data[i]['all_cities'])) - 1) * 100

    output.append({
        "date_start": date_start.strftime("%Y-%m-%d"),
        "date_end": date_end.strftime("%Y-%m-%d"),
        "price_change": price_change
    })

# Output the result
print(json.dumps(output, indent=2))
