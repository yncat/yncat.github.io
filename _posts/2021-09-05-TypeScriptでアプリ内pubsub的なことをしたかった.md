---
layout: post
tags: typescript

---

# intro

TypeScript ぱねえ便利という話です。

今、オンラインゲームのフロントエンドを作ってます。で、サーバーからとあるメッセージが来たら、複数の画面を色々いじりたくなりました。

今回、画面表示と通信部分のロジックを完全に分離しているので、なんか起きたことをフロントのコンポーネントが知るために、 subscriber 的な関数をロジックに渡して、それを呼んでもらうようにしていました。こんな感じ。

```
export default function ChatMessageList(props: Props) {
  const [messageList, setMessageList] = React.useState<ChatMessage[]>([]);
  React.useEffect(() => {
    props.chatMessageListLogic.subscribe((chatMessageList: ChatMessage[]) => {
      setMessageList(chatMessageList);
      // ほんとはここで音を鳴らしたりとかしてる
    return () => {
      props.chatMessageListLogic.unsubscribe();
    };
  }, []);
```

でも、これだけだといくつか問題があることに気づきました。まず、関数を1個しか渡せないので、特定のイベントに複数のコンポーネントが subscribe したいときに詰んでました。というか、それで詰んだので、今回のちょい型パズルをやることにしました。

次に、イベントの種類がいっぱい増えていくルと、いちいち subscribe なんとかとか unsubscribe なんとかとかいうメソッドを作るのがいやになってきます。

最後に、 subscribe はあとからでもできる想定なので、 subscriber を保持しておく場所は、 `subscriber:SubscriberFunc|null;` みたいに定義するしかないです。こうすると、いちいち constructor で `this.ogehogeSubscriber = null;` みたいに書かなきゃいけないです。だるいです。しかも、 subscriber を呼ぶときに、いちいち null チェックをしないと TypeScript が「nullかもよ、ヌルポするかもよ危ないよねえねえマズいんじゃない？てかマズいよ直しなさいよ」って怒ります。当然ですね。やっぱりだるいです。

# オレオレ汎用pubsub的なやつを作った

本当は、 pub/sub ってもっと登場人物が多いはずなんですが、まあやってることは似てるので、オレオレpubsubとでも言っておきます。ようは、なんかが更新されたぞってことを通知して、それを知りたいやつは、関数を登録しておいたら、そいつら全員に勝手に教えてくれる便利屋がほしかったということです。ただ、いちいち書くのはだるいので、いい感じに汎用化します。

## 登録と削除 (subscribe / unsubscribe)

たとえば、プレイヤーの人数の増減イベントだったら、たぶん通知されるのは number になるでしょう。はたまた、チャットが来たぜっていうイベントだったら、通知されるのは string になるでしょう。だれかがどっかのルームに入ったぴょんっていうイベントだったら、通知されるのは、プレイヤー名の string みたいなやつと、ルームの識別子的な number とかになるでしょう。こいつらも全部ひっくるめて一般化したいです。

でも、これはまあ簡単ですね。クラス定義の時にこんな感じにして、あとは、必要なところで T を参照して、関数の型に置き換えればいいだけです。

```
export class Pubsub<T extends (...args: any) => void> {
  public subscribe(subscriber:T){
    // なんかいい感じに登録する
  }
}
```

これで、関数の型を渡すと、勝手にそれ用の pubsub オブジェクトを作ってくれるようになります。

たとえば、プレイヤーの人数を処理刷るpubsubを作りたかったら、事前にこんな感じでタイプを定義してあげて

```
type PlayerCountSubscriber = (newPlayerCount:number)=>void;
```

pubsubくんをこうやって作れば

```
const playerCountPubsub = new Pubsub<PlayerCountSubscriber>();
```

subscribe の型は PlayerCountSubscriber になるので、 playerCountPubsub.subscribe と打ったら、 vscode がちゃーんと subscriber:PlayerCountSubscriber とヒントを出してくれます。

最初、 `<T extends Function>` にしてたんですが、これだとだめっぽかったです。このあと、 `Parameters<T>` っていう組み込みの generics を使うんですが、それが `<T extends (...args: any) => any>` としてたので、それを拝借してきました。最後を any から void に変えたのは、基本的に subscriber から値を返すことはないからです。

unsubscribe は書いてないけど、結局同じです。

## publish がちょっとめんどかった

次に、 publish を呼んで、そこに更新情報を載せたら、 subscribe してる人たちに全員通知が行くというやつを作ります。

書き始めるまで忘れてたんですが、 publish を作るということは、更新情報の引数を一般化できる必要があります。さっきの例でいえば、プレイヤーの人数だったら、 publish の引数は number 1個になってほしいし、ルーム入ったぜっていうやつだったら、 publish の引数は string と number になってほしいです。これ一筋縄じゃいかなくねって思って調べたら、 `Parameters<T>` とかいう組み込みの generics がありました。これの中身がどうなってるかは知りません。たぶん俺は読んでも理解できないです。

`Parameters<T>` は、関数のタイプを渡すと、それの引数を、型のタプルにして返してくれるらしいです。

TypeScriptでは、型のタプルがあるとき、 `...` を使えば、それを引数に転回することができます。これを使うと、なんとさっきのことそのままが実現しちゃいます。こんな感じ。

```
public publish(...args: Parameters<T>) {
  // 関数を呼ぶときに func(...args) で渡せばこれも分割代入で引数リストに転回できる
}
```

こうするとですね、なんと、型が違おうが引数の数が違おうが、 subscriber の関数の型定義さえあれば、 publish の引数リストの型を導出できちゃいます、でもって、 vscode でも思った通りのヒントを出してくれちゃいます!!!!!!!!

TypeScript を使いこなしまくっている人にとっては序の口なんでしょうが、これは便利だ。感動しちゃったわ。

## 完成品
```
export class Pubsub<T extends (...args: any) => void> {
  private readonly subscribersMap: Map<number, T>;
  private internalCount: number;
  constructor() {
    this.subscribersMap = new Map<number, T>();
    this.internalCount = 0;
  }

  public subscribe(func: T): number {
    this.internalCount++;
    this.subscribersMap.set(this.internalCount, func);
    return this.internalCount;
  }

  public unsubscribe(id: number): boolean {
    if (!this.subscribersMap.has(id)) {
      return false;
    }
    this.subscribersMap.delete(id);
    return true;
  }

  public publish(...args: Parameters<T>) {
    this.subscribersMap.forEach((f) => {
      f(...args);
    });
  }
}
```

subscribe したら、 id が帰ってきます。この id を指定して unsubscribe することができます。 Map を使っているのは、 unsubscribe で、他の subscriber の id がずれないようにするためです。( array を使うとずれちゃいます)

## 動作確認

```
import { Pubsub } from "./pubsub";

type TestSubscriberFunc = (num: number) => void;

function makeFuncs() {
  const f1 = jest.fn((num: number) => {});
  const f2 = jest.fn((num: number) => {});
  return [f1, f2];
}

describe("pubsub", () => {
  it("can get subscriber indices", () => {
    const pubsub = new Pubsub<TestSubscriberFunc>();
    const [f1, f2] = makeFuncs();
    expect(pubsub.subscribe(f1)).toBe(1);
    expect(pubsub.subscribe(f2)).toBe(2);
  });

  it("calls all subscribers when published", () => {
    const pubsub = new Pubsub<TestSubscriberFunc>();
    const [f1, f2] = makeFuncs();
    pubsub.subscribe(f1);
    pubsub.subscribe(f2);
    pubsub.publish(1);
    expect(f1).toHaveBeenCalledWith(1);
    expect(f2).toHaveBeenCalledWith(1);
  });

  it("does not call unsubscribed function", () => {
    const pubsub = new Pubsub<TestSubscriberFunc>();
    const [f1, f2] = makeFuncs();
    pubsub.subscribe(f1);
    const f2id = pubsub.subscribe(f2);
    pubsub.unsubscribe(f2id);
    pubsub.publish(1);
    expect(f1).toHaveBeenCalledWith(1);
    expect(f2).not.toHaveBeenCalled();
  });
});
```

## これで楽になるぞたぶん
まだ実際にアプリに入れてないんですが、今までよりは楽になるはず。
