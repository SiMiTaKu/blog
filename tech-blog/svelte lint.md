# svelteのeslintおすすめ設定

## 0.はじめに
最近lintの有用性を実感している清水琢巳です。
私は、株式会社ネクストビートで「おもてなしHR」という地方創生に関わる、宿泊業界に特化した転職支援プロダクトの開発をしております。

現在おもてなしHRでは、svelteを採用して開発しています。
その開発する中で、svelteのlint設定について調査をしました。
今回はその結果を共有します。

※ 本記事は[eslint-plugin-svelte](https://github.com/sveltejs/eslint-plugin-svelte?tab=readme-ov-file)［1］を参考にし、私が翻訳した内容を記載しております。そのため言い回しが異なっている場合がありますが、ご了承ください。

## 目次
本記事の構成は以下のとおりです。

1. svelteのlintの導入方法
2. plugin:svelte/recommendedの設定内容
3. 個別に追加したおすすめの設定
4. 僕の中のベスト3
5. まとめ
6. 参考文献
7. 告知

## 1. svelteのlintの導入方法
### 1.1. インストール

インストールは以下のコマンドで行います。

```bash
npm install --save-dev eslint eslint-plugin-svelte svelte
```

### 1.2. recommended設定

以下の設定を`.eslintrc.json`に追加します。これによってplugin:svelte/recommendedの設定が適用されます。

```json
{
  "extends": ["plugin:svelte/recommended"]
}
```

## 2. plugin:svelte/recommendedの設定内容
recommendedは基本的な設定が多く含まれているため、これをベースに設定を追加していくと良いでしょう。
以下にrecommendedで設定されている内容を示します。

|ルール名| 説明                                  |
|---|-------------------------------------|
|[svelte/no-dupe-else-if-blocks](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-dupe-else-if-blocks/)| else ifブロックの重複を禁止します。               |
|[svelte/no-dupe-style-properties](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-dupe-style-properties/)| スタイルプロパティの重複を禁止します。                 |
|[svelte/no-dynamic-slot-name](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-dynamic-slot-name/)| 動的なスロット名を禁止します。                     |
|[svelte/no-not-function-handler](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-not-function-handler/)| 関数でないハンドラーを禁止します。                   |
|[svelte/no-object-in-text-mustaches](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-object-in-text-mustaches/)| テキストマスタッシュ内のオブジェクトを禁止します。           |
|[svelte/no-shorthand-style-property-overrides](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-shorthand-style-property-overrides/)| 省略されたスタイルプロパティでの上書きを禁止します。          |
|[svelte/valid-compile](https://sveltejs.github.io/eslint-plugin-svelte/rules/valid-compile/)| svelteのコンパイラーを使用してソースコードをチェックします。   |
|[svelte/no-at-html-tags](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-at-html-tags/)| HTMLタグ内の@属性を禁止します。                  |
|[svelte/no-at-debug-tags](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-at-debug-tags/)| `@debug`タグを禁止します。                   |
|[svelte/no-unused-svelte-ignore](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-unused-svelte-ignore/)| 不必要な`<!-- svelte-ignore`コメントを禁止します。 |
|[svelte/no-inner-declarations](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-inner-declarations/)| ネストされた関数や変数の宣言を禁止します。               |
|[svelte/comment-directive](https://sveltejs.github.io/eslint-plugin-svelte/rules/comment-directive/)| テンプレートHTMLにeslint-disable機能を提供します。  |
|[svelte/system](https://sveltejs.github.io/eslint-plugin-svelte/rules/system/)| このルールはこのプラグインを動作させるためのシステムルールです。    |

## 3. 個別に追加した設定
ここからは、私が個別に追加した設定を紹介します。
私個人としてはルールはガチガチにしておいて、後からもっと柔軟にしたいなと思ったときに変更するほうが好きなので、「普通そんな書き方しないよ！」というものも含めて設定してあります。

### 3.1 [svelte/no-export-load-in-svelte-module-in-kit-pages](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-export-load-in-svelte-module-in-kit-pages/)
`<script context="module">`内で予期せずエクスポートされたload関数を報告してくれます。
load関数は、ページ用のjsファイルまたはtsファイルでのみ使用されるため、コンポーネントで使用する必要はありません。

```html
<script context="module">
  /* eslint svelte/no-export-load-in-svelte-module-in-kit-pages: "error" */
  /* ✓ GOOD  */
  export function foo() {}
  /* ✗ BAD  */
  export function load() {}
  // export const load = () => {}
</script>
```
### 3.2 [svelte/no-reactive-reassign](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-reactive-reassign/)
このルールは、バインドや再割り当てによって引き起こされる意図しない動作を防いでくれます。
svelteのリアクティブ変数は、割り当てすると再割り当てされた変数がリアクティブ変数として機能しなくなります。

```sveltehtml
<script>
  /* eslint svelte/no-reactive-reassign: "error" */
  let value = 0;
  $: reactiveValue = value * 2;

  function handleClick() {
    /* ✓ GOOD */
    value++;
    /* ✗ BAD */
    reactiveValue = value * 3;
    reactiveValue++;
  }
</script>

<!-- ✓ GOOD -->
<input type="number" bind:value />
<!-- ✗ BAD -->
<input type="number" bind:value={reactiveValue} />
```
### 3.3 [svelte/no-store-async](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-store-async/)
このルールは、svelteストア内でのasync/awaitの使用をすべて報告してくれます。自動購読解除機能（`$:`）で問題が発生するためです。

```typescript
/* eslint svelte/no-store-async: "error" */
import { writable } from 'svelte/store';

/* ✓ GOOD */
const w1 = writable(false, () => {});

/* ✗ BAD */
const w2 = writable(false, async () => {});
```

### 3.4 [svelte/require-store-reactive-access](https://sveltejs.github.io/eslint-plugin-svelte/rules/require-store-reactive-access/)
このルールは、ストア自体をオペランド（非演算子）として使用することを禁止します。ストアの値へのアクセスには、$接頭辞かget関数を使用しなければなりません。

```sveltehtml
/* eslint svelte/require-store-reactive-access: "error" */
<script>
  import { writable, get } from 'svelte/store';
  const storeValue = writable('world');
  /* ✓ GOOD */
  $: message = `Hello ${$storeValue}`;
  /* ✗ BAD */
  $: message = `Hello ${storeValue}`;
</script>

<!-- ✓ GOOD -->
<p>{$storeValue}</p>
<p>{get(storeValue)}</p>
<!-- ✗ BAD -->
<p>{storeValue}</p>
```

### 3.5 [svelte/no-target-blank](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-target-blank/)
このルールは、rel="noopener noreferrer "なしでtarget="_blank "属性を使用することを禁止しています。
「target=”_blank”」によって開かれたページは対策をしていないと、 javascriptを使って親ページ（元ページ）の操作ができてしまうためです。

>target="_blank"を使って新しいリンクを開いた場合、新しいタブからwindow.openerというオブジェクトを用いてリンク元の操作ができてしまいます。
リンク先のサイトが悪意のあるサイトや、ハッキングされているサイトだった場合に、リンク先から自分の見ていたリンク元ページにアクセス・操作される可能性があります。また、フィッシング詐欺に利用されるリスクもあるのです。
openerを否定するnoをつけたnoopenerを使うことで、元ページは操作されません。これにより、サイトのページを不正に改ざんされるのを防ぐことが可能です。[3]

```sveltehtml
/* eslint svelte/no-target-blank: "error" */
<!-- ✓ GOOD -->
<a href="http://example.com" target="_blank" rel="noopener noreferrer">link</a>

<!-- ✗ BAD -->
<a href="http://example.com" target="_blank">link</a>
```

### 3.6 [svelte/block-lang](https://sveltejs.github.io/eslint-plugin-svelte/rules/block-lang/)
このルールは、すべてのsvelteコンポーネントがscriptとstyleに同じ言語を設定することを強制します。
こちらのルールはなくても問題ないですが、コードに統一感を持たせるために設定しています。

```sveltehtml
/* eslint svelte/block-lang: ["error", { "script": "ts", "style": "scss" }] */
<!-- ✓ GOOD -->
<script lang="ts">
</script>

<style lang="scss">
</style>

<!-- ✗ BAD -->
<script>
</script>

<style>
</style>
```

### 3.7 [svelte/button-has-type](https://sveltejs.github.io/eslint-plugin-svelte/rules/button-has-type/)
このルールは、ボタンのtype属性にtypeが使われていないか、無効なtypeが使われている場合に警告を出します。
buttonはデフォルトでsubmitボタンとして動作するため、type属性を指定しないと意図しない動作を引き起こす可能性があります。
また明示的にtype属性を指定することで、ボタンの動作を明確にできるメリットもあります。

```sveltehtml
/* eslint svelte/button-has-type: "error" */
<!-- ✓ GOOD -->
<button type="button">Hello World</button>

<!-- ✗ BAD -->
<button>Hello World</button>
<button type="">Hello World</button>
<button type="foo">Hello World</button>
```

### 3.8 [svelte/no-ignored-unsubscribe](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-ignored-unsubscribe/)
このルールは、`subscribe()`の呼び出しによって返されたサブスクライバーが、変数やプロパティに代入されていないか、関数に渡されていない場合に警告を出します。
ストアは不要になったら、常にサブスクライブを解除すべきです。でなければサブスクリプションが有効なままとなり、メモリーリークの原因となります。

```sveltehtml
<script>
  /* eslint svelte/no-ignored-unsubscribe: "error" */
  import myStore from './my-stores';
  
  /* ✓ GOOD */
  const unsubscribe = myStore.subscribe(() => {});
  
  /* ✗ BAD */
  myStore.subscribe(() => {});
</script>
```

### 3.9 [svelte/no-immutable-reactive-statements](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-immutable-reactive-statements/)
このルールは、リアクティブ・ステートメントで参照されるすべての変数が不変である場合に警告を出します。
リアクティブ変数は、変数が変更されるたびに再評価されるため、不変な変数を使用すると、無駄な再評価が発生します。

```sveltehtml
<script>
  /* eslint svelte/no-immutable-reactive-statements: "error" */
  import myStore from './my-stores';
  import myVar from './my-variables';
  let mutableVar = 'hello';
  export let prop;
  
  /* ✓ GOOD */
  $: computed1 = mutableVar + ' ' + mutableVar;
  $: computed2 = fn1(mutableVar);
  $: console.log(mutableVar);
  $: console.log(computed1);
  $: console.log($myStore);
  $: console.log(prop);
  
  const immutableVar = 'hello';
  
  /* ✗ BAD */
  $: computed3 = fn1(immutableVar);
  $: computed4 = fn2();
  $: console.log(immutableVar);
  $: console.log(myVar);
  
  /** ignore */
  $: console.log(unknown);

  function fn1(v) {
    return v + ' ' + v;
  }
  function fn2() {
    return mutableVar + ' ' + mutableVar;
  }
</script>

<input bind:value={mutableVar} />
```

### 3.10 [svelte/no-reactive-functions](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-reactive-functions/)
このルールは、関数がリアクティブ・ステートメントで定義されるたびに警告を出します。
関数は実行されるたびに、必要な最新の値にアクセスしているので、リアクティブに定義する必要はありません。
リアクティブ文の中で関数を再定義することは、CPUの無駄遣いにしかなりません。

```sveltehtml
<script>
  /* eslint svelte/no-reactive-functions: "error" */

  /* ✓ GOOD */
  const arrowFn = () => {
    /* ... */
  };

  /* ✗ BAD */
  $: arrowFn = () => {
    /* ... */
  };
</script>
```

### 3.11 [svelte/no-reactive-literals](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-reactive-literals/)
このルールは、リアクティブ・ステートメント内での静的で不変な値の代入を報告します。
こちらもリアクティブ変数は、変数が変更されるたびに再評価されるため、不変な値を使用すると、無駄な再評価が発生します。

```sveltehtml
<script>
  /* eslint svelte/no-reactive-literals: "error" */
  /* ✓ GOOD */
  let foo = 'bar';

  /* ✗ BAD */
  $: foo = 'bar';
</script>
```

### 3.12 [svelte/no-unused-class-name](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-unused-class-name/)
このルールは、HTMLテンプレートで使用されていないクラスを報告します。
svelte-checkは、`<style>`ブロックにテンプレートで使用されていないクラスが含まれている場合、css-unused-selectorを生成しますが、このルールはその逆で、`<style>`ブロックで参照されていないクラスがテンプレートに含まれているケースを報告します。

`svelte/valid-compile`で未使用のstyleを警告しているため、相性が良いです。不要なコードは削除していきましょう。

```sveltehtml
/* eslint svelte/no-unused-class-name: "error" */

<!-- ✓ GOOD -->
<div class="first-class">Hello</div>
<div class="second-class third-class">Hello</div>

<!-- ✗ BAD -->
<div class="fifth-class">Hello</div>

<style>
  .first-class, .second-class {
    color: red;
  }
  
  .third-class {
    font-size: 16px;
  }
</style>
```

### 3.13 [svelte/require-each-key](https://sveltejs.github.io/eslint-plugin-svelte/rules/require-each-key/)
このルールはキーのない {#each} ブロックを報告します。
Svelteのデフォルトでは、eachブロックの値を変更すると、ブロックの末尾の項目が追加・削除され、変更された値が更新されます［4］
そのため、キーを指定することで、変更された値のみが更新されるようになります。

```sveltehtml
/* eslint svelte/require-each-key: "error" */
<!-- ✓ GOOD -->
{#each things as thing (thing.id)}
  <Thing name={thing.name} />
{/each}

<!-- ✗ BAD -->
{#each things as thing}
  <Thing name={thing.name} />
{/each}
```

### 3.14 [ svelte/valid-each-key](https://sveltejs.github.io/eslint-plugin-svelte/rules/valid-each-key/)
このルールは{#each}ブロックのキーが{#each}ブロックで、定義された変数名ではない場合に報告します。
eachブロック内でキーは一意である必要があります。そのため、変数名をキーとして使用することが推奨されています。

```sveltehtml
<script>
  /* eslint svelte/valid-each-key: "error" */

  let things = [
    { id: 1, name: 'apple' },
    { id: 2, name: 'banana' },
    { id: 3, name: 'carrot' },
  ];
  let foo = 42;
</script>

<!-- ✓ GOOD -->
{#each things as thing (thing.id)}
  <Thing name={thing.name} />
{/each}

<!-- ✗ BAD -->
{#each things as thing (foo)}
  <Thing name={thing.name} />
{/each}
```

### 3.15 [svelte/html-self-closing](https://sveltejs.github.io/eslint-plugin-svelte/rules/html-self-closing/)
このルールは、HTML要素が自己終了しているかどうかを報告します。中身がない場合は自己終了タグを使うことで、コードがスッキリします。

```sveltehtml
/* eslint svelte/html-self-closing: "error" */
<!-- ✓ GOOD -->
<div />
<p>Hello</p>
<img />
<svelte:head />

<!-- ✗ BAD -->
<div></div>
<p> </p>
<img>
<svelte:body></svelte:body>
```

### 3.16 [svelte/indent](https://sveltejs.github.io/eslint-plugin-svelte/rules/indent/)
このルールは`.svelte`で一貫したインデントスタイルを強制します。デフォルトのスタイルは2スペースです。
やっぱりインデントって大事ですよね。

```sveltehtml
<script>
  /* eslint svelte/indent: "error" */
  function click() {}
</script>

<!-- ✓ GOOD -->
<button
  type="button"
  on:click={click}
  class="my-button primally"
>
  CLICK ME!
</button>

<!-- ✗ BAD -->
<button
type="button"
    on:click={click}
     class="my-button primally"
  >
CLICK ME!
</button>
```

私は以下のように設定しています。

```json
{
  "svelte/indent": [
    "error",
    {
      "indent": 2,
      "ignoredNodes": [],
      "switchCase": 1,
      "alignAttributesVertically": true
    }
  ]
}
```

### 3.17 [svelte/max-attributes-per-line](https://sveltejs.github.io/eslint-plugin-svelte/rules/max-attributes-per-line/)
このルールは、可読性を向上させるために、一行あたりの属性/ディレクティブの最大数を制限します。

```sveltehtml
<script>
  /* eslint svelte/max-attributes-per-line: "error" */
</script>

<!-- ✓ GOOD -->
<input
  type="text"
  bind:value={text}
  {maxlength}
  {...attrs}
  readonly
  size="20"
/>

<!-- ✗ BAD -->
<input type="text" bind:value={text} {maxlength} {...attrs} readonly />
```

こちらデフォルトでは`1`に設定されていますが、私は行が増えすぎるのも嫌なので`2`に設定しました。
ここは個人の好みに合わせて設定してください。

```json
{
  "svelte/max-attributes-per-line": [
    "error",
    {
      "singleline": 2,
      "multiline": 1
    }
  ]
}
```

### 3.18 [svelte/mustache-spacing](https://sveltejs.github.io/eslint-plugin-svelte/rules/mustache-spacing/)
このルールは、マスタッシュ内（中括弧内）のスペースを統一します。

```sveltehtml
/* eslint svelte/mustache-spacing: "error" */
<!-- ✓ GOOD -->
{name}
<input {id} {...attrs} />
{@html page}
{#if c1}...{:else if c2}...{:else}...{/if}
{#each list as item}...{/each}

<!-- ✗ BAD -->
{ name }
<input { id } { ...attrs } />
{ @html page }
{ #if c1 }...{ :else if c2 }...{ :else }...{ /if }
{ #each list as item }...{ /each }
```

### 3.19[svelte/shorthand-attribute](https://sveltejs.github.io/eslint-plugin-svelte/rules/shorthand-attribute/) & [svelte/shorthand-directive](https://sveltejs.github.io/eslint-plugin-svelte/rules/shorthand-directive/)
このルールは、ディレクティブで省略構文を使うことを強制します。
svelteは短く書けるのが魅力なので、コードがスッキリさせるためにも設定しておきましょう。

```sveltehtml
/* eslint svelte/shorthand-attribute: "error" */
<!-- ✓ GOOD -->
<button {disabled}>...</button>

<!-- ✗ BAD -->
<button disabled={disabled}>...</button>

<script>
  /* eslint svelte/shorthand-directive: "error" */
  let value = 'hello!'
  let active = true
  let color = 'red'
</script>

<!-- ✓ GOOD -->
<input bind:value>
<div class:active>...</div>
<div style:color>...</div>

<!-- ✗ BAD -->
<input bind:value={value}>
<div class:active={active}>...</div>
<div style:color={color}>...</div>
```

### 3.20 [svelte/sort-attributes](https://sveltejs.github.io/eslint-plugin-svelte/rules/sort-attributes/)
このルールは属性の順序を強制します。 デフォルトの順序は以下です。
- `this`
- `bind:this`
- `id`
- `name`
- `slot`
- `--style-props`（同じグループ内ではアルファベット順）
- `style`, `style:`
- `class`
- `class:`（同じグループ内ではアルファベット順）
- 他（同じグループ内ではアルファベット順）
- `bind:`, `bind:this`, `on:`
- `use:`（同じグループ内ではアルファベット順）
- `transition:`
- `in:`
- `out:`
- `animate:`
- `let:`（同じグループ内ではアルファベット順）

私は特に順序にこだわりはないですが、揃ってる方が好きなので、デフォルトのまま使用しています。

## 4. 僕の中のベスト3
### 4.1 第3位 [svelte/sort-attributes](https://sveltejs.github.io/eslint-plugin-svelte/rules/sort-attributes/)
第3位は、`svelte/sort-attributes`です。理由は、チーム開発には欠かせないルールだと考えたからです。
チーム開発では、コードの統一感が重要です。ですが、属性の順序は個人の好みによって異なることが多いです。
そのため属性の順序を統一することで、コードの見通しが良くなり、コードの品質を保つことができます。
また深く考えずとも、pre-commit時に自動で並び替えてくれるので、手間はかかりません。

### 4.2 第2位  [svelte/no-unused-class-name](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-unused-class-name/)
第2位は、`svelte/no-unused-class-name`です。cssが割り当てられていないクラスを報告してくれるため、コードの見通しが良くなります。
また[`svelte-check`](https://www.npmjs.com/package/svelte-check/v/1.1.9)にある`css-unused-selector`とも相性が良いです。
cssやクラスは不要なものが実装されていても気づきにくいため、このルールを設定しておくことで、コードの品質を保つことができます。

### 4.3 第1位 [svelte/require-each-key](https://sveltejs.github.io/eslint-plugin-svelte/rules/require-each-key/)
第1位は、`svelte/require-each-key`です。理由は、svelteのベストプラクティスに則っているためです。
svelteは学習コストが低く、扱いやすい点が魅力ですが、新しいフレームワークのため、ベストプラクティスを把握することが難しいです。
svelteのlint内には、ベストプラクティスをサポートしてくれるルール多く含まれていますが、その中でも`require-each-key`は実装者が予期していない挙動を防いでくれるため、第1位に選びました。

## 5. まとめ
今回はsvelteのlint設定について調査とまとめを行いました。
svelteのlintはかなりルールが豊富で驚きました。
特にベストプラクティスを助けてくれるようなルールが多いので、細かく観て設定しておくことは、コード品質の担保と開発スピードの向上に繋がるはずです。
初期設定が大変なイメージでしたが、svelteの仕様を理解するきっかけにもなり、かなり勉強になりました。
ぜひ、svelteのlint設定を導入してみてください。

## 6. 参考文献
- [1]https://github.com/sveltejs/eslint-plugin-svelte?tab=readme-ov-file
- [2]https://sveltejs.github.io/eslint-plugin-svelte/rules/
- [3]https://myajo.net/tips/10672/
- [4]https://svelte.dev/tutorial/keyed-each-blocks
 

## 7. 告知
We are hiring!
本記事をご覧いただき、ネクストビートの技術や組織についてもっと話を聞いてみたいと思われたかた、カジュアルにお話しませんか？

・今後のキャリアについて悩んでいる
・記事だけでなく、より詳しい内容について知りたい
・実際に働いている人の声を聴いてみたい

など、まだ転職を決められていないかたでも、ネクストビートに少しでもご興味をお持ちいただけましたら、ぜひカジュアルにお話しましょう！

🔽申し込みはこちら
https://hrmos.co/pages/nextbeat/jobs/1000008

また、ネクストビートについてはこちらもご覧ください。

🔽エントランスブック
https://note.nextbeat.co.jp/n/nd6f64ba9b8dc