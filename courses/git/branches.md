# ブランチでの作業

2つのコミットがあるローカルリポジトリに戻りましょう。これまでのところ、一本の履歴だけがあります。コミットは一本の線でつながっています。しかし、ときには同じリポジトリで二つの異なる機能を並行して作業する必要があるかもしれません。ここで、同じコードで新しいフォルダ／リポジトリを作成し、それを別の機能の開発に使用するという選択肢があります。しかし、もっと良い方法があります。gitはコミットを木のような構造にしているので、ブランチを使って異なる機能に取り組むことができます。ひとつのコミットから2つ以上のブランチを作成したり、ブランチをマージしたりすることができます。

ブランチを使うと、複数の履歴が存在することになり、そのうちのどれかにチェックアウトして作業を進めることができます。チェックアウトとは、先ほど説明したように、ディレクトリ（リポジトリ）の内容を、チェックアウトしたバージョンのスナップショットで置き換えることです。

ブランチを作成して、どうなるか見てみましょう。

```bash
$ git branch b1
$ git log --oneline --graph
* 7f3b00e (HEAD -> master, b1) adding file 2
* df2fb7a adding file 1
```

`b1`というブランチを作成しました。git logによると、b1は最後のコミット (7f3b00e) も指していますが、`HEAD`はまだmasterを指しています。覚えておいてほしいのですが、HEAD はチェックアウト先のコミット/参照を指しています。つまり、`b1`にチェックアウトした場合、HEADは`b1`を指しているはずです。確認してみましょう。

```bash
$ git checkout b1
Switched to branch 'b1'
$ git log --oneline --graph
* 7f3b00e (HEAD -> b1, master) adding file 2
* df2fb7a adding file 1
```

`b1`はまだ同じコミットを指していますが、HEADは`b1` を指すようになりました。コミット`7f3b00e`でブランチを作成しているので、このコミットの最初には2つの履歴が存在します。どちらのブランチでチェックアウトされたかによって、履歴の行が進みます。

現時点では、ブランチ`b1`でチェックアウトされているので、新しいコミットを作成すると、ブランチ参照`b1`がそのコミットに進み、現在の`b1`のコミットがその親になります。これを実行してみましょう。

```bash
# ファイルの作成とコミットの実行
$ echo "I am a file in b1 branch" > b1.txt
$ git add b1.txt
$ git commit -m "adding b1 file"
[b1 872a38f] adding b1 file
1 file changed, 1 insertion(+)
create mode 100644 b1.txt

# 履歴の新しい行
$ git log --oneline --graph
* 872a38f (HEAD -> b1) adding b1 file
* 7f3b00e (master) adding file 2
* df2fb7a adding file 1
$
```

masterはまだ古いコミットを指していることに注意しましょう。これで、masterブランチにチェックアウトして、そこにコミットすることができます。この結果、コミット7f3b00eから始まる別の履歴が表示されます。

```bash
# master ブランチへのチェックアウト
$ git checkout master
Switched to branch 'master'

# master ブランチに新しいコミットを作成
$ echo "new file in master branch" > master.txt
$ git add master.txt
$ git commit -m "adding master.txt file"
[master 60dc441] adding master.txt file
1 file changed, 1 insertion(+)
create mode 100644 master.txt

# 履歴
$ git log --oneline --graph
* 60dc441 (HEAD -> master) adding master.txt file
* 7f3b00e adding file 2
* df2fb7a adding file 1
```

ここではmasterにいるので、ブランチ`b1`が見えていないことに注目してください。全体像を把握するために、両方を視覚化してみましょう。

```bash
$ git log --oneline --graph --all
* 60dc441 (HEAD -> master) adding master.txt file
| * 872a38f (b1) adding b1 file
|/
* 7f3b00e adding file 2
* df2fb7a adding file 1
```

上記のツリー構造により、状況が明確になります。コミット7f3b00eから明確に分岐していることに注目してください。これがブランチを作る方法です。このようにして、2つのブランチはそれぞれ独立した機能開発を行うことができます。

**繰り返しになりますが、gitは内部的にはコミットのツリーにすぎません。ブランチの名前 (人間が読める名前) は、ツリー内のコミットへのポインターです。私たちはさまざまなgitコマンドを使って、ツリーの構造や参照を操作します。それに応じて、gitはそれに従ってリポジトリの内容を変更します。**

## マージ

ブランチ`b1`で作業していた機能が完成したので、それをmasterブランチにマージする必要があるとしましょう。そこで、まずmasterブランチにチェックアウトし、上流ブランチ（GitHubなど）から最新のコードを取得します。次に、`b1`のコードをmasterにマージする必要があります。この作業には2つの方法があります。

現在の履歴は以下の通りです。

```bash
$ git log --oneline --graph --all
* 60dc441 (HEAD -> master) adding master.txt file
| * 872a38f (b1) adding b1 file
|/
* 7f3b00e adding file 2
* df2fb7a adding file 1
```

**選択肢1: ブランチを直接マージする。** ブランチb1をmasterにマージすると、新しいマージコミットが発生します。これは、2つの異なる行の履歴からの変更をマージし、その結果の新しいコミットを作成します。

```bash
$ git merge b1
Merge made by the 'recursive' strategy.
b1.txt | 1 +
1 file changed, 1 insertion(+)
create mode 100644 b1.txt
$ git log --oneline --graph --all
*   8fc28f9 (HEAD -> master) Merge branch 'b1'
|\
| * 872a38f (b1) adding b1 file
* | 60dc441 adding master.txt file
|/
* 7f3b00e adding file 2
* df2fb7a adding file 1
```

新しいマージコミットが作成されているのがわかります（8fc28f9）。コミットメッセージの入力が促されます。リポジトリにたくさんのブランチがある場合、この結果はたくさんのマージコミットになってしまいます。これは、一本の開発履歴に比べて見づらいです。そこで、別のアプローチを考えてみましょう。

まず、最後のマージを[リセット](https://git-scm.com/docs/git-reset)して、以前の状態に戻しましょう。

```bash
$ git reset --hard 60dc441
HEAD is now at 60dc441 adding master.txt file
$ git log --oneline --graph --all
* 60dc441 (HEAD -> master) adding master.txt file
| * 872a38f (b1) adding b1 file
|/
* 7f3b00e adding file 2
* df2fb7a adding file 1
```

**選択肢2：リベース** ここで、ベースが似ている2つのブランチをマージする（コミット: 7f3b00e）のではなく、ブランチb1を現在のマスターにリベースしてみましょう。**これは、ブランチ`b1`（コミット7f3b00eからコミット872a38fまで）をリベースして、マスター（60dc441）の上に置くことを意味しています**。

```bash
# b1 に切り替える
$ git checkout b1
Switched to branch 'b1'

# (現在のブランチであるb1を）masterにリベースする
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: adding b1 file

# 結果
$ git log --oneline --graph --all
* 5372c8f (HEAD -> b1) adding b1 file
* 60dc441 (master) adding master.txt file
* 7f3b00e adding file 2
* df2fb7a adding file 1
```

1つのコミットがあった`b1`がわかります。このコミットの親は`7f3b00e` です。しかし、master(`60dc441`)にリベースしたので、このコミットが親になりました。副次的な効果として、1行の履歴になったこともわかります。`b1`を`master`にマージするとしたら、単純に`master`が`b1`である`5372c8f`を指すように変更するだけです。試してみましょう。

```bash
# コードをmasterにマージしたいので、masterにチェックアウトします。
$ git checkout master
Switched to branch 'master'

# 現在の履歴。b1はmasterをベースにしています。
$ git log --oneline --graph --all
* 5372c8f (b1) adding b1 file
* 60dc441 (HEAD -> master) adding master.txt file
* 7f3b00e adding file 2
* df2fb7a adding file 1


# マージを実行します。"fast-forward"のメッセージに注目してください。
$ git merge b1
Updating 60dc441..5372c8f
Fast-forward
b1.txt | 1 +
1 file changed, 1 insertion(+)
create mode 100644 b1.txt

# 結果
$ git log --oneline --graph --all
* 5372c8f (HEAD -> master, b1) adding b1 file
* 60dc441 adding master.txt file
* 7f3b00e adding file 2
* df2fb7a adding file 1
```

これで、`b1`と`master`が同じコミットを指していることがわかります。これで、あなたのコードはmasterブランチにマージされ、プッシュできるようになりました。また、履歴もきれいに残っていますね！:D
