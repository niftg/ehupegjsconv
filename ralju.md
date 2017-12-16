# ロジバンパーサのPEG.js用PEG文法ファイルから“素のPEG”を再現するのに役立つツールについて
<!----><!---->

## ロジバンの形式文法
- [Lojban Formal Grammars](https://mw.lojban.org/papri/Lojban_Formal_Grammars)
	- 公式の形式文法はYACCまたはEBNF
	- PEGは提案段階（だが今現在は“事實上の標準”か？）
<!----><!---->

## PEG文法を基にしたロジバンパーサ
- [PEG](https://mw.lojban.org/papri/PEG)
	- ブラウザで試せるやつ:
		- [visual camxes](http://camxes.lojban.org): Javaベース
		- [camxes.js](http://masatohagiwara.net/camxes.js/): JavaScriptベース([PEG.js](https://pegjs.org))
			- 派生文法のパーサはだいたいcamxes.jsの改造版
				- [ilmentufa](http://lojban.github.io/ilmentufa/camxes-exp.html)
				- [zantufa](https://mw.lojban.org/papri/zantufa)
<!----><!---->

## camxes.js系パーサのPEG.js用形式文法ファイルを見て困る事
- 文法そのものと比べると枝葉末節である後處理用JavaScriptコードが深く食ひ込んでゐて、割と讀み辛い
  - 素のパースツリーには殘されない、品詞等の文法情報を示すのに缺かせないものではあるが…
<!----><!---->

[ロジバン向けPEG文法](http://users.digitalkingdom.org/~rlpowell/hobbies/lojban/grammar/lojban.peg.txt)の一部分:
```
text-1 <-
	I-clause (jek / joik)? (stag? BO-clause)? free* text-1?
	/ NIhO-clause+ free* su-clause* paragraphs?
	/ paragraphs
paragraphs <-
	paragraph (NIhO-clause+ free* su-clause* paragraphs)?

paragraph <-
	(statement / fragment)
	(I-clause !jek !joik !joik-jek free* (statement / fragment)?)*
```
<!----><!---->

[camxes.js.peg](https://github.com/mhagiwara/camxes.js/blob/master/camxes.js.peg)の一部分:
```
text_1 =
	expr:(
		I_clause (jek / joik)? (stag? BO_clause)? free* text_1?
		/ NIhO_clause+ free* su_clause* paragraphs?
		/ paragraphs
	) {return _node("text_1", expr);}
paragraphs =
	expr:(
		paragraph (NIhO_clause+ free* su_clause* paragraphs)?
	) {return _node("paragraphs", expr);}
paragraph =
	expr:(
		(statement / fragment)
		(I_clause !jek !joik !joik_jek free* (statement / fragment)?)*
	) {return _node("paragraph", expr);}
```
<!----><!---->

## PEG.js用の文法ファイルの追加的要素

- `:`でタグづけされた部分式にマッチしたものを`{`... `}`内のJavaScriptコードで後から處理
  - "parser action"とよばれる機能
	  - 實はオリジナルのJava版camxesも、同樣のコード埋め込み機能を用ゐる[Rats!](https://cs.nyu.edu/rgrimm/xtc/rats-intro.html)なるパーサジェネレータを利用
		  - [Index of /~rlpowell/hobbies/lojban/grammar/rats](http://users.digitalkingdom.org/~rlpowell/hobbies/lojban/grammar/rats/)
<!----><!---->

## 埋め込みコードの除去に有用なツール
讀み易さのために素のPEGを再現したい時に
<!----><!---->

### [`pegpoho.sh`](https://github.com/guskant/gerna_cipra/blob/master/pegpoho.sh)
- zantufaリポジトリのシェルスクリプト
  - PEG.js用のコード入り文法ファイルを讀込んで、單純な文字列置換によりコード無しの文法ファイルを書出し
    - 非常に明快
    - タイムスタンプも押してくれる
    - 不要な括弧が殘つてしまふ
    - 不可逆的操作である
<!----><!---->

`pegpoho.sh`の出力例( [`maltufa-1.2.peg`](https://github.com/guskant/gerna_cipra/blob/master/maltufa-1.2.peg)より):

```
; la'o zoi maltufa-1.2.peg zoi peg zei gerna de'i li 20170502 tede'i UTC

; i ku'i la'o peg zoi_open zoi_word zoi_close peg na'e ka'e se tamgau va'o la peg po'o fa'o



text <- (intro_null NAI_clause* (CMEVLA_clause+ / indicators?) free* paragraphs? si_clause? SI_clause* faho_clause EOF?)

intro_null <- (initial_spaces? su_clause* (!paragraphs si_clause SI_clause*)?)

faho_clause <- ((FAhO_clause dot_star)?)

paragraphs <- ((NIhO_clause+ free* paragraphs_1?)+ / paragraphs_1 (NIhO_clause+ free* paragraphs_1?)*)
```
...
<!----><!---->

### [`pegjs_conv.js`](https://github.com/lojban/ilmentufa/blob/master/pegjs_conv.js)
- ilmentufaリポジトリのNode.jsスクリプト
  - "PEG <-> PEGJS CONVERTER"である：可逆的…かどうかはともかく雙方向的
	  - PEGJS→PEGの場合は、不要な括弧の除去もしてくれる
	  - PEG→PEGの場合は、コード埋め込みを自動化
	- 内部處理の把握は單純明快…とまではいかない
<!----><!---->

`pegjs_conv.js`の出力例([`camxes.js.peg`](https://github.com/mhagiwara/camxes.js/blob/master/camxes.js.peg)を入力した結果の一部):
```
text_1 <- I_clause (jek / joik)? (stag? BO_clause)? free* text_1? / NIhO_clause+ free* su_clause* paragraphs? / paragraphs

paragraphs <- paragraph (NIhO_clause+ free* su_clause* paragraphs)?

paragraph <- (statement / fragment) (I_clause !jek !joik !joik_jek free* (statement / fragment)?)*
```
...
<!----><!---->

函數"`peg_add_js_parser_actions`"の一部:
```
/* ZOI handling parser actions */
```
...
```
/* Parser action for elidible terminators */
```
...
```
/* Others */
peg = peg.replace(/^(initial[-_]spaces|dot[-_]star) *= *([^\r\n]+)/gm,
                  '$1 = expr:($2) {return ["$1", _join(expr)];}');
peg = peg.replace(/^(space[-_]char) *= *([^\r\n]+)/gm,
                  '$1 = expr:($2) {return _join(expr);}');
peg = peg.replace(/^(comma) *= *([^\r\n]+)/gm,
                  '$1 = expr:($2) {return ",";}');
/* Default parser action */
peg = peg.replace(/([0-9a-zA-Z_-]+) *= *(([^: \r\n]+ )*[^: \r\n]+)( *)(?=\r|\n|$)/gm,
                '$1 = expr:($2) {return _node("$1", expr);}$4');
return peg;
```
<!----><!---->

- 埋め込みコード（と埋め込み先ルール）を調整などすれば、自分好みにできるかも
- Node.jsを入れてる人は是非利用してみてください

mu'o mi'e .niftyg.
