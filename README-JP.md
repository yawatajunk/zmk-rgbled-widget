# RGB LED を使った LED インジケーター

> [!IMPORTANT]
> このモジュールは、ZMK のバージョンに対応したバージョニング方式を使っています。
> 原則として、`main` ブランチは ZMK の `main` との互換性を対象にしています。
>
> **ZMK の最新リリース版 (`v0.3` など) でビルドに失敗する場合は、`west.yml` の `zmk-rgbled-widget` に対して [対応するリビジョン](#インストール) を使ってください。**

これは、3 本の個別制御される LED で構成された RGB LED を使って状態表示を行う [ZMK module](https://zmk.dev/docs/features/modules) です。
主に、バッテリー残量と BLE 接続状態をミニマルに表示するために使います。

このリポジトリは、元の [caksoylar/zmk-rgbled-widget](https://github.com/caksoylar/zmk-rgbled-widget) をフォークしたものです。
ここで使っているフォークは [yawatajunk/zmk-rgbled-widget](https://github.com/yawatajunk/zmk-rgbled-widget) で、元の機能を維持したまま、`pwm-leds` を使うボード向けに PWM による LED 調光機能を追加しています。

> [!WARNING]
> この fork では、`v0.3-branch` はテスト済みです。
> fork の `main` ブランチはまだテストしていません。

## フォークとライセンス

- 元プロジェクト: [caksoylar/zmk-rgbled-widget](https://github.com/caksoylar/zmk-rgbled-widget)
- この構成で使うフォーク: [yawatajunk/zmk-rgbled-widget](https://github.com/yawatajunk/zmk-rgbled-widget)
- ライセンス: MIT

このフォークは、元プロジェクトと同じ MIT ライセンスで配布されています。再配布や改変を行う場合は、このリポジトリに含まれている [LICENSE](LICENSE) の著作権表示と MIT ライセンス文を保持してください。

## 機能

- `gpio-leds` を使う従来の GPIO ベース RGB LED をサポート
- `pwm-leds` を使う PWM ベース RGB LED をサポート
- `CONFIG_RGBLED_WIDGET_BRIGHTNESS` による明るさ設定を追加

<details>
  <summary>短いデモ動画</summary>
  以下の動画では、電源投入、プロファイル切り替え、電源オフ時の動作を簡単に確認できます。

  https://github.com/caksoylar/zmk-rgbled-widget/assets/7876996/cfd89dd1-ff24-4a33-8563-2fdad2a828d4
</details>

### バッテリー状態

- 起動時に、バッテリー残量に応じて 🟢 / 🟡 / 🔴 を点滅表示します。しきい値は `CONFIG_RGBLED_WIDGET_BATTERY_LEVEL_HIGH` と `CONFIG_RGBLED_WIDGET_BATTERY_LEVEL_LOW` で設定します。
  - 分割キーボードでの表示については [分割キーボードでのバッテリー表示](#分割キーボードでのバッテリー表示) を参照してください。
- バッテリー残量がクリティカルしきい値 `CONFIG_RGBLED_WIDGET_BATTERY_LEVEL_CRITICAL` を下回ると、残量変化のたびに 🔴 を点滅表示します。

### 接続状態

- 起動時のバッテリー表示のあと、および BT プロファイル切り替え時に、接続済みは 🔵、広告中は 🟡、未接続は 🔴 で点滅表示します。分割キーボードではセントラル側のみが対象です。
  - `CONFIG_RGBLED_WIDGET_CONN_SHOW_USB` を有効にすると、BLE 状態の代わりに USB 優先時をシアンで点滅表示します。
- 分割キーボードのペリフェラル側では、接続済みを 🔵、未接続を 🔴 で点滅表示します。

### レイヤー状態

最上位のアクティブレイヤーを表示する方法は、以下のいずれかを選べます。デフォルトでは無効です。

- `CONFIG_RGBLED_WIDGET_SHOW_LAYER_CHANGE` を有効にすると、レイヤーが有効化されるたびに、レイヤー番号のゼロ始まりインデックスを N として、シアンを N 回点滅表示します。
- `CONFIG_RGBLED_WIDGET_SHOW_LAYER_COLORS` を有効にすると、各レイヤーに色を割り当て、そのレイヤーが最上位アクティブな間はその色を点灯し続けます。

レイヤー情報は分割キーボードのペリフェラル側では把握できないため、これらの表示はセントラル側でのみ有効です。

> [!TIP]
> バッテリー状態や接続状態を任意のタイミングで表示するには、下の [任意のタイミングで状態を表示する](#任意のタイミングで状態を表示する) も参照してください。

## インストール

まず、このモジュールを `config/west.yml` の `projects` に追加します。

```yaml west.yml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: v0.3           # 使用中の ZMK バージョン
      import: app/west.yml
    - name: zmk-rgbled-widget  # <-- 追加
      url: https://github.com/yawatajunk/zmk-rgbled-widget
      revision: v0.3-branch    # ZMK のバージョンと合わせること
  self:
    path: config
```

ローカルビルドを含む詳しい手順は、ZMK 公式ドキュメントの [building with modules](https://zmk.dev/docs/features/modules#building-with-modules) を参照してください。

`rgbled_adapter` シールドに対応しているボードを使う場合、たとえば Xiao BLE では、`build.yaml` に追加シールドとして `rgbled_adapter` を加えるだけで使えます。

```yaml build.yaml
---
include:
  - board: seeeduino_xiao_ble
    shield: hummingbird rgbled_adapter
```

それ以外のキーボードについては、下の [カスタム boards/shields での対応追加](#カスタム-boardsshields-での対応追加) を参照してください。

## 任意のタイミングで状態を表示する

このモジュールは、任意のタイミングでバッテリー状態や接続状態を表示するためのキー map [behaviors](https://zmk.dev/docs/keymaps/behaviors) も提供しています。

```dts
#include <behaviors/rgbled_widget.dtsi>  // behavior を使うために必要

/ {
    keymap {
        ...
        some_layer {
            bindings = <
                ...
                &ind_bat  // バッテリー残量を表示
                &ind_con  // 接続状態を表示
                ...
            >;
        };
    };
};
```

対応するキーやコンボを押して behavior を呼び出すと、LED 表示が実行されます。
分割キーボードでは全パーツで動作するため、有効化したらすべてのパーツにファームウェアを書き込んでください。

> [!NOTE]
> 分割キーボードで、すべてのコントローラがこのウィジェットに対応していない場合でも、behavior 自体は使えます。
> `rgbled_adapter` シールドを使う、または `CONFIG_RGBLED_WIDGET` を有効にするのは、このウィジェットに対応しているパーツだけにしてください。

## 分割キーボードでのバッテリー表示

分割キーボードでは、デフォルトでは各パーツが自分自身のバッテリー状態を 1 回だけ点滅表示します。
ただし、ドングル構成やペリフェラル側に RGB LED widget がない構成では、セントラル側がペリフェラル側のバッテリー残量もまとめて表示したい場合があります。
その場合は、以下のいずれかを有効にします。

- `CONFIG_RGBLED_WIDGET_BATTERY_SHOW_PERIPHERALS`: 自分自身のあとに、ペリフェラル側の残量も順に点滅表示
- `CONFIG_RGBLED_WIDGET_BATTERY_SHOW_ONLY_PERIPHERALS`: ペリフェラル側の残量のみを順に点滅表示

この 2 つは、分割キーボードのセントラル側でのみ有効です。
ペリフェラル側の表示順は、最初にペアリングした順序で決まります。
パーツが切断されている場合は、マゼンタ / 紫色の点滅で表示されます。

## 設定詳細

<details>
<summary>一般</summary>

| Name                               | 説明                                 | Default |
| ---------------------------------- | ------------------------------------ | ------- |
| `CONFIG_RGBLED_WIDGET_INTERVAL_MS` | 2 回の点滅の間に待つ最小時間 (ms)    | 500     |
| `CONFIG_RGBLED_WIDGET_BRIGHTNESS`  | `pwm-leds` 使用時の明るさの割合      | 100     |

</details>

<details>
<summary>バッテリー関連</summary>

| Name                                          | 説明                                                       | Default       |
| --------------------------------------------- | ---------------------------------------------------------- | ------------- |
| `CONFIG_RGBLED_WIDGET_BATTERY_BLINK_MS`       | バッテリー残量表示の点滅時間 (ms)                          | 2000          |
| `CONFIG_RGBLED_WIDGET_BATTERY_LEVEL_HIGH`     | 高バッテリー残量のしきい値 (%)                             | 80            |
| `CONFIG_RGBLED_WIDGET_BATTERY_LEVEL_LOW`      | 低バッテリー残量のしきい値 (%)                             | 20            |
| `CONFIG_RGBLED_WIDGET_BATTERY_LEVEL_CRITICAL` | クリティカル残量のしきい値。下回ると定期点滅対象になる     | 5             |
| `CONFIG_RGBLED_WIDGET_BATTERY_LEVEL_CRITICAL` | クリティカル残量のしきい値。下回ると定期点滅対象になる     | 5             |
| `CONFIG_RGBLED_WIDGET_BATTERY_COLOR_HIGH`     | 高残量時 (`LEVEL_HIGH` より上) の色                        | Green (`2`)   |
| `CONFIG_RGBLED_WIDGET_BATTERY_COLOR_MEDIUM`   | 中残量時 (`LEVEL_LOW` 以上 `LEVEL_HIGH` 未満) の色         | Yellow (`3`)  |
| `CONFIG_RGBLED_WIDGET_BATTERY_COLOR_LOW`      | 低残量時 (`LEVEL_LOW` 未満) の色                           | Red (`1`)     |
| `CONFIG_RGBLED_WIDGET_BATTERY_COLOR_CRITICAL` | クリティカル残量時 (`LEVEL_CRITICAL` 未満) の色            | Red (`1`)     |
| `CONFIG_RGBLED_WIDGET_BATTERY_COLOR_MISSING`  | バッテリー未検出またはペリフェラル側未接続時の色                   | Magenta (`5`) |

以下のうち有効にできるのは 1 つだけです。
デフォルト以外の 2 つ目と 3 つ目は、分割キーボードのセントラル側でのみ動作します。

| Name                                                 | 説明                                           | Default |
| ---------------------------------------------------- | ---------------------------------------------- | ------- |
| `CONFIG_RGBLED_WIDGET_BATTERY_SHOW_SELF`             | 自分自身のバッテリー残量のみ表示               | `n`     |
| `CONFIG_RGBLED_WIDGET_BATTERY_SHOW_PERIPHERALS`      | 分割セントラル側で、ペリフェラル側の残量も表示               | `n`     |
| `CONFIG_RGBLED_WIDGET_BATTERY_SHOW_ONLY_PERIPHERALS` | 分割セントラル側で、ペリフェラル側の残量のみ表示             | `n`     |

</details>

<details>
<summary>接続関連</summary>

| Name                                           | 説明                                               | Default      |
| ---------------------------------------------- | -------------------------------------------------- | ------------ |
| `CONFIG_RGBLED_WIDGET_CONN_BLINK_MS`           | BLE 接続状態表示の点滅時間 (ms)                    | 1000         |
| `CONFIG_RGBLED_WIDGET_CONN_SHOW_USB`           | USB が優先のとき BLE 状態の代わりに USB を表示     | `n`          |
| `CONFIG_RGBLED_WIDGET_CONN_COLOR_CONNECTED`    | BLE 接続済み時の色                                 | Blue (`4`)   |
| `CONFIG_RGBLED_WIDGET_CONN_COLOR_ADVERTISING`  | BLE 広告中時の色                                   | Yellow (`3`) |
| `CONFIG_RGBLED_WIDGET_CONN_COLOR_DISCONNECTED` | BLE 未接続時の色                                   | Red (`1`)    |
| `CONFIG_RGBLED_WIDGET_CONN_COLOR_USB`          | USB エンドポイント有効時の色                       | Cyan (`6`)   |

</details>

<details>
<summary>レイヤー関連</summary>

レイヤー表示は、非キーボード、または分割キーボードのセントラル側でのみ動作します。

以下は、点滅回数で示すレイヤー表示の設定です。

| Name                                     | 説明                                                             | Default    |
| ---------------------------------------- | ---------------------------------------------------------------- | ---------- |
| `CONFIG_RGBLED_WIDGET_SHOW_LAYER_CHANGE` | レイヤー変更時に、点滅シーケンスで最上位アクティブレイヤーを表示 | `n`        |
| `CONFIG_RGBLED_WIDGET_LAYER_BLINK_MS`    | レイヤー表示の点滅時間と待機時間                                 | 100        |
| `CONFIG_RGBLED_WIDGET_LAYER_COLOR`       | レイヤー表示に使う色                                             | Cyan (`6`) |
| `CONFIG_RGBLED_WIDGET_LAYER_DEBOUNCE_MS` | レイヤー変更後に表示するまで待つ時間                             | 100        |

以下は、色で示すレイヤー表示の設定です。

| Name                                     | 説明                                                 | Default       |
| ---------------------------------------- | ---------------------------------------------------- | ------------- |
| `CONFIG_RGBLED_WIDGET_SHOW_LAYER_COLORS` | 各レイヤーに対応する色を点灯して最上位レイヤーを表示 | `n`           |
| `CONFIG_RGBLED_WIDGET_LAYER_0_COLOR`     | ベースレイヤーの色                                   | Black (`0`)   |
| `CONFIG_RGBLED_WIDGET_LAYER_1_COLOR`     | レイヤー 1 の色                                      | Red (`1`)     |
| `CONFIG_RGBLED_WIDGET_LAYER_2_COLOR`     | レイヤー 2 の色                                      | Green (`2`)   |
| `CONFIG_RGBLED_WIDGET_LAYER_3_COLOR`     | レイヤー 3 の色                                      | Yellow (`3`)  |
| `CONFIG_RGBLED_WIDGET_LAYER_4_COLOR`     | レイヤー 4 の色                                      | Blue (`4`)    |
| `CONFIG_RGBLED_WIDGET_LAYER_5_COLOR`     | レイヤー 5 の色                                      | Magenta (`5`) |
| `CONFIG_RGBLED_WIDGET_LAYER_6_COLOR`     | レイヤー 6 の色                                      | Cyan (`6`)    |
| `CONFIG_RGBLED_WIDGET_LAYER_7_COLOR`     | レイヤー 7 の色                                      | White (`7`)   |
| `CONFIG_RGBLED_WIDGET_LAYER_xx_COLOR`    | レイヤー xx の色。xx を対象レイヤー番号に置き換える  | Black (`0`)   |

</details>

<details>
<summary>色値の対応表</summary>
色設定には以下の整数値を使います。

| 色           | 値    |
| ------------ | ----- |
| Black (なし) | `0`   |
| Red          | `1`   |
| Green        | `2`   |
| Yellow       | `3`   |
| Blue         | `4`   |
| Magenta      | `5`   |
| Cyan         | `6`   |
| White        | `7`   |

</details>

これらの設定は、たとえば `config/hummingbird.conf` のようなキーボード用 `.conf` ファイルに追加して変更できます。

```ini
CONFIG_RGBLED_WIDGET_INTERVAL_MS=250
CONFIG_RGBLED_WIDGET_BRIGHTNESS=25
CONFIG_RGBLED_WIDGET_BATTERY_LEVEL_HIGH=50
CONFIG_RGBLED_WIDGET_BATTERY_LEVEL_CRITICAL=10
```

PWM 対応フォークを使う場合、`.conf` ファイルでの主な明るさ設定は `CONFIG_RGBLED_WIDGET_BRIGHTNESS` です。目安は以下です。

- `10` から `20`: かなり暗め
- `25` から `40`: 中程度
- `100`: 最大

## カスタム boards/shields での対応追加

このウィジェットを使うには、3 個の GPIO 制御 LED が必要です。スマート LED ではなく、理想的には赤・緑・青の 3 色 LED を想定しています。
ボードまたはシールド側にこれらの LED 定義があれば、対応する `aliases` を RGB LED ノードラベルへ向けるだけで利用できます。

nRF52840 コントローラで、VCC と各 GPIO の間に LED を接続した例は以下です。

```dts
/ {
    aliases {
        led-red = &led0;
        led-green = &led1;
        led-blue = &led2;
    };

    leds {
        compatible = "gpio-leds";
        status = "okay";
        led0: led_0 {
            gpios = <&gpio0 26 GPIO_ACTIVE_LOW>;  // red LED, connected to P0.26
        };
        led1: led_1 {
            gpios = <&gpio0 30 GPIO_ACTIVE_LOW>;  // green LED, connected to P0.30
        };
        led2: led_2 {
            gpios = <&gpio0 6 GPIO_ACTIVE_LOW>;  // blue LED, connected to P0.06
        };
    };
};
```

LED が GPIO と GND の間に配線されている場合は、代わりに `GPIO_ACTIVE_HIGH` を使ってください。

### PWM LED 対応

このフォークは、`pwm-leds` を使う PWM 制御 RGB LED もサポートしています。
ボード側で RGB 各色が PWM コントローラ経由で公開されていて、明るさを調整したい場合はこちらを使います。

この構成では、`.conf` ファイルで PWM ドライバとウィジェットの明るさを有効化し、ボードまたはシールドの `.overlay` / `.dtsi` で PWM ピンと LED ノードを定義します。

`.conf` 設定例:

```ini
CONFIG_PWM=y
CONFIG_LED_PWM=y
CONFIG_RGBLED_WIDGET=y
CONFIG_RGBLED_WIDGET_BRIGHTNESS=25
```

devicetree 設定例:

```dts
#include <dt-bindings/pinctrl/nrf-pinctrl.h>
#include <dt-bindings/pwm/pwm.h>

/ {
  aliases {
    led-red = &red_pwm_led;
    led-green = &green_pwm_led;
    led-blue = &blue_pwm_led;
  };
};

&pinctrl {
  pwm0_default: pwm0_default {
    group1 {
      psels = <NRF_PSEL(PWM_OUT0, 0, 26)>, <NRF_PSEL(PWM_OUT1, 0, 30)>,
          <NRF_PSEL(PWM_OUT2, 0, 6)>;
    };
  };

  pwm0_sleep: pwm0_sleep {
    group1 {
      psels = <NRF_PSEL(PWM_OUT0, 0, 26)>, <NRF_PSEL(PWM_OUT1, 0, 30)>,
          <NRF_PSEL(PWM_OUT2, 0, 6)>;
      low-power-enable;
    };
  };
};

&pwm0 {
  status = "okay";
  pinctrl-0 = <&pwm0_default>;
  pinctrl-1 = <&pwm0_sleep>;
  pinctrl-names = "default", "sleep";
};

/ {
  pwm_leds {
    compatible = "pwm-leds";
    status = "okay";

    red_pwm_led: red_pwm_led {
      pwms = <&pwm0 0 PWM_MSEC(10) PWM_POLARITY_INVERTED>;
    };

    green_pwm_led: green_pwm_led {
      pwms = <&pwm0 1 PWM_MSEC(10) PWM_POLARITY_INVERTED>;
    };

    blue_pwm_led: blue_pwm_led {
      pwms = <&pwm0 2 PWM_MSEC(10) PWM_POLARITY_INVERTED>;
    };
  };
};
```

PWM 構成時の注意点:

- `CONFIG_RGBLED_WIDGET_BRIGHTNESS` は、すべての表示で使う実行時の明るさを制御します。
- `.conf` で `CONFIG_PWM=y` と `CONFIG_LED_PWM=y` を必ず有効にしてください。
- `overlay` または `.dtsi` では、PWM 用 pinctrl、PWM コントローラの有効化、`led-red` / `led-green` / `led-blue` の alias 定義が必要です。
- RGB LED が active-low 配線なら `PWM_POLARITY_INVERTED` を使い、それ以外は `PWM_POLARITY_NORMAL` を使ってください。

最後に、設定でウィジェット自体を有効にします。

```ini
CONFIG_RGBLED_WIDGET=y
```