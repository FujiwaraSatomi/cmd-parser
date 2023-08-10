# cmd-parser

cmd-parserは、スペース区切りで構成されたコマンド (例: `!say username msg`) のメッセージをパースするために便利なライブラリです。\
主にWebSocketやircのようなチャットで役に立ちます。

## 推奨環境

Node.js v18.16.1 以上

## 簡単な使い方

まず、cmd-parserをインポートします。
```js
import CmdParser from "cmd-parser";
```
次に、インスタンスを作成します。
```js
const parser = new CmdParser();
```
そして、コマンドを設定します (詳しくは下記) 。
```js
parser.set(...);
```
最後に、メッセージがコマンドを使用しているかを判定します (詳しくは下記) 。
```js
parser.parse(...);
```

## コマンドの設定

`parser.set`の引数にはオブジェクトを指定し、その中で接頭辞、コマンド等を指定します。

例えば、接頭辞が`!`のコマンドを作りたいときは以下のようにします。\
```js
parser.set({
  prefix: "!"
})
```
※以下`parser.set()`を省略します。

コマンドを設定します。\
例えば、`!info`というコマンドを作りたい場合、
```js
{
  prefix: "!",
  commands: [
    {
      name: "info",
      description: "コマンドの概要"
    }
  ]
}
```
のようにします。\
具体的には、commandsに配列を作り、配列の中のオブジェクトに、nameはコマンド名を、descriptionにはコマンドの概要を表示します。descriptionは省略可能です。\
複数コマンドを作りたい場合は、このようなオブジェクトを複数指定します。ただし、同じコマンド名は複数指定できません。

そして、コマンドを実行したときにメッセージが帰るようにするには、callback関数を指定します。\
returnしたものが、`parser.parse`の返値で帰ります (returnせずにcallback内で完結する方法は下記で説明します) 。
```js
{
  prefix: "!",
  commands: [
    {
      name: "info",
      description: "コマンドの概要",
      callback: () => {
        return "このボットは、コマンドをパースします。";
      }
    }
  ]
}
```
そのコマンドが静的なレスポンスしか返さないのなら、単純に文字列だけでもいけます。これは上記と同じ動作をします。
```js
callback: "このボットは、コマンドをパースします。"
```

次に、コマンドに引数を持たせるようにします。\
例えば、`!say message`などのように、指定したメッセージをオウム返しするコマンドを作るとします。\
結論から言えば以下のようになります。
```js
{
  prefix: "!",
  commands: [
    {
      name: "say",
      description: "メッセージをオウム返しします",
      options: [
        {
          name: "message",
          type: "string",
          description: "オウム返しするメッセージです",
          required: true
        }
      ],
      callback: (options) => {
        return options.getString("message");
      }
    }
  ]
}
```
まず、引数を指定できるようにするには、optionsに配列を作り、その中のオブジェクトに、nameは引数の名前、typeは引数の型、descriptionは引数の説明、requiredはその引数は必須かどうかを指定します。\
descriptionとrequiredは省略可能です。

### typeの説明

引数の型は5つあります。
| 型 | 説明 |
| :-: | :-: | :-: |
| string | 文字列を指定します |
| number | 数値を指定します |
| bool | 真偽値を指定します |
| select | 複数の値の中の一つを指定します |
| subcmds | サブコマンドを指定します |
