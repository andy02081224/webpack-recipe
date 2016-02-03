Webpack是一個模組打包工具，目的是把在JavaScript模組系統下的JS模組打包成實際在browser上執行的一到多個JS檔，並處理各模組間的相依關係，另外透過原生或第三方的模組，Webpack也可以做到如程式碼轉化、編譯等原本經常是使用前端建置工具如gulp、grunt來完成的工作。

## 環境設定
##### 整體設定
首先安裝Node，安裝完成後即可使用Node附帶的NPM套件管理工具安裝Webpack及其他相關的plugin。接下來可以選擇性的安裝全域的Webpack： `npm install -g webpack`，透過`-g`option來指定全域安裝，讓webpack可以直接在命令列當中使用。
##### 專案設定
使用`npm init`指令，並依照提示輸入資料後NPM會自動生成package.json。準備好package.json後即可開始安裝local端的webpack：`npm install --save-dev webpack`，雖然webpack可以透過命令列執行，但是為了管理方便，通常會在專案的根目錄下在新增一個webpack專用的設定檔webpack.config.js，接下來所有的webpack設定皆會寫在此檔案中。

## 設定Babel
Babel是可以將最新標準的JavaScript程式碼轉化為瀏覽器可執行之程式碼的工具

##### 基本設定

首先安裝需要的套件

```
npm install --save-dev babel-core babel-preset-es2015 babel-loader
```

babel-core是babel的核心，但其不包括任何轉化功能，需要自己手動加入plugins去處理，而為了方便，babel也提供了所謂的presets，也就是plugins的組合，在這邊加上的是babel-preset-es2015，基本包括了絕大部分es2015(es6)中需要**轉化**(有些功能不是用轉化，而需要以polyfill的方式加上，見下[模擬es6環境以及helpers設定])的功能，也可以自行設定模組set，最後babel-loader將babel外掛進webpack。

> 這邊要注意的是在某些專案的package.json中不會看到babel-core這個dependency，但是node_modules當中卻確實有安裝，這是因為babel-core被列在babel-loader的[peerDependencies](https://nodejs.org/en/blog/npm/peer-dependencies/)當中，在NPM@3之前peerDependencies當中的套件預設會自動安裝在project的node_modules當中，但是此行為在NPM@3之後的版本已[取消](http://blog.npmjs.org/post/110924823920/npm-weekly-5)，peerDependencies不會自動安裝，而是改為如果沒有安裝則發出警告。

如下列範例，實際使用時，只需將'babel-loader'(或'babel')加入loaders即可。

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
##### 模擬es6環境以及helpers設定
上面的提到的基本設定其實指的是對無法用es5去實做之es6功能的轉化，因為根據ES6功能的不同，有些程式碼需要用到轉化，如arrow function，因為其用現有es5程式無法實做出來，因此需要像babel這種compiler將他轉為模擬他的功能、browser看得懂的es5程式碼，而有些功能，如string的一些新的prototype方法，他是有辦法用現有的es5程式碼直接去實作並封裝成新的API，對於這種功能需要的就是所謂的polyfill(shim，如[es6-shim](https://github.com/paulmillr/es6-shim))，他的目的是用現有的標準去模擬es6環境，而在babel當中，這需要手動處理，方法基本上有兩個:

1. 使用babel-polyfill模組將模擬es6環境外掛至global使用
2. 使用babel-runtime和babel-plugin-transform-runtime將模擬es6環境封裝至module使用

###### 使用babel-polyfill模組將模擬es6環境外掛至global使用:
首先安裝babel-polyfill模組

`npm install --save-dev babel-polyfill`

接著將其加入至設定檔的entry point當中即可

webpack.config.js

```javascript
module.exports = {
  entry: ['babel-polyfill'],
  // ...
}
```

###### 使用babel-runtime和babel-plugin-transform-runtime將模擬es6環境封裝至module使用:

安裝babel-runtime和babel-plugin-transfrom-runtime

`npm install --save-dev babel-runtime babel-plugin-transform-runtime`

將`transform-runtime`加到.babelrc的plugins當中

.babelrc
```json
{
	"presets": ['es2015'],
	"plugins": ["transform-runtime"]
}
```

在這邊實際函有polyfill環境的是babel-runtime，將他require進需要使用的檔案即可，如`require(‘babel-runtime/core-js/promise’)`即可使用promise，但是這樣會有ㄧ個問題就是每當要使用這些功能時就必須再require一次，非常麻煩，因此babel提供了babel-plugin-transform-runtime這個搭配的模組，使用者只需要像在全域環境下一樣直接使用這些API，此模組會對照[definition.js](https://github.com/babel/babel/blob/472ad1e6a6d4d0dd199078fdb08c5bc16c75b5a9/packages/babel-plugin-transform-runtime/src/definitions.js)自動把這些有使用API的地方轉化為require適當模組，所以假如使用者撰寫如下es6程式碼(範例取自舊官網)：
```javascript
var sym = Symbol();

var promise = new Promise;

console.log(arr[Symbol.iterator]());
```

babel-plugin-transform-runtime會將其轉化為：
```javascript
"use strict";

var _core = require("babel-runtime/core-js");

var sym = _core.Symbol();

var promise = new _core.Promise();

console.log(_core.$for.getIterator(arr));
```

###### helpers設定：
babel會使用許多helper function來幫助執行，預設會在每個模組當中都寫入這些function，在有多個模組的應用情境下容易造成redundency，所以babel額外提供了一個集合這些helper functions的模組讓這些helpers可以透過這個統一的介面被重用。在上面第二個方法「使用babel-runtime和babel-plugin-transform-runtime」的情況下，這些helper functions已經包括在babel-runtime當中毋須而外設定，而在第一個方法「使用babel-polyfill」中，需要在額外安裝`babel-plugin-external-helpers-2`這個模組：

`npm install --save-dev babel-plugin-external-helpers-2`

將`external-helpers-2`加到.babelrc的plugins當中
```json
{
	"presets": ["es2015"],
	"plugins": ["external-helpers-2"]
}
```

###### babel-polyfill或是babel-runtime + babel-plugin-transform-runtime?
參考[這篇](https://medium.com/@jcse/clearing-up-the-babel-6-ecosystem-c7678a314bf3#.dz9pj64vb)有非常詳細的說明，基本上第一個方案(babel-polyfill)會在全域環境中加入額外的環境，有可能會與其他的library或module發生衝突，而第二個方案是封裝在模組當中require進來使用，不會有此問題，不過要注意有些「ambiguous code」不會被轉化，如:

```javascript
console.log('123'.repeat(3));
```

在編譯過後仍然會是:
```javascript
'use strict';
    
console.log('123'.repeat(3));
```


##### 參考資料
- [Setting up ES6](https://leanpub.com/setting-up-es6/read)
- [Babel 6: configuring ES6 standard library and helpers](http://www.2ality.com/2015/12/babel6-helpersstandard-library.html)
- [Clearing up the Babel 6 Ecosystem](https://medium.com/@jcse/clearing-up-the-babel-6-ecosystem-c7678a314bf3#.fcez3o447)



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

## 設定ESLint

安裝所需套件

`npm install --save-dev eslint babel-eslint eslint-loader`

新增ESLint專案設定檔`.eslintrc`，新增`parser` key，value指定為`babel-eslint`，剩下的設定則參考[官網文件](http://eslint.org/docs/user-guide/configuring)自行依據專案設定。

.eslintrc
```json
{
  "extends": "eslint:recommended",
  "parser": "babel-eslint",
  "env": {
    "es6": true,
    "browser": true,
    "node": true,
    "jquery": true,
    "mocha": true
  },
  "ecmaFeatures": {
    "jsx": true,
    "modules": true
  },
  "rules": {
    "quotes": 0,
    "no-console": 0,
    "no-debugger": 1,
    "semi": [1, "always"],
    "no-trailing-spaces": 0,
    "eol-last": 0,
    "no-unused-vars": 0,
    "no-underscore-dangle": 0,
    "no-alert": 0,
    "no-lone-blocks": 0,
    "jsx-quotes": 1
  }
}

````

最後在webpack.config.js當中將`eslint-loader`加入`module.preLoaders`並新增`eslint` entry指定global的ESLint設定檔。

webpack.config.js
```javascript
module.exports = {
 module: {
    preLoaders: [{
      include: dir_app,
      test: /\.(js|jsx)$/,
      loaders:['eslint-loader']
    }],
  },
  eslint: {
    configFile: './.eslintrc'
  }
}
```
