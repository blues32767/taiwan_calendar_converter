# 臺灣學年度、民國、農曆、西元年曆 日期對照表
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
![Python Version](https://img.shields.io/badge/python-3.6%2B-blue)

這是一個多功能的台灣年曆日期對照產生工具，可以同時處理學年度、民國、農曆和西元年份的對照。支援節日顯示和多種輸出格式。
特別適合作為 AI 模型的參考資料庫，協助校正常見的日期轉換錯誤。


## 主要用途
### AI 模型參考資料庫
- 解決 AI 在處理台灣特有日期系統時的常見錯誤：
  - 民國年份轉換錯誤
  - 星期對照錯誤
  - 學年度判斷錯誤
- 提供標準化的日期對照表
- 支援完整的節日判斷


### 快速使用方式
1. **直接使用現有資料**：
   - 在 `output` 資料夾中已提供預先產生的對照表2024-2050年
   - 格式包含 CSV 和 TXT 兩種版本
   - 可直接導入系統使用，無需重新執行程式

2. **自定義產生資料**：
   - 可依需求修改年份範圍
   - 支援自訂輸出格式和內容

## 功能特點
- 支援學年度、民國、農曆、西元年份轉換
- 自動判斷台灣國定假日和農曆節日
- 支援 CSV 和 TXT 格式輸出
- 可選擇按年份分割或合併輸出檔案
- 完整支援繁體中文輸出




## 使用方式
1. 直接下載【2024-2050合併版(民國113-139年)-taiwan_calendar】使用

2. 或使用python執行code設定基本參數：
 ## 安裝需求
```bash
pip install lunar-python
```
```python
START_YEAR = 2024  # 開始年(可調整)
END_YEAR = 2026    # 結束年(可調整)

# 輸出選項設定 (0:關閉 1:開啟)
SHOW_CONSOLE = 0   # 終端機顯示
EXPORT_CSV = 1     # CSV檔案輸出
EXPORT_TXT = 1     # TXT檔案輸出
SPLIT_FILES = 1    # 檔案分割選項（1:分割 0:合併）
```

2. 執行程式：
```bash
python taiwan_calendar_converter.py
```

## 輸出範例
### CSV 格式
```csv
西元,星期,學年度,民國,生肖年,農曆年,農曆月,農曆日,臺灣國曆節假日,臺灣農曆節假日
2024/1/1,一,112,113/1/1,龍,甲辰,冬月,二十,開國紀念日,
```

### TXT 格式
```
西元：2024/1/1(一)[112學年度]
民國：113/1/1(一)[112學年度]
農曆：(龍) 甲辰年 冬月二十
節日：開國紀念日
```

## 支援的節日
### 國定假日
- 開國紀念日 (1/1)
- 和平紀念日 (2/28)
- 兒童節 (4/4)
- 國慶日 (10/10)

### 農曆節日
- 春節 (農曆1/1-1/3)
- 端午節 (農曆5/5)
- 中秋節 (農曆8/15)
- 除夕

## 配置選項說明
| 參數 | 說明 | 可選值 |
|------|------|--------|
| SHOW_CONSOLE | 終端機顯示輸出 | 0:關閉 1:開啟 |
| EXPORT_CSV | CSV檔案輸出 | 0:關閉 1:開啟 |
| EXPORT_TXT | TXT檔案輸出 | 0:關閉 1:開啟 |
| SPLIT_FILES | 檔案分割選項 | 0:合併 1:分割 |

## 完整程式碼
```python
#臺灣學年度、西元年、民國年、農曆年、對照產生csv、txt -20250206-huangka
# 請先透過終端機安裝【lunar-python】套件，指令在第三行
# pip install lunar-python

from datetime import datetime, timedelta
from lunar_python import Lunar, Solar
import csv
import os

# 設定年份範圍
START_YEAR = 2024  # 開始年
END_YEAR = 2025  # 結束年

# 輸出選項 0關閉 1開啟 ，預設輸出csv跟txt可匯入給AI套用。
SHOW_CONSOLE = 0  # 直接顯示在終端機
EXPORT_CSV = 1  # 輸出csv
EXPORT_TXT = 1  # 輸出txt
SPLIT_FILES = 0  # 每年分割為1個檔案（1為分割，0為合併）

# 建立輸出目錄
OUTPUT_DIR = "calendar_output"
if not os.path.exists(OUTPUT_DIR):
    os.makedirs(OUTPUT_DIR)

# 生肖轉換字典（簡體轉繁體）
zodiac_convert = {
    '鼠': '鼠', '牛': '牛', '虎': '虎', '兔': '兔',
    '龙': '龍', '蛇': '蛇', '马': '馬', '羊': '羊',
    '猴': '猿', '鸡': '雞', '狗': '狗', '猪': '豬'
}


def convert_zodiac(zodiac):
    """轉換生肖為繁體中文"""
    return zodiac_convert.get(zodiac, zodiac)


def get_lunar_holiday(lunar_date):
    """判斷農曆節日"""
    month = lunar_date.getMonth()
    day = lunar_date.getDay()

    lunar_holidays = {
        (1, 1): "春節",
        (1, 2): "春節",
        (1, 3): "春節",
        (5, 5): "端午節",
        (8, 15): "中秋節"
    }

    next_year_first_day = Lunar.fromYmd(lunar_date.getYear() + 1, 1, 1)
    prev_day = next_year_first_day.getSolar().next(-1).getLunar()

    if month == prev_day.getMonth() and day == prev_day.getDay():
        return "除夕"

    return lunar_holidays.get((month, day), "")


def is_holiday(month, day):
    """判斷國定節日"""
    holidays = {
        (1, 1): "開國紀念日",
        (2, 28): "和平紀念日",
        (4, 4): "兒童節",
        (10, 10): "國慶日"
    }
    return holidays.get((month, day), "")


def get_school_year(date):
    """判斷學年度"""
    if date.month >= 8:
        return date.year - 1911
    else:
        return date.year - 1912


def export_to_csv(data, filename):
    """輸出CSV檔案"""
    fieldnames = [
        '西元', '星期', '學年度',
        '民國', '生肖年', '農曆年',
        '農曆月', '農曆日', '臺灣國曆節假日',
        '臺灣農曆節假日'
    ]
    with open(filename, 'w', encoding='utf-8-sig', newline='') as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(data)


def export_to_txt(data, filename):
    """輸出TXT檔案"""
    with open(filename, 'w', encoding='utf-8') as f:
        for row in data:
            f.write(f"西元：{row['西元']}({row['星期']})[{row['學年度']}學年度]\n")
            f.write(f"民國：{row['民國']}({row['星期']})[{row['學年度']}學年度]\n")
            f.write(f"農曆：({row['生肖年']}) {row['農曆年']}年 {row['農曆月']}{row['農曆日']}\n")
            if row['臺灣國曆節假日'] or row['臺灣農曆節假日']:
                f.write(f"節日：{row['臺灣國曆節假日']} {row['臺灣農曆節假日']}".strip() + "\n")
            f.write("\n")


def generate_taiwan_calendar():
    """生成臺灣年曆資料"""
    all_data = []

    for year in range(START_YEAR, END_YEAR + 1):
        weekday_chinese = {
            0: '一', 1: '二', 2: '三', 3: '四',
            4: '五', 5: '六', 6: '日'
        }

        current_date = datetime(year, 1, 1)
        end_date = datetime(year, 12, 31)
        year_data = []

        while current_date <= end_date:
            year_ad = current_date.year
            year_roc = year_ad - 1911
            month = current_date.month
            day = current_date.day
            weekday = weekday_chinese[current_date.weekday()]
            school_year = get_school_year(current_date)

            solar = Solar.fromDate(current_date)
            lunar = solar.getLunar()

            national_holiday = is_holiday(month, day)
            lunar_holiday = get_lunar_holiday(lunar)

            row_data = {
                '西元': f"{year_ad}/{month}/{day}",
                '星期': f"{weekday}",
                '學年度': f"{school_year}",
                '民國': f"{year_roc}/{month}/{day}",
                '生肖年': convert_zodiac(lunar.getYearShengXiao()),
                '農曆年': lunar.getYearInGanZhi(),
                '農曆月': lunar.getMonthInChinese(),
                '農曆日': lunar.getDayInChinese(),
                '臺灣國曆節假日': national_holiday,
                '臺灣農曆節假日': lunar_holiday
            }
            year_data.append(row_data)
            all_data.append(row_data)
            current_date += timedelta(days=1)

        if SPLIT_FILES:
            # 輸出單年檔案
            if EXPORT_CSV:
                csv_filename = os.path.join(OUTPUT_DIR, f'taiwan_calendar_{year}.csv')
                export_to_csv(year_data, csv_filename)

            if EXPORT_TXT:
                txt_filename = os.path.join(OUTPUT_DIR, f'taiwan_calendar_{year}.txt')
                export_to_txt(year_data, txt_filename)

    if not SPLIT_FILES:
        # 輸出合併檔案
        if EXPORT_CSV:
            csv_filename = os.path.join(OUTPUT_DIR, f'taiwan_calendar_{START_YEAR}-{END_YEAR}.csv')
            export_to_csv(all_data, csv_filename)

        if EXPORT_TXT:
            txt_filename = os.path.join(OUTPUT_DIR, f'taiwan_calendar_{START_YEAR}-{END_YEAR}.txt')
            export_to_txt(all_data, txt_filename)

    return len(all_data)


def main():
    output_types = []
    if SHOW_CONSOLE: output_types.append("終端機顯示")
    if EXPORT_CSV: output_types.append("CSV檔案")
    if EXPORT_TXT: output_types.append("TXT檔案")

    print(f"=== {START_YEAR}-{END_YEAR} 臺灣學年度年曆 ===")
    print(f"輸出方式：{', '.join(output_types)}")
    print(f"檔案分割：{'是' if SPLIT_FILES else '否'}")
    print("=" * 50)

    total_days = generate_taiwan_calendar()

    print(f"\n完成！總共處理了 {total_days} 天的資料")
    print(f"檔案已儲存在 {OUTPUT_DIR} 目錄下")


# 執行程式
if __name__ == "__main__":
    main()

```

## 注意事項
1. 本工具特別適合作為 AI 模型的參考資料庫使用
2. 如遇到特殊日期轉換需求，建議參考 `output` 資料夾中的標準對照表

## 授權協議
本專案採用 MIT 授權協議。詳情請見 [LICENSE](LICENSE) 檔案。

## 作者
[Huang Jun Ci]

## 更新日誌
### v1.0.0 (2025-02-06)
- 初始版本發布
- 支援基本年曆轉換功能
- 加入檔案分割選項
- 提供預產生的標準對照表
