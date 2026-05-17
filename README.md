
import undetected_chromedriver as uc
from bs4 import BeautifulSoup
import csv
import time

def scrape_fbref_undetected():
    url = "https://fbref.com/en/comps/9/stats/Premier-League-Stats"
    
    print("Đang khởi động trình duyệt chống phát hiện...")
    
    options = uc.ChromeOptions()
    driver = uc.Chrome(options=options)
    
    try:
        print("Đang truy cập trang web...")
        driver.get(url)
       
        print("Đang chờ tải trang và vượt qua Cloudflare (15 giây)...")
        time.sleep(15) 
        
       
        html_content = driver.page_source
        soup = BeautifulSoup(html_content, 'html.parser')
        
        table = soup.find('table', {'id': 'stats_standard'})
        if not table:
            print("Vẫn không tìm thấy bảng dữ liệu. Hãy thử tăng thời gian chờ.")
            return

        print("Đã lấy được HTML, đang phân tích dữ liệu...")
        
        
        thead_rows = table.find('thead').find_all('tr')
        headers_html = thead_rows[-1].find_all('th')
        
        csv_headers = ["STT"]
        for th in headers_html[1:]:
            csv_headers.append(th.text.strip())

       
        tbody = table.find('tbody')
        rows = tbody.find_all('tr')
        
        player_data_list = []
        stt = 1

        for row in rows:
            if 'thead' in row.get('class', []):
                continue

            minutes_cell = row.find('td', {'data-stat': 'minutes'})
            if not minutes_cell or not minutes_cell.text.strip():
                continue 
                
            minutes_played = int(minutes_cell.text.replace(',', '').strip())

            if minutes_played > 90:
                cells = row.find_all(['th', 'td'])
                row_data = [stt] 
                
                for cell in cells[1:]:
                    val = cell.text.strip()
                    if val == "":
                        val = "N/a"
                    row_data.append(val)
                
                player_data_list.append(row_data)
                stt += 1 

        
        filename = "EPL_Players_Stats_Undetected.csv"
        with open(filename, mode='w', newline='', encoding='utf-8') as file:
            writer = csv.writer(file)
            writer.writerow(csv_headers)
            writer.writerows(player_data_list)

        print(f"Thành công! Đã thu thập {len(player_data_list)} cầu thủ và lưu vào {filename}.")

    except Exception as e:
        print(f"Có lỗi xảy ra: {e}")
    finally:
        print("Đang đóng trình duyệt...")
        driver.quit()

if __name__ == "__main__":
    scrape_fbref_undetected()
