Anomaly チュートリアル (Java)
============================================

ここではJava版のAnomalyサンプルプログラムの解説をします。

--------------------------------
ソースコード
--------------------------------

このサンプルプログラムでは、学習の設定をするconfig.jsonと外れ値検知を行うlof.javaを利用します。以下にソースコードを記載します。

**config.json**

.. code-block:: python
 :linenos:

 {
  "method" : "lof",
  "parameter" : {
   "nearest_neighbor_num" : 10,
   "reverse_nearest_neighbor_num" : 30,
   "method" : "euclid_lsh",
   "parameter" : {
    "hash_num" : 8,
    "table_num" : 16,
    "probe_num" : 64,
    "bin_width" : 10,
    "seed" : 1234,
    "retain_projection" : true
   }
  },
 
  "converter" : {
   "string_filter_types": {},
   "string_filter_rules": [],
   "num_filter_types": {},
   "num_filter_rules": [],
   "string_types": {},
   "string_rules": [{"key":"*", "type":"str", "global_weight" : "bin", "sample_weight" : "bin"}],
   "num_types": {},
   "num_rules": [{"key" : "*", "type" : "num"}]
  }
 }

**anomaly.java**

.. code-block:: java
 :linenos:

 package lof;
 
 import java.io.BufferedReader;
 import java.io.FileNotFoundException;
 import java.io.FileReader;
 import java.io.IOException;
 import java.util.ArrayList;
 import java.util.Arrays;
 import java.util.List;
 
 import us.jubat.anomaly.*;
 
 public class Lof {
 	public static final String HOST = "127.0.0.1";
 	public static final int PORT = 9199;
 	public static final String NAME = "anom_kddcup";
 	public static final String FILE_PATH = "./src/main/resources/";
 	public static final String TEXT_NAME = "kddcup.data_10_percent.txt";
 
 	// TEXTのカラム名定義
 	public static String[] TEXT_COLUMN = {
 		"duration",
 		"protocol_type",
 		"service",
 		"flag",
 		"src_bytes",
 		"dst_bytes",
 		"land",
 		"wrong_fragment",
 		"urgent",
 		"hot",
 		"num_failed_logins",
 		"logged_in",
 		"num_compromised",
 		"root_shell",
 		"su_attempted",
 		"num_root",
 		"num_file_creations",
 		"num_shells",
 		"num_access_files",
 		"num_outbound_cmds",
 		"is_host_login",
 		"is_guest_login",
 		"count",
 		"srv_count",
 		"serror_rate",
 		"srv_serror_rate",
 		"rerror_rate",
 		"srv_rerror_rate",
 		"same_srv_rate",
 		"diff_srv_rate",
 		"srv_diff_host_rate",
 		"dst_host_count",
 		"dst_host_srv_count",
 		"dst_host_same_srv_rate",
 		"dst_host_diff_srv_rate",
 		"dst_host_same_src_port_rate",
 		"dst_host_srv_diff_host_rate",
 		"dst_host_serror_rate",
 		"dst_host_srv_serror_rate",
 		"dst_host_rerror_rate",
 		"dst_host_srv_rerror_rate",
 		"label"
 	};
 
 	// String型の項目
 	public static String[] STRING_COLUMN = {
 		"protocol_type",
 		"service",
 		"flag",
 		"land",
 		"logged_in",
 		"is_host_login",
 		"is_guest_login"
 	};
 
 	// Double型の項目
 	public static String[] DOUBLE_COLUMN = {
 		"duration",
 		"src_bytes",
 		"dst_bytes",
 		"wrong_fragment",
 		"urgent",
 		"hot",
 		"num_failed_logins",
 		"num_compromised",
 		"root_shell",
 		"su_attempted",
 		"num_root",
 		"num_file_creations",
 		"num_shells",
 		"num_access_files",
 		"num_outbound_cmds",
 		"count",
 		"srv_count",
 		"serror_rate",
 		"srv_serror_rate",
 		"rerror_rate",
 		"srv_rerror_rate",
 		"same_srv_rate",
 		"diff_srv_rate",
 		"srv_diff_host_rate",
 		"dst_host_count",
 		"dst_host_srv_count",
 		"dst_host_same_srv_rate",
 		"dst_host_same_src_port_rate",
 		"dst_host_diff_srv_rate",
 		"dst_host_srv_diff_host_rate",
 		"dst_host_serror_rate",
 		"dst_host_srv_serror_rate",
 		"dst_host_rerror_rate",
 		"dst_host_srv_rerror_rate"
 	};
 
 	public void execute() throws Exception {
 		// 1. Jubatus Serverへの接続設定
 		AnomalyClient client = new AnomalyClient(HOST, PORT, 5);
 
 		// 2. 学習用データの準備
 		Datum datum = null;
 		TupleStringFloat result = null;
 
 		try {
 			BufferedReader br = new BufferedReader(new FileReader(FILE_PATH + TEXT_NAME));
 
 			List<String> strList = new ArrayList<String>();
 			List<String> doubleList = new ArrayList<String>();
 
 			String line = "";
 
 			// 最終行までループでまわし、1行ずつ読み込む
 			while ((line = br.readLine()) != null) {
 				strList.clear();
 				doubleList.clear();
 
 				// 1行をデータの要素に分割
 				String[] strAry = line.split(",");
 
 				// StringとDoubleの項目ごとにListを作成
 				for (int i = 0; i < strAry.length; i++) {
 					if (Arrays.toString(STRING_COLUMN).contains(TEXT_COLUMN[i])) {
 						strList.add(strAry[i]);
 					} else if (Arrays.toString(DOUBLE_COLUMN).contains(TEXT_COLUMN[i])) {
 						doubleList.add(strAry[i]);
 					}
 				}
 				// datumを作成
 				datum = makeDatum(strList, doubleList);
 
 				// 3. データの学習（学習モデルの更新）
 				result = client.add(NAME, datum);
 
 				// 4. 結果の出力
 				if ( !(Float.isInfinite(result.second)) && result.second != 1.0) {
 					System.out.print( "('" + result.first + "', " + result.second + ") " + strAry[strAry.length -1] + "\n" );
 				}
 			}
 			br.close();
 
 		} catch (FileNotFoundException e) {
 			// Fileオブジェクト生成時の例外捕捉
 			e.printStackTrace();
 		} catch (IOException e) {
 			// BufferedReaderオブジェクトのクローズ時の例外捕捉
 			e.printStackTrace();
 		}
 		return;
 	}
 
 
 	// Datumを指定された名称で、リスト分作成
 	private Datum makeDatum(List<String> strList, List<String> doubleList) {
 
 		Datum datum = new Datum();
 		datum.string_values = new ArrayList<TupleStringString>();
 		datum.num_values = new ArrayList<TupleStringDouble>();
 
 		for (int i = 0; i < strList.size(); i++) {
 			TupleStringString data = new TupleStringString();
 			data.first = STRING_COLUMN[i];
 			data.second = strList.get(i);
 
 			datum.string_values.add(data);
 		}
 
 		try {
 			for (int i = 0; i < doubleList.size(); i++) {
 				TupleStringDouble data = new TupleStringDouble();
 				data.first = DOUBLE_COLUMN[i];
 				data.second = Double.parseDouble(doubleList.get(i));
 
 				datum.num_values.add(data);
 			}
 		} catch (NumberFormatException e) {
 			e.printStackTrace();
 			return null;
 		}
 
 		return datum;
 	}
 
 	// メインメソッド
 	public static void main(String[] args) throws Exception {
 
 		new Lof().execute();
 		System.exit(0);
 	}
 }

--------------------------------
解説
--------------------------------

**config.json**

設定は単体のJSONで与えられます。JSONの各フィールドは以下のとおりです。

* method

 分類に使用するアルコリズムを指定します。
 Regressionで指定できるのは、現在"LOF"のみなので"LOF"（Local Outlier Factor）を指定します。


* converter

 特徴変換の設定を指定します。
 ここでは、"num_rules"と"string_rules"を設定しています。
 
 "num_rules"は数値特徴の抽出規則を指定します。
 "key"は"*"つまり、すべての"key"に対して、"type"は"num"なので、指定された数値をそのまま重みに利用する設定です。
 具体的には、valueが"2"であれば"2"を、"6"であれば"6"を重みとします。
 
 "string_rules"は文字列特徴の抽出規則を指定します。
 "key"は"*"、"type"は"str"、"sample_weight"は"bin"、"global_weight"は"bin"としています。
 これは、すべての文字列に対して、指定された文字列をそのまま特徴として利用し、各key-value毎の重みと今までの通算データから算出される、大域的な重みを常に"1"とする設定です。

* parameter（要修正）

 ･･･

  

**anomaly.java**

 anomaly.javaでは、textから読み込んだデータをJubatusサーバ与え、外れ値を検出し出力します。

 1. Jubatus Serverへの接続設定

  Jubatus Serverへの接続を行います（117行目）。
  Jubatus ServerのIPアドレス、Jubatus ServerのRPCポート番号、接続待機時間を設定します。
  
 2. 学習用データの準備

  AnomalyClientでは、Datumをaddメソッドに与えることで、学習および外れ値検知が行われます。
  今回はKDDカップ（Knowledge Discovery and Data Mining Cup）の結果（TEXTファイル）を元に学習用データを作成していきます。
  まず、学習用データの元となるTEXTファイルを読み込みます。
  ここでは、FileReaderとBuffererdReaderを利用して1行ずつループで読み込んで処理します（132-157行目）。
  このTEXTファイルはカンマ区切りで項目が並んでいるので、取得した1行を’,’で分割し要素ごとに分けます（137行目）。
  定義したTEXTファイルの項目リスト（TEXT_COLUMN）とStringとDoubleの項目を定義したリスト（STRING_COLUMN、DOUBLE_COLUMN）を用い、型ごとにリストを作成します（140-145行目）。
  作成した２つのリストを引数としてDatumを作成するprivateメソッド「makeDatum」を呼び出します（91行目）。
   
  「makeDatum」では、引数のString項目のリストとDouble項目のリストから、String項目はTupleStringStringのListを、Double項目はTupleStringDoubleのListを作成します（172-200行目）。
  まず、Datumクラスを生成してDatumの要素であるstring_valuesとnum_valuesのListをそれぞれ生成します（174-176行目）。
  次に、定義しているString項目リスト（STRING_COLUMN）と引数のstrListの順番は対応しているので、ループでTupleStringStringを生成し、要素firstにキー（カラム名）をsecondにバリュー（値）を設定してstring_valuesのListに追加します（178-184行目）。
  Double項目リストもString項目と同様にループでTupleStringDoubleを生成し、要素を設定してからnum_valuesに追加します。ここで注意する点は、引数はString型のListですがDatumのnum_valuesはDouble型の為、変換が必要になります（190行目）。
  これで、Datumの作成が完了しました。

  
 3. データの学習（学習モデルの更新）

  AnomalyClientのaddメソッドに2. で作成したデータを渡します（151行目）。
  addメソッドの第1引数は、タスクを識別するZookeeperクラスタ内でユニークな名前を指定します。（スタンドアロン構成の場合、空文字（""）を指定）
  第2引数として、先ほど2. で作成したDatumを指定します。
  戻り値として、tuple<string, float>型で点IDと異常値を返却します。
  
 4. 結果の出力

  addメソッドの戻り値である異常値から外れ値かどうかを判定します（154行目）。
  異常値が無限ではなく、1.0以外の場合は外れ値と判断し出力します（155行目）。

-------------------------------------
サンプルプログラムの実行
-------------------------------------

**［Jubatus Serverでの作業］**

 jubaanomalyを起動します。
 
 ::
 
  $ jubaanomaly --configpath config.json
 

**［Jubatus Clientでの作業］**

 必要なパッケージとJavaクライアントを用意し、実行します。
 
**［実行結果］**

::

 ('574', 0.99721104) normal.
 ('697', 1.4958459) normal.
 ('1127', 0.79527026) normal.
 ('1148', 1.1487594) normal.
 ('1149', 1.2) normal.
 ('2382', 0.9994011) normal.
 ('2553', 1.2638165) normal.
 ('2985', 1.4081864) normal.
 ('3547', 1.275244) normal.
 ('3557', 0.90432936) normal.
 ('3572', 0.75777346) normal.
 ('3806', 0.9943142) normal.
 ('3816', 1.0017062) normal.
 ('3906', 0.5671135) normal.
 …
 …（以下略）
