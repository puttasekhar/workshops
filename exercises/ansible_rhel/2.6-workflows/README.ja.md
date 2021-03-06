# 演習 - ワークフロー

**Read this in other languages**:
<br>![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png)[日本語](README.ja.md), ![brazil](../../../images/brazil.png) [Portugues do Brasil](README.pt-br.md), ![france](../../../images/fr.png) [Française](README.fr.md), ![Español](../../../images/col.png) [Español](README.es.md).

* [目的](#目的)
  * [ラボシナリオ](#ラボシナリオ)
  * [プロジェクトの設定](#プロジェクトの設定)
  * [ジョブテンプレートの作成](#ジョブテンプレートの作成)
  * [ワークフローの設定](#ワークフローの設定)
  * [実行してみましょう](#実行してみましょう)

# 目的

ワークフローの基本的な考え方は、複数のジョブテンプレートをリンクし実行できることです。各ジョブテンプレートの実行は、例えば以下の様な実行条件を付与することができます。  

  - ジョブテンプレート A が成功すると、ジョブテンプレート B が自動的に実行される  

  - ただし、失敗した場合は、ジョブテンプレート C が実行される  

また、ワークフローはジョブテンプレートだけではなくプロジェクトやインベントリの更新を行うこともできます。  

これにより、異なるジョブテンプレートをが互いに対して構築するようなことが可能になります。例えば、ネットワークチームは独自のコンテンツを含むPlaybookを独自の Git リポジトリで作成し、独自のインベントリをターゲットにしつつ、運用チームも独自のリポジトリ、Playbook、インベントリを持つことができます。

このラボでは、ワークフローの設定方法を学びます。  

# ガイド

## ラボシナリオ  

組織には以下の2つの部門があります。  

  - `webops` Git ブランチでPlaybookを開発しているWeb運用チーム  

  - `webdev` Git ブランチでPlaybookを開発しているWeb開発チーム  

新しい Node.js サーバーをデプロイする必要がある場合、以下の 2 つのことを行う必要があります  

**Web運用チーム**
  - node.js をインストールし、ファイアウォール設定を行い、node.jsを開始する  

**Web開発チーム**
  - 最新バージョンのWebアプリケーションをデプロイする  

Playbook、JSP ファイルなど、必要なものはすべて Git リポジトリーに存在します。それを利用してラボを行います。  

物事を簡単にするために、プレイブック、JSPファイルなど必要な資材はすべてGitHubのリポジトリに存在しています。あなたはそれを組み合わせる必要があります。

---

> **メモ**
>
> シナリオでは、2 つの異なる Git リポジトリの利用を想定していますが、このラボでは同じリポジトリの2つの異なるブランチにアクセスしています。実際には、SCMリポジトリの構造は多くの要因により異なっている可能性があります。

## プロジェクトの設定

先のラボで実施した通り、まずはプロジェクトを作成し、 Git リポジトリを登録する必要があります。必要な情報は以下です。ご自身で設定してみてください。  

> **注意**
>
> このラボは admin アカウントで実施します。 **wweb** ユーザーでログインしている場合は、ログアウトして **admin** でログインしなおしてください！

Web運用チーム用のプロジェクトを作成します。**プロジェクト** 画面で緑色の「+」ボタンをクリックし、以下の内容を入力します。 

<table>
  <tr>
    <th>パラメーター名</th>
    <th>設定値</th>
  </tr>
  <tr>
    <td>名前</td>
    <td>Webops Git Repo</td>
  </tr>
  <tr>
    <td>組織</td>
    <td>Default</td>
  </tr>
  <tr>
    <td>SCMタイプ</td>
    <td>Git</td>
  </tr>  
  <tr>
    <td>SCM URL</td>
    <td><code>https://github.com/ansible/workshop-examples.git</code></td>
  </tr>
  <tr>
    <td>SCM ブランチ/タグ/コミット</td>
    <td><code>webops</code></td>
  </tr>
  <tr>
    <td>SCM 更新オプション</td>
    <td><ul><li>✓ クリーニング</li><li>✓ 更新時のデプロイ</li><li>✓ 起動時のリビジョン更新</li></ul></td>
  </tr>             
</table>

- **保存** をクリックします

---
Web開発チーム用のプロジェクトを作成します。**プロジェクト** 画面で緑色の「+」ボタンをクリックし、以下の内容を入力します。 

<table>
  <tr>
    <th>パラメーター名</th>
    <th>設定値</th>
  </tr>
  <tr>
    <td>名前</td>
    <td>Webdev Git Repo</td>
  </tr>
  <tr>
    <td>組織</td>
    <td>Default</td>
  </tr>
  <tr>
    <td>SCMタイプ</td>
    <td>Git</td>
  </tr>  
  <tr>
    <td>SCM URL</td>
    <td><code>https://github.com/ansible/workshop-examples.git</code></td>
  </tr>
  <tr>
    <td>SCM ブランチ/タグ/コミット</td>
    <td><code>webdev</code></td>
  </tr>
  <tr>
    <td>SCM 更新オプション</td>
    <td><ul><li>✓ クリーニング</li><li>✓ 更新時のデプロイ</li><li>✓ 起動時のリビジョン更新</li></ul></td>
  </tr>             
</table>

- **保存** をクリックします    

## ジョブテンプレートの作成

最終目標はワークフローの作成ですが、まず、通常のジョブテンプレートを作成する必要があります。

**テンプレート** 画面で緑色の「+」ボタンをクリックし、**ジョブテンプレート** を選択します。

 <table>
    <tr>
      <th>パラメーター名</th>
      <th>設定値</th>
    </tr>
    <tr>
      <td>名前</td>
      <td>Web App Deploy</td>
    </tr>
    <tr>
      <td>ジョブタイプ</td>
      <td>実行</td>
    </tr>
    <tr>
      <td>インベントリー</td>
      <td>Workshop Inventory</td>
    </tr>  
    <tr>
      <td>プロジェクト</td>
      <td>Webops Git Repo</td>
    </tr>
    <tr>
      <td>PLAYBOOK</td>
      <td><code>rhel/webops/web_infrastructure.yml</code></td>
    </tr>
    <tr>
      <td>認証情報</td>
      <td>Workshop Credentials</td>
    </tr>
    <tr>
      <td>制限</td>
      <td>web</td>
    </tr>    
    <tr>
      <td>オプション</td>
      <td>✓ 権限昇格の有効化</td>
    </tr>                     
  </table> 

  - **保存** をクリックします  

  ---

  **テンプレート** 画面で緑色の「+」ボタンをクリックし、**ジョブテンプレート** を選択します。

   <table>
    <tr>
      <th>パラメーター名</th>
      <th>設定値</th>
    </tr>
    <tr>
      <td>名前</td>
      <td>Node.js Deploy</td>
    </tr>
    <tr>
      <td>ジョブタイプ</td>
      <td>実行</td>
    </tr>
    <tr>
      <td>インベントリー</td>
      <td>Workshop Inventory</td>
    </tr>  
    <tr>
      <td>プロジェクト</td>
      <td>Webdev Git Repo</td>
    </tr>
    <tr>
      <td>PLAYBOOK</td>
      <td><code>hel/webdev/install_node_app.yml</code></td>
    </tr>
    <tr>
      <td>認証情報</td>
      <td>Workshop Credentials</td>
    </tr>
    <tr>
      <td>制限</td>
      <td>web</td>
    </tr>    
    <tr>
      <td>オプション</td>
      <td>✓ 権限昇格の有効化</td>
    </tr>                     
  </table> 

  - **保存** をクリックします  

> **ヒント**  
>
> Playbook の中身をご覧になりたい方は、 Github URL を確認して、適切なブランチに切り替えてご覧ください。  

## ワークフローの設定

ワークフローを設定します。ワークフローは **テンプレート** 画面で設定できます。
テンプレートを追加する際に、**ジョブテンプレート** と **ワークフローテンプレート** のどちらかを選択することができます。

  ![workflow add](images/workflow_add.png)

  - **テンプレート** を選択し、緑色の「+」ボタンをクリックして、**ワークフローテンプレート**を選択します。

  <table>
    <tr>
      <td><b>名前</b></td>
      <td>Deploy Webapp Server</td>
    </tr>
    <tr>
      <td><b>組織</b></td>
      <td>Default</td>
    </tr>    
</table>      

  - **保存** をクリックします  

テンプレートを保存すると、**ワークフロー・ビジュアライザー** が開き、ワークフローを作成することができます。テンプレートの詳細ページにあるボタンを使用して、**ワークフロー・ビジュアライザー** を再度開くことができます。

  - **開始** ボタンをクリックすると、Node 画面が開きます。右側で、ノードにアクションを割り当てることができます。**ジョブ**、**プロジェクトの同期**、**インベントリー同期**のいずれかが選択できます。

  - この実習ラボでは、先に作成したジョブテンプレートをリンクします。そのため、**Tomcat Deploy** ジョブを選んで**選択**をクリックします。  

  - 左側にノードが現れます。ノードにはジョブの名前が入っています。ノードの上にマウスポインターを合わせると、赤い**x**と緑の **+** 記号、真ん中には鎖のような青い記号が表示されます。

   ![workflow node](images/workflow_node.png)

> **ヒント**
>
> 赤い「x」を使用するとノードを削除でき、緑のプラスを使用すると次のノードを追加できます。青は他のノードへのリンク作成を行う際に使います。  

  - 緑の **+** を選択します

  - 次のジョブとして **Node.js Deploy** を選択します（次のページに切り替える必要がある場合があります）  

  - **実行** は**成功時**のままにします。

> **ヒント**
>
> この実行を使うことにより、より複雑なワークフローが可能になります。Playbook の実行が成功した場合と失敗した場合に、異なる実行パスをレイアウトできます。

  - **選択**  をクリック

  - **ワークフロービジュアライザー**画面で**保存**をクリックします  

  - **ワークフローテンプレート**画面で**保存**をクリックします

## 実行してみましょう

作成が完了しましたので早速動作させてみましょう♪  

  - **起動** ボタンを直接クリックしても良いですし、**テンプレート画面**で**Deploy Webapp Server**のロケットアイコンをクリックしても起動ができます。  

  ![launch](images/launch.png)

ジョブビューでワークフローの実行がどのように表示されるかに注意してください。今回の通常のジョブテンプレートジョブの実行とは対照的に、右側にはプレイブックの出力はありませんが、複数のジョブステップの実行状況が表示されます。各ジョブで実行されたプレイブックの状況を確認したい場合は、各ステップの**詳細**をクリックしてください。再度ワークフロー実行画面に戻りたい場合は、左の画面の Web App Deloy の右隣にある小さな ![w-button](images/w_button.png) をクリックしてください。

![jobs view of workflow](images/job_workflow.png)

ジョブが完了した後、すべてがうまく働いたかどうかを確認します。コントロールホストから`node1`、`node2`または`node3`のいずれかにログインし、以下を実行します。

```bash
$ curl http://localhost/nodejs
```

また、コントロールホスト上でcurlを実行し、ノードに向けて `nodejs` のパスを問い合わせれば、シンプルなnodejsアプリケーションも表示されるはずです。  

----
**ナビゲーション**
<br>
[前のエクササイズ](../2.5-rbac/README.ja.md) - [次のエクササイズ](../2.7-wrap/README.ja.md)

[Ansible Tower ワークショップ表紙に戻る](../README.ja.md#section-2---ansible-towerの演習)
