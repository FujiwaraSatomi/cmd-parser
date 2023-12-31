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
※以下`parser.set()`を省略します。\
接頭辞は、コマンド名との衝突を避けるため、基本的に記号を指定してください。

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
具体的には、commandsに配列を作り、配列の中のオブジェクトに、nameはコマンド名を、descriptionにはコマンドの概要を指定します。descriptionは省略可能です。

複数のコマンドを作りたい場合は、このようなオブジェクトを複数指定します。以下のような感じです。
```js
{
  prefix: "!",
  commands: [
    {
      name: "command1",
      description: "コマンド1の概要"
    },
    {
      name: "command2",
      description: "コマンド2の概要"
    }
  ]
}
```
これで、`!command1`と`!command2`というコマンドが使えるようになります。ただし、同じコマンド名は複数指定できません。

そして、コマンドを実行したときにメッセージが帰るようにするには、callback関数を指定します。\
returnしたものが、`parser.parse`の返値で帰ります (returnせずにcallback内で完結する方法は下記で説明します) 。
```js
parser.set({
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
});
console.log(parser.parse("!info")); // "このボットは、コマンドをパースします。"
```

次に、コマンドに引数を持たせるようにします。\
例えば、`!say <message>`で、指定したメッセージをオウム返しするコマンドを作るとします。\
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
      callback: (err, options) => {
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
| :-: | :-: |
| string | 文字列を指定します any型を使用したい場合はこの型を使用してください |
| integer | 整数値を指定します 小数値が指定された場合は、整数未満を切り捨てます |
| number | 小数値を指定します |
| bool | 真偽値を指定します |
| select | 複数の値から一つを指定します |
| subcmd | サブコマンドを指定します |
| group | 複数の引数がまとめられたグループを指定します groupの中にgroupを入れることはできません |

### プロパティの説明
| プロパティ | データ型 | 説明 | 対応する型 |
| :-: | :-: | :-: | :-: |
| match | `RegExp` | 正規表現に一致する文字列のみを受け付けます | `string` |
| minlength | `Number` | 文字列の最小の長さを指定します | `string` |
| maxlength | `Number` | 文字列の最大の長さを指定します | `string` |
| min | `Number` | 数値の最小値を指定します | `integer` `number` |
| max | `Number` | 数値の最大値を指定します | `integer` `number` |
| maxpoint | `Number` | 小数点以下の最大桁数を指定します もし最大桁数を超えていたら、その桁未満は切り捨てられます | `number` |
| values | `Object` | trueやfalseの場合の名前を指定します | `bool` |
| choices | `Array` | 受け付ける値を指定します | `select` |
| commands | `Array` | サブコマンドを指定します | `subcmd` |
| options | `Array` | 引数を指定します | `group` |
| variadic | `Boolean` | 引数を可変長引数にするかを指定します | `string` `integer` `number` `bool` `select` |
| maxarglength | `Number` | 可変長引数にした時の最大の個数を指定します | `string` `integer` `number` `bool` `select` |
| required | `Boolean` | 引数を必須にします | すべて |
| define | `Array` | optionを指定します のちにoptionsで使用することができます | `subcmd` |
| use | `String` | nameを指定するとdefineで定義したoptionを使用することができます | なし |

※`required = false`は、引数を「省略可能」ではなく、「空文字にすることができる」だということに注意してください。例えば、`!test [arg1] <arg2>`というコマンドなら、`!test arg2`ではなく、`!test  arg2`と、区切り分のスペースを入れる必要があります。

### callbackの引数
```js
callback: (err, options, raw, command) => {
  
}
```
- `err` - エラーであるかどうか、またその詳細を取得します。
  - `err.is_err` - エラーであるかどうかです。
  - `err.option` - エラーがある箇所のオプションのオブジェクトを取得します。
    `err.option.value`にはエラーとなった文字列 (どの型であっても) が入ります。
  - `err.type` - どのようなエラーが起こったかを数値として取得します。
  - `err.type_p` - 大体、どのようなエラーが起ったのかを数値として取得します。
    詳しくは下記
- `options` - コマンドの引数を取得します。
  `options.getString()` - 文字列の引数を取得します。このメソッドの第一引数には`name`を指定します (以下省略)。返値は`String`です。
  `options.getInteger()` - 小数値の引数を取得します。返値は`Number`です。
  `options.getNumber()` - 整数値の引数を取得します。返値は`Number`です。
  `options.getBool()` - 真偽値の引数を取得します。返値は`Boolean`です。
  `options.getSelect()` - 選択式の引数を取得します。返値は`String`です。
  `options.getSubcmd()` - サブコマンドの引数を取得します。返値は`String`です。
  `options.getAll()` - すべての引数を取得します。
  ※可変長引数の場合はすべての型で配列が返ります。
- `raw` - parseの引数に指定した生のメッセージを取得します。
  `raw.getArgs()` - メッセージを引数で分けて配列にします。詳細を取得したい場合は`options.getAll()`を使用してください。
- `command` - nameやdescriptionがあるコマンドのオブジェクトを取得します。

#### `err.type`
| エラー番号 | 説明 |
| :-: | :-: |
| 0 | エラーはありません |
| 1 | matchにマッチしていません |
| 2 | 文字数がminlengthより少ないです |
| 3 | 文字数がmaxlengthより多いです |
| 4 | 有効な数値ではありません |
| 5 | 数値がminよりも小さいです |
| 6 | 数値がmaxよりも大きいです |
| 7 | boolで、存在しない値が指定されています |
| 8 | selectで、存在しない値が指定されています |
| 9 | 存在しないサブコマンドです |
| 10 | 必須な引数には何も指定されていません |

#### `err.type_p`
| タイプ | 説明 |
| :-: | :-: |
| 0 | エラーはありません |
| 1 | matchにマッチしていません |
| 2 | 文字数がminlengthより少ないか、maxlengthより大きいです |
| 3 | 数値がminより小さいか、maxより大きいです |
| 4 | 存在しない値やサブコマンドが指定されています |
| 5 | 必須な引数には何も指定されていません |

### ignore
引数のエラーがある場合、ignoreに数値を指定するとその数値に対応したエラーは無視され、デフォルトのエラーが出力されます。\
例えば、以下のようなことを考えます。
```js
{
  prefix: "!",
  commands: [
    {
      name: "test",
      description: "引数のテストです",
      options: [
        {
          name: "arg",
          type: "string",
          required: true,
        }
      ],
      callback: (err) => {
        if(err.type == 10) return "引数argは必須です";
        return "テストを通過しました";
      }
    }
  ]
}
```
```js
parser.parse("!test"); // "引数argは必須です"
```
この例は、必須の引数が指定されてなかったときにcallbackでエラーを解消させます。\
ただ、デフォルトにもエラーを解消させるコードはあります。\
以下のように変更します。
```js
{
  name: "test",
  description: "引数のテストです",
  options: [
    {
      name: "arg",
      type: "string",
      required: true,
    }
  ],
  ignore: [10],
  callback: (err) => {
    return "テストを通過しました";
  }
}
```
```js
parser.parse("!test"); // "String型の引数'arg'は必須です"
parser.parse("!test a"); // "テストを通過しました"
```
この例では、エラー番号10を無視し、callbackを発生させないようにし、デフォルトのエラーを発生させています。\
すべてのエラーを無視するには、
```js
ignore: parser.all
```
と指定します。

#### デフォルトのエラーを指定する
デフォルトのエラーを指定することもできます。\
例えば、以下のように変更できます。
```js
{
  prefix: "!",
  errors: {
    10: (type, name) => {
      return `引数${name}は必須で、${type}型を指定する必要があります`
    }
  },
  commands: [
    {
      name: "test",
      description: "引数のテストです",
      options: [
        {
          name: "arg",
          type: "string",
          required: true,
        }
      ],
      ignore: [10],
      callback: () => {
        return "テストを通過しました";
      }
    }
  ]
}
```
```js
parser.parse("!test"); // "引数argは必須で、String型を指定する必要があります"
```
具体的には、最初のオブジェクトにerrorsオブジェクトを作り、対応する番号をプロパティ名として関数を指定するだけです。※エラー番号0は指定できません。

| エラー番号 | デフォルトのエラーの形式 |
| :-: | :-: |
| 1 | `${type}型の引数'${name}'は指定された形式にマッチしていません` |
| 2 | `${type}型の引数'${name}'は${arg}文字より長くする必要があります` |
| 3 | `${type}型の引数'${name}'は${arg}文字より短くする必要があります` |
| 4 | `${type}型の引数'${name}'は有効な数値の形式ではありません`
| 5 | `${type}型の引数'${name}'は${arg}以上にする必要があります` |
| 6 | `${type}型の引数'${name}'は${arg}以下にする必要があります` |
| 7 | `${type}型の引数'${name}'は指定された値にする必要があります` |
| 8 | `${type}型の引数'${name}'は指定された値にする必要があります` |
| 9 | `${type}型の引数'${name}'にはそのようなサブコマンドはありません` |
| 10 | `${type}型の引数'${name}'は必須です` |

```js
errors: {
  n: (type, name, arg, value) => {
    
  }
}
```

| 引数 | 説明 |
| :-: | :-: |
| n | エラー番号です (上記参照) |
| type | 型の名前です |
| name | 引数の名前です |
| arg | エラーに対応した条件の値です (下記参照) |
| value | 実際に指定された値です。必須の引数に指定されていなかった場合でも、空文字(`""`)で返ります |

#### argに格納される値
| エラー番号 | プロパティ |
| :-: | :-: |
| 1 | match |
| 2 | minlength |
| 3 | maxlength |
| 4 | min |
| 5 | max |
| 6 | values (`{ true: name, false: name }`が格納されている) |
| 7 | choices (`[ name, name, ... ]`が格納されている) |
| 8 | commands (`[ name, name, ... ]`が格納されている) |
| 9 | `null` |

### 具体的な仕様 (BNF風)
```js
parser.set(<settings>);
```
```
<settings> → {
               prefix: <prefix>,
               [helpcmd: <helpcmd>,]
               [errors: <errors>,]
               commands: <commands>
             }
<prefix> → <string-witout-spaces>
<helpcmd> → <string-alphanumeric>
<string-witout-spaces> → (<character> - " ")*
<string-alphanumeric> → (a..z | A..Z | 0..9)+
<errors> → {
             (<error-code>: <error-format>,)*
           }
<error-code> → 1..9
<error-format> → (type, name, arg, value) => {
                   return <string>
                 }
<commands> → [
               (<command>,)*
             ]
<command> → {
              name: <string-witout-spaces>,
              [description: <string>,]
              [options: <options>,]
              [ignore: <ignore>,]
              [callback: <callback>]
            }
<options> → [
              (<option>,)*
            ]
<ignore> → [
             (<error-code>,)*
           ]
<callback> → (err, options, raw, command) => {
               return <any>
             }
<option> → {
             [use: <string>,]
             [name: <string-witout-spaces>,]
             [description: <string|function>,]
             [type: <type>,]
             [define: <options>,]
             [match: <match>,]
             [minlength: <number>,]
             [maxlength: <number>,]
             [min: <number>,]
             [max: <number>,]
             [maxpoint: <number>,]
             [values: <values>,]
             [choices: <choices>,]
             [commands: <commands>,]
             [options: <options>,]
             [maxarglength: <number>,]
             [required: <boolean>]
           }
<type> → string | integer | number | bool | select | subcmd | group
<values> → {
             true: <value>,
             false: <value>
           }
<value> → {
            name: <name>
          }
<choices> → [
              (<choice>,)*
            ]
<choice> → {
             name: <string-witout-spaces>,
             [value: <string>]
           }
```
```
<settings> → Object
<prefix> → String
<helpcmd> → String
<errors> → Object
<error-code> → Number
<error-format> → Function
<commands> → Array
<command> → Object
<name> → String
<description> → String
<string-witout-spaces> → String
<string-alphanumeric> → String
<options> → Array
<option> → Object
<ignore> → Array
<callback> → Function
<type> → String
<values> → Object
<value> → Object
<choices> → Array
<choice> → Object

<string> → String
<number> → Number
<boolean> → Boolean
```

### 引数指定の方法
ここからは、実際に例を交えて説明していきます。

#### 特定の形式の文字しか受け付けない
この例は、0-9とa-fのみで構成された10文字の文字列であるかを判定するコマンドです。
```js
{
  prefix: "!",
  commands: [
    {
      name: "hash",
      description: "ハッシュかどうかを判定します",
      options: [
        {
          name: "hash",
          type: "string",
          match: /^[a-f0-9]{10}$/,
          required: true,
        }
      ],
      ignore: [10],
      callback: (err) => {
        if(err.is_err) return "ハッシュではありません";
        return "ハッシュです";
      }
    }
  ]
}
```
```js
parser.parse("!hash f7be91b38a"); // "ハッシュです"
parser.parse("!hash test"); // "ハッシュではありません"
```

#### 最小文字数と最大文字数を指定する
この例では、3文字以上15文字以下の文字列であるかを判定するコマンドです。
```js
{
  prefix: "!",
  commands: [
    {
      name: "name",
      description: "ユーザー名が指定した長さあるかを判定します",
      options: [
        {
          name: "username",
          type: "string",
          minlength: 3,
          maxlength: 15,
          required: true,
        }
      ],
      ignore: [10],
      callback: (err) => {
        if(err.type == 2) return "ユーザー名が短すぎます";
        if(err.type == 3) return "ユーザー名が長すぎます";
        return "ユーザー名は指定した長さあります";
      }
    }
  ]
}
```
```js
parser.parse("!name a"); // "ユーザー名が短すぎます"
parser.parse("!name yuito2525"); // "ユーザー名は指定した長さあります"
```
