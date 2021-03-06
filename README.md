# DIO_Trans

子機のDI(ディジタル入力)を高頻度(標準1Khz/4ポート, 12.8KHz/1ポートまで設定可)でサンプルし、親機のDO(ディジタル出力)に再現します。



### App_Twelite(標準アプリ)との違い

App_Tweliteのディジタル入出力は、手でボタンを押すと言った比較的頻度が低い変化を想定した設計になっています。DIO_Transは非常に早い変化を検出・再現することに焦点をあてています。

* App_Twelite の対応可能な周波数はおよそ10Hz以下（片方向通信では32Hzまで対応）ですが、DIO_Transは１ポートのみの伝送では最大10Khz程度まで伝送できます。
* App_Twelite にはチャタリングの抑制を行うための連照アルゴリズムなども含まれていますが、DIO_Trans にはそういったアルゴリズムは含まれていません。
* 親機から子機への遅延は、App_Twelite では連照アルゴリズムを利用した標準的な場合は30ms前後になります。定レイテンシモードでの最短は10ms未満になります。DIO_Transは連続的なデータを途切れを軽減しながら送信するためバッファリングを行っています。1kHz, 4ポート 伝送時の標準的な遅延は250ms前後です。
* DIO_Trans は送信頻度が高いアプリで、そのまま中継動作を行うと実用的になりません。中継については OFF としています。
  * 中継が必要な場合は 子機1 ..(無線).. 親機1--(DIOを配線)--子機2 ..(無線).. 親機2 のような配置を考えてください。
    　この場合、子機1・親機1の無線チャネルを子機2・親機2の無線チャネルと違うものにしてください。例えば１側をチャネル12, ２側を22とします。



## License

MW-SLA-1 (MONO WIRELESS SOFTWARE LICENSE AGREEMENT) を適用します。



## アプリケーションの書き換え

### TWELITE STAGE の導入とアップデート

MWSTAGE2020_05（または今後リリースされるより新しい版）をインストールしてください。

MWSTAGE2020_05ではライブラリ更新が必要です（今後のリリースでは更新は不要になります）。以下のファイルをダウンロード、展開して、MWSTAGE/MWSDK/TWENET ディレクトリと置き換えてください。

> https://github.com/monowireless/MWSDK_COMMON_SNAP/releases/tag/MWSDK2020_07_UNOFFICIAL



### TWELITE STAGEによる書き換え

DIO_TransはTWELITE SDKのmwxライブラリ上で記述された Act です。ActEx_DIO_Transディレクトリを`MWSTAGE/MWSDK/Act_samples/`に格納します。TWELITE STAGE 上でのビルドと書き換えが可能になります。

DIO_Trans は親機と子機で同じファームウェアを書き込んでください。

ファームウェアを書き込む手順は以下となります。

1. ファームウェアプログラムを書き込む対象(TWELITE DIPなど)を TWELITE R 経由で接続しておきます。
2. TWELITE STAGEを起動します。起動時に TWELITE R のポートを指定します。
3.  [アプリ書き換え] > [Actビルド&書換] > [ActEx_DIO_trans] を選択します。



TWELITE STAGE 0.8.9 の画面表示例です。

```
1: ビューア                                 
2: アプリ書換                    <-----選択
3: インタラクティブモード                          
4: TWELITE® STAGEの設定                    
5: シリアルポート選択[MWXXXXXXX]                  
```

```
1: BINから選択                                           
2: Actビルド&書換                <-----選択    
3: TWELITE APPSビルド&書換                                
4: 指定 [ﾌｫﾙﾀﾞをﾄﾞﾛｯﾌﾟ]                                 
5: 再書換 [無し]                                          
```

```
1: act0                                              
2: act1                                              
3: act2                                              
4: act3                                              
5: act4                                                 
6: ActEx_DIO_trans             <-----選択    
7: BRD_APPTWELITE                                       
8: PAL_AMB                                           
9: PAL_AMB-behavior                                  
0: PAL_AMB-usenap                                    
q: PAL_MAG                                           
```

DIO_Transを選択後ビルドが始まります。無事ビルドが終わるとファームウェアが書き込まれます。

```

[ActEx_DIO_trans_BLUE_L1304_V0-1-0.bin]

ファームウェアを書き込んでいます...
|                                |
書き込みに成功しました(2823ms)

中ボタンまたは[Enter]で
ﾀｰﾐﾅﾙ(ｲﾝﾀﾗｸﾃｨﾌﾞﾓｰﾄﾞ用)を開きます。
```



## 使用方法

親機と子機の設定 (M1, M2, M3, BPS ピン) を行います。

* 親機は M1=G(GND)、子機は M1=O(オープンまたはVccプルアップ) とします。
* M2/M3/BPS ピンは動作のための設定で、**親機と子機は同じ設定**でなければなりません。すべてオープンの場合、4ch, 1Khz で動作します。



## 使用するピン

App_Tweliteのピン配置に倣います。

| ピン名 | DIP# | DIO# | 意味                             |
| ------ | :--: | :--: | -------------------------------- |
| DI1    |  15  |  12  | 子機の入力1 (1ch, 2ch, 4ch)      |
| DI2    |  16  |  13  | 子機の入力2 (2ch, 4ch)           |
| DI3    |  17  |  11  | 子機の入力3 (4ch)                |
| DI4    |  18  |  16  | 子機の入力4 (4ch)                |
| DO1    |  5   |  18  | 親機の出力1 (1ch, 2ch, 4ch)      |
| DO2    |  8   |  19  | 親機の出力2 (2ch, 4ch)           |
| DO3    |  9   |  4   | 親機の出力3 (4ch)                |
| DO4    |  12  |  9   | 親機の出力4 (4ch)                |
| M1     |  13  |  10  | 親機子機設定                     |
| M2     |  26  |  2   | 設定                             |
| M3     |  27  |  3   | 設定                             |
| BPS    |  20  |  17  | 設定（サンプル周波数を倍にする） |



## 設定

| ピン名 |                                                              |
| ------ | ------------------------------------------------------------ |
| M1     | G:親機を指定します。<br />O:子機を指定します。               |
| M2,M3  | M2=O M3=O --> 4ch, 1Khz<br />M2=G M3=O --> 2ch, 2Khz<br />M2=O M3=G --> 1ch, 4Khz<br />M2=G M3=G --> 1ch, 6.4Khz |
| BPS    | サンプル周波数を倍にします。                                 |

G: GND, O: オープンまたはVccプルアップ



### チャネル・アプリケーションID

common.h を編集してから[Actビルド&書換]を行ってください。

```C++
// application ID
const uint32_t APP_ID = 0x1234abcd;

// channel
const uint8_t CHANNEL = 13;
```



### 再送回数

myAppBhvChild.cpp の `MWX_STATE(MY_APP_CHILD::STATE_TX, uint32_t ev, uint32_t evarg)`内の`rx_retry()`内のパラメータを変更します。例えば `0x2` から `0x1` に変更すると、再送回数は１回となり都合２回の送信が行われます。

```C++
		if (auto&& pkt = the_twelite.network.use<NWK_SIMPLE>().prepare_tx_packet()) {
			// set tx packet behavior
			pkt << tx_addr(0x00)  // 0..0xFF (LID 0:parent, FE:child w/ no id, ...
				<< tx_retry(0x2)  // set retry (0x2 send three times in total)
				<< tx_packet_delay(0, 0, 4)  // send packet w/ delay
				<< tx_process_immediate()    // transmit immediately
				;

```

一般にパケットを１回のみ送信する場合、条件のいい場合でも目安として１０回に１回程度失敗する場合が想定されます。安定した通信のために、再送回数は最低１とすることを推奨します。



## 例外的な動作



* 子機側で順番を入れ替えて送信するようなコードになっていないため、親機側ではパケットの順序が入れ替わった場合の対処はしていません。
* パケットが抜けて連続しないような場合には、親機の出力が維持され、次に受信したパケットデータに基づき出力します。
* 親機と子機の時計と言える水晶振動子の相互誤差により、同じサンプル周波数で動作させたとしても、子機側と親機側で少しずつ時計がずれていきます。この誤差を吸収するため、子機からのパケットを２つ分バッファしてから親機の出力を行っています。時間とともに誤差は蓄積し、親機のバッファが足りなくなってしまう（アンダーラン）または過剰に蓄積され蓄積できなくなってしまう（オーバラン）が発生します。アンダーランまたはオーバーランした場合には、適切なバッファ蓄積をやりなおします。



## 運用について

* 無線チャネルをほぼ専有します。DIO_Trans の親機子機を同じチャネル内で複数ペア動作させることはできません。またTWELITEなどIEEE802.15.4を利用する同種のシステムに対しても深刻な影響を与えることが想定されます。
* DIO_Transは中継を想定した実装ではないため、単にmwxライブラリ用の中継機を設置しただけではうまく動作しません。サンプル周波数を落とす (4chで250Hz, 1chで1kHz程度) ことで、対応できる可能性はあります。



## パケットの伝送について

IEEE802.15.4の規格では、データはパケットという単位で伝送されます。DIO_Transのような連続データの送受信を行う場合は、パケットの送受信の失敗や、タイミングのゆらぎを考慮して、受信側ではバッファリングを行ってからデータを再現します。

DIO_Transでは１パケットあたり64バイト(512ビット)を単位としてデータ送信を行っています。1Khzでサンプルした場合 4ch では128msデータが揃うまで時間がかかります。データが揃ったら速やかにパケットを送信します。次の64バイトが揃うまでの間に送信を完了します。受信側は、このパケットを２つ分受信し内部に格納してからIOポートの出力を開始します。この例では、２パケット分が受信されるまで、つまり 256ms に加えて無線送受信にかかる時間 (10〜20ms程度) の遅延が発生します。

サンプル周波数が大きくなればその分だけ遅延は減りますが、無線送信にかけられる時間が短くなります。処理が間に合わない場合は、再送回数を減らすなどの対処が考えられますが、それでも時間が足りない場合は、うまく動作しなくなることが考えられます。いずれにせよデータの欠損等が発生しやすくなります。



## シリアルポートの表示

シリアルポートに動作を確認するための表示が出ています。以下にはパケットの送受信状態を示す１文字出力について解説します。



### 子機側

送信が終了すると `t` が表示されます。１ブロック 512bit 分が蓄積され、これが無線送信されたことを示します。



### 親機側

パケットの受信状況に応じて以下の表示が行われます。

| 表示        | 内容                                       |
| ----------- | ------------------------------------------ |
| `.`         | パケットの受信                             |
| `U`  (反転) | バッファのアンダーラン                     |
| `S`  (反転) | バッファのスキップ（続き番号が連続しない） |
| `O`  (反転) | バッファのオーバーラン                     |

