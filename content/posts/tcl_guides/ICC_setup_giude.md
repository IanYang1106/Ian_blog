---
title: "Synopsys ICC 設定檔教學文件"
date: 2025-07-07T16:20:00+08:00
draft: false
categories: ["ICC"]
tags: ["學習筆記", "ICC"]
---

# Synopsys ICC 設定檔教學文件

## 概述

本文件基於 Synopsys ICC（IC Compiler）的 `.synopsys_ICC.setup` 設定檔，說明如何配置 ICC 工具的各項參數。該設定檔採用 Tcl 格式，用於定義 ICC 工具的基本設定、路徑配置、以及各種編譯選項。

## 文件結構

### 1. 基本資訊設定

```tcl
# 版本資訊
set host_options -max_cores 16

# 設計者和公司資訊
set designer    {Devoting Engineers}
set company     {XXXXXX Inc.}
```

### 2. 搜尋路徑設定 (Search Path)

搜尋路徑定義了 ICC 工具尋找各種檔案的位置：

```tcl
set search_path "
    /app/cad/synopsys/CoreSynthesis/Q-2018.06-SP1/libraries/syn \
    /app/cad/synopsys/CoreSynthesis/Q-2018.06-SP1/minpower/syn \
    /app/lib/tsmc/artl6/TSMCHOME/digital/Front_End/timing_power_noise/NLDM/tcb016ndhwpwc_270a \
    /app/lib/tsmc/artl6/TSMCHOME/digital/Front_End/timing_power_noise/NLDM/tpd016nvwc_280a \
    ../00_IPfolder/hard_macro/sram/db/slow \
    ../00_IPfolder/hard_macro/sram/db/fast \
    ../00_IPfolder/hard_macro/IP_Name/db \
    ../00_IPfolder/hard_macro/IP_Name/db \
"
```

### 3. 函式庫設定

#### 符號函式庫 (Symbol Library)
```tcl
set symbol_library      {tsmc18.sdb}
```

#### 目標函式庫 (Target Library)
```tcl
set target_library      { tcb016ndhwpwc.db \
                          tpd016nvwc.db \
                        }
```

#### 合成函式庫 (Synthetic Library)
```tcl
set synthetic_library   { standard.sldb dw_foundation.sldb }
```

#### 連結函式庫 (Link Library)
```tcl
set link_library        {* \
                         tcb016ndhwpwc.db \
                         tpd016nvwc.db \
                         dw_foundation.sldb \
                         RA1SHD_1Kx8_M16_ss_1.62_125.0.db \
                         RA1SHD_3Kx16_M8_ss_1.62_125.0.db \
                         RF1SH_256x8_M4_ss_1p62v_125c.db \
                         RF1SH_656x40_M4_ss_1p62v_125c.db \
                         RF1_128x10_W2M4_ss_1p62v_125c.db \
                         RF1_128x8_W1M4_ss_1p62v_125c.db \
                         RF1_512x64_W8M4_ss_1p62v_125c.db \
                         RF1_64x4_W1M4_ss_1p62v_125c.db \
                         RF1_832x32_W1M8_ss_1p62v_125c.db \
                         RODSHD_OSD_Font_slow.db \
                        }
```

### 4. 不使用清單 (Don't Use Lists)

定義不希望工具使用的特定元件：

```tcl
set_dont_use [list "tcb016ndhwpwc/BUFT*" \
                   "tcb016ndhwpwc/ANTENNA" \
                   "tcb016ndhwpwc/BHD*" \
                   "tcb016ndhwpwc/CK*" \
                   "tcb016ndhwpwc/DCA*" \
                   "tcb016ndhwpwc/DEL*" \
                   "tcb016ndhwpwc/LH*" \
                   "tcb016ndhwpwc/LM*" \
                   "tcb016ndhwpwc/G*" \
                   "tcb016ndhwpwc/CKLM" \
                   "tcb016ndhwpwc/TIEH*" \
                   "tcb016ndhwpwc/TIEL*" \
                   "tcb016ndhwpwc/*D8H8RMP" \
                   "tcb016ndhwpwc/*D1H8RMP" \
                   "tcb016ndhwpwc/*D1P5H8RMP" \
                   "tcb016ndhwpwc/SOFBND" \
             ]
```

### 5. 技術檔案設定

```tcl
set_mw_technology_file -technology /user/cadwayi/lib/tsmc/artl6/TSMCHOME/digital/Back_End/milkyway/tcb016ndhwpwc_270a/techfiles/tsmc016_5lm.tf
```

### 6. 最小函式庫檔案設定

```tcl
set MIN_LIBRARY_FILES  "tcb016ndhwpwc.db tcb016ndhwpbc.db \
                        tpd016nvwc.db tpd016nvbc.db \
                        RA1SHD_1Kx8_M16_ss_1.62_125.0.db RA1SHD_1Kx8_M16_ff_1.98_-40.0.db \
                        RA1SHD_3Kx16_M8_ss_1.62_125.0.db RA1SHD_3Kx16_M8_ff_1.98_-40.0.db \
                        RF1SH_256x8_M4_ss_1p62v_125c.db RF1SH_256x8_M4_ff_1p98v_m40c.db \
                        RF1SH_656x40_M4_ss_1p62v_125c.db RF1SH_656x40_M4_ff_1p98v_m40c.db \
                        RF1_128x10_W2M4_ss_1p62v_125c.db RF1_128x10_W2M4_ff_1p98v_m40c.db \
                        RF1_128x8_W1M4_ss_1p62v_125c.db RF1_128x8_W1M4_ff_1p98v_m40c.db \
                        RF1_512x64_W8M4_ss_1p62v_125c.db RF1_512x64_W8M4_ff_1p98v_m40c.db \
                        RF1_64x4_W1M4_ss_1p62v_125c.db RF1_64x4_W1M4_ff_1p98v_m40c.db \
                        RF1_832x32_W1M8_ss_1p62v_125c.db RF1_832x32_W1M8_ff_1p98v_m40c.db \
                        RODSHD_OSD_Font_slow.db RODSHD_OSD_Font_fast.db \
                        "
```

### 7. Verilog 輸出設定

```tcl
# 保留未連接的腳位
set verilogout_show_unconnected_pins
set hwlin_auto_save_templates             {true}
                                          {true}

# 寫入網表設定
set write_name_nets_same_as_ports         {true}
set verilogout_no_tri                     {true}

# 編輯輸出設定
set edifout_netlist_only                  {true}
set edifout_power_net_name                {VCC}
set edifout_ground_net_name               {GND}
set edifout_no_array                      {true}
set edifout_power_and_ground_representation {net}
```

### 8. 匯流排命名設定

```tcl
set bus_dimension_separator_style         {[]}
set bus_naming_style                      {%s_%d}
set bus_range_separator_style             {:}
```

### 9. 閘級網表規則

```tcl
#define name_rules GateLevelNetlist -special_verilog
#define name_rules GateLevelNetlist -allowed {a-z A-Z 0-9 _[]} -type_port
#define name_rules GateLevelNetlist -allowed {a-z A-Z 0-9 _} -type_net
#define name_rules GateLevelNetlist -allowed {a-z A-Z 0-9 _} -type_cell
#define name_rules GateLevelNetlist -first_restricted {_0-9 []} -last_restricted {[]} -equal_ports_nets -inout_ports_equal_nets -repl
```

### 10. 編譯選項

```tcl
set auto_wire_load_selection              {TRUE}
set compile_use_fast_delay_model          {TRUE}

# 保留子設計介面
set compile_preserve_subdesign_interfaces {true}
# 禁用階層反向器優化
set compile_disable_hierarchical_inverter_opt {true}

# 模板命名樣式
set template_naming_style "%s_%d"
set template_parameter_style "%s%d"
# %s 代表模板名稱，%d 代表參數值
set template_parameter_style "%d"
set template_separator_style "_"

# 唯一命名樣式
set uniquify_naming_style "%s_UH_%d"
```

## 使用說明

1. **檔案位置**：將此設定檔放在用戶家目錄下，命名為 `.synopsys_ICC.setup`

2. **路徑配置**：根據您的環境修改 `search_path` 中的路徑

3. **函式庫配置**：確保所有引用的 `.db` 檔案都存在於指定路徑中

4. **技術檔案**：確認技術檔案路徑正確

5. **參數調整**：根據設計需求調整各項參數

## 注意事項

- 確保所有路徑都正確且檔案存在
- 檢查函式庫版本兼容性
- 根據目標製程技術調整參數
- 定期備份設定檔

## 常見問題

1. **找不到函式庫檔案**：檢查 `search_path` 設定
2. **編譯錯誤**：確認目標函式庫和連結函式庫設定正確
3. **效能問題**：調整 `max_cores` 參數以利用多核心處理器

---

**檔案格式**：本文件已為 Markdown 格式，可直接儲存為 `.md` 檔案使用。