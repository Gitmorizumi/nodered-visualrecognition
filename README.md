## Watsonの画像認識をBluemixのNode-REDから呼び出そう!(Visual Recognition V3)
***
### 概要

IBM Watsonのサービスの一つである画像認識APIを呼びだす簡単なサンプルです。

Node-REDを使って簡単に呼び出しているのが特徴です。   
IBM Bluemixを使えば簡単に、そして迅速にアプリケーションを作ることが可能です。  

プログラミングが初めて、あるいは始めたばかりの方にもわかりやすいように解説させていただきます。

***
ノード解説
#### Watson Visual Recognition

IBMがBluemixは様々なコグニティブAPIを提供しています。その中でもVisual Recognition<img src="images/visual-recognition-node.png" width="90px">は画像解析から年齢や人物判定まで行う機能を持ったサービスです。IBM Watsonのサービスなので確認してみてください。

***
### 全体フロー概要

画像のURL（例："http://xxxxx.jpg" ）をVisual RecognitionのAPIにかけると画像解析を行い、顔認識の結果を返してくれるサンプルアプリです。

![Nodeの全体像](images/vr-overallnode.png)


***
## 1.BluemixでNode-REDサービス、WatsonのAPIを設定する

### 1-1.Node-RED/Watson APIの準備
まず、Node-REDのサービスを立ち上げます。

カタログをクリック。
![node-red_1](images/node-red_1.png)

次にボイラーテンプレートからNode-RED Starterをクリックします。
![node-red_2](images/node-red_2.png)

画面が変わるのでアプリケーション名を他人と被らないように入れます。（英文字とハイフンはOKです）
名前を入れたらそのまま右下にある作成ボタンをクリック。
![node-red_3](images/node-red_3.png)

その後、みなさんのクラウド環境上にNode-REDのアプリケーションが作成されます。アプリケーション名のすぐ右側のステータスが'実行中'に変わったら、Visit App URLをクリックします。
![node-red_4](images/node-red_4.png)

Welcome画面が表示されたら成功です！
次に右下のNextをクリックして先に進めます。
![node-red_5](images/node-red_5.png)

ここでは便宜上、ユーザー認証を設定しません。（通常はお勧めしません）
Allow anyone〜　をクリックしてすぐしたのチェックボックスにチェックを入れます。
右下のNextをクリックして先に進めます。
![node-red_6](images/node-red_6.png)

Flow Libraryの説明画面になるので、そのまま右下のNextで先に進みます。
![node-red_7](images/node-red_7.png)

最後はFinishをクリックしてNode-REDアプリの完成です！
![node-red_8](images/node-red_8.png)

Node-REDの初期画面が出てきたら成功です。さっそくGo to your Node-RED flow editorをクリックしてみましょう。
![node-red_9](images/node-red_9.png)

フローエディタが出てきました！これで完成です。
![node-red_10](images/node-red_10.png)

***
## 2.Visual Recognition APIを追加する
さて、次にWatsonのAPIを準備するため、Bluemixのコンソール画面に戻ります。左側のメニューから接続をクリック。その後、Node-REDにバインドされているサービス一覧が表示されるので、右側にある新規に接続をクリック。
![node-red_11](images/node-red_11.png)

そのままWatsonのVisual Recognitionを探してクリックしてください
![node-red_12](images/node-red_12.png)

Visual Recognitionのサービス説明画面になるので、右下の作成ボタンをクリック。
![node-red_13](images/node-red_13.png)

再ステージのポップアップが表示されるので、OKをクリックして再起動させます。

Bluemixのダッシュボードの"概要"メニューをクリックすると、Visual  Recognitionが接続されていることを確認できます。
![node-red_15](images/node-red_15.png)

アプリケーションが再起動されたら、ダッシュボードのステータスを確認し実行中のステータスであることを確認しurlをクリックします。
![node-red_14](images/node-red_14.png)

これで準備が整いました。
![node-red_9](images/node-red_9.png)


## 3.Node-REDでプログラミング

### 3-1.HTTP Input node

Visual RecognitionはRESTのGETメソッドでアクセスして画像を解析します。  
まずは左側のパレットのInputカテゴリ内のhttpのnode<img src="images/httpinput-node.png" width="120px">をドラッグ&ドロップし、キャンバス内に配置します。
プロパティー内のURL欄にアクセスポイントを記載します。ここでは/callwatson　とでもしておきます。
![httpinput-property](images/htttpinput-property.png)


Nameの欄はノードの名前をわかりやすいようにしておくために記述しておきます。任意ですが、ここではHTTP Inputにしておきます。

### 3-2.switch node

画像のURLをチェックするノードを準備します。
左側のリソースパレットのfunctionカテゴリ内のswitchノード![switch-node](images/switch-node.png)をフローエディタ中央のキャンバスにドラッグ&ドロップします。  
プロパティー内の左下にある+ruleをクリックして、分岐ロジックを2つ用意します。
Propertyは以下の通りにimagurl属性に含まれるペイロードのnullチェックを行います。
nullであれば、"1"にそれ以外であれば"2"に値が渡されます。![swich-property](images/switch-property.png)


### 3-3 template node (初期画面)

画面のHTMLを表示したり、Inputとなる画像を送信するためのメニューを提供するためにHTMLを記述します。
temlpalteノード![template-node](images/template-node.png)をフローエディタ中央のキャンバスにドラッグ&ドロップします。  
プロパティを以下のように記述します。


```
<h1>Welcome to a Watson Visual Recognition sample Face Detection app</h1>
<H2>Recognize anyone?</H2>
<form  action="{{req._parsedUrl.pathname}}">
    <img src="http://sysrun.haifa.il.ibm.com/ibm/history/exhibits/chairmen/images/watsonsr.jpg" height='200'/>
    <img src="http://www.awaken.com/wp-content/uploads/2015/05/forbes.jpg" height='200'/>
    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/52/LinuxCon_Europe_Linus_Torvalds_03.jpg/220px-LinuxCon_Europe_Linus_Torvalds_03.jpg" height='200'/>
    <img src="http://smashinghub.com/wp-content/uploads/2012/01/nb5.jpg" height='200'/>
        <br/>Right-click one of the above images and select Copy image location and paste the URL in the box below.<br>Do an image search for faces, try multiple faces. After you click on an image, to the right it usually says "View image" click that to get the URL.<br/>
    <br>Image URL: <input type="text" name="imageurl"/>
    <input type="submit" value="Analyze"/>
</form>
```

![templateoutput-property1](images/vr-temp1prop.png)


### 3-4.change node

入力画面から画像URLを抽出するchangeノードを定義します。左側のリソースパレットのfunctionカテゴリ内のchangeノード
![change-node](images/change-node.png)をフロー・エディタ中央のキャンバスにドラッグ&ドロップします。
ここからpayload属性をimageurl属性に変換します。以下の通りにプロパティを設定します。
![changenode-property](images/changenode-property.png)


### 3-5.Image Analysis

画像解析のためのImage Analysisノードを定義します。左側のリソースパレットのWatsonカテゴリ内のImage Analysisノード![imageanalysis](images/visual-recognition-node.png) をフロー・エディタ中央のキャンバスにドラッグ&ドロップします。
プロパティーでは顔認識を行うため、以下の通りにDetectをDetect Faceに設定します。
![imageanalysis-property](images/vr-nodeproperty.png)


### 3-6. template node (結果)

WatsonのImage Analysisから返ってきた結果を表示させるためのHTMLを記載します。temlpalteノード![template-node](images/template-node.png)をフローエディタ中央のキャンバスにドラッグ&ドロップします。  
プロパティを以下のように記述します。


```
<h1>Visual Recognition v3 Image Analysis</h1>
    <p>Analyzed image: {{result.images.0.resolved_url}}<br/><img id="image" src="{{result.images.0.resolved_url}}" height="200"/></p>
    {{^result}}
        <P>No Face detected</P>
    {{/result}}
    <p>Images Processed: {{result.images_processed}}</p>
    <table border='1'>
        <thead><tr><th>Age Range</th><th>Confidence</th><th>Gender</th><th>Confidence</th><th>Name</th></tr></thead>
        {{#result.images.0.faces}}<tr>
            <td><b>{{age.min}} - {{age.max}}</b></td><td><i>{{age.score}}</i></td>
            <td>{{gender.gender}}</td><td>{{gender.score}}</td>
            <td>{{identity.name}} ({{identity.score}})</td>
        </tr>{{/result.images.0.faces}}
    </table>
    <form  action="{{req._parsedUrl.pathname}}">
        <br><input type="submit" value="Try again or go back to the home page"/>
    </form>
```

![templateoutput-property2](images/vr-temp2prop.png)


### 3-7.フローをつなげる

出来上がった客ノードをつなげて、右上のDepoyをクリックすれば完成です!エラーが出ていないことを確認してください。
![deploy](images/deployv3.png)

## 4.動作確認
ブラウザのURL欄にhttp://xxxx.mybluemix.net/callwatson をインプットして呼び出してみましょう。
Image URLの入力欄にWatsonに読ませたい画像URLを入れてみてください。

顔の認識や人物名が出てきます！さすがWatson！
![steve](images/steve.png)
