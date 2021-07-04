# Git

## 前提条件

1. Gitがインストールされていること [https://git-scm.com/downloads](https://git-scm.com/downloads)
2. gitの上級のチュートリアルまたは以下のLinkedInの学習コースを受講したこと
      - [https://www.linkedin.com/learning/git-essential-training-the-basics/](https://www.linkedin.com/learning/git-essential-training-the-basics/)
      - [https://www.linkedin.com/learning/git-branches-merges-and-remotes/](https://www.linkedin.com/learning/git-branches-merges-and-remotes/)
      - [The Official Git Docs](https://git-scm.com/doc)

## このコースで扱うこと

コンピュータサイエンス分野のエンジニアとして、バージョン管理ツールの知識を持つことはほぼ必須となります。現在、SVNやMercurialなど多くのバージョン管理ツールが存在しますが、Gitはおそらく最も使用されているツールであり、本コースではGitを扱います。このコースはGit 101から始まるわけではなく、前提条件としてGitの基本的な知識を求めていますが、様々なGitコマンドを実行する際に内部で何が起こっているかを詳細に説明しながら、皆さんが知っているGitの概念を再導入します。次回、gitコマンドを実行する際には、より自信を持ってエンターキーを押せるようになるでしょう。

## このコースでは扱わないこと

Gitの高度な使い方や内部実装の詳細について。

## コース内容

 1. [Gitの基本](/git/git-basics/#git-basics)
 2. [ブランチの扱い方](/git/branches/)
 3. [GithubとGit](/git/github-hooks/#git-with-github)
 4. [フック](/git/github-hooks/#hooks)

## Gitの基本

すでにご存知かもしれませんが、なぜバージョン管理システムが必要なのかを再確認してみましょう。プロジェクトが大きくなり、複数の開発者が作業するようになると、効率的なコラボレーションのための方法が必要になります。Git は、チームの共同作業を容易にし、コードベースの変更履歴を管理します。

### Gitリポジトリの作成

任意のフォルダをgitリポジトリに変換できます。以下のコマンドを実行すると、フォルダ内に`.git`フォルダが作成され、これがあることでフォルダはgitリポジトリになります。**gitのすべての魔法は、`.git`フォルダによって実現しています。**

```bash
# 空のフォルダを作成し、現在のディレクトリをそこに変更します。
$ cd /tmp
$ mkdir school-of-sre
$ cd school-of-sre/

# git repo の初期化
$ git init
Initialized empty Git repository in /private/tmp/school-of-sre/.git/
```

出力にあるように、空のgitリポジトリが私たちのフォルダに初期化されました。何が入っているのか見てみましょう。

```bash
$ ls .git/
HEAD        config      description hooks       info        objects     refs
```

`.git`フォルダの中にはたくさんのフォルダやファイルがあります。先ほど言ったように、これらがあるからこそ gitは魔法を使えるのです。これらのフォルダやファイルのいくつかを見ていきましょう。しかし今のところ、私たちが持っているのは空のgitリポジトリです。

### ファイルの追跡

もう知っているかもしれませんが、リポジトリに新しいファイルを作成してみましょう (ここではフォルダを _リポジトリ_ と呼びます)。そしてgitの状態を見てみましょう。

```bash
$ echo "I am file 1" > file1.txt
$ git status
On branch master

No commits yet

Untracked files:
 (use "git add <file>..." to include in what will be committed)

       file1.txt

nothing added to commit but untracked files present (use "git add" to track)
```

現在のgitステータスは`No commits yet`で、追跡されていないファイルがひとつあります。ファイルを作成したばかりなので、gitはそのファイルを追跡していません。ファイルやフォルダを追跡するようにgitに明示的に指示する必要があります。([gitignore](https://git-scm.com/docs/gitignore) も確認しましょう)。そのためには、上の出力にあるように`git add`コマンドを使います。続いて、コミットを作成します。

```bash
$ git add ファイル1.txt
$ git status
On branch master

No commits yet

Changes to be committed:
 (use "git rm --cached <file>..." to unstage)

       new file:   file1.txt

$ git commit -m "adding file 1"
[master (root-commit) df2fb7a] adding file 1
1 file changed, 1 insertion(+)
create mode 100644 file1.txt
```

ファイルを追加した後、gitステータスが`Changes to be committed:`と表示されていることに注目しましょう。これが意味するところは、ここに表示されているものはすべて次のコミットに含まれるということです。次に、コミットを作成して`-m`でメッセージを添付します。

### コミットの詳細

コミットはリポジトリのスナップショットです。コミットが行われるたびに、現在のリポジトリの状態（フォルダ）のスナップショットが取られ、保存されます。各コミットには固有のIDがあります。(前のステップで行ったコミットは`df2fb7a`)。中身をどんどん追加・変更し、コミットを繰り返していくと、それらのスナップショットがすべてgitに保存されていきます。繰り返しになりますが、これらの魔法はすべて`.git`フォルダの中で行われます。このようにして、すべてのスナップショットやバージョンが効率的な方法で保存されるのです。

### 変更の追加

もうひとつファイルを作成して、変更をコミットしてみましょう。先ほどのコミットと同じようになります。

```bash
$ echo "I am file 2" > file2.txt
$ git add file2.txt
$ git commit -m "adding file 2"
[master 7f3b00e] adding file 2
1 file changed, 1 insertion(+)
create mode 100644 file2.txt
```

IDが`7f3b00e`の新しいコミットが作成されました。リポジトリの状態を確認するには、いつでも`git status`を発行することができます。

**重要: コミットIDは長い文字列(SHA)ですが、最初の数文字(8文字以上)でもコミットを参照することができることに注意してください。ここでは、短いコミットIDと長いコミットIDを同じ意味で使用します。**

さて、2つのコミットがあるので、それらを視覚化してみましょう。

```bash
$ git log --oneline --graph
* 7f3b00e (HEAD -> master) adding file 2
* df2fb7a adding file 1
```

`git log`はその名のとおり、すべてのgitコミットのログを表示します。ここでは2つの追加の引数があります。`--oneline`はログの短いバージョン、つまりコミットメッセージのみを表示し、誰がいつコミットしたのかは表示しません。--graph` は、ログをグラフ形式で表示します。

**さて、この時点ではコミットが一行にひとつずつ並んでいるように見えるかもしれませんが、すべてのコミットはgitの内部でツリー状のデータ構造として保存されています。つまり、あるコミットには2つ以上の子コミットが存在する可能性があるということです。一行だけのコミットではありません。この部分については、ブランチのセクションで詳しく説明します。今のところ、これが私たちのコミット履歴です。**

```bash
   df2fb7a ===> 7f3b00e
```

### コミットは本当にリンクされているの？

先ほどの2つのコミットは、木のようなデータ構造でリンクされていて、リンクされている様子もわかりました。実際にそれを検証してみましょう。gitではすべてがオブジェクトです。新しく作成されたファイルはオブジェクトとして保存されます。ファイルへの変更はオブジェクトとして保存され、コミットもオブジェクトです。オブジェクトの内容を見るには、オブジェクトのIDを指定して次のコマンドを実行します。ここでは、2つ目のコミットの内容を見てみましょう。

```bash
$ git cat-file -p 7f3b00e
tree ebf3af44d253e5328340026e45a9fa9ae3ea1982
parent df2fb7a61f5d40c1191e0fdeb0fc5d6e7969685a
author Sanket Patel <spatel1@linkedin.com> 1603273316 -0700
committer Sanket Patel <spatel1@linkedin.com> 1603273316 -0700

adding file 2
```

上の出力では、`parent`属性に注目してください。これは、私たちが最初に行ったコミットのコミットIDを指しています。つまり、この2つのコミットがリンクしていることを示しています。さらに、このオブジェクトには2つ目のコミットのメッセージが表示されています。先ほど言ったように、これらの魔法はすべて`.git`フォルダによって有効になり、今見ているオブジェクトもそのフォルダの中にあります。

```bash
$ ls .git/objects/7f/3b00eaa957815884198e2fdfec29361108d6a9
.git/objects/7f/3b00eaa957815884198e2fdfec29361108d6a9
```

`.git/objects/`フォルダに格納されています。すべてのファイルとそれらへの変更もこのフォルダに格納されます。

### Git のバージョン管理の部分

すでにgitのログには2つのコミット（バージョン）が表示されています。バージョン管理ツールには、履歴をさかのぼって参照できる機能があります。たとえば、あるユーザーが古いバージョンのコードを使っていて問題を報告してきたとしましょう。その問題をデバッグするためには、古いコードにアクセスする必要があります。現在のリポジトリにあるのは最新のコードです。この例では、あなたは2つ目のコミット（7f3b00e）で作業していて、誰かがコミット（df2fb7a）のコードスナップショットに問題を報告しました。古いコミットのコードにアクセスするには、このようにします。

```bash
# 現在の状態、2つのファイルが存在
$ ls
file1.txt file2.txt

# (古い) コミットへのチェックアウト
$ git checkout df2fb7a
Note: checking out 'df2fb7a'.

# ここは'detached HEAD'の状態です。いろいろ見て回ったり、実験的な変更をしたり、それをコミットしたりすることができます。
# また、この状態でコミットした内容を破棄することもできます。この状態で行ったコミットは、別のチェックアウトを行うことで、どのブランチにも影響を与えることなく破棄することができます。

# 作成したコミットを保持するために新しいブランチを作成したい場合は、次のようにします。checkoutコマンドで-bを再度使用することで、（今でも後でも）そうすることができます。例:

 git checkout -b <新しいブランチ名>

HEAD is now at df2fb7a adding file 1

# 内容を確認すると、古い状態であることがわかります。
$ ls
file1.txt
```

このようにして、古いバージョンやスナップショットにアクセスすることができます。必要なのは、そのスナップショットへの _参照_ だけです。`git checkout ...`を実行すると、gitは`.git`フォルダを使用して、そのバージョン/参照での状態 (ファイルやフォルダ) を確認し、カレントディレクトリの内容をその内容で置き換えます。その時点での内容はローカルディレクトリ(リポジトリ)には存在しなくなりますが、それでもそのファイルにはアクセスすることができます。なぜなら、それらはgit commitによって追跡され、`.git`フォルダに保存されているからです。

### 参照

前のセクションで、バージョンの _参照_ が必要だと言いました。デフォルトでは、gitリポジトリはコミットのツリー構造になっています。そして、それぞれのコミットは一意のIDを持っています。しかし、ユニークなIDを通してコミットを参照する方法はそれだけではありません。コミットを参照する方法は複数あります。例えば、`HEAD`は現在のコミットを参照しています。`HEAD~1`は前のコミットへの参照です。ですから、上のセクションで前のバージョンをチェックアウトするときに、`git checkout HEAD~1`と実行できたはずです。

同じように、masterも（ブランチへの）参照となります。gitは木のような構造でコミットを保存するので、当然ながらブランチが存在します。そして、デフォルトのブランチは`master`と呼ばれます。（訳者注: デフォルトのブランチ名は将来的に変更される可能性があります。）master（あるいは他のブランチの参照）は、そのブランチの最新のコミットを指します。外のリポジトリで前のコミットにチェックアウトしたとしても、`master`は最新のコミットを指します。そして、`master`を参照してチェックアウトすれば、最新のバージョンに戻ることができます。

```bash
Previous HEAD position was df2fb7a adding file 1
Switched to branch 'master'

# これで、2つのファイルを含む最新のコードを見ることができます。
$ ls
file1.txt file2.txt
```

なお、上のコマンドでは、`master`の代わりに、コミットIDを使うこともできます。

### 参考文献と魔法

状況を見てみましょう。2つのコミット、`master`と`HEAD`の参照が最新のコミットを指しています。

```bash
$ git log --oneline --graph
* 7f3b00e (HEAD -> master) add file 2
* df2fb7a ファイル1の追加
```

魔法？これらのファイルを見てみましょう。

```bash
$ cat .git/refs/heads/master
7f3b00eaa957815884198e2fdfec29361108d6a9
```

masterが指している場所はファイルに保存されています。**gitがmasterの参照先を知りたくなったとき、あるいはgitがmasterの参照先を更新したくなったときには、上のファイルを更新すればよいのです。**したがって、新しいコミットを作成すると、現在のコミットの上に新しいコミットが作成され、マスターファイルが新しいコミットのIDで更新されます。

同様に、`HEAD`を参照してください。

```bash
$ cat .git/HEAD
ref: refs/heads/master
```

`HEAD`が`refs/heads/master`という名前のリファレンスを指していることがわかります。つまり、`HEAD`は `master`が指している場所を指すことになります。

### 小さな冒険

コマンドを実行したとき、gitがどのようにファイルを更新するのかを説明しました。しかし、実際に自分でやってみてどうなるかを見てみましょう。

```bash
$ git log --oneline --graph
* 7f3b00e (HEAD -> master) adding file 2
* df2fb7a adding file 1
```

では、masterを前の/最初のコミットを指すように変更しましょう。

```bash
$ echo df2fb7a61f5d40c1191e0fdeb0fc5d6e7969685a > .git/refs/heads/master
$ git log --oneline --graph
* df2fb7a (HEAD -> master) adding file 1

# 最初の状態に戻す
$ echo 7f3b00eaa957815884198e2fdfec29361108d6a9 > .git/refs/heads/master
$ git log --oneline --graph
* 7f3b00e (HEAD -> master) adding file 2
* df2fb7a adding file 1
```

`master`の参照ファイルを編集したところ、git logには最初のコミットだけが表示されるようになりました。ファイルへの変更を元に戻すと、元の状態に戻ります。魔法というほどのものではありませんが、いかがでしょうか？