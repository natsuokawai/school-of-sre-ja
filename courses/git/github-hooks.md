# GitとGithub

これまでの作業はすべてローカルのリポジトリで行われていましたが、gitは共同作業の環境でも役に立ちます。GitHubはインターネット上のひとつの場所で、あなたのgitリポジトリを集中的にホストして他の開発者と共同作業を行うことができます。

ワークフローの大部分はこれまで説明したものと同じですが、いくつかの点が追加されます。

 1. Pull: 最新の変更点をGitHub（中央）のリポジトリから引き出す
 2. Push: 自分の変更をGitHubリポジトリに反映して、他の人が利用できるようにする

GitHubには、これらに関するすばらしいガイドやチュートリアルが用意されています。

- [GitHub Hello World](https://guides.github.com/activities/hello-world/)
- [Git Handbook](https://guides.github.com/introduction/git-handbook/)

## フック

Gitには、フックと呼ばれるもうひとつの優れた機能があります。フックは基本的に、あるイベントが発生したときに呼び出されるスクリプトです。フックの場所は次のとおりです。

```bash
$ ls .git/hooks/
applypatch-msg.sample     fsmonitor-watchman.sample pre-applypatch.sample     pre-push.sample           pre-receive.sample        update.sample
commit-msg.sample         post-update.sample        pre-commit.sample         pre-rebase.sample         prepare-commit-msg.sample
```

名前を見れば一目瞭然ですね。これらのフックは、特定のイベントが発生したときに、特定の処理を行いたい場合に便利です。コードをプッシュする前にテストを実行したい場合は、`pre-push`フックを設定するとよいでしょう。コミット前のフックを作ってみましょう。

```bash
$ echo "echo this is from pre commit hook" > .git/hooks/pre-commit
$ chmod +x .git/hooks/pre-commit
```

基本的には、hooksフォルダの中に`pre-commit`というファイルを作成し、それを実行可能にします。これで、コミットすると次のようなメッセージが表示されるようになりました。

```bash
$ echo "sample file" > sample.txt
$ git add sample.txt
$ git commit -m "adding sample file"
this is from pre commit hook     # <===== フック実行のメッセージ
[master 9894e05] adding sample file
1 file changed, 1 insertion(+)
create mode 100644 sample.txt
```
