# Goでenumのswitchをcheckできるツールを作りました

Goでenumのswitchをcheckできるツール [switchecker](https://github.com/bannzai/switchecker) を作りました。

## switchecker
**switchecker** は Goのソースコード内でswitch文で `enum` を用いた値のパターンマッチングにおいて `enum` で宣言されている `case` をすべて網羅できているかをチェックしてくれるツールです。
と、ここまで言いましたが、`Go` で `enum` ？と思いますよね。厳密には`Go`で`enum`は存在しないです。ここで言っている

