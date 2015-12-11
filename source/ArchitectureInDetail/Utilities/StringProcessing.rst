文字列処理(String Processing)
--------------------------------------------------------------------------------

.. only:: html

 .. contents:: 目次
    :depth: 4
    :local:

|

Overview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| 
|

How to use
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

全角半角文字列変換
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

全角文字列に変換
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| \ ``FullHalfConverter#toFullwidth``\ は、引数の文字列を全角文字列へ変換するメソッドである。
| 以下に半角文字列から全角文字列への変換方法を示す。

.. code-block:: java

   String fullwidth = DefaultFullHalf.INSTANCE.toFullwidth("ｱﾞ!A8ｶﾞザ");    // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``org.terasoluna.gfw.common.fullhalf.DefaultFullHalf``\ クラスを使用し、デフォルトの全角文字列と半角文字列の組合せが登録された\ ``FullHalfConverter#toFullwidth``\ にて、引数に渡した文字列を、全角文字列へ変換する。
       | 本例では、"ア゛！Ａ８ガザ" に変換される。
       | なお、本例の"ザ"のように半角文字ではない（デフォルトの組合せに無い）文字は、そのまま返却される。
       | \ ``DefaultFullHalf``\ クラスが定義しているデフォルトの全角文字列と半角文字列の組合せについては `ソース <https://github.com/terasolunaorg/terasoluna-gfw/blob/master/terasoluna-gfw-string/src/main/java/org/terasoluna/gfw/common/fullhalf/DefaultFullHalf.java>`_ を参照されたい。

.. note::

    独自の全角文字列と半角文字列の組合せを登録した\ ``FullHalfConverter``\ を使用する場合は :ref:`StringOperationsHowToDesignCustomFullHalfConverter` を参照されたい。

|


半角文字列に変換
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| \ ``FullHalfConverter#toHalfwidth``\ は、引数の文字列を半角文字列へ変換するメソッドである。
| 以下に全角文字列から半角文字列への変換方法を示す。

.. code-block:: java

   String halfwidth = DefaultFullHalf.INSTANCE.toHalfwidth("Ａ！アガｻ");    // (1)

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``DefaultFullHalf``\ クラスを使用し、デフォルトの全角文字列と半角文字列の組合せが登録された\ ``FullHalfConverter#toHalfwidth``\ にて、引数に渡した文字列を、半角文字列へ変換する。
       | 本例では、"A!ｱｶﾞｻ" に変換される。
       | なお、本例の"ｻ"のように全角文字ではない（デフォルトの組合せに無い）文字は、そのまま返却される。
       | \ ``DefaultFullHalf``\ クラスが定義しているデフォルトの全角文字列と半角文字列の組合せについては `ソース <https://github.com/terasolunaorg/terasoluna-gfw/blob/master/terasoluna-gfw-string/src/main/java/org/terasoluna/gfw/common/fullhalf/DefaultFullHalf.java>`_ を参照されたい。

.. note::

    独自の全角文字列と半角文字列の組合せを登録した\ ``FullHalfConverter``\ を使用する場合は :ref:`StringOperationsHowToDesignCustomFullHalfConverter` を参照されたい。

.. note::

    \ ``FullHalfConverter``\ は、2文字以上で1文字を表現する結合文字（例："シ（\\u30b7）" + "濁点（\\u3099）"）を半角文字（例："ｼﾞ"）へ変換することが出来ない。
    このような場合、テキスト正規化を行い、結合文字を合成文字（例："ジ（\\u30b8）"）に変換してから \ ``FullHalfConverter``\ を使用する必要がある。
    
    テキスト正規化を行う場合は、\ ``java.text.Normalizer``\ を使用する。
    以下に、使用方法を示す。
    
    正規化形式 ： NFD（正準等価性によって分解する）の場合
    
      .. code-block:: java

         String str1 = Normalizer.normalize("モジ", Normalizer.Form.NFD); // str1 = "モシ + Voiced sound mark(\\u3099)"
         String str2 = Normalizer.normalize("ﾓｼﾞ", Normalizer.Form.NFD);  // str2 = "ﾓｼﾞ"

    正規化形式 ： NFC（正準等価性によって分解し、再度合成する）の場合
    
      .. code-block:: java

         String mojiStr = "モシ\\u3099";                                   // "モシ + Voiced sound mark(\\u3099)"
         String str1 = Normalizer.normalize(mojiStr, Normalizer.Form.NFC); // str1 = "モジ（\\u30b8）"
         String str2 = Normalizer.normalize("ﾓｼﾞ", Normalizer.Form.NFC);   // str2 = "ﾓｼﾞ"
    
    正規化形式 ： NFKD（互換等価性によって分解する）の場合
    
      .. code-block:: java

         String str1 = Normalizer.normalize("モジ", Normalizer.Form.NFKD); // str1 = "モシ + Voiced sound mark(\\u3099)"
         String str2 = Normalizer.normalize("ﾓｼﾞ", Normalizer.Form.NFKD);  // str2 = "モシ + Voiced sound mark(\\u3099)"
    
    正規化形式 ： NFKC（互換等価性によって分解し、再度合成する）の場合
    
      .. code-block:: java

         String mojiStr = "モシ\\u3099";                                    // "モシ + Voiced sound mark(\\u3099)"
         String str1 = Normalizer.normalize(mojiStr, Normalizer.Form.NFKC); // str1 = "モジ（\\u30b8）"
         String str2 = Normalizer.normalize("ﾓｼﾞ", Normalizer.Form.NFKC) ;  // str2 = "モジ"
    
    
    上記のように、結合文字を合成文字に変換する場合などは、正規化形式 ： NFC または NFKC を利用する。
    
    詳細は \ `JavaDoc <https://docs.oracle.com/javase/8/docs/api/java/text/Normalizer.html>`_\ を参照されたい。

|


.. _StringOperationsHowToDesignCustomFullHalfConverter:

独自の全角文字列と半角文字列の組合せを登録したFullHalfConverterクラスの作成
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

| \ ``DefaultFullHalf``\ を使用せず、独自に作成した全角文字列と半角文字列の組合せを使って \ ``FullHalfConverter``\ を使用することも出来る。
| 以下に、独自の全角文字列と半角文字列の組合せを使って \ ``FullHalfConverter``\ を使用する方法を示す。

* | 独自の組合せを使った \ ``FullHalfConverter``\ を提供するクラスの実装

 .. code-block:: java
 
    public class CustomFullHalf {
        
        private static final int FULL_HALF_CODE_DIFF = 0xFEE0;
        
        public static final FullHalfConverter INSTANCE;
        
        static {
            // (1)
            FullHalfPairsBuilder builder = new FullHalfPairsBuilder();
        
            // (2)
            builder.pair("ー", "-");
            
            // (3)
            for (char c = '!'; c <= '~'; c++) {
                String fullwidth = String.valueOf((char) (c + FULL_HALF_CODE_DIFF));
                builder.pair(fullwidth, String.valueOf(c));
            }
            
            // (4)
            builder.pair("。", "｡").pair("「", "｢").pair("」", "｣").pair("、", "､")
                    .pair("・", "･").pair("ァ", "ｧ").pair("ィ", "ｨ").pair("ゥ", "ｩ")
                    .pair("ェ", "ｪ").pair("ォ", "ｫ").pair("ャ", "ｬ").pair("ュ", "ｭ")
                    .pair("ョ", "ｮ").pair("ッ", "ｯ").pair("ア", "ｱ").pair("イ", "ｲ")
                    .pair("ウ", "ｳ").pair("エ", "ｴ").pair("オ", "ｵ").pair("カ", "ｶ")
                    .pair("キ", "ｷ").pair("ク", "ｸ").pair("ケ", "ｹ").pair("コ", "ｺ")
                    .pair("サ", "ｻ").pair("シ", "ｼ").pair("ス", "ｽ").pair("セ", "ｾ")
                    .pair("ソ", "ｿ").pair("タ", "ﾀ").pair("チ", "ﾁ").pair("ツ", "ﾂ")
                    .pair("テ", "ﾃ").pair("ト", "ﾄ").pair("ナ", "ﾅ").pair("ニ", "ﾆ")
                    .pair("ヌ", "ﾇ").pair("ネ", "ﾈ").pair("ノ", "ﾉ").pair("ハ", "ﾊ")
                    .pair("ヒ", "ﾋ").pair("フ", "ﾌ").pair("ヘ", "ﾍ").pair("ホ", "ﾎ")
                    .pair("マ", "ﾏ").pair("ミ", "ﾐ").pair("ム", "ﾑ").pair("メ", "ﾒ")
                    .pair("モ", "ﾓ").pair("ヤ", "ﾔ").pair("ユ", "ﾕ").pair("ヨ", "ﾖ")
                    .pair("ラ", "ﾗ").pair("リ", "ﾘ").pair("ル", "ﾙ").pair("レ", "ﾚ")
                    .pair("ロ", "ﾛ").pair("ワ", "ﾜ").pair("ヲ", "ｦ").pair("ン", "ﾝ")
                    .pair("ガ", "ｶﾞ").pair("ギ", "ｷﾞ").pair("グ", "ｸﾞ")
                    .pair("ゲ", "ｹﾞ").pair("ゴ", "ｺﾞ").pair("ザ", "ｻﾞ")
                    .pair("ジ", "ｼﾞ").pair("ズ", "ｽﾞ").pair("ゼ", "ｾﾞ")
                    .pair("ゾ", "ｿﾞ").pair("ダ", "ﾀﾞ").pair("ヂ", "ﾁﾞ")
                    .pair("ヅ", "ﾂﾞ").pair("デ", "ﾃﾞ").pair("ド", "ﾄﾞ")
                    .pair("バ", "ﾊﾞ").pair("ビ", "ﾋﾞ").pair("ブ", "ﾌﾞ")
                    .pair("べ", "ﾍﾞ").pair("ボ", "ﾎﾞ").pair("パ", "ﾊﾟ")
                    .pair("ピ", "ﾋﾟ").pair("プ", "ﾌﾟ").pair("ペ", "ﾍﾟ")
                    .pair("ポ", "ﾎﾟ").pair("ヴ", "ｳﾞ").pair("\u30f7", "ﾜﾞ")
                    .pair("\u30fa", "ｦﾞ").pair("゛", "ﾞ").pair("゜", "ﾟ").pair("　", " ");
            
            // (5)
            INSTANCE = new FullHalfConverter(builder.build());
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | \ ``org.terasoluna.gfw.common.fullhalf.FullHalfPairsBuilder``\ を使用して、全角文字列と半角文字列の組合せとなる \ ``org.terasoluna.gfw.common.fullhalf.FullHalfPairs``\ を作成する。
    * - | (2)
      - | \ ``DefaultFullHalf``\ では、全角文字列"ー"に対する半角文字列を"ｰ(\uFF70)"と設定していたところを、"-(\u002D)"に変更して設定。
        | なお、"-(\u002D)"は、下記(3)でのマッピングに含まれるため、本マッピングを先に定義し、本マッピングが優先されるようにする。
    * - | (3)
      - | Unicodeの全角"！"から"～"までと半角"!"から"~"までのコード値は、コード定義の順番が同じであるため、ループ処理にてマッピングする。
    * - | (4)
      - | 上記(3)以外の文字はコード定義の順番が全角文字列と半角文字列で一致しない。そのため、それぞれ個別にマッピングする。
    * - | (5)
      - | \ ``FullHalfPairsBuilder``\ より作成した \ ``FullHalfPairs``\ を使用して、 \ ``FullHalfConverter``\ を作成する。

 .. note::

    \ ``FullHalfPairsBuilder#pair``\ メソッドは、以下の条件を満たさない場合、\ ``java.lang.IllegalArgumentException``\ をスローする。
     * 第1引数の全角文字は1文字
     * 第2引数の半角文字は1文字または2文字

|

* | 独自の組合せを使った \ ``FullHalfConverter``\ の使用例

 .. code-block:: java
 
    String halfwidth = CustomFullHalf.INSTANCE.toHalfwidth("ハローワールド！"); // (1)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | 実装した \ ``CustomFullHalf``\ を使用し、 独自の組合せが登録された \ ``FullHalfConverter#toHalfwidth``\ にて、引数に渡した文字列を、半角文字列へ変換する。
        | 本例では、"ﾊﾛ-ﾜ-ﾙﾄﾞ!" （"-" は (\u002D)）に変換される。


|


.. raw:: latex

   \newpage

