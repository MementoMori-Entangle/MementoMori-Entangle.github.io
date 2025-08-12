---
layout: post
title: "next.js学習"
date: 2025-08-13 10:00:00 +0900
---

https://nextjs.org/learn/ で  
1. React Foundations  
2. App Router  
3. Pages Router  
4. SEO

の4つを全て受けましたが、  
そこで"App Router"の第6章で問題がありましたので書き留めておく

nextjs-dashboard\app\seed\route.ts

const result = await sql.begin((sql) => [  
　seedUsers(),  
　seedCustomers(),  
　seedInvoices(),  
　seedRevenue(),  
]);

// 上記4つ関数で定義  
await sql`CREATE EXTENSION IF NOT EXISTS "uuid-ossp"`;

sql.beginは1つのトランザクションで並列処理するため、  
"CREATE EXTENSION IF NOT EXISTS "uuid-ossp"が同時に実行されることがあるので  
return Response.json({ error }, { status: 500 });が呼ばれることがある。  
マルチスレッドを考慮していない実装

その他にtry {} catch ()でnotFound追加部分や  
state.messageに関する部分は初心者には不親切?
