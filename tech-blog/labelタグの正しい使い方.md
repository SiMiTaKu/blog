<label>について

<label>は単体で使用するのはHTML構文的に不適切。<input>や<textarea>に紐づけて使用するのが正しい。

<input>1に対し<label>は1以上関連づけられる。（複数可能）

<label>1に対し<input>は1つしか関連づけれない（単数のみ）

関連づけれるタグ

<button>

<input>

<meta> 関連付けてどうするんやろ？

<output> このタグあんま知らん。

<progress> このタグあんま知らん。

<select>

<textarea>

<label>をつけるメリット

視覚的にも・プログラム的にもフォームの要素に紐づけられる。

（例）フォーカス変更時でも読み上げソフトが読み上げてくれるようになる。

入力欄クリック時と同じようにクリックすると入力欄にフォーカスが当たるようになる。

紐づける方法

内包型



<label>
  テキスト
  <input type="text" name="text" />
</label>
この場合は<label>のfor属性と<input>のid属性によるの関連付けは省略可能

非内包型



<label　for="text">テキスト</label>
<input id="text" type="text" name="text" />
上記を調べたことによりわかったこと

radioやcheckboxは選択肢それぞれを<label><input>で表現しているためグルーピング全体を<label>で表すのは不適切である。

なぜならグルーピング全体を表す<label>は紐づく対象が存在しないため

radioとcheckboxをグルーピングしたものの見出しはどう表現すればいいの？

<fieldset>と<legend>を使う

（この二つはしっかり調べきれていないため調べる予定）