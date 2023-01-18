---
title: "Block（基本編）"
---

::: message alert
本書では`modId`を`examplemod`、`displayName`を`ExampleMod`としています。適宜ご自分の環境に合わせて読み替えてください。
:::

# `Block`と`BlockItem`とは
`Block`はワールド上に配置されているブロックです。基本的な`Block`は1x1x1のもので、インベントリに格納できないものを指します。注意としてブロックをインベントリに格納でき、再設置を可能にしているのは`BlockItem`です。`BlockItem`は`Block`と`Item`を繋ぐ役割を持つものです。この「基本編」では特に機能を持たない簡単な`Block`をMODに追加します。

# MODへの追加について
`Item`と同様に二種類の方法が存在します。詳しくは[Item（基本編）#MODへの追加について](https://zenn.dev/cyber_hacnosuke/books/minecraft-modding/viewer/basic-item#mod%E3%81%B8%E3%81%AE%E8%BF%BD%E5%8A%A0%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)をご覧ください。

# `Block`を登録する
`BlockInit.java`をお好きな場所に生成してください。`BlockInit`では`Block`をまとめて登録したり管理します。

::: message
ここでは`DeferredRegister`を使用した方法のみ取り扱います。`RegisterEvent`を使用したやり方については[Item（基本編）##RegisterEventを使う](https://zenn.dev/cyber_hacnosuke/books/minecraft-modding/viewer/basic-item#registerevent%E3%82%92%E4%BD%BF%E3%81%86)が参考になるかと思います。
:::

```java:BlockInit.java
// 前略

public class BlockInit {
    public static final DeferredRegister<Block> BLOCKS =
              DeferredRegister.create(ForgeRegistries.BLOCKS, MODID);
    public static final RegistryObject<Block> EXAMPLE_BLOCK =
              BLOCKS.register("example_block", () -> new Item(new Item.Properties().tab(CreativeModeTab.TAB_MISC)));
}
```

`Item`と同様に`ExampleMod.java`でイベントを追加します。

```java:ExampleMod.java
// 前略

@Mod(ExampleMod.MODID)
public class ExampleMod {

    // 中略

    public ExampleMod() {
        IEventBus bus = FMLJavaModLoadingContext.get().getModEventBus();
        ItemBlock.BLOCKS.register(bus);
    }
}
```

# `BlockItem`を登録する
[`Block`と`BlockItem`とは](#blockとblockitemとは)で書いた通り、ブロックを手に持ち、配置できるようにするには`Block`を`Item`として扱う`BlockItem`が必要になります。ただこれは`Block`の登録と同程度ソースコードを書く必要があり、ブロックの数だけ書く必要があります。そこで、`Event`を使って`Block`の登録時に`BlockItem`の登録も済ますようにします。

```java:BlockInit.java
@Mod.EventBusSubscriber(bus = Mod.EventBusSubscriber.Bus.MOD)
public class BlockInit {

    // 中略

    @SubscribeEvent
    public static void onRegisterItems(final RegisterEvent event) {
      if (event.getRegistryKey().equals(ForgeRegistries.Keys.ITEMS)){
          BLOCKS.getEntries().forEach( (blockRegistryObject) -> {
              Block block = blockRegistryObject.get();
              Item.Properties properties = new Item.Properties().tab(new Item.Properties().tab(CreativeModeTab.TAB_MISC));
              Supplier<Item> blockItemFactory = () -> new BlockItem(block, properties);
              event.register(ForgeRegistries.Keys.ITEMS, blockRegistryObject.getId(), blockItemFactory);
          });
      }
  }
}
```

少し長いかもしれませんがやっていることは簡単です。`onRegisterItems`は`Forge`が`Item`を登録する時に実行するメソッドです。`@EventBusSubscriber`環境下のクラスで書かなければならないので一行目を忘れないでください。次に`BLOCKS`に対して一つずつループを行い、`Item.Properties`を準備して関数型インターフェースを使用して登録しています。（関数型インターフェースは高度なJavaの技術ですが、一種のルーティンですからここでは読み飛ばしても構いません。）

# ParchmentMCについて
`Gradle`を使用してダウンロードした`Forge`のライブラリファイルは暗号化されています。変数の名前が`p_<五桁の整数>`となっており、`JavaDoc`が不十分です。`ParchmentMC`は変数名を意味のある本来（あるべき姿）のものに変更され、`JavaDoc`が追加されたソースをインポートしてくれます。`IDEA`を使っている人は`Alt+左クリック`でクラスの定義を見ることができるようになります。こちらの記事で紹介しているのでぜひご覧ください。