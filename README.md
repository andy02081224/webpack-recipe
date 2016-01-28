Webpack是一個模組打包工具，目的是把在JavaScript模組系統下的JS模組打包成實際在browser上執行的一到多個JS檔，並處理各模組間的相依關係，另外webpack還可以透過內建或第三方的工具去做編譯SASS、轉化ES6程式碼等，本來經常是使用前端建置工具如gulp、grunt來完成的工作。

## 環境設定
##### 整體設定
首先安裝Node，安裝完成後即可使用Node附帶的NPM套件管理工具安裝Webpack及其他相關的plugin。接下來可以選擇性的安裝全域的Webpack： `npm install -g webpack`，透過`-g`option來指定全域安裝，讓webpack可以直接在命令列當中使用。
##### 專案設定
使用`npm init`指令，並依照提示輸入資料後NPM會自動生成package.json。準備好package.json後即可開始安裝local端的webpack：`npm install --save-dev webpack`，雖然webpack可以透過命令列執行，但是為了管理方便，通常會在專案的根目錄下在新增一個webpack專用的設定檔webpack.config.js，接下來所有的webpack設定皆會寫在此檔案中。




