---
layout: default
title: データーベース操作処理をレポジトリに整理しよう
---

---

# {{ page.title }}

## 本章メニュー

- 本章では以下を行います。

    1. レポジトリの作成の仕方を説明します。

    1. データーべース構造定義ファイルとレポジトリの関係を説明します。

    1. コントローラー上のデーターベース操作処理をレポジトリに移設します。

## レポジトリとデータベース操作

- 前章では、データベースから情報を取得して表示する方法を説明しました。

- 本章では、ここまで行なってきたデーターベースに関連する処理をレポジトリに移設します。

## レポジトリファイルの確認

- まずは以前の章で設定した、データーベース構造定義ファイルを確認してみましょう。
- 以下のファイルを確認します。
    - /src/Eccube/Resource/doctrine/Eccube.Entity.Crud.dcm.yml

    1. ファイルを開いて以下の様に修正します。

        - **Eccube.Entity.Crud.dcm.yml**

<script src="http://gist-it.appspot.com/https://github.com/EC-CUBE/ec-cube.github.io/blob/master/Source/tutorial_11/dcm_yml_view_repository.yml"></script>

<!--
```
Eccube\Entity\Crud:
type: entity
table: dtb_crud
repositoryClass: Eccube\Repository\CrudRepository ★レポジトリ定義
id:
    id:
        type: integer
        nullable: false
        unsigned: false
        id: true
        column: id
        generator:
            strategy: AUTO
fields:
    reason:
        type: smallint
        nullable: false
    name:
        type: string
        lenght: 255
        nullable: false
    title:
        type: string
        lenght: 255
        nullable: false
    notes:
        type: text
        nullable: false
    create_date:
        type: datetime
        nullable: false
    update_date:
        type: datetime
        nullable: false
lifecycleCallbacks: {  }
```
-->

- 上記がデーターベース構造定義ファイルの内容です。
- ★で示した箇所にレポジトリの定義がされています。

- この定義によって、テーブル、エンティティ、テーブルとエンティティの関係、さらに今回のレポジトリで**データーベース操作**クラスの関連付けが行われています。

### レポジトリの作成

#### ファイルの作成

- 以下フォルダ内にレポジトリが保存されています。

  - /src/Eccube/Repository

  1. 次に**CrudRepository.php**を作成します。

  - AuthorityRoleRepository.phpをコピー、リネームします。

    - **CrudRepository.php**( 中身はAuthorityRoleRepository.phpのコピー )

<script src="http://gist-it.appspot.com/https://github.com/EC-CUBE/ec-cube.github.io/blob/master/Source/tutorial_11/CrudRepository_before.php"></script>

<!--
```
<?php
/*
 * This file is part of EC-CUBE
 *
 * Copyright(c) 2000-2015 LOCKON CO.,LTD. All Rights Reserved.
 *
 * http://www.lockon.co.jp/
 *
 * This program is free software; you can redistribute it and/or
 * modify it under the terms of the GNU General Public License
 * as published by the Free Software Foundation; either version 2
 * of the License, or (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program; if not, write to the Free Software
 * Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.
 */


namespace Eccube\Repository;

use Doctrine\ORM\EntityRepository;

/**
 * AuthorityRoleRepository ★リネーム
 * 
 * This class was generated by the Doctrine ORM. Add your own custom
 * repository methods below.
 */
class AuthorityRoleRepository extends EntityRepository ★リネーム
{

    /**
     * 権限、拒否URLでソートする
     *
     * @return array
     */
    public function findAllSort()
    {
        return $this->findBy(array(), array('Authority' => 'ASC', 'deny_url' => 'ASC'));
    }
}
```
-->

- 上記のソースを以下の様に修正します。

<script src="http://gist-it.appspot.com/https://github.com/EC-CUBE/ec-cube.github.io/blob/master/Source/tutorial_11/CrudRepository_after.php"></script>

<!--
```
<?php
/*
 * This file is part of EC-CUBE
 *
 * Copyright(c) 2000-2015 LOCKON CO.,LTD. All Rights Reserved.
 *
 * http://www.lockon.co.jp/
 *
 * This program is free software; you can redistribute it and/or
 * modify it under the terms of the GNU General Public License
 * as published by the Free Software Foundation; either version 2
 * of the License, or (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program; if not, write to the Free Software
 * Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.
 */


namespace Eccube\Repository;

use Doctrine\ORM\EntityRepository;

/**
 * CrudRepository ★コメントのクラス名を修正
 * 
 * This class was generated by the Doctrine ORM. Add your own custom
 * repository methods below.
 */
class CrudRepository extends EntityRepository ★クラス名を修正
{
    ★メソッドは一度全て削除
}
```
-->


## サービスプロバイダへの登録

- 次にコントローラーからレポジトリを呼び出すために、サービスプロバイダに登録します。

- 以下のサービスプロバイダを修正していきます。
    - /src/Eccube/ServiceProvider/EccubeServiceProvider.php

    1. ファイルを開いて以下の様に修正します。

        - **EccubeServiceProvider.php**

    1. 次に「Repository」のコメントを検索します。

<script src="http://gist-it.appspot.com/https://github.com/EC-CUBE/ec-cube.github.io/blob/master/Source/tutorial_11/EccubeServiceProvider_before.php"></script>

<!--
```
        // Repository
        $app['eccube.repository.master.authority'] = $app->share(function () use ($app) {
            return $app['orm.em']->getRepository('Eccube\Entity\Master\Authority');
        });
        $app['eccube.repository.master.tag'] = $app->share(function () use ($app) {
            return $app['orm.em']->getRepository('Eccube\Entity\Master\Tag');
        });
        $app['eccube.repository.master.pref'] = $app->share(function () use ($app) {
            return $app['orm.em']->getRepository('Eccube\Entity\Master\Pref');
        });
        ...
        ..
        .
```
-->

- 上記を検索で見つけたら、次に「Repository」の一番最後の行に**CrudRepository**をサービスプロバイダへ登録します。

<script src="http://gist-it.appspot.com/https://github.com/EC-CUBE/ec-cube.github.io/blob/master/Source/tutorial_11/EccubeServiceProvider_after.php"></script>

<!--
```
        $app['eccube.repository.help'] = $app->share(function () use ($app) {
            return $app['orm.em']->getRepository('Eccube\Entity\Help');
        });
        $app['eccube.repository.plugin'] = $app->share(function () use ($app) {
            return $app['orm.em']->getRepository('Eccube\Entity\Plugin');
        });
        $app['eccube.repository.plugin_event_handler'] = $app->share(function () use ($app) {
            return $app['orm.em']->getRepository('Eccube\Entity\PluginEventHandler');
        });

        $app['eccube.repository.crud'] = $app->share(function () use ($app) { ★この行を追加
            return $app['orm.em']->getRepository('Eccube\Entity\Crud');
        });

        // em
```
-->

- 上記の追加で、**$app**が利用できる箇所であれば、「**$app['eccube.repository.crud']**」で呼び出す事が可能となりました。


## Doctrine定義のマジックメソッドで情報取得

- レポジトリにはあらかじめ、以下の様な検索系メソッドが標準で容易されています。

    - **find ～**

- 本項ではまず、一覧データー取得を**エンティティーマネージャー**、**クエリビルダー**で抽出していた箇所を、**レポジトリのマジックメソッド**に置き換えてみます。

- 以下のコントローラーを修正していきます。
    - /src/Controller/Default/CrudController.php

    1. ファイルを開いて以下の様に修正します。

        - **CrudController.php**

<script src="http://gist-it.appspot.com/https://github.com/EC-CUBE/ec-cube.github.io/blob/master/Source/tutorial_11/CrudController_add_repository.php"></script>

<!--
```
<?php
/*
 * This file is part of EC-CUBE
 *
 * Copyright(c) 2000-2015 LOCKON CO.,LTD. All Rights Reserved.
 *
 * http://www.lockon.co.jp/
 *
 * This program is free software; you can redistribute it and/or
 * modify it under the terms of the GNU General Public License
 * as published by the Free Software Foundation; either version 2
 * of the License, or (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program; if not, write to the Free Software
 * Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.
 */


namespace Eccube\Controller\Tutorial;

use Eccube\Application;
use Eccube\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;

class CrudController extends AbstractController
{
    public function index(Application $app, Request $request)
    {
        $app->clearMessage();

        $Crud = new \Eccube\Entity\Crud();

        $builder = $app['form.factory']->createBuilder('crud', $Crud);

        $form = $builder->getForm();

        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $app->addSuccess('データーが保存されました');
            $app['orm.em']->persist($Crud);
            $app['orm.em']->flush($Crud);
        } elseif ($form->isSubmitted() && !$form->isValid()) {
            $app->addSuccess('入力内容をご確認ください');
        }

        $crudList = $app['eccube.repository.crud']->findAll(); ★クエリビルダーから置き換え

        return $app->render(
            'Tutorial/crud_top.twig',
            array(
                'forms' => $form->createView(),
                'crudList' => $crudList,
            )
        );
    }
}
```
-->

### 表示内容の確認

- 最後に確認のためにブラウザにアクセスしてみましょう。

    1. ブラウザのURLに「http://[ドメイン + インストールディレクトリ]/tutorial/crud」を入力してください。

    1. クエリビルダーで一覧情報を取得していた際と同じ結果が表示されています。
        - ※正確には、**order**を指定していないため、複数のレコードがある際は、表示順序が異なるかも知れません。

---

![レポジトリで一覧取得](/images/img-tutorial11-view-rendar-repository-findall.png)

---

### Doctine定義のマジックメソッド

- 本章ではレポジトリが保有しているメソッド**findAll**を使用して、一覧表示用のデーターを取得しました。
- findAllメソッドは**Doctrine定義のマジックメソッド**で他にも下記参照先の様な種類があります。
    - <a href="http://docs.symfony.gr.jp/symfony2/book/doctrine.html#id8" target="_blank">データベースからのオブジェクトのフェッチ</a>

- **マジックメソッドは便利**なため、**よく利用**しがちです。
- その一種類である**findBy～**ですが、**オーバーライドによる、危険性**があるため、**EC-CUBE3では、利用する事を推奨していません。**

## レポジトリのメソッド定義
  - 上記でレポジトリのマジックメソッドについて説明を行いました。
  - 簡易な情報であれば、**findAll**で問題ないかも知れませんが、複雑なクエリでの情報取得などの場合、またはケースバイケースにはなりますが、メンテナンス性を考慮し、通常はレポジトリ内に、データーベース操作のロジックを定義し、コントローラーは必要情報の取得、条件のを受け渡しのみとし構築する事が多いかと思います。
  - ここでは、一覧表示の情報取得、データーベースへの登録部分を、レポジトリにメソッドして定義していきます。

### レポジトリファイルの修正

- 以下のファイルを開きます。
    - /src/Eccube/Repository/CrudRepository.php

    1. ファイルを開いて以下の様に修正します。

        - **CrudRepository.php**

<script src="http://gist-it.appspot.com/https://github.com/EC-CUBE/ec-cube.github.io/blob/master/Source/tutorial_11/CrudRepository_modified.php"></script>

<!--
```
<?php
/*
 * This file is part of EC-CUBE
 *
 * Copyright(c) 2000-2015 LOCKON CO.,LTD. All Rights Reserved.
 *
 * http://www.lockon.co.jp/
 *
 * This program is free software; you can redistribute it and/or
 * modify it under the terms of the GNU General Public License
 * as published by the Free Software Foundation; either version 2
 * of the License, or (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program; if not, write to the Free Software
 * Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.
 */


namespace Eccube\Repository;

use Doctrine\ORM\EntityRepository;

/**
 * CrudRepository
 *
 * This class was generated by the Doctrine ORM. Add your own custom
 * repository methods below.
 */
class CrudRepository extends EntityRepository
{
    /**
     * getAllDataSortByUpdateDate
     * dtb_crudの値を全て返却、引数により更新日をソート
     *
     * @param string $order
     * @return array|bool
     */
    public function getAllDataSortByUpdateDate($order = 'asc') ★更新日付で昇順・降順でソートし全てのデーターを返却するメソッドを作成します。
    {
        $qb = $this->createQueryBuilder('dc'); ★クエリビルダーの取得

        $qb->select('dc') ★クエリ作成
            ->orderBy('dc.update_date', $order);

        $res = $qb->getQuery()->getResult(); ★一覧表示情報の取得

        return count($res) > 0 ? $res : false; ★取得情報の判定と対応値の返却
    }

    /**
     * save
     *
     * @param \Eccube\Entity\Crud $entity
     * @return bool
     */
    public function save(\Eccube\Entity\Crud $entity) ★引数のエンティティを保存するメソッド
    {
        $em = $this->getEntityManager(); ★エンティティマネージャーを取得

        try { ★リレーションをしていないため、トランザクションの必要性は低いがトランザクション定義
            $em->beginTransaction();
            $em->persist($entity);
            $em->flush($entity); ★エンティティの保存
            $em->commit();
        } catch (\Exception $e) {
            $em->rollback(); ★予想外のエラーが発生した際はロールバック

            return false;
        }

        return true;
    }
}
```
-->

- 上記の説明を行います。

### 一覧表示データーの取得

- 一覧表示データー取得用メソッドをレポジトリに定義します。

    1. まずメソッドを定義です。
        - 今回のメソッドは、**引数**で**更新日時の昇順・降順を指定**し、**データーがテーブルに一件でもあれば**、**ソート済みのオブジェクト配列を返却**します。

    1. クエリビルダーの取得。
        - **コントローラーでクエリビルダーを取得する際**は、**エンティティマネージャを取得**してから、**クエリビルダーを取得**していましたが、**レポジトリでは、$thisで取得可能**です。
        - その際の相違点として、**対象テーブルは、データベース構造定義ファイルで指定されている**ため、**fromメソッド**必要なく、**createQueryBuilderの引数**に**エイリアス**を定義します。

    1. **クエリ生成部は、コントローラー記述時とほとんど差異がなく**、**オーダーバイの条件を変数で渡している**箇所のみ**相違**があります。

    1. 最後に取得オブジェクト配列を返却していますが、その際にカウントでレコードがあるかどうか判定し、三項演算子で返却値を生成しています。

### 情報の保存

- **コントローラー**でサブミット値、入力値判定成功後に**エンティティマネージャーで保存**していた内容を、本**レポジトリに移設**します。

    1. まず**引数**ですが、**タイプヒンティング**を用い、**dtb_crudのエンティティしか受け付けない**様にしています。

    1. 次にエンティティマネージャーを取得します。
        - 情報**取得処理**とは**違い**、本メソッドでは、**エンティティマネージャーを利用**します。

    1. 次に、応用として、**トランザクション**処理のために、**try ～ catch**を利用しています。
        - 通常本ケースであれば、リレーションもないために、トランザクションの必要性は低いですが、**例題としてあえて利用**しています。

    1. トランザクションの開始です。
        - トランザクションの開始には、以下メソッドを利用します。
        
        ```
        $em->beginTransaction();
        ```

        - **エンティティマネージャー**から**上記メソッド**を呼び出すだけで、トランザクション処理が**開始**されます。

    1. 情報の保存
        - 情報の保存は、コントローラーで定義していた内容と、ほぼ同じで、**transaction**処理のためにに**$em->commit();**を呼び出し、コミットを行なっています。

    1. もし保存中に、不明なエラーが発生した時のために、catchでエラーを補足し、**$em->rollback()**を呼び出し、ロールバックを行なっています。

## コントローラーファイルからレポジトリメソッドを呼び出す
  - 上記でレポジトリにメソッドを定義する作業が完了しました。
  - 次にコントローラーファイルを修正して、**情報の取得、保存処理を完成**させます。

### コントローラーの修正

- 以下のファイルを開きます。
    - /src/Eccube/Controller/Tutorial/CrudController.php

    1. ファイルを開いて以下の様に修正します。

        - **CrudController.php**

<script src="http://gist-it.appspot.com/https://github.com/EC-CUBE/ec-cube.github.io/blob/master/Source/tutorial_11/CrudController_modified_repository.php"></script>

<!--
```
<?php
/*
 * This file is part of EC-CUBE
 *
 * Copyright(c) 2000-2015 LOCKON CO.,LTD. All Rights Reserved.
 *
 * http://www.lockon.co.jp/
 *
 * This program is free software; you can redistribute it and/or
 * modify it under the terms of the GNU General Public License
 * as published by the Free Software Foundation; either version 2
 * of the License, or (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program; if not, write to the Free Software
 * Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.
 */


namespace Eccube\Controller\Tutorial;

use Eccube\Application;
use Eccube\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;

class CrudController extends AbstractController
{
    const LIST_ORDER = 'desc'; ★ソート順切り替えに定数を宣言

    public function index(Application $app, Request $request)
    {
        $app->clearMessage();

        $Crud = new \Eccube\Entity\Crud(); ★フォーム生成部は処理かわらず

        $builder = $app['form.factory']->createBuilder('crud', $Crud);

        $form = $builder->getForm();

        $defaultForm = clone $form; ★処理成功時の画面で保持値をクリアするために、空のエンティティを格納したフォームを一時保存

        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $saveStatus = $app['eccube.repository.crud']->save($Crud); ★サブミット値をエンティティマネージャーではなくレポジトリのメソッドに渡す

            if ($saveStatus) {
                $app->addSuccess('データーが保存されました');
                $form = $defaultForm; ★フォームオブジェクトを一時保存しておいた空エンティティのフォームオブジェクトで上書き
            } else {
                $app->addError('データーベースの保存中にエラーが発生しました'); ★登録後一覧情報が取得出来ない際は、エラーメッセージを表示
            }
        } elseif ($form->isSubmitted() && !$form->isValid()) {
            $app->addError('入力内容をご確認ください');
        }

        $crudList = $app['eccube.repository.crud']->getAllDataSortByUpdateDate(self::LIST_ORDER); ★レポジトリに作成したメソッドで更新日時降順の全レコード取得

        return $app->render(
            'Tutorial/crud_top.twig',
            array(
                'forms' => $form->createView(),
                'crudList' => $crudList,
            )
        );
    }
}
```
-->

- 上記の説明を行います。

1. **フォーム生成部のロジックに変更はありません**。

1. 今回のレポジトリへのロジック移設とは直接関係ありませんが、**エンティティが空のフォームオブジェクトをクローン**します。
    - **保存成功時に、ユーザー入力データーをクリアするため**に、**new直後のエンティティ**と紐付いたフォームオブジェクトを一時保存しておきます。
    - データーベース登録にはクローン元のフォームオブジェクトを利用します。

1. **エンティティマネージャーで保存**していた、ユーザー入力値を、**レポジトリのメソッドに置き換え**ます。

1. エラー判定を行い、以下を返却します。
    - 成功時
        - 成功メッセージ
        - 空エンティティフォーム
    - 失敗時
        - エラーメッセージ

1. **エンティティマネージャー**と**クエリビルダー**で取得していた**一覧表示用配列オブジェクト**を、**レポジトリのメソッドに置き換え**ます。

### 表示内容の確認

- 最後に確認のためにブラウザにアクセスしてみましょう。

#### 初期画面

1. ブラウザのURLに「http://[ドメイン + インストールディレクトリ]/tutorial/crud」を入力してください。

1. レコードがあれば入力画面とリストが表示されているはずです。

---

![一覧画面](/images/img-tutorial11-view-rendar-default-has.png)

---

#### データー保存時

1. ブラウザのURLに「http://[ドメイン + インストールディレクトリ]/tutorial/crud」を入力してください。

1. 各項目に正しい値を入力して登録を行なってください。

1. 以下が表示されるはずです。

---

![データー保存後データ登録画面](/images/img-tutorial11-view-rendar-adddata.png)

---

#### エラー内容入力時

1. ブラウザのURLに「http://[ドメイン + インストールディレクトリ]/tutorial/crud」を入力してください。

1. 投稿者ハンドルネームに「テスト3」を入力してください。

1. 以下が表示されるはずです。

---

![エラーデーター登録時データ登録画面](/images/img-tutorial11-view-rendar-error.png)

---

## 本章で学んだ事

1. データーベース構造定義ファイルのレポジトリ定義の確認を行いました。
1. レポジトリファイルの作成を行いました。
1. 作成レポジトリのサービスプロバイダへの登録を行いました。
1. Doctrine定義のマジックメソッド(取得系)の説明を行いました。
1. マジックメソッドを使ったコントローラーからの情報取得を行いました。
1. レポジトリでのメソッド定義を行いました。
1. レポジトリ内でのクエリビルダーの取得構築を行いました。
1. レポジトリ内メソッドでの保存方法(エンティティマネージャー利用)を説明しました。
1. レポジトリ内メソッドでのトランザクション利用方法を説明しました。
1. コントローラーからレポジトリメソッドの呼び出し方法を説明しました。