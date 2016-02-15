.. _SpringSecurityCsrf:

CSRF対策
================================================================================

.. only:: html

 .. contents:: 目次
    :local:

Overview
--------------------------------------------------------------------------------

本節では、Spring Securityが提供しているCross site request forgeries(以下、CSRFと略す）対策の機能について説明する。

CSRFとは、Webサイトにスクリプトや自動転送(HTTPリダイレクト)を実装することにより、
ユーザーが、ログイン済みの別のWebサイト上で、意図しない何らかの操作を行わせる攻撃手法のことである。

サーバ側でCSRFを防ぐには、以下の方法が知られている。

* 秘密情報(トークン)の埋め込み
* パスワードの再入力
* Refererのチェック

CSRF対策機能は、攻撃者が用意したWebページから送られてくる偽造リクエストを不正なリクエストとして扱うための機能である。
CSRF対策が行われていないWebアプリケーションを利用すると、以下のような方法で攻撃を受ける可能性がある。

* 利用者は、CSRF対策が行われていないWebアプリケーションにログインする。
* 利用者は、攻撃者からの巧みな誘導によって、攻撃者が用意したWebページを開いてしまう。
* 攻撃者が用意したWebページは、フォームの自動送信などのテクニックを使用して、偽造したリクエストをCSRF対策が行われていないWebアプリケーションに対して送信する。
* CSRF対策が行われていないWebアプリケーションは、攻撃者が偽造したリクエストを正規のリクエストとして処理してしまう。


.. tip::

    OWASP\ [#fSpringSecurityCSRF1]_\では、\ `トークンパターンを使用する方法が推奨されている。 <https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet#General_Recommendation:_Synchronizer_Token_Pattern>`_\
    
      .. [#fSpringSecurityCSRF1] Open Web Application Security Projectの略称であり、信頼できるアプリケーションや、セキュリティに関する  効果的なアプローチなどを検証、提唱する、国際的な非営利団体である。
       https://www.owasp.org/index.php/Main_Page

.. note:: **ログイン時におけるCSRF対策**

    CSRF対策はログイン中のリクエストだけではなく、ログイン処理でも行う必要がある。
    ログイン処理に対してCSRF対策を怠った場合、攻撃者が用意したアカウントを使って知らぬ間にログインさせられ、ログイン中に行った操作履歴などを盗まれる可能性がある。


Spring SecurityのCSRF対策
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、セッション単位にランダムに生成される固定トークン値(CSRFトークン)を払い出し、払い出されたCSRFトークンをリクエストパラメータ(HTMLフォームのhidden項目)として送信する。
これにより正規のWebページからのリクエストなのか、攻撃者が用意したWebページからのリクエストなのかを判断する仕組みを採用している。

.. figure:: ./images_CSRF/Csrf.png
    :width: 100%

    **Spring SecurityのCSRF対策の仕組み**

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | クライアントは、HTTPのGETメソッドを使用してアプリケーションサーバにアクセスする。
    * - | (2)
      - | Spring Securityは、CSRFトークンを生成しHTTPセッションに格納する。
        | 生成したCSRFトークンは、HTMLフォームのhiddenタグを使ってクライアントと連携する。
    * - | (3)
      - | クライアントは、HTMLフォーム内のボタンを押下してアプリケーションサーバーにリクエストを送信する。
        | HTMLフォーム内のhidden項目にCSRFトークンが埋め込まれているため、CSRFトークン値はリクエストパラメータとして送信される。
    * - | (4)
      - | Spring Securityは、HTTPのPOSTメソッドを使ってアクセスされた際は、リクエストパラメータに指定されたCSRFトークン値とHTTPセッション内に保持しているCSRFトークン値が同じ値であることをチェックする。
        | トークン値が一致しない場合は、不正なリクエスト(攻撃者からのリクエスト)としてエラーを発生させる。
    * - | (5)
      - | クライアントは、HTTPのGETメソッドを使用してアプリケーションサーバにアクセスする。
    * - | (6)
      - | Spring Securityは、GETメソッドを使ってアクセスされた際は、CSRFトークン値のチェックは行わない。

.. note:: **Ajax使用時のCSRFトークン**

    Spring Securityは、リクエストヘッダにCSRFトークン値を設定することができるため、Ajax向けのリクエストなどに対してCSRF対策を行うことが可能である。

.. _csrf_ckeck-target:

トークンチェックの対象リクエスト
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityのデフォルト実装では、以下のHTTPメソッドを使用したリクエストに対して、CSRFトークンチェックを行う。

* POST
* PUT
* DELETE
* PATCH

.. note:: **CSRFトークンチェックを行わない理由**

    GET, HEAD, OPTIONS, TRACE メソッドがチェック対象外となっている理由は、これらのメソッドがアプリケーションの状態を変更するようなリクエストを実行するためのメソッドではないためである。

.. _csrf_spring-security-setting:

How to use
--------------------------------------------------------------------------------

CSRF対策機能の適用
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

CSRFトークン用の\ ``RequestDataValueProcessor``\ 実装クラスを利用し、Springのタグライブラリの\ ``<form:form>``\ タグを使うことで、自動的にCSRFトークンを、hiddenに埋め込むことができる。

* spring-mvc.xmlの設定例

.. code-block:: xml

    <bean id="requestDataValueProcessor"
        class="org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor"> <!-- (1)  -->
        <constructor-arg>
            <util:list>
                <bean
                    class="org.springframework.security.web.servlet.support.csrf.CsrfRequestDataValueProcessor" /> <!-- (2)  -->
                <bean
                    class="org.terasoluna.gfw.web.token.transaction.TransactionTokenRequestDataValueProcessor" />
            </util:list>
        </constructor-arg>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ 共通ライブラリから提供されている、\ ``org.springframework.web.servlet.support.RequestDataValueProcessor``\ を複数定義可能な
       | \ ``org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor``\ をbean定義する。
   * - | (2)
     - | コンストラクタの第1引数に、\ ``org.springframework.security.web.servlet.support.csrf.CsrfRequestDataValueProcessor``\ のbean定義を設定する。

Spring Security 4.0からは、上記設定により、デフォルトでCSRF対策機能が有効となる。このため、CSRF対策機能を適用したくない場合は、明示的に無効化する必要がある。 

CSRF対策機能を使用しない場合は、以下のようなbean定義を行う。

* spring-security.xmlの定義例

.. code-block:: xml

    <sec:http>
        <!-- omitted -->
        <sec:csrf disabled="true"/> <!-- disabled属性にtrueを設定して無効化 -->
        <!-- omitted -->
    </sec:http>

CSRFトークン値の連携
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Securityは、CSRFトークン値をクライアントとサーバー間で連携する方法として、以下の2種類の方法を提供している。

* HTMLフォームのhidden項目としてCSRFトークン値を出力し、リクエストパラメータとして連携する
* HTMLのmetaタグとしてCSRFトークンの情報を出力し、Ajax通信時にリクエストヘッダにトークン値を設定して連携する

.. _csrf_formtag-use:

Spring MVCを使用した連携
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Spring Securityは、Spring MVCと連携するためのコンポーネントをいくつか提供している。
ここでは、CSRF対策機能と連携するためのコンポーネントの使い方を説明する。

hidden項目の自動出力
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

HTMLフォームを作成する際は、以下のようなJSPの実装を行う。

* JSPの実装例

.. code-block:: jsp

    <%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>

    <c:url var="loginUrl" value="/login"/>
    <form:form action="${loginUrl}"> <!-- (1) -->
        <!-- omitted -->
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | HTMLフォームを作成する際は、Spring MVCから提供されている\ ``<form:form>``\ 要素を使用する。

Spring MVCから提供されている\ ``<form:form>``\ 要素を使うと、以下のようなHTMLフォームが作成される。

* HTMLの出力例

.. code-block:: html

    <form id="command" action="/login" method="post">
        <!-- omitted -->
        <!-- Spring MVCの機能と連携して出力されたCSRFトークン値のhidden項目 -->
        <div>
            <input type="hidden"
                   name="_csrf" value="63845086-6b57-4261-8440-97a3c6fa6b99" />
        </div>
    </form>

.. tip:: **出力されるCSRFトークンチェック値**

    Spring 4上で\ ``CsrfRequestDataValueProcessor``\ を使用すると、\ ``<form:form>``\ タグの\ ``method``\ 属性に指定した値がCSRFトークンチェック対象の
    HTTPメソッド(Spring Securityのデフォルト実装ではGET,HEAD,TRACE,OPTIONS以外のHTTPメソッド)と一致する場合に限り、CSRFトークンが埋め込まれた\ ``<input type="hidden">``\ タグが出力される。

    例えば、以下の例のように \ ``method``\ 属性にGETメソッドを指定した場合は、CSRFトークンが埋め込まれた\ ``<input type="hidden">``\ タグは出力されない。

        .. code-block:: jsp

            <form:form method="GET" modelAttribute="xxxForm" action="...">
                <%-- ... --%>
            </form:form>

    これは、\ `OWASP Top 10 <https://code.google.com/p/owasptop10/>`_\ で説明されている、

        The unique token can also be included in the URL itself, or a URL parameter. However, such placement runs a greater risk that the URL will be exposed to an attacker, thus compromising the secret token.

    に対応している事を意味しており、セキュアなWebアプリケーション構築の手助けとなる。

.. _csrf_htmlformtag-use:

HTMLフォーム使用時の連携
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

\ :ref:`Spring MVCと連携<csrf_formtag-use>` せずに、HTMLフォームを使用してCSRFトークン値を連携することも可能である。
HTMLフォームを使ってリクエストを送信する場合は、HTMLフォームのhidden項目としてCSRFトークン値を出力し、リクエストパラメータとして連携する。

* JSPの実装例

.. code-block:: jsp

    <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>

    <form action="<c:url value="/login" />" method="post">
        <!-- omitted -->
        <sec:csrfInput /> <!-- (1) -->
        <!-- omitted -->
    </form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | HTMLの\ ``<form>``\ 要素の中に\ ``<sec:csrfInput>``\ 要素を指定する。

Spring Securityから提供されている\ ``<sec:csrfInput>``\ 要素を指定すると、以下のようなhidden項目が出力される。
HTMLフォーム内にhidden項目を出力することで、CSRFトークン値がリクエストパラメータとして連携される。
デフォルトでは、CSRFトークン値を連携するためのリクエストパラメータ名は\ ``_csrf``\ になる。

* HTMLの出力例

.. code-block:: html

    <form action="/login" method="post">
        <!-- omitted -->
        <!-- CSRFトークン値のhidden項目 -->
        <input type="hidden"
               name="_csrf"
               value="63845086-6b57-4261-8440-97a3c6fa6b99" />
        <!-- omitted -->
    </form>

.. warning:: **GETメソッド使用時の注意点**

    HTTPメソッドとしてGETを使用する場合、\ ``<sec:csrfInput>``\ 要素を指定しないこと。
    \ ``<sec:csrfInput>``\ 要素を指定してしまうと、URLにCSRFトークン値が含まれてしまうため、CSRFトークン値が盗まれるリスクが高くなる。

.. _csrf_ajax-token-setting:

Ajax使用時の連携
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Ajaxを使ってリクエストを送信する場合は、HTMLのmetaタグとしてCSRFトークンの情報を出力し、metaタグから取得したトークン値をAjax通信時のリクエストヘッダに設定して連携する。

まず、Spring Securityから提供されているJSPタグライブラリを使用して、HTMLのmetaタグにCSRFトークンの情報を出力する。

* JSPの実装例

.. code-block:: jsp

    <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>

    <head>
        <!-- omitted -->
        <sec:csrfMetaTags /> <!-- (1) -->
        <!-- omitted -->
    </head>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | HTMLの\ ``<head>``\ 要素内に\ ``<sec:csrfMetaTags>``\ 要素を指定する。

\ ``<sec:csrfMetaTags>``\ 要素を指定すると、以下のようなmetaタグが出力される。
デフォルトでは、CSRFトークン値を連携するためのリクエストヘッダ名は\ ``X-CSRF-TOKEN``\ となる。

* HTMLの出力例

.. code-block:: html

    <head>
        <!-- omitted -->
        <meta name="_csrf_parameter" content="_csrf" />
        <meta name="_csrf_header" content="X-CSRF-TOKEN" /> <!-- ヘッダ名 -->
        <meta name="_csrf"
              content="63845086-6b57-4261-8440-97a3c6fa6b99" /> <!-- トークン値 -->
        <!-- omitted -->
    </head>

つぎに、JavaScriptを使ってmetaタグからCSRFトークンの情報を取得し、Ajax通信時のリクエストヘッダ
にCSRFトークン値を設定する。(ここではjQueryを使った実装例となっている)

* JavaScriptの実装例

.. code-block:: javascript

    $(function () {
        var headerName = $("meta[name='_csrf_header']").attr("content"); // (1)
        var tokenValue = $("meta[name='_csrf']").attr("content"); // (2)
        $(document).ajaxSend(function(e, xhr, options) {
            xhr.setRequestHeader(headerName, tokenValue); // (3)
        });
    });

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - | CSRFトークン値を連携するためのリクエストヘッダ名を取得する。
    * - | (2)
      - | CSRFトークン値を取得する。
    * - | (3)
      - | リクエストヘッダにCSRFトークン値を設定する。

.. _csrf_token-error-response:

トークンチェックエラー時のレスポンス
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

CSRFトークンチェックでエラーが発生した場合、Spring Securityは\ ``AccessDeniedHandler``\
インタフェースを使用してエラーのレスポンスを行う。

CSRFトークンチェックでエラーが発生した際に、専用のエラーページに遷移させたい場合は、Spring Securityから提供されている\ ``DelegatingAccessDeniedHandler``\ クラスを利用して、例外毎に\ ``AccessDeniedHandler``\ インタフェースの実装クラスを指定する。

CSRFのトークンチェック処理では、エラーの内容に応じて以下の2つの例外を使用する。

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **CSRFトークンチェックで使用される例外クラス**
    :header-rows: 1
    :widths: 35 65

    * - クラス名
      - 説明
    * - | \ ``InvalidCsrfTokenException``\
      - | クライアントから送られたトークン値と、サーバー側で保持しているトークン値が一致しない場合に使用する例外クラス。
    * - | \ ``MissingCsrfTokenException``\
      - | サーバー側にトークン値が保存されていない場合に使用する例外クラス。

CSRFトークンチェックエラー時に専用のエラー画面に遷移させたい場合のBean定義は「 :ref:`認可機能とCSRF対策の連動<SpringSecurityAuthorizationOnErrorWithCSRF>` 」を参照されたい。

.. _csrf_403-webxml-setting:

.. tip:: **<sec:access-denied-handler>の設定を省略した場合のエラーハンドリングについて**

  web.xmlに以下の設定を行うことで、任意のページに遷移させることができる。

    * web.xmlの設定例

    .. code-block:: xml

        <error-page>
            <error-code>403</error-code>  <!-- (1) -->
            <location>/WEB-INF/views/common/error/accessDeniedError.jsp</location>  <!-- (2) -->
        </error-page>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - 項番
         - 説明
       * - | (1)
         - | error-code要素に、ステータスコード403を設定する。
       * - | (2)
         - | location要素に、遷移先のパスを設定する。

.. note::

  **ステータスコード403以外を返却したい場合**

  リクエストに含まれるCSRFトークンが一致しない場合、ステータスコード403以外を返却したい場合は、\ ``org.springframework.security.web.access.AccessDeniedHandler``\ インタフェースを実装した、独自のAccessDeniedHandlerを作成する必要がある。

Appendix
--------------------------------------------------------------------------------

マルチパートリクエスト(ファイルアップロード)時の留意点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ファイルアップロードなど、マルチパートリクエストを送信する場合、formから送信されるCSRFトークンを\ ``springSecurityFilterChain``\ 内では取得できないため、CSRFトークンエラーが発生する。

そのため、マルチパートリクエストを送信する場合は、以下の方法によって、CSRFトークンを取得できるようにする必要がある。

* \ ``org.springframework.web.multipart.support.MultipartFilter``\ を使用する
* クエリのパラメータでCSRFトークンを送信する

.. note::

    それぞれメリット・デメリットが存在するため、システム要件を考慮して、採用する対策方法を決めて頂きたい。

ファイルアップロードの詳細については、\ :doc:`FileUpload <../ArchitectureInDetail/FileUpload>`\ を参照されたい。


.. _csrf_use-multipart-filter:

MultipartFilterを使用する方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
\ ``org.springframework.web.multipart.support.MultipartFilter``\ を使用することで、マルチパートリクエストでも、\ ``springSecurityFilterChain``\ 内で、formから送信されたCSRFトークンを取得することができる。

.. warning:: **MultipartFilterを使用する際の留意点**

    \ ``MultipartFilter``\ を使用した場合、\ ``springSecurityFilterChain``\による認証・認可処理が行われる前にアップロード処理が行われるため、認証又は認可されていないユーザーからのアップロード(一時ファイル作成)を許容してしまうことになる。

\ ``MultipartFilter``\ を使用するには、以下のように設定する。

* web.xmlの設定例

.. code-block:: xml

    <!-- (1) -->
    <filter>
        <filter-name>MultipartFilter</filter-name>
        <filter-class>org.springframework.web.multipart.support.MultipartFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>MultipartFilter</filter-name>
        <servlet-name>/*</servlet-name>
    </filter-mapping>

    <!-- (2) -->
    <filter>
        <filter-name>springSecurityFilterChain</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>springSecurityFilterChain</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``org.springframework.web.multipart.support.MultipartFilter``\ を 定義する。
   * - | (2)
     - | \ ``springSecurityFilterChain``\ より前に、\ ``MultipartFilter``\ を定義すること。

* JSPの実装例

.. code-block:: jsp

    <!-- (1) -->
    <form:form action="${pageContext.request.contextPath}/fileupload"
        method="post" modelAttribute="fileUploadForm" enctype="multipart/form-data">  
        <table>
            <tr>
                <td width="65%"><form:input type="file" path="uploadFile" /></td>
            </tr>
            <tr>
                <td><input type="submit" value="Upload" /></td>
            </tr>
        </table>
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<form:form>``\ タグを使用することで、CSRFトークンが埋め込まれた\ ``<input type="hidden">``\ タグが自動的に追加されるため、JSPの実装でCSRFトークンを意識する必要はない。
       |
       | **<html> タグを使用する場合**
       | :ref:`csrf_htmlformtag-use`\ を参照して、CSRFトークンを設定すること。


クエリパラメータでCSRFトークンを送る方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

認証又は認可されていないユーザーからのアップロード(一時ファイル作成)を防ぎたい場合は、\ ``MultipartFilter``\ は使用せず、クエリパラメータでCSRFトークンを送る必要がある。

.. warning:: **クエリパラメータでCSRFトークンを送る際の留意点**

    この方法でCSRFトークンを送信した場合、以下の危険性があるため、\ ``MultipartFilter``\ を使用する方法と比べると、攻撃者にCSRFトークンを悪用されるリスクが高くなる。
    なお、Spring Securityのデフォルト実装では、CSRFトークンの値としてランダムなUUIDを生成しているため、仮にCSRFトークンが漏洩してもセッションハイジャックされる事はないという点を補足しておく。

    * ブラウザのアドレスバーにCSRFトークンが表示される
    * ブックマークした場合、ブックマークにCSRFトークンが記録される
    * WebサーバのアクセスログにCSRFトークンが記録される


以下に、CSRFトークンをクエリパラメータとして送る実装例を示す。

* JSPの実装例

.. code-block:: jsp

    <form:form action="${pageContext.request.contextPath}/fileupload?${f:h(_csrf.parameterName)}=${f:h(_csrf.token)}"
        method="post" modelAttribute="fileUploadForm" enctype="multipart/form-data"> <!-- (1) -->
        <table>
            <tr>
                <td width="65%"><form:input type="file" path="uploadFile" /></td>
            </tr>
            <tr>
                <td><input type="submit" value="Upload" /></td>
            </tr>
        </table>
    </form:form>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - 項番
     - 説明
   * - | (1)
     - | \ ``<form:form>``\ タグのaction属性に、CSRFトークンのパラメータ名と、CSRFトークン値をクエリパラメタとして付加する。

.. raw:: latex

   \newpage

