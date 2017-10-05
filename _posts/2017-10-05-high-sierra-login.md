---
layout: post
title: "macOS 10.13 High Sierraのログイン画面で、Othersが表示されるのを直す方法"
date: 2017-10-05
categories: development
---

# Others

High Sierraにアップグレード後、ログイン画面に自分のユーザー以外に「Others」という選択肢が表示されるようになりました。

ゲストログインはオフにしていたのでゲストではありません。

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"
     style="display:block; text-align:center;"
     data-ad-layout="in-article"
     data-ad-format="fluid"
     data-ad-client="ca-pub-3655474149264343"
     data-ad-slot="9606645212"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>


結論をかくと、rootユーザーがなぜか有効化されていました。

修正方法は

- システム環境設定を開く
- ユーザー&グループを開く
- 鍵マークを押してロックを解除する
- ログインオプションのネットワークアカウントサーバーのJoinをおす
- Open Disk Utilityをおす
- 鍵マークを押してロックを解除する
- メニューバーのEditからDisable Root Userを選択する

これでOthersが表示されなくなりました。

