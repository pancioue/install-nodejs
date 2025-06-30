## 安裝
fnm 套件管理安裝 node 非常方便 [官方網站](https://nodejs.org/zh-tw/download)
> fnm 與 nvm相比號稱更為快速，跨平台相容性更佳，版本幾乎瞬間切換

## 特色
各自角色安裝 fnm，node，與 npm並不會打架，這點很重要，可以用不同角色切換任意版本。

## 關於 npm install 安裝套件
專案裡執行 `npm install` 時建議用 www 角色，避免 root。  
`npm install` 會載入、建立檔案，若用 root 執行，套件中的某些 install script 可取得 root 權限，造成安全風險。
因此最好使用 www 下 `npm install`

## 用www安裝完，可能無法使用 fnm
* `sudo -u www fnm install 22` => sudo: fnm: command not found
* `sudo -u www ~/.local/share/fnm/fnm ls` => sudo: /root/.local/share/fnm/fnm: command not found 會找 root 的而不是 www
* `sudo -u www bash -c 'fnm ls'` => bash: line 1: fnm: command not found  ，www 沒吃到 .bashrc 底下的設定
* `sudo -u www bash -c '~/.local/share/fnm/fnm ls'` => 正確

## 安裝 node 
`sudo -u www bash -c '~/.local/share/fnm/fnm install 22'`
安裝完之後下 `sudo -u www bash -c 'node -v'` 依然會有問題，可以在

__/usr/local/bin/www-node.sh__
```
#!/bin/bash
export PATH="/home/www/.local/share/fnm:$PATH"
eval "$(/home/www/.local/share/fnm/fnm env)"
exec node "$@"
```
之後只要下 `sudo -u www bash -c '/usr/local/bin/www-node.sh -v'` 即可  
若在程式中 
``` php
exec('/usr/local/bin/www-node.sh /path/srcipt.js', $output, $returnVar);
```

## npm install
至於 npm install 因為大部分是一次性的，之後應該不太會用到，所以可以用偷吃步的方法就好
```
sudo -u www bash -c 'export PATH="$HOME/.local/share/fnm:$PATH"; eval "$(fnm env)"; npm install'
```

### Puppeteer
* 執行node 的 script 以後不指定瀏覽器的話，會抓預設的，但會缺少套件  
  `... error while loading shared libraries: libatk-1.0.so.0: cannot open shared object file: No such file or directory`  
  - 安裝套件 => `sudo apt-get install -y libatk1.0-0`  
    安裝時若看到 update-alternatives: error: alternative path等錯誤，
    那是 apt-get 在處理套件資料庫或 alternatives 更新時出現的訊息，通常是某次安裝或套件殘留設定造成的，
    和 libatk1.0-0 完全無關，也不影響 puppeteer 執行
  
* 結果中間會一直缺套件，直接裝完全部指令
  ```
  sudo apt-get update && \
  sudo apt-get install -y \
    libatk1.0-0 \
    libatk-bridge2.0-0 \
    libcups2 \
    libnss3 \
    libxcomposite1 \
    libxdamage1 \
    libxfixes3 \
    libxrandr2 \
    libgbm1 \
    libasound2 \
    libpango1.0-0 \
    libgtk-3-0 \
    && sudo rm -rf /var/lib/apt/lists/*
  ```
* 不過這上面指令安裝會出錯，原因是版本的問題，有些套件是不同名稱，如果使用的是 `Ubuntu 24.04.2 LTS`，指令應改成
  ```
  sudo apt-get update && \
  sudo apt-get install -y \
    libatk1.0-0t64 \
    libatk-bridge2.0-0t64 \
    libcups2t64 \
    libnss3 \
    libxcomposite1 \
    libxdamage1 \
    libxfixes3 \
    libxrandr2 \
    libgbm1 \
    libpango-1.0-0 \
    libgtk-3-0t64 \
    libasound2t64 \
    && sudo rm -rf /var/lib/apt/lists/*
  ```
  若不處理音訊，可以省略 libasound2t64
