---
title: "Train plugin API (IRuntimeTrain) and Route plugin API (IRuntimeRoute)"
hidden: true
---

これは車両プラグインと路線プラグインのドキュメントです。車両プラグインを作成するためには名前空間OpenBveApi.RuntimeからIRuntimeTrainインターフェースを実装します。路線プラグインを作成するためには名前空間OpenBveApi.RuntimeからIRuntimeRouteインターフェースを実装します。

{{% warning %}}

#### IRuntime vs. IRuntimeTrain and IRuntimeRoute

IRuntimeとIRuntimeTrainには互換性がありません。IRuntimeTrainとIRuntimeRouteはopenBVE 1.5.4.X以上で使用可能です。以前のバージョンで読み込むとエラーが発生するので注意してください。

{{% /warning %}}

## ■ Overview

プラグインがロードされると、次の関数がこの順序で呼び出されます。

- IRuntimeTrain
  - Load
  - SetVehicleSpecs
  - Initialize
  - SetPower
  - SetBrake
  - SetReverser
- IRuntimeRoute
  - Load
  - Initialize

毎フレームに、次の関数がこの順序で呼び出されます。

- IRuntimeRoute
  - Elapse
- IRuntimeTrain
  - Elapse

次の関数はいつでも呼び出されることができます。

- IRuntimeTrain
  - SetPower
  - SetBrake
  - SetReverser
  - KeyDown
  - KeyUp
  - HornBlow
  - DoorChange
  - SetSignal
  - SetBeacon
  - PerformAI

(S520注: 現時点ではこれらの関数の前に路線プラグインは呼び出されません。実装した方がよい場合はご相談ください。)

IRuntimeTrainのイベント *Transmitter* は任意の関数内で呼び出すことができます。このイベントはIRuntimeRouteの関数 *Receiver* を呼び出します。

プラグインがアンロードされると、次の関数がこの順序で呼び出されます。

- IRuntimeTrain
  - Unload
- IRuntimeRoute
  - Unload

## ■ Function calls - IRuntimeTrain

IRuntimeTrainはIRuntimeを拡張したインターフェースのため、ここでは異なる関数のみ一覧とその動作について説明します。記述されていない関数はIRuntimeと同様ですので、[そちら]({{< ref "/plugins/iruntime/_index.md" >}})を参照してください。

------

**void Elapse(ElapseData data, byte[] recieveData)**

この関数は全てのフレームで呼ばれます。電車の現在の状態を車両プラグインに通知し、ハンドルの状態を設定できます。加えて、路線プラグインからのデータを受け取ることができます。

引数:

{{% table-nonheader %}}

| ElapseData | data        | 本体からプラグインに渡されたデータ           |
| ---------- | ----------- | -------------------------------------------- |
| byte[]     | recieveData | 路線プラグインから受け取るデータを取得する。 |

{{% /table-nonheader %}}

------

**event EventHandler\<TxEventArgs> Transmitter**

これはIRuntimeRouteの関数 *Receiver* を呼び出します。

TxEventArgs (クラス):

{{% table-nonheader %}}

| byte[] | SendData | 路線プラグインに送るデータを設定する。 |
| ------ | -------- | -------------------------------------- |
|        |          |                                        |

{{% /table-nonheader %}}

<br/>

{{% code "実装例" %}}

```c#
public event EventHandler<TxEventArgs> Transmitter;

protected virtual void UseTransmitter(TxEventArgs e) {
    if (Transmitter != null) {
        Transmitter(this, e);
    }
}

// 使用方法
using System;
using System.Linq;
bool A = true;
int B = 123;
byte[] sendData = BitConverter.GetBytes(A).Concat(BitConverter.GetBytes(B)).ToArray();
UseTransmitter(new TxEventArgs(sendData));
```

{{% /code %}}

## ■ Function calls - IRuntimeRoute

以下は全ての関数の一覧とその動作に関する説明です。

------

**bool Load()**

この関数はプラグインがロードされた後に最初に呼び出されます。

(S520注: 引数については仮のものです。ロード時に取得した方がよいものがあれば、ご指摘ください。)

------

**void Unload()**

この関数はプラグインがアンロードされる前に呼び出される最後のものです。

------

**void Initialize()**

この関数は *Load* の後に呼び出され、プラグインを初期化します。ユーザーが「駅へジャンプ」機能を使用する際に、この関数は車両を新しい場所へ移動する前にも呼び出されます。

(S520注: 引数については仮のものです。初期化時に取得した方がよいものがあれば、ご指摘ください。)

------

**void Elapse(ElapseDataRoute data)**

この関数は全てのフレームで呼ばれます。路線の現在の状態を車両プラグインに通知し、信号インデックスを設定できます。加えて、車両プラグインへデータを送ることができます。

引数:

{{% table-nonheader %}}

| ElapseDataRoute | data | 本体からプラグインに渡されたデータ |
| --------------- | ---- | ---------------------------------- |
|                 |      |                                    |

{{% /table-nonheader %}}

ElapseDataRoute (クラス):

{{% table-nonheader %}}

| byte[]    | SendData | 車両プラグインに送るデータを設定する。 |
| --------- | -------- | -------------------------------------- |
| Section[] | Sections | 路線上の全ての閉塞の情報               |

{{% /table-nonheader %}}

Section (構造体)

{{% table-nonheader %}}

| SectionAspect[] | Aspects | 各閉塞の全ての信号インデックスの情報 |
| --------------- | ------- | ------------------------------------ |
|                 |         |                                      |

{{% /table-nonheader %}}

SectionAspect (構造体)

{{% table-nonheader %}}

| int    | Number | 各信号インデックスの数字を取得および設定する。             |
| ------ | ------ | ---------------------------------------------------------- |
| double | Speed  | 各信号インデックスに対応する制限速度を取得および設定する。 |

{{% /table-nonheader %}}

(S520注: クラスや構造体のメンバーについては仮のものです。取得可能にした方がよいものがあれば、ご指摘ください。)

------

**void Receiver(byte[] receiveData)**

この関数はIRuntimeTrainのイベント *Transmitter* に呼び出されます。車両プラグインからのデータを受け取ることができます。

引数:

{{% table-nonheader %}}

| byte[] | recieveData | 車両プラグインから受け取るデータを取得する。 |
| ------ | ----------- | -------------------------------------------- |
|        |             |                                              |

{{% /table-nonheader %}}

<br/>

{{% code "実装例" %}}

```c#
using System;

public void Receiver(byte[] receiveData) {
    if (receiveData.Length < 5) {
        return;
    }
    bool A = BitConverter.ToBoolean(receiveData, 0);
    int B = BitConverter.ToInt32(receiveData, 1);
}
```

{{% /code %}}