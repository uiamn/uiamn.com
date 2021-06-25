---
title: Rustでtarアーカイバを作った
date: "2021-06-08"
description: "Rustが来るらしい．"
---

今一緒に仕事をして居る方の知人のフリーランスの方と先日食事をする機会があったのですが，その人は何年も前からプログラミング言語をウォッチして居るらしく，次に来る言語はRustだ！！！！と仰られて居ましたので，勉強することにしました．プログラミングの勉強は取敢ず何か作ればわかる様になってくるので，前々から実装したいな〜と思ってたtarアーカイバを作りました．

パスが100文字を超えると上手く動作しません．多分．

[これ](https://github.com/uiamn/tar_archiver)

## Checksum
ヘッダに含まれるChecksumはそのヘッダの各バイトを1つの数字として見たときに，それらの和を8進数で表したものなのですが，計算時はChecksumの領域は全部空白(つまり0x20)として計算すればいい様です．つまりヘッダの各バイトの総和 = Checksumは誤りであり，ヘッダのChecksum以外の各バイトの総和 + 32(0x20) x 8 = Checksumが正解となります．0でもないのは何故なのか．


## 参考にした
* [NimでTarファイル作成を実装する ～ちゃぶだい返し(理論編)～ - Kapibara Tech Blog](https://kapibara-sos.net/archives/442)
* [TAR(5) - ファイルフォーマット - YOS OPENSONAR](http://www.yosbits.com/opensonar/rest/man/freebsd/man/ja/man5/tar.5.html?l=ja)
