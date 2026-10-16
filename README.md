# T0⁺ YOLOv7-tiny 脈衝神經網路（SNN）demo 影片

**線上觀看：** https://pekoranoinu.github.io/snn-yolov7tiny-demo/

把 VOC 訓練的 YOLOv7-tiny 轉成**整數脈衝神經網路（SNN）**，在 34 支公開授權的影片上逐幀做物件偵測，並和同一個網路的浮點版（ANN）比較。每支影片都附上 SNN 的偵測結果、各類別「有框的畫面比例」和每幀的估計能耗。

## 模型

**訓練資料：兩個模型都只用 PASCAL VOC 訓練**（VOC2007＋2012 trainval，16,551 張、20 類；從 COCO 預訓練的 YOLOv7-tiny 權重開始），在 VOC2007 test（4,952 張）上評估。下面的 demo 影片都不在訓練資料裡。

| 模型 | 訓練資料 | VOC07 test mAP@0.5 ↑ | mAP@0.5:0.95 ↑ | 說明 |
|---|---|---|---|---|
| T0⁺ YOLOv7-tiny 浮點 ANN（基準） | PASCAL VOC 07＋12 | **0.848** | **0.607** | 6.07 M 參數 |
| T0⁺ YOLOv7-tiny SNN D=4／2 全整數（本 demo） | PASCAL VOC 07＋12（由上一列轉換，再用 VOC 微調） | 0.798 | 0.531 | 時間步 T＝1；LeakyReLU 拆成正 4 階＋負 2 階兩種神經元；每通道整數門檻；INT8 權重；偵測頭輸出 16 位元，由主機解框 |

SNN 的每一層只送出 0 到 4（正半邊）或 0 到 2（負半邊）的整數脈衝數；推論時拆成 0/1 脈衝，卷積只剩加法。影片裡的每一幀都走這條全整數路徑。

## 在 34 支影片上的結果

影片沒有標註框，所以下表的數字是「預期會出現的類別，有被框到的畫面比例」，取各組所有「影片 × 預期類別」（共 82 組）的平均，**不是 mAP**。第一列是基準；兩個模型都只用 PASCAL VOC 07＋12 訓練。

| 模型 | VOC07 mAP@0.5 | 台灣行車視角 ↑ | 道路（機車＋行人） ↑ | 其他 VOC 類別 ↑ | vKITTI2 合成行車 ↑ | 全部 ↑ | 框數／張 |
|---|---|---|---|---|---|---|---|
| T0⁺ YOLOv7-tiny 浮點 ANN（基準） | **0.848** | 60.1% | 61.1% | 69.7% | 86.4% | 65.2% | 4.85 |
| T0⁺ YOLOv7-tiny SNN D=4／2 全整數（本 demo） | 0.798 | 54.9% | 57.3% | 64.0% | 79.8% | 60.4% | 4.07 |

讀法：SNN 比自己的浮點 ANN 少 4.7 個百分點，和 VOC07 上 mAP 掉的 5.0 分同一級；框數也比較少，SNN 比較保守。差最多的是小、遠、被擋住或低對比（霧、夜）的物件。

**每幀能耗**（運算次數估計，INT8 常數：第 0 層吃像素的乘加 0.23 pJ，之後每個脈衝 × 扇出各算一次加法 0.1 pJ；對照是同一張圖跑 INT8 ANN 的乘加；不含記憶體存取）：

| 每幀平均 | 台灣行車視角 | 道路（機車＋行人） | 其他 VOC 類別 | vKITTI2 合成行車 | 全部 |
|---|---|---|---|---|---|
| INT8 ANN 的能耗 mJ（基準） | 0.910 | 0.914 | 0.908 | 0.531 | 0.889 |
| SNN 的能耗 mJ ↓ | 0.402 | 0.407 | 0.390 | 0.228 | 0.388 |
| SNN ÷ INT8 ANN ↓ | 0.44× | 0.45× | 0.43× | 0.43× | 0.44× |

影片輸入是 letterbox 後的長方形（例如 640×384），所以絕對值比 VOC 的 640×640 小，看比例即可（VOC07 上是 0.41×）。

## 設定

- letterbox 640，信心門檻 0.3，NMS IoU 0.45，每一幀都跑。
- 影片縮成 854×480、H.264 重新壓縮，方便網頁播放。

## 影片來源與授權

這裡的影片都是改作：每一幀加上了偵測框和上方的狀態列，並縮成 854×480 重新壓縮。依原授權提供：

- **CC BY-SA 3.0／4.0：** 改作以同一授權（CC BY-SA，同版本）提供。
- **CC BY 2.0／3.0／4.0：** 需署名。
- **vKITTI2（Cabon et al. 2020, Naver Labs Europe）：** CC BY-NC-SA 3.0，只限非商業用途。
- **CC0、Pexels License、Pixabay Content License：** 不需署名。

| 組 | 影片 | 內容 | 原作者／來源 | 授權 |
|---|---|---|---|---|
| 台灣行車視角 | `c01_taiwan_city_scooter_pov` | 台灣市區捷運站前，騎機車往前拍，手持 | [Video by 宋 小天 on Pexels (attribution optional)](https://www.pexels.com/video/footage-of-a-busy-road-from-a-moving-vehicle-4403833/) | Pexels License |
| 台灣行車視角 | `c02_tainan_scooter_crowd_red_light` | 台南路口，停在機車群裡、騎士眼睛高度，鏡頭不動 | [Video by jeffplay from Pixabay (attribution optional)](https://pixabay.com/videos/streetcar-street-taiwan-163554/) | Pixabay Content License |
| 台灣行車視角 | `c03_taroko_moped_pov` | 花蓮太魯閣中橫，騎機車往前拍，山路 | [Taroko - from moped - June 18 2011 by Nesnad, CC BY 4.0, via Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Taroko_-_from_moped_-_June_18_2011.ogg) | CC BY 4.0 |
| 台灣行車視角 | `c04_taichung_bus_night_windshield` | 台中大坑中興嶺下坡，夜間，公車內隔擋風玻璃往前拍 | [Video: Jidanni, CC0, via Wikimedia Commons (attribution optional)](https://commons.wikimedia.org/wiki/File:20240315豐原客運270中興嶺下坡.webm) | CC0 |
| 道路（機車＋行人） | `a01_hcmc_overpass_scooters` | 越南 胡志明市，高架橋上整片機車；手持、會晃 | [Saigon traffic-oVbn3HeLDA0 by Karel Bilek, CC BY-SA 4.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/c/c6/Saigon_traffic-oVbn3HeLDA0.webm) | CC BY-SA 4.0 |
| 道路（機車＋行人） | `a02_taichung_xuetian_crossing` | 台灣 台中烏日 學田路平交道：機車等柵欄、台鐵電聯車通過 | [Video: Saimmx, CC0, via Wikimedia Commons (attribution optional)](https://upload.wikimedia.org/wikipedia/commons/6/6f/20260720_183051_Xuetian_Road_Level_Crossing.webm) | CC0 |
| 道路（機車＋行人） | `a03_taipei_bridge_waterfall` | 台灣 台北橋機車道出口的「機車瀑布」，上班尖峰 | [June 2023 motorcycle waterfall on Taipei Bridge by rush hour commuters 06 by Tze Chiang Hao, CC BY-SA 4.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/a/a1/June_2023_motorcycle_waterfall_on_Taipei_Bridge_by_rush_hour_commuters_06.webm) | CC BY-SA 4.0 |
| 道路（機車＋行人） | `a04_raxaul_rail_crossing` | 印度 Raxaul 平交道，人和機車擠滿 | [A train stuck in traffic at a railway crossing in Raxaul, Bihar in 2018 by Saddam19, CC BY-SA 4.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/5/57/A_train_stuck_in_traffic_at_a_railway_crossing_in_Raxaul%2C_Bihar_in_2018.webm) | CC BY-SA 4.0 |
| 道路（機車＋行人） | `a05_bangalore_banashankari_circle` | 印度 班加羅爾 圓環，高處俯拍、固定機位，公車多 | [Banashankari traffic circle, Banashankari, Bangalore (2024) 01 by Gpkp, CC BY-SA 4.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/1/1e/Banashankari_traffic_circle%2C_Banashankari%2C_Bangalore_%282024%29_01.ogg) | CC BY-SA 4.0 |
| 道路（機車＋行人） | `a06_bengaluru_bellary_road` | 印度 班加羅爾 Bellary Road，天橋俯拍；嘟嘟車多（VOC 沒這類） | [Bellary Road, Bengaluru (2026) 01 by Gpkp, CC BY-SA 4.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/f/f3/Bellary_Road%2C_Bengaluru_%282026%29_01.webm) | CC BY-SA 4.0 |
| 道路（機車＋行人） | `a07_bangalore_rajkumar_road` | 印度 班加羅爾 Dr. Rajkumar Road，天橋俯拍、晴天 | [Dr. Rajkumar Road, Bangalore (2025) by Gpkp, CC BY-SA 4.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/c/c6/Dr._Rajkumar_Road%2C_Bangalore_%282025%29.webm) | CC BY-SA 4.0 |
| 道路（機車＋行人） | `a08_hue_motorbike_ride` | 越南 順化近郊，第一人稱騎機車、會晃 | [Veturado sur motorciklo apud Hue by RG72, CC BY-SA 4.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/3/39/Veturado_sur_motorciklo_apud_Hue.webm) | CC BY-SA 4.0 |
| 道路（機車＋行人） | `a09_chiangmai_intersection` | 泰國 清邁路口，固定機位，汽車為主 | [A intersection with traffic in Chiang Mai, Thailand by Amada44, CC BY-SA 4.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/2/21/A_intersection_with_traffic_in_Chiang_Mai%2C_Thailand.ogv) | CC BY-SA 4.0 |
| 道路（機車＋行人） | `a10_taiwan_motorcycles_city` | 台灣，天橋下路口，機車停等、行人穿越 | [Video by Allen Low on Pexels (attribution optional)](https://www.pexels.com/download/video/4740223/) | Pexels License |
| 道路（機車＋行人） | `a11_taipei_traffic_light` | 台灣 台北，機車停等紅燈、行人走斑馬線 | [Video by Levi Wedge on Pexels (attribution optional)](https://www.pexels.com/download/video/13269014/) | Pexels License |
| 道路（機車＋行人） | `a12_taipei_busy_street` | 台灣 台北路口 | [Video by Levi Wedge on Pexels (attribution optional)](https://www.pexels.com/download/video/13269015/) | Pexels License |
| 道路（機車＋行人） | `a13_bali_scooter_street` | 印尼 峇里島窄巷，迎面而來的機車；騎士的臉看得清楚 | [Video by Sergei Starostin on Pexels (attribution optional)](https://www.pexels.com/download/video/39547675/) | Pexels License |
| 其他 VOC 類別 | `b01_cargo_747_landing` | 747 貨機起降（機場沒標） | [Plane Spotting China airlines Cargo 747-400 B-2425 CK262 takeoff and landing (Typhoon delayed) by small Jia, CC BY 3.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/9/96/Plane_Spotting_China_airlines_Cargo_747-400_B-2425_CK262_takeoff_and_landing_%28Typhoon_delayed%29.webm) | CC BY 3.0 |
| 其他 VOC 類別 | `b02_kerala_boat_race` | 印度 喀拉拉邦 龍舟賽，船身很長、坐滿划手 | [NehruTrophyBoatRaceIruttukuthiAGradeFinal2025 by Varghesepunnamada, CC BY-SA 4.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/6/67/NehruTrophyBoatRaceIruttukuthiAGradeFinal2025.webm) | CC BY-SA 4.0 |
| 其他 VOC 類別 | `b03_tokyo_rock_dove` | 日本 東京，停在欄杆上的鴿子 | [Rock dove (Columba livia) in Tokyo by The Nature Box, CC BY-SA 3.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/a/a6/Rock_dove_%28Columba_livia%29_in_Tokyo.webm) | CC BY-SA 3.0 |
| 其他 VOC 類別 | `b04_estonia_cat_dog` | 愛沙尼亞，貓抓老鼠、狗（公開前先看過內容） | [Hiirepyyk kass koer 2022 by Sillerkiil, CC BY-SA 4.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/c/c9/Hiirepyyk_kass_koer_2022.webm) | CC BY-SA 4.0 |
| 其他 VOC 類別 | `b05_sf_dog_walking` | 美國 舊金山，人行道上遛狗，跟拍 | [Dog walking in the Mission in San Francisco by Sage Ross, CC BY-SA 3.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/6/67/Dog_walking_in_the_Mission_in_San_Francisco.ogv) | CC BY-SA 3.0 |
| 其他 VOC 類別 | `b06_horses_pasture` | 美國 伊利諾州，牧場奔馬，馬在畫面裡偏小 | [Horses running in a pasture by Field of Dreams, CC BY 3.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/a/af/Horses_running_in_a_pasture.webm) | CC BY 3.0 |
| 其他 VOC 類別 | `b07_jura_cow_herd` | 法國 Jura，牛群走在鄉間小路 | [Video: Benoit Prieur, CC0, via Wikimedia Commons (attribution optional)](https://upload.wikimedia.org/wikipedia/commons/a/ab/Passage_d%27un_troupeau_de_montb%C3%A9liardes_%C3%A0_Boissia_%28Jura%29_en_juillet_2018.webm) | CC0 |
| 其他 VOC 類別 | `b08_armenia_sheep_road` | 亞美尼亞 往 Sevan 湖的公路，羊群占滿馬路，後面跟著公車和汽車 | [Traffic chaos by gails_pictures (Flickr), CC BY 2.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/c/c8/Traffic_chaos.webm) | CC BY 2.0 |
| 其他 VOC 類別 | `b09_utrecht_bicycles` | 荷蘭 Utrecht，大量停放的腳踏車 | [Utrecht Centrumzijde 101802 by Quahadi, CC BY-SA 4.0, via Wikimedia Commons](https://upload.wikimedia.org/wikipedia/commons/6/68/Utrecht_Centrumzijde_101802.webm) | CC BY-SA 4.0 |
| 其他 VOC 類別 | `b10_living_room_tv_sofa` | 沒人的客廳，鏡頭慢慢移（地點沒標） | [Video by Curtis Adams on Pexels (attribution optional)](https://www.pexels.com/download/video/16641477/) | Pexels License |
| 其他 VOC 類別 | `b11_cafe_interior` | 泰國 普吉 咖啡店內部 | [Video by Peggy Anke on Pexels (attribution optional)](https://www.pexels.com/download/video/4349485/) | Pexels License |
| 其他 VOC 類別 | `b12_bar_bottles` | 酒吧酒櫃，瓶子小、背景逆光（地點沒標） | [Video by K (kelly) on Pexels (attribution optional)](https://www.pexels.com/download/video/2912918/) | Pexels License |
| vKITTI2 合成行車 | `vkitti2_scene01_clone` | Virtual KITTI 2 合成（Unity）行車畫面，Scene01 clone（原版、不加天氣）；有廂型車、卡車，沒有機車和行人 | [Virtual KITTI 2 - Cabon et al. 2020 - Naver Labs Europe](https://europe.naverlabs.com/research/computer-vision/proxy-virtual-worlds-vkitti-2/) | CC BY-NC-SA 3.0 |
| vKITTI2 合成行車 | `vkitti2_scene02_rain` | 同上，Scene02 雨天 | [Virtual KITTI 2 - Cabon et al. 2020 - Naver Labs Europe](https://europe.naverlabs.com/research/computer-vision/proxy-virtual-worlds-vkitti-2/) | CC BY-NC-SA 3.0 |
| vKITTI2 合成行車 | `vkitti2_scene06_fog` | 同上，Scene06 霧 | [Virtual KITTI 2 - Cabon et al. 2020 - Naver Labs Europe](https://europe.naverlabs.com/research/computer-vision/proxy-virtual-worlds-vkitti-2/) | CC BY-NC-SA 3.0 |
| vKITTI2 合成行車 | `vkitti2_scene18_sunset` | 同上，Scene18 夕陽 | [Virtual KITTI 2 - Cabon et al. 2020 - Naver Labs Europe](https://europe.naverlabs.com/research/computer-vision/proxy-virtual-worlds-vkitti-2/) | CC BY-NC-SA 3.0 |
| vKITTI2 合成行車 | `vkitti2_scene20_morning` | 同上，Scene20 早晨 | [Virtual KITTI 2 - Cabon et al. 2020 - Naver Labs Europe](https://europe.naverlabs.com/research/computer-vision/proxy-virtual-worlds-vkitti-2/) | CC BY-NC-SA 3.0 |

## 檔案

- `index.html`：demo 頁面，用瀏覽器打開即可；GitHub Pages 直接提供。
- `taiwan_dashcam/`、`road/`、`voc_other/`、`vkitti2/`：加了偵測框的影片。
