# Logi Web 設定工具

用瀏覽器直接調整羅技滑鼠／鍵盤的設定，不需要安裝 G HUB 或 Logi Options+。
整個工具只有一個 `index.html`，不用安裝套件，也不用建置。

## 原理
- **WebHID**：Chrome / Edge 內建的 API，網頁經使用者同意後可以和 HID 裝置溝通。
- **HID++ 2.0**：羅技裝置的廠商私有協定。所有功能都以 *feature* 的形式提供（例如 `0x2201` 是 DPI、`0x8060` 是回報率）。
  工具會先問裝置支援哪些 feature，再只顯示它支援的設定面板。

## 目前支援
| 功能 | Feature |
|---|---|
| 裝置名稱、類型、韌體版本 | 0x0005 / 0x0003 |
| 電池（百分比／電壓） | 0x1004 / 0x1000 / 0x1001 |
| DPI 調整（滑桿、預設值、快速按鈕）、離地高度 LOD | 0x2201 / 0x2202 |
| 回報率 125–8000 Hz | 0x8060 / 0x8061 |
| 板載設定檔：內建記憶體模式／主機模式 | 0x8100 |
| SmartShift 滾輪（MX 系列） | 0x2110 / 0x2111 |
| 高解析度滾輪、反向捲動 | 0x2121 |
| Easy-Switch 主機切換 | 0x1814 |
| 按鍵列表（唯讀） | 0x1B04 |
| 支援 feature 清單、原始指令主控台 | — |

連線方式：USB 有線、藍牙、Unifying／Bolt／Lightspeed 接收器（會掃描槽位 1–6）。

## 使用方式
1. 用桌面版 **Chrome、Edge 或 Opera** 開啟（Firefox、Safari 不支援 WebHID）。
2. WebHID 只能在安全環境使用，所以請用下面其中一種方式開啟頁面：
   - 在這個資料夾執行 `python3 -m http.server 8000`，再開啟 `http://localhost:8000`
   - 或部署到任何 HTTPS 網站，例如 GitHub Pages
   - 直接雙擊用 `file://` 開啟，在多數 Chrome 版本也可以使用
3. 按「連線裝置」並選擇你的裝置。用接收器連線時，如果找不到滑鼠，先動一下滑鼠喚醒它，再按「重新掃描」。

## 注意事項
- **先關閉 G HUB／Options+／Solaar**，否則它們可能會佔用裝置或覆蓋設定。
- G 系列滑鼠在「內建記憶體模式」下會套用板載設定檔，DPI 修改可能沒效果。請切換到「主機模式」。
  主機模式的設定通常在裝置斷電後就會還原，所以每次開機可能要重新套用。
- **Linux** 需要給 hidraw 權限，例如新增 `/etc/udev/rules.d/42-logitech.rules`：
  ```
  KERNEL=="hidraw*", ATTRS{idVendor}=="046d", MODE="0660", TAG+="uaccess"
  ```
  然後執行 `sudo udevadm control --reload && sudo udevadm trigger`，再重新插拔裝置。
- 舊的 HID++ 1.0 裝置（大約 2012 年以前）不在支援範圍內。

## 尚未實作（可以擴充的方向）
- RGB 燈效（0x8070 / 0x8071）
- 按鍵重新指派（0x1B04 setCidReporting）
- 將設定寫入板載設定檔，讓設定斷電後也保留（0x8100 記憶體讀寫）
- 鍵盤 Fn 反轉（0x40A3）、背光（0x1982）

協定可參考 Solaar 的原始碼（`lib/logitech_receiver/`）以及 Linux kernel 的 `hid-logitech-hidpp.c`。
