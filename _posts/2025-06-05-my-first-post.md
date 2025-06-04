---
layout: post
title: "覚え書き始めました(そろそろ冷やし中華時期か・・・)"
date: 2025-06-05 00:00:00 +0900
---

flutter + androidでOpenCVを使用してみたく、  
実行環境構築で困惑したので覚え書きとして残しておきます。

結論から言うと、OpenCV4.11.0(2025年6月5日時点で最新版)で動作することを確認しました。

最終的に成功した実行環境構築  
flutter + android実装はWindows  
1. VSCode  
2. Android Studio  
3. Dart  
4. Flutter

Platform Channelを使用するためのAAR(+so)ファイルビルド環境構築  
VMware Workstation (Ubuntu 64ビット)
1. VSCode  
2. Android Studio  
　　(ndk18.xとcmake3.3xインストール android studioのcmakeを優先的に参照するので必須)

https://github.com/opencv/opencv/wiki/Custom-OpenCV-Android-SDK-and-AAR-package-build  
の指示にしたがって作業をする。  
結果、ビルド失敗する。(一先ず各バージョンを変えて再ビルド)  
まずはJavaSDKのバージョンがデフォルトで21だったので、公式では、  
17をインストールする指示があったので確認して変更  

sudo update-alternatives --config java (21→17)  
sudo update-alternatives --config javac (21→17)

JavaSDKを変えただけではまだビルドに失敗するので、  
メッセージを元にapi-level=21を24に変更(ndk-18-api-level-24.config.py採用)  
build_java_shared_aar.pyのparser.add_argument('--android_min_sdk', default="21")も  
parser.add_argument('--android_min_sdk', default="24")に変更(引数で渡すなら変更不要)  

上記を修正して、再度ビルドで成功しました。

Windows環境でビルドしていた時は、python3ではなくCMakeでビルドしていました。  
これが良くなかったようです。(AI君あ〇～なに〇じてたのに～)
