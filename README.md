Webpack是一個模組打包工具，目的是把在JavaScript模組系統下的JS模組打包成實際在browser上執行的一到多個JS檔，並處理各模組間的相依關係，另外透過原生或第三方的模組，Webpack也可以做到如程式碼轉化、編譯等原本經常是使用前端建置工具如gulp、grunt來完成的工作。

## 環境設定
##### 整體設定
首先安裝Node，安裝完成後即可使用Node附帶的NPM套件管理工具安裝Webpack及其他相關的plugin。接下來可以選擇性的安裝全域的Webpack： `npm install -g webpack`，透過`-g`option來指定全域安裝，讓webpack可以直接在命令列當中使用。
##### 專案設定
使用`npm init`指令，並依照提示輸入資料後NPM會自動生成package.json。準備好package.json後即可開始安裝local端的webpack：`npm install --save-dev webpack`，雖然webpack可以透過命令列執行，但是為了管理方便，通常會在專案的根目錄下在新增一個webpack專用的設定檔webpack.config.js，接下來所有的webpack設定皆會寫在此檔案中。

## 設定Babel
Babel是可以將最新標準的JavaScript程式碼編譯為瀏覽器可執行之程式碼的工具

##### 基本設定

首先安裝需要的套件

```
npm install --save-dev babel-core babel-preset-es2015 babel-loader
```

babel-core是babel的核心、babel-preset-es2015是babel的plugin組合，基本包括了絕大部分es2015(es6)中的功能，也可以自行設定模組set，最後babel-loader是實際webpack在執行時用來將程式碼做轉化的外掛工具。

> 這邊要注意的是在某些專案的package.json中不會看到babel-core這個dependency，但是node_modules當中卻確實有安裝，這是因為babel-core被列在babel-loader的[peerDependencies](https://nodejs.org/en/blog/npm/peer-dependencies/)當中，在NPM@3之前peerDependencies當中的套件預設會自動安裝在project的node_modules當中，但是此行為在NPM@3之後的版本已[取消](http://blog.npmjs.org/post/110924823920/npm-weekly-5)，peerDependencies不會自動安裝，而是改為如果沒有安裝則發出警告。

query在這邊是做為[Secondary sources of configuration data](https://leanpub.com/setting-up-es6/read#leanpub-auto-sources-of-configuration-data)，也就是說只有在.babelrc沒有指定presets(或是根本沒有.babelrc)的情況下才會使用這裡的設定

webpack.config.js

```javascript
module.exports = {
  // ...
  module: {
    loaders: [{
      include: path.resolve(__dirname, 'app'),
      test: /\.(js|jsx)$/,
      loaders: ['babel-loader'],
      query: {
        presets: ['es2015']
      }
    }]
  }
}
```
##### runtime設定

##### 參考資料
- [Setting up ES6](https://leanpub.com/setting-up-es6/read)




## 設定CSS/SCSS
安裝需要的loader與module

`npm install --save-dev style-loader css-loader sass-loader postcss-loader autoprefixer`

在這邊sass-loader用來將sass編譯成css，透過postcss-loader與autoprefixer plugin將css轉化(加入瀏覽器prefix)、css-loader讀取分析css檔案，最後透過style-loader將css插入到html的style標籤中。

另外可以在loader後加上參數啟用常用的設定:
- 在css-loader和sass-loader後面加上sourceMap參數啟用sourcemap功能
- 在css-loader後加上minimize參數啟用Minification

webpack.config.js

```javascript
var autoprefixer = require('autoprefixer');

module.exports = {
  // ...
  module: {
    loaders:[{
      test: /\.(css|scss)$/,
      loaders: ['style', 'css?sourceMap&minimize', 'postcss', 'sass?sourceMap'] 
    }]
  },
  postcss: function() {
    return [autoprefixer];
  }
}
```



