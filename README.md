# xarm-motion-controller-touchdesigner <!-- omit in toc -->
<img src="https://img.shields.io/badge/Python-3.11 or Later-blue?&logo=Python"> <img src="https://img.shields.io/badge/TouchDesigner-2023-blue?"> <img src="https://img.shields.io/badge/License-BSD 3 Clause-green">

[UFACTORY Inc.](https://www.ufactory.us/)が販売しているxArmをTouchDesignerから制御するためのサンプルです。

# Table Of Contents <!-- omit in toc -->
<details>
<summary>Details</summary>

- [Copyright](#copyright)
- [Environment](#environment)
- [Installation](#installation)
  - [xArm Python SDK](#xarm-python-sdk)
  - [TouchDesigner](#touchdesigner)
- [Usage](#usage)
  - [xArmの設置](#xarmの設置)
  - [xArmの初期位置設定](#xarmの初期位置設定)
  - [ライブラリの準備](#ライブラリの準備)
  - [xArmを操作する](#xarmを操作する)
- [Description](#description)
  - [xArmへのデータ送信](#xarmへのデータ送信)
    - [メソッド](#メソッド)
    - [送信するデータについて](#送信するデータについて)
    - [送信する配列の順番](#送信する配列の順番)
    - [安全に操作するための設定](#安全に操作するための設定)
      - [動作範囲の制限](#動作範囲の制限)
      - [急激な動きに対する制限](#急激な動きに対する制限)
- [References](#references)
- [Troubleshooting](#troubleshooting)
- [免責事項](#免責事項)
- [Versions](#versions)
- [Author](#author)
- [License](#license)
</details>


# Copyright 
このプログラムは[UFACTORY Inc.](https://www.ufactory.us/)が開発、公開している[xArm-Python-SDK](https://github.com/xArm-Developer/xArm-Python-SDK.git)を利用して作成しています。

This program is created using the [xArm-Python-SDK](https://github.com/xArm-Developer/xArm-Python-SDK.git) developed and published by [UFACTORY Inc.](https://www.ufactory.us/)

Copyright (c) 2018, UFACTORY Inc. All Rights Reserved.



# Environment
- Windows 11
- TouchDesigner 2023
- Python 3.11 or later
- xArm 7
- xArm Gripper
- xArm Python SDK 1.12.1


# Installation
## xArm Python SDK
[公式の手順](https://github.com/xArm-Developer/xArm-Python-SDK?tab=readme-ov-file#installation)に従ってインストールしてください

## TouchDesigner
事前にTouchDesignerがインストールされており、Pythonの外部モジュールを読み込むパスが正しく通っていることを確認してください。


# Usage
## xArmの設置
1. まずはUFACTORY Studioを用いて、xArmに接続できることを確認してください
1. SettingsのMountingが「Floor」になっていることを確認してください
    - 本プログラムではFloorになっていることを想定しています
    - それ以外のMountingの場合は予期せぬ動作になる場合がありますので注意してください

## xArmの初期位置設定
1. UFACTORY StudioのSettingsを開く
1. Motionを選択する
1. Initial PositionのSetを押下し、任意の位置にxArmを調整する
1. OKを押下すると、その位置を初期位置として設定する

本プログラムでは、初期位置に移動させる動作を行うので、事前に任意の値を設定してください。
デフォルトでは全て0になります。

## ライブラリの準備
TouchDesignerのPython外部モジュール読み込みを利用すると、うまく読み込めない場合があります。
そのため、やや強引ですが、toeファイルのあるディレクトリにxArm Python SDKを配置して、そのディレクトリをモジュール読み込みパスに追加する手法を用いています。

以下のフォルダ構成になるように配置してください (このリポジトリにSDKは含まれていません)。
お使いのPythonのsite-packagesディレクトリにインストールされたライブラリをコピーして配置します。
xArm-Python-SDK-1.XX.Xの部分はご自身の環境にインストールしたバージョンになります。

<pre>
xarm-motion-controller-touchdesigner/
├── xArmController.toe
└── Lib/
    └── site-packages/
        └── xArm-Python-SDK-1.XX.X
</pre>


## xArmを操作する
<span style="font-size: 120%; color: red;">**【注意】 まずはUFACTORY Studioの「Simulated Robot」で動作を確認してください**</span>

いきなり実際のロボットを動かす場合、万が一予期せぬ値が送信された場合に怪我や故障につながる恐れがあります。
必ずSimulated Robotで確認してから実際のロボットを動かしてください。

1. xArmController.toeを実行します
2. hostにxArmのIPアドレスを指定します
3. Connect xArmボタンを押下します
    - xArm Gripperを使用する場合はトグルをOnにします
    - 接続中はブロッキングします
4. 接続が成功したらMove Up -> Initial Positionボタンを押下します
    - デフォルトでは現在位置から50mm上に上がった後、UFACTORY Studioで設定した初期位置に移動します
5. Send dataをOnにすると、xArmへの値の送信を開始します
    - スライダーを用いて値を送信するサンプルを用意しています
    - 「container_sample_transform」を右クリックし、「View...」をクリックすることで別ウィンドウとしてスライダーを表示します
    - 値を少しずつ変化させて、xArmが動くことを確認してください
6. Send dataをOffにすると、xArmへの値の送信を停止します
7. Disconnect xArmボタンを押下すると接続を解除します


# Description
## xArmへのデータ送信
### メソッド
xArmの位置と回転制御には ``set_servo_cartesian`` メソッドを使用しています。

```python
from xarm.wrapper import XArmAPI

host = 'localhost'
arm = XArmAPI(host)
arm.connect()

arm.motion_enable(enable=True)
arm.set_mode(1)
arm.set_state(0)
time.sleep(0.1)

mvpose = [200, 0, 200, 180, 0, 0]
arm.set_servo_cartesian(mvpose)
```

### 送信するデータについて
本プログラムでは、``current_relative_transform``オペレータから受け取った値に**初期位置を加算する**処理を行っています。
そのため、このオペレータに流す値は、**初期位置からの相対位置**にしてください。

例えば、このオペレータの値が全て0の場合は、初期位置から動きません。
xを30にした場合、**初期位置から+x方向に30mm移動します**。

### 送信する配列の順番
配列の順番を以下の通り指定しています。
かっこは単位です。

```python
[x(mm), y(mm), z(mm), roll(deg), pitch(deg), yaw(deg)]
```

### 安全に操作するための設定
危険な動作を避けるために、デフォルトで2種類の制限を設けています。

#### 動作範囲の制限
``min_transform``、``max_transform`` オペレータで指定した範囲を超える値を送信しようとした場合、この値に制限されます。
ご自身の環境に合わせて調整してください。
この値はConnect xArmをした段階で読み込まれます。そのため、動作中に動的に変更することはできません。

例えば、最大x座標を250と指定して、270を送信しようとした場合、送信されるのは250になります。

#### 急激な動きに対する制限
1フレーム前の位置と回転と、現在のフレームの位置と回転との差分が、``moving_diff_limit`` を超える場合、強制的にSend dataをOffにします。
この場合、Move Up -> Initial Positionボタンを押下し、初期位置に移動させてから、再度Send dataをOnにしないとデータの送信を再開しません。

デフォルトで位置は各軸50mm、回転は各軸15度を超えた場合、送信を停止します。
ご自身の環境や実験設計に合わせて調整してください。

高いフレームレートで動作させている場合、急激な動きの検知が正しく動作しない場合があります。
そのため、**本プログラムの安全のための制限を過信せず、ご自身の実験プログラムにおいても安全に扱うための処理を含めることを強くおすすめします**。


# References
- [xArm-Python-SDK](https://github.com/xArm-Developer/xArm-Python-SDK.git)


# Troubleshooting

# 免責事項
- 本プログラムを用いたことによる怪我や故障、損害等について、著者は一切の責任を負いかねますのでご了承ください
- その他、いかなる損害や紛争についても著者は責任を負いません

# Versions
- 1.0.0: 2025/8/8


# Author
- Takayoshi Hagiwara
    - National Institute of Technology (KOSEN), Nagano College
    - Graduate School of Media Design, Keio University
    - Toyohashi University of Technology


# License
- BSD 3-Clause License