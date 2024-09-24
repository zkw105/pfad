import requests
import parsel
import matplotlib.pyplot as plt
from datetime import datetime

headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
    "Referer": "https://tianqi.2345.com/"
}

response = requests.get(
    "https://tianqi.2345.com/Pc/GetHistory?areaInfo%5BareaId%5D=54511&areaInfo%5BareaType%5D=2&date%5Byear%5D=2024&date%5Bmonth%5D=7",
    headers=headers)

print("HTTP 状态码:", response.status_code)

if response.status_code == 200:
    json_data = response.json()
    html_data = json_data['data']
    selector = parsel.Selector(html_data)

    dates = []
    high_temperatures = []
    low_temperatures = []

    for tr in selector.css('tr')[1:]:
        td = tr.css('td::text').getall()
        if len(td) >= 3:
            date_str = td[0].strip()
            high_temp_str = td[1].strip('°')
            low_temp_str = td[2].strip('°')

            try:
                high_temp = float(high_temp_str)
                low_temp = float(low_temp_str)
                date_cleaned = date_str.split(' ')[0]
                date = datetime.strptime(date_cleaned, "%Y-%m-%d")
            except ValueError as e:
                print(f"无法转换温度数据: {e}")
                continue

            dates.append(date)
            high_temperatures.append(high_temp)
            low_temperatures.append(low_temp)

else:
    print("请求数据失败")


print("提取的数据:")
for d, ht, lt in zip(dates, high_temperatures, low_temperatures):
    print(f"日期: {d.date()}, high: {ht}, high: {lt}")


plt.figure(figsize=(10, 5))
plt.plot(dates, high_temperatures, marker='o', label='high')
plt.plot(dates, low_temperatures, marker='o', label='low')


plt.title("202407everyday temperature of Beijing")
plt.xlabel("date")
plt.ylabel("temp (°C)")

plt.xticks(rotation=45)
plt.grid(True)
plt.legend()
plt.tight_layout()

plt.show()
