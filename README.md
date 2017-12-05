VASPで簡単に物性を計算するために作りました。
プログラムはcshで書かれています。Linux上で動作します。

スパコンではなくデスクトップなどの普通の人が購入するPCで最適な計算速度になるようにしています。
もし、スパコンでされる方は$num_coreを指定するように書き換えてください。
cshの部分はMITライセンスです。

必要なものは、VASPとcsh, python, cif2cellです。その他には、phonopyとphono3py, boltztrapです。

vasp2cif.pyはpythonです。この部分は他の方のものでApache License, Version 2.0です。
http://www.apache.org/licenses/LICENSE-2.0 に従ってください。
