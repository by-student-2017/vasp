VASPで簡単に物性を計算するために作りました。
プログラムはcshで書かれています。Linux上で動作します。

大切：改行コードがWindows用のCR+LFになっている場合があります。その場合には改行コードをLFにしてください。

スパコンではなくデスクトップなどの普通の人が購入するPCで最適な計算速度になるようにしています。
もし、スパコンでされる方はnum_coreを指定するように書き換えてください。
cshの部分はMITライセンスです。

名前にautosetと付いたものはvaspで必要な入力ファイルを準備します。
cshの中に記述していますので対応する部分を書き換えれば好きな入力の設定で計算できます。
autosetと記述の付いたものはcshの中でなくても入力ファイルの修正が可能です。
どちらが良いかは好みですが、とりあえずautosetと付いたものを使用してみてください。

必要なものは、VASPとOpenMPI, csh, python, cif2cell, gnuplotです。

その他には、phonopyとphono3py, boltztrapです。

名称にbandと付いたものは動作未確認です。
いまの私は計算をするための環境が非常に悪いのでこれ以上の作業があまり進んでいません。
cshを読める人に書き直して貰いたいです。

vasp2cif.pyはpythonです。この部分は他の方のものでApache License, Version 2.0に従ってください。

大変申し訳ないのですが、この０スクリプトを使用して生じた損害は補償できませんので、 
自己責任で参考にしてください。
