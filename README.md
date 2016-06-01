## Alchemy APIをBluemixのNode-REDから呼び出そう!
***
### 概要

IBM Watsonのサービスの一つであるAlchemy APIを呼びだす簡単なサンプルです。

Node-REDを使って簡単に呼び出しているのが特徴です。   
IBM Bluemixを使えば簡単に、そして迅速にアプリケーションを作ることが可能です。  

プログラミングが初めて、あるいは始めたばかりの方にもわかりやすいように解説させていただきます。

***
ノード解説
#### Alchemy Image Analysis node

IBMがBluemixは様々なコグニティブAPIを提供しています。その中でもAlchemyの画像認識サービスであるImage Analysis<img src="images/Node-RED___mz-nodered-z002_eu-gb_mybluemix_net.png" width="70px">は画像解析から年齢や人物判定まで行う機能を持ったサービスです。IBM Watsonのカテゴリに入っているので確認してみてください。

***
### 全体フロー概要

画像のURL（例："http://xxxxx.jpg" ）をImage AnalysisのAPIにかけると画像解析を行い、顔認識の結果を返してくれるサンプルアプリです。

![Nodeの全体像](images/Node-RED_Image_Analysis_all.png)


***
## 1.BluemixでNode-REDサービス、WatsonのAPIを設定する

### 1-1.Node-RED/Watson APIの準備
[こちら](https://www.ibm.com/developerworks/community/wikis/home?lang=ja#!/wiki/Wdec01e50fbfa_493c_8a88_6dd85c4d983f/page/%E5%AD%A6%E7%BF%92%E7%94%A8%E6%95%99%E6%9D%90)のp.38までを参考にHello WorldをNode-REDで

***


## 2.Node-REDでプログラミング

### 2-1.HTTP Input node

AlchemyAPIはRESTのGETメソッドでアクセスして画像を解析します。  
まずは左側のパレットのInputカテゴリ内のhttpのnode<img src="images/httpinput-node.png" width="120px">をドラッグ&ドロップし、キャンバス内に配置します。
プロパティー内のURL欄にアクセスポイントを記載します。ここでは/callwatson　とでもしておきます。
![httpinput-property](images/htttpinput-property.png)


Nameの欄はノードの名前をわかりやすいようにしておくために記述しておきます。任意ですが、ここではHTTP Inputにしておきます。

### 2-2.switch node

画像のURLをチェックするノードを準備します。
左側のリソースパレットのfunctionカテゴリ内のswitchノード![switch-node](images/switch-node.png)をフローエディタ中央のキャンバスにドラッグ&ドロップします。  
プロパティー内の左下にある+ruleをクリックして、分岐ロジックを2つ用意します。
Propertyは以下の通りにimagurl属性に含まれるペイロードのnullチェックを行います。
nullであれば、"1"にそれ以外であれば"2"に値が渡されます。![swich-property](images/switch-property.png)


### 2-3 template node (初期画面)

画面のHTMLを表示したり、Inputとなる画像を送信するためのメニューを提供するためにHTMLを記述します。
temlpalteノード![template-node](images/template-node.png)をフローエディタ中央のキャンバスにドラッグ&ドロップします。  
プロパティを以下のように記述します。


```
<h1>Welcome to the Alchemy Vision Face Detection Demo on Node-RED</h1>
<H2>Select an image</H2>
<form  action="{{req._parsedUrl.pathname}}">
    <img src="https://si.wsj.net/public/resources/images/MK-CK494_SELFIE_GR_20140303174816.jpg" height='100'/>
    <img src="https://upload.wikimedia.org/wikipedia/commons/f/f1/34th_G8_summit_member_20080707.jpg" height='100'/>
    <img src="http://demo1.alchemyapi.com/images/vision/politicians.jpg" height='100'/>
        <br/>Copy above image location URL or enter any image URL:<br/>
    Image URL: <input type="text" name="imageurl"/>
    <input type="submit" value="Analyze"/>
</form>
```


### 2-4.change node

入力画面から画像URLを抽出するchangeノードを定義します。左側のリソースパレットのfunctionカテゴリ内のchangeノード
![change-node](images/change-node.png)をフロー・エディタ中央のキャンバスにドラッグ&ドロップします。
ここからpayload属性をimageurl属性に変換します。以下の通りにプロパティを設定します。
![changenode-property](images/changenode-property.png)


### 2-5.Image Analysis

画像解析のためのImage Analysisノードを定義します。左側のリソースパレットのWatsonカテゴリ内のImage Analysisノード![imageanalysis](images/Node-RED___mz-nodered-z002_eu-gb_mybluemix_net.png) をフロー・エディタ中央のキャンバスにドラッグ&ドロップします。
プロパティーでは顔認識を行うため、以下の通りにDetectをFaceに設定します。
![imageanalysis-property](images/imageanalysis-property.png)


### 2-6. template node (結果)

WatsonのImage Analysisから返ってきた結果を表示させるためのHTMLを記載します。temlpalteノード![template-node](images/template-node.png)をフローエディタ中央のキャンバスにドラッグ&ドロップします。  
プロパティを以下のように記述します。


```
<h1>Alchemy Image Analysis</h1>
    <p>Analyzed image: {{payload}}<br/><img id="alchemy_image" src="{{payload}}" height="50"/></p>
    {{^result}}
        <P>No Face detected</P>
    {{/result}}
    <table border='1'>
        <thead><tr><th>Age Range</th><th>Score</th><th>Gender</th><th>Score</th><th>Name</th></tr></thead>
        {{#result}}<tr>
            <td><b>{{age.ageRange}}</b></td><td><i>{{age.score}}</i></td>
            <td>{{gender.gender}}</td><td>{{gender.score}}</td>
            {{#identity}}<td>{{identity.name}} ({{identity.score}})</td>{{/identity}}
        </tr>{{/result}}
    </table>
    <form  action="{{req._parsedUrl.pathname}}">
        <input type="submit" value="Try again"/>
    </form>
```

![templateoutput-property](images/templateoutput-property.png)


### 2-7.フローをつなげる

出来上がった客ノードをつなげて、右上のDepoyをクリックすれば完成です!エラーが出ていないことを確認してください。
![deploy](images/deploy.png)

## 3.動作確認
