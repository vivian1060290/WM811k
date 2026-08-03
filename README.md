# 以影像辨識技術實作半導體晶圓篩選系統之研究

本專案為國立虎尾科技大學資訊工程系之實務專題成果。專案基於 **WM811K 晶圓圖資料集**，針對半導體製程中的晶圓缺陷進行自動化偵測與分類。系統採用 **Two-stage（雙階段）分類架構**，並結合 **貪婪演算法（Greedy-based）** 進行資料增強篩選與**集成學習（Ensemble Learning）** 多數決投票，有效解決資料集嚴重的類別不平衡問題。

## 📌 功能特色

- **雙階段分類架構 (Two-stage Classification)**：
  - **Stage 1**：判斷晶圓是否有缺陷（None vs Defect 二元分類）。
  - **Stage 2**：針對缺陷晶圓進行 8 種缺陷型態分類（Center, Donut, Edge-loc, Edge-ring, Loc, Random, Scratch, Near-Full）。
- **智慧資料增強搜尋**：使用 Greedy 策略動態篩選出最佳資料增強組合（如 Sharpen, Noise, Erosion 等）。
- **類別權重補償**：利用各類別樣本比例之倒數作為損失函數權重，改善少數缺陷類別的辨識效果。
- **高效集成學習**：整合 VGG11-BN、DenseNet121 與 Swin Transformer 三大模型，並透過多數決投票大幅提升整體 F1 分數至 **0.89**。

## 💻 實驗環境

專案主要於本機端執行，核心環境配置如下：

| 類別 | 項目 | 規格 |
| --- | --- | --- |
| **硬體** | CPU / GPU | Intel i7-10700 / NVIDIA GeForce RTX 4060 |
| | 記憶體 | 64GB |
| **軟體** | 作業系統 | Windows 11 |
| | 程式語言與平台 | Python (Anaconda3, Jupyter Notebook) |
| | 深度學習框架 | PyTorch |

## 📁 檔案結構

```text
├── data/
│   └── WM811K.p1k              # 原始晶圓資料集
├── src/
│   ├── preprocess.py          # LUT顏色對應、補正方形、Resize 64x64
│   ├── dataset.py             # 類別權重計算與資料分割 (70%:15%:15%)
│   ├── train_single.py        # 單一模型訓練與 Greedy 資料增強搜尋
│   └── ensemble.py            # 集成學習多數決投票流程
├── notebooks/
│   └── main_experiment.ipynb  # 實驗操作與結果分析手冊
└── README.md
```

## 🚀 快速上手

### 1. 安裝環境依賴
請確保已安裝 Anaconda，並透過環境設定檔或指令安裝 PyTorch：
```bash
pip install torch torchvision timm jupyter
```

### 2. 資料前處理
將原始的 `WM811K.pkl` 數值矩陣透過 LUT 查表轉換為彩色影像（0:背景/黑、1:正常/綠、2:缺陷/紅），並縮放至 64x64 像素：
```bash
python src/preprocess.py
```

### 3. 執行單一模型訓練
訓練過程中會自動透過 Greedy 策略尋找最合適的資料增強組合：
```bash
python src/train_single.py --model vgg11_bn --batch_size 64
python src/train_single.py --model densenet121 --loss_weight_strength 0.5
python src/train_single.py --model swin_tiny --lr 5e-5
```

### 4. 執行集成學習預測
結合三大最佳單一模型進行多數決投票預測：
```bash
python src/ensemble.py
```

## 📊 實驗結果

在完整測試集下，各模型與集成學習的平均 F1 分數表現對比：

| 模型 / 方法 | 主要超參數設定 | 最佳平均 F1 分數 | 說明 |
| --- | --- | --- | --- |
| **VGG11-BN** | Batch Size = 64 | **0.83** | 推論時間最快 (91.96秒) |
| **DenseNet121** | Loss Weight Strength = 0.5 | **0.82** | 調整初始特徵擷取層以保留細節 |
| **Swin Transformer** | Learning Rate = 5e-5 | **0.82** | 影像 Resize 至 224x224 微調 |
| 🏆 **集成學習** | 結合上述三組最佳模型 | ⭐ **0.89** | 採多數決投票，具顯著模型互補性 |

> **註**：集成學習在特定困難類別（如 **Loc 局部缺陷**與 **Scratch 刮痕缺陷**）的改善效果最為顯著，其子集 F1 分數分別可由單一模型的 0.86 與 0.76 提升至 **0.91** 與 **0.90**。

## 👥 組員貢獻 (工作分配)

- **范芷紜** (學號: 41243104)：負責專題全面性技術研究、資料前處理、模型設計、實驗測試與論文撰寫 (100%)。
- **郭俞汎** (學號: 41043104)：參與開會討論、單一模型設計與測試。
- **指導教授**：黃建宏 教授。
