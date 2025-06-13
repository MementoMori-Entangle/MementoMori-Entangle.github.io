---
layout: post
title: "Flutter gRPC関連覚え書き"
date: 2025-06-13 00:00:00 +0900
---

FlutterでgRPCをmTLS使用する場合の注意点  
gRPCのmTLS、TLSに対応していないのでWebは  
プロキシー(grpcwebproxyなど)、  
Windowsはブリッジ(C#.net自作)、  
AndroidはPlatform Channelで対応する必要がある。

Webは  
[Flutter Web] --gRPC-Web/HTTP1.1-->   
[grpcwebproxy:8080] --gRPC/HTTP2-->   
[Spring Boot gRPC:9090]  
のようにプロキシーを通して処理する必要がある。  
サーバー側で設定するサーバー証明書を作成する時、  
SAN付きにしないと認証できないので注意

クライアントHTTPの場合  
--run_tls_server=false

クライアントHTTPSの場合  
--run_tls_server=true  
--server_tls_cert_file=プロキシー用client.crt  
--server_tls_key_file=プロキシー用client.key  
--server_http_tls_port=HTTPSポート(9443など被らない数値)指定

HTTPSの場合は、証明書が恒久ではない場合、  
初回ブラウザでhttps://localhost:9443/  
などにアクセスして信用許可  
(または証明書のインストール、セキュリティ的注意)

WindowsはネイティブアプリでブリッジしてmTLS対応する。  
(proto定義変更で対応が必要)

パイプモードか標準入出力モードのどちらかでjsonの送受信を想定  
Flutterでパイプ送受信は面倒なので、標準入出力で処理

Androidのproto自動生成をする場合、build.gradle.ktsなどに  
記述する際、debug、release、profile毎にsrcDirsの設定を  
定義しないとパスが認識できない場合がある。  
task.builtinsに、javaとgrpcの指定も忘れないこと

android {  
	sourceSets {  
        getByName("main") {  
            java.srcDirs(  
                "src/main/kotlin",  
                "src/main/java",  
                "build/app/generated/source/proto/main/java",  
                "build/app/generated/source/proto/main/grpc"  
            )  
        }  
        getByName("debug") {  
            java.srcDirs(  
                "src/main/kotlin",  
                "src/main/java",  
                "build/app/generated/source/proto/main/java",  
                "build/app/generated/source/proto/main/grpc"  
            )  
        }  
        getByName("release") {  
            java.srcDirs(  
                "src/main/kotlin",  
                "src/main/java",  
                "build/app/generated/source/proto/main/java",  
                "build/app/generated/source/proto/main/grpc"  
            )  
        }  
        getByName("profile") {  
            java.srcDirs(  
                "src/main/kotlin",  
                "src/main/java",  
                "build/app/generated/source/proto/main/java",  
                "build/app/generated/source/proto/main/grpc"  
            )  
        }  

        ...

		generateProtoTasks {  
	        all().forEach { task ->  
	            task.builtins {  
	                create("java")  
	            }  
	            task.plugins {  
	                create("grpc")  
	            }  
	        }  
	    }  

Androidビルドするとき、コード難読化で影響が出る場合は、  
proguard-rules.proファイルで定義すること

	buildTypes {  
        release {  
			isMinifyEnabled = true  
            proguardFiles(  
                getDefaultProguardFile("proguard-android-optimize.txt"),  
                "proguard-rules.pro"  
            )  
        }  
    }

protoのベース(クライアント、サーバーの参照元)からコピーして、  
タスク処理する定義は、register、named、afterEvaluateで  
リストでfindByNameする。以下例

tasks.register<Copy>("copyProto") {  
    from("../../../../proto/analysis_service.proto")  
    into("src/main/proto")  
}

tasks.named("preBuild") {  
    dependsOn("copyProto")  
}

afterEvaluate {  
    listOf("generateDebugProto", "generateReleaseProto",   
           "generateProfileProto").forEach { taskName ->  
        tasks.findByName(taskName)?.dependsOn("copyProto")  
    }  
}

# 課題
androidでCA証明書をassets経由で読み込むとき、なぜかBOM付になる!?  
参照元ファイルにはBOMは付いていない。  
エミュレータ環境依存なのか、使用しているイメージやAPI、SDKなどの依存問題なのか、  
原因が変わらない。AIや各技術系サイトでも同じ現象が発生していることが確認できない。  
一先ず、リソースやダウンロードしてストレージから読み込んでBOMが付いてるか要確認

2025/06/14追記  
　→ androidエミュレータのCold Bootしたら、BOM消えてました。(謎)

BOMの問題は解決?しましたが、今度は別の問題が発生  
NettyではmTLSを実装するのが難しいようなので、  
gRPC-OkHttpを使用して実装する必要があるようです。  
また、gRPCサーバーはPCのローカルホストで稼働しているため、    
androidエミュレータからは10.0.2.2(PCのローカルホスト)で接続する。  
証明書のSANに「IP.1 = 10.0.2.2」が必須です。  
これを追加しないと、TLSハンドシェイク時に「証明書のホスト名が一致しない」ため接続できません。
