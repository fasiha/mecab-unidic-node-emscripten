I assume you have UniDic installed on your computer already (on macOS with Homebrew, `brew install mecab-unidic`), as well as [Git](https://git-scm.com/).

First, install and activate Emscripten [`emsdk`](http://kripken.github.io/emscripten-site/docs/getting_started/).

Confirm `emcc` is available:
```
emcc -v
```

Create the follow `pre.js` script somewhere:
```js
Module['preInit'] =
    function() {
  FS.mkdir('/asset_dir');
  FS.mount(NODEFS, {root: './asset_dir'}, '/asset_dir');
  // Module.arguments.push('-h');
  Module.arguments.push('-r');
  Module.arguments.push('/asset_dir/mecabrc');
  Module.arguments.push('-d');
  Module.arguments.push('/asset_dir/unidic');
}
```

```
git clone https://github.com/taku910/mecab.git
cd mecab
emconfigure ./configure
emmake make
cp ./src/.libs/mecab mecab.bc
cp src/.libs/libmecab.a .
emcc -O2 mecab.bc libmecab.a -o mecab.js -s ALLOW_MEMORY_GROWTH=1 --pre-js /PATH/TO/pre.js
```
Make sure you replace the `/PATH/TO/pre.js` with the correct value to the `pre.js` described above.

Next, create and fill an `asset_dir` folder to store MeCab configuration file and UniDic:
```
mkdir -p asset_dir/unidic
cp /usr/local/etc/mecabrc asset_dir
cp /usr/local/lib/mecab/dic/unidic/* asset_dir/unidic
```

Now you can run MeCab purely in Node!
```
echo 日本語が好きです。 | node mecab.js
```
will produce the following output:
```
日本	ニッポン	ニッポン	日本	名詞-固有名詞-地名-国
語	ゴ	ゴ	語	名詞-普通名詞-一般
が	ガ	ガ	が	助詞-格助詞
好き	スキ	スキ	好き	形状詞-一般
です	デス	デス	です	助動詞	助動詞-デス	終止形-一般
。			。	補助記号-句点
EOS
```

To make *this* Git repo/npm package:
```
mkdir -p mecab-unidic-node-emscripten
cp -r mecab.wasm mecab.js asset_dir mecab-unidic-node-emscripten
```

