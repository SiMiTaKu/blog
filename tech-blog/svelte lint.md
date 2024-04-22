# svelteのeslintおすすめ設定

## はじめに
最近lintの有用性を実感している清水琢巳です。
私は、株式会社ネクストビートで「おもてなしHR」という地方創生に関わる、宿泊業界に特化した転職支援プロダクトの開発をしております。

現在おもてなしHRでは、svelteを採用して開発を行っています。
その開発を行う中で、svelteのlint設定について調査を行いました。
今回はその結果を共有したいと思います。

## アジェンダ
本記事の構成は以下のとおりです。

1. svelteのlintの導入方法
2. plugin:svelte/recommendedの設定内容
3. 個別に追加したおすすめの設定
4. まとめ
5. 参考文献

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
svelteのリアクティブ変数は、再割り当てを行うと再割り当てされた変数がリアクティブ変数として機能しなくなります。

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
また明示的にtype属性を指定することで、ボタンの動作を明確にすることができるメリットもあります。

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
このルールは、`subscribe()`の呼び出しによって返された サブスクライバーが、変数やプロパティに代入されていないか、関数に渡されていない場合に警告を出します。
ストアは不要になったら、常にサブスクライブを解除すべきです。でなければサブスクリプションが有効なままとなり、メモリリークの原因となります。

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
svelte-checkは、<style>ブロックにテンプレートで使用されていないクラスが含まれている場合、css-unused-selectorを生成しますが、このルールはその逆で、<style>ブロックで参照されていないクラスがテンプレートに含まれているケースを報告します。

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
> このルールはキーのない {#each} ブロックを報告する。

> [!INFORMATION]
> Svelteのデフォルトでは、eachブロックの値を変更すると、ブロックの末尾の項目が追加・削除され、変更された値が更新される。[4]

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

### 3.14 [svelte/require-stores-init](https://sveltejs.github.io/eslint-plugin-svelte/rules/require-stores-init/)
> このルールは、Svelteストアを初期化する際に初期値を設定していない場合を報告します。

```typescript
/* eslint svelte/require-stores-init: "error" */

import { writable, readable, derived } from 'svelte/store';

/* ✓ GOOD */
export const w1 = writable(false);
export const r1 = readable({});
export const d1 = derived([a, b], () => {}, false);

/* ✗ BAD */
export const w2 = writable();
export const r2 = readable();
export const d2 = derived([a, b], () => {});
```

### 3.15 [ svelte/valid-each-key](https://sveltejs.github.io/eslint-plugin-svelte/rules/valid-each-key/)
> このルールは{#each}ブロックのキーが{#each}ブロックで定義された変数を使用しないことを報告します。

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

### 3.16 [svelte/derived-has-same-inputs-outputs](https://sveltejs.github.io/eslint-plugin-svelte/rules/derived-has-same-inputs-outputs/)
> このルールは、変数名とコールバック関数の引数名が異なる場合に報告する。これは主に実装上の混乱を避けるための推奨ルールです。

```typescript
/* eslint svelte/derived-has-same-inputs-outputs: "error" */

import { derived } from 'svelte/store';

/* ✓ GOOD */
derived(a, ($a) => {});
derived(a, ($a, set) => {});
derived([a, b], ([$a, $b]) => {});

/* ✗ BAD */
derived(a, (b) => {});
derived(a, (b, set) => {});
derived([a, b], ([one, two]) => {});
```

### 3.17 [svelte/html-closing-bracket-spacing](https://sveltejs.github.io/eslint-plugin-svelte/rules/html-closing-bracket-spacing/)
> このルールは、HTML要素の終了タグの角括弧の前のスペースをフォーマットを強制します。
> 閉じ括弧の前の間隔は、次の2つのスタイルから選択できます。
> - always: <div />
> - never: <div/>

```sveltehtml
/* eslint svelte/html-closing-bracket-spacing: "error" */
<!-- ✓ GOOD -->
<div />
<p>Hello</p>
<div>
</div>

<!-- ✗ BAD -->
<div/>
<p >Hello</p >
<div  >
</div >
```

### 3.18 [svelte/html-self-closing](https://sveltejs.github.io/eslint-plugin-svelte/rules/html-self-closing/)
> このルールは、HTML要素が自己終了しているかどうかを報告します。

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

### 3.19 [svelte/indent](https://sveltejs.github.io/eslint-plugin-svelte/rules/indent/)
> このルールは.svelteで一貫したインデントスタイルを強制します。デフォルトのスタイルは2スペースです。

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

私は以下のように設定しました。

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

### 3.20 [svelte/max-attributes-per-line](https://sveltejs.github.io/eslint-plugin-svelte/rules/max-attributes-per-line/)
> このルールは、可読性を向上させるために、一行あたりの属性/ディレクティブの最大数を制限します。

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

### 3.21 [svelte/mustache-spacing](https://sveltejs.github.io/eslint-plugin-svelte/rules/mustache-spacing/)
> このルールは、マスタッシュ内のスペースを統一します。

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

### 3.22 [svelte/no-extra-reactive-curlies](https://sveltejs.github.io/eslint-plugin-svelte/rules/no-extra-reactive-curlies/)
> このルールは、単一の式のみを含むリアクティブ・ステートメント本体を囲む中括弧`（{`と`}）`が不必要に使用されている場合報告します。

```sveltehtml
<script>
  /* eslint svelte/no-extra-reactive-curlies: "error" */

  /* ✓ GOOD */
  $: foo = 'red';

  /* ✗ BAD */
  $: {
    foo = 'red';
  }
</script>
```

### 3.23 [svelte/shorthand-attribute](https://sveltejs.github.io/eslint-plugin-svelte/rules/shorthand-attribute/)
このルールは、アトリビュートの省略構文の使用を強制します。
svelteは短く書けるのが魅力なので、コードがスッキリさせるためにも設定しておきましょう。

```sveltehtml
  /* eslint svelte/shorthand-attribute: "error" */
<!-- ✓ GOOD -->
<button {disabled}>...</button>

<!-- ✗ BAD -->
<button disabled={disabled}>...</button>
```

### 3.24 [svelte/shorthand-directive](https://sveltejs.github.io/eslint-plugin-svelte/rules/shorthand-directive/)
このルールは、ディレクティブで省略構文を使うことを強制します。
svelteは短く書けるのが魅力なので、コードがスッキリさせるためにも設定しておきましょう。

```sveltehtml
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

### 3.25 [svelte/sort-attributes](https://sveltejs.github.io/eslint-plugin-svelte/rules/sort-attributes/)
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

### 3.26 [svelte/spaced-html-comment](https://sveltejs.github.io/eslint-plugin-svelte/rules/spaced-html-comment/)
このルールは、HTMLコメントの前後にスペースを強制します。
コメントの書き方は、人によって異なる場合が多いため、チーム開発では統一するために設定すると良いでしょう。

```sveltehtml
/* eslint svelte/spaced-html-comment: "error" */
<!-- ✓ GOOD -->
<!--✗ BAD-->
```

## 5. 参考文献
- https://github.com/sveltejs/eslint-plugin-svelte?tab=readme-ov-file
- https://sveltejs.github.io/eslint-plugin-svelte/rules/
- [3]https://myajo.net/tips/10672/
- [4]https://svelte.dev/tutorial/keyed-each-blocks