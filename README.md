# Exploring-WattTime-API
Code Related to Functionality of WattTime API. The WattTime API provides access to real-time, forecast, and historical data for electric grids around the world, including marginal emissions data. 

Device makers and software providers can incorporate one or more of WattTime’s data signals into their control algorithms and user interfaces to enable impact-aware load-shifting. Organizations procuring renewable energy can also use these data to select renewable energy projects in locations where they avoid more emissions (i.e., emissionality). These data allow you to optimize (or co-optimize) multiple end-user objectives beyond simply cost reduction, including reduction of carbon emissions or human health impacts.
<hr>

## References 
1. WattTime API Documentation: https://docs.watttime.org/
2. WattTime Python Client: https://github.com/WattTime/watttime-python-client
3. WattTime Data Signals: https://watttime.org/data-science/data-signals/
<hr>

## Endpoints 

#### `/register`
```python
register_url = 'https://api.watttime.org/register'
params = {'username': 'your_username',
         'password': 'your_password',
         'email': 'your_email_id',
         'org': 'optional_param'}
rsp = requests.post(register_url, json=params)
print(rsp.text)
```
#### `/login`
```python
login_url = 'https://api.watttime.org/login'
rsp = requests.get(login_url, auth=HTTPBasicAuth('your_username', 'your_password'))
TOKEN = rsp.json()['token']
print(TOKEN)
```
#### Checking Access granted for the account `/v3/my-acess`
```python
url = "https://api.watttime.org/v3/my-access"
headers = {"Authorization": f"Bearer {TOKEN}"}
params = {}
response = requests.get(url, headers=headers, params=params)
response.raise_for_status()
print(response.json())
```
`parsing it to get better understanding`
```python
# Assuming response.json() returns the JSON data
json_data = response.json()

# Extract the 'signal_types' key
data = json_data['signal_types']


# Normalize the JSON data
# Normalize the JSON data
df = pd.json_normalize(data, record_path=['regions', 'endpoints', 'models'],
                       meta=['signal_type', ['regions', 'region'],
                             ['regions', 'region_full_name'], ['regions', 'parent'],
                             ['regions', 'data_point_period_seconds'],
                             ['regions', 'endpoints', 'endpoint']])

# Display the DataFrame
for i in df.columns:
    print(f"\n----{i}------\n {df[i].value_counts()} \n----------\n")
```
#### Information about Grid Regions based on Latitude and Longitude `/v3/region-from-loc`
```python
url = "https://api.watttime.org/v3/region-from-loc"

headers = {"Authorization": f"Bearer {TOKEN}"}
params = {"latitude": "32.372", "longitude": "-87.519", "signal_type": "co2_moer"}
response = requests.get(url, headers=headers, params=params)
response.raise_for_status()
print(response.json())
```
```python
url = "https://api.watttime.org/v3/region-from-loc"

headers = {"Authorization": f"Bearer {TOKEN}"}
params = {"latitude": "42.372", "longitude": "-76.519", "signal_type": "health_damage"}
response = requests.get(url, headers=headers, params=params)
response.raise_for_status()
print(response.json())
```
#### Forecasting Data `/v3/forecast`
```python
url = "https://api.watttime.org/v3/forecast"

headers = {"Authorization": f"Bearer {TOKEN}"}
params = {
    "region": "CAISO_NORTH",
    "signal_type": "co2_moer",
}
response = requests.get(url, headers=headers, params=params)
response.raise_for_status()

json_data = response.json()
data = json_data['data']
data
```
<hr>

## Structuring and Analysing Data to Extract Insights 

`Structuring Data`
```python
df = pd.DataFrame(data)

df['Date'] = pd.to_datetime(df['point_time']).dt.date
df['Time'] = pd.to_datetime(df['point_time']).dt.time
df['Hour'] = pd.to_datetime(df['point_time']).dt.hour
df.drop(columns=['point_time'], inplace=True)
df
```
![Screenshot 2024-10-03 at 11 51 34 AM](https://github.com/user-attachments/assets/f46e9f7a-f409-438d-83c8-25251b886fe7)

`Analysing Dates`
```python
df['Date'].value_counts()
```
![Screenshot 2024-10-03 at 11 52 25 AM](https://github.com/user-attachments/assets/bb6e29d1-4032-4d9e-8c74-65818b22219d)

```python
unique_dates = df['Date'].unique()
unique_dates
```
`Visualizing data for each date to comprehend the trend of carbon intensity and make informed decisions about when to reduce or increase workload`<br><br>
`Work more when carbon intensity is less, Work less when carbon intensity is high `<br><br>
`For 1st Date of Prediction`<br>
```python
df_date_01 = df[pd.to_datetime(df['Date']) == pd.to_datetime(unique_dates[0])]
df_date_01.shape
```
```python
plt.figure(figsize=(10, 6))
plt.plot(df_date_01['Hour'], df_date_01['value'], marker='o', linestyle="--", color='red', label='Value')

# Rotate date labels for better readability
plt.gcf().autofmt_xdate()

# Add labels and title
plt.xlabel('Time in Hour')
plt.ylabel('Value')
plt.title('Value over Time')

# Show the plot
plt.legend()
plt.grid(True)
plt.show()
```
![image](https://github.com/user-attachments/assets/cdea0326-8189-4ecf-b875-adc1257f5abc)

`For 2nd Date of Forecast`
```python
df_date_02 = df[pd.to_datetime(df['Date']) == pd.to_datetime(unique_dates[1])]
df_date_02.shape
```
```python
plt.figure(figsize=(10, 6))
plt.plot(df_date_02['Hour'], df_date_02['value'], marker='o', linestyle="--", color='red', label='Value')

# Rotate date labels for better readability
plt.gcf().autofmt_xdate()

# Add labels and title
plt.xlabel('Time in Hour')
plt.ylabel('Value')
plt.title('Value over Time')

# Show the plot
plt.legend()
plt.grid(True)
plt.show()
```
![image](https://github.com/user-attachments/assets/441efbdb-67e9-4d98-9fda-0b4f145c2fa1)

