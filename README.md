# Vue + Vue Router を使ってタブ切り替えを作成する

## 機能・要件
- Vue.jsでカテゴリ別にタブ切り替えをさせる
- 各カテゴリは、表示されている件数を表示
- 表示の並びは、五十音順でソートする
- ID、氏名、サムネイルは、jsonファイルで管理
- imgは、VueLazyloadで遅延ロードさせる
- Vue Routerを使って、ブラウザバックに対応

### ダブ切り替えのビューとロジックを作成

```HTML
<transition-group class="nameList__body" name="fade" tag="ul">
  <li class="nameList__item" v-for="(item, index) in items" :key="index"
    v-show="(item.category === currentCtg || currentCtg === 'ALL')">
    <article class="nameCard">
      <a class="nameCard__link" href="#" :href="item.url">
        <h2 class="nameCard__name">{{item.name}}</h2>
        <p :class="['nameCard__label',
      {'nameCard__label--A': (item.category === 'Category A')},
      {'nameCard__label--B': (item.category === 'Category B')},
      {'nameCard__label--C': (item.category === 'Category C')}
      ]">{{item.category}}</p>
        <div class="nameCard__img">
          <div class="nameCard__img__inner">
            <img src="/" v-lazy="item.logo" alt="">
          </div>
        </div>
      </a>
    </article>
    <!-- /.nameCard -->
  </li>
</transition-group>
```

```js
var list = new Vue({
  router: router,
  el: '#js-nameList',
  data: {
    items: [],
    sum: 0,
    categories: [
      { text: 'ALL', num: 0 },
      { text: 'Category A', num: 0 },
      { text: 'Category B', num: 0 },
      { text: 'Category C', num: 0 },
    ],
    currentCtg: 'ALL'
  },
  computed: {
  },
  methods: {
    queryPage: function () {
      if (typeof this.$route.query.label !== 'undefined' && this.$route.query.label !== null && this.$route.query.label !== '') {
        this.currentCtg = this.$route.query.label;
      }
      else {
        this.currentCtg = 'ALL';
      }
    }
  },
  mounted: function () {
    this.queryPage();
  },
  watch: {
    '$route': function () {
      this.queryPage();
    }
  },
  created: function () {
    axios.get('/json/list.json')
      .then(function (res) {
        list.items = res.data;
        for (var i = 0; i < list.items.length; i++) {
          for (var j = 1; j < list.categories.length; j++) {
            if (list.categories[j].text === list.items[i].category) {
              list.categories[j].num++;
            }
          }
          list.categories[0].num++;
          list.items[i].phonetic = replaceDakuToHan(list.items[i].phonetic);
        }
        list.items.sort(function (a, b) {
          if (a.phonetic < b.phonetic) return -1;
          else if (a.phonetic > b.phonetic) return 1;
          return 0;
        });
        list.sum = list.items.length;
      }).catch(function (error) {
      }).finally(function () {
      });
  }
});
```

### 表示の並びは、五十音順でソートする
ふりがなに濁音（半濁音）が含まれる場合は、清音に置換する。

```js
function replaceDakuToHan(str) {
  if (str.length > 0) {
    var data = [
      ['が', 'か'], ['ぎ', 'き'], ['ぐ', 'く'], ['げ', 'け'], ['ご', 'こ'],
      ['ざ', 'さ'], ['じ', 'し'], ['ず', 'す'], ['ぜ', 'せ'], ['ぞ', 'そ'],
      ['だ', 'た'], ['ぢ', 'ち'], ['づ', 'つ'], ['で', 'て'], ['ど', 'と'],
      ['ば', 'は'], ['び', 'ひ'], ['ぶ', 'ふ'], ['べ', 'へ'], ['ぼ', 'ほ'],
      ['ぱ', 'は'], ['ぴ', 'ひ'], ['ぷ', 'ふ'], ['ぺ', 'へ'], ['ぽ', 'ほ']
    ];
    for (var i = 0; i < data.length; i++) {
      str = str.replace(new RegExp(data[i][0], 'g'), function () {
        return data[i][1];
      });
    }
  }
  return str;
}
```

### 画像はVueLazyloadで遅延ロード

遅延ロードしたいimgタグのsrcに`v-lazy`を指定。
error用の画像とloading用の画像を、Vue.use()で設定する。

```HTML
<img src="/" v-lazy="item.logo" alt="">
```

```js
Vue.use(VueLazyload, {
  error: 'http://placehold.jp/BFBFBF/222222/320x210.png?text=ERROR',
  loading: '/img/loading.gif'
});
```

### Vue Routerを使って、ブラウザバックに対応

```js
//各ルートを定義する
var routes = [{
  path: location.pathname,
  meta: {
    title: document.title
  }
}];
//ルーターインスタンスを作成して、ルートオプションを渡す
var router = new VueRouter({
  mode: 'history',
  routes: routes
})
//アプリケーション全体がルーターを認知できるように、ルーターをインジェクトする
var list = new Vue({
  router: router,
  .
  .
  .
```