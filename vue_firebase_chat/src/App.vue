<template>
  <div id="app">
    <!-- 名前の入力欄 -->
    <label for="nameInput">名前</label>
    <input type="txt" id="nameInput" v-model="name">

    <!-- メッセージの入力欄 -->
    <label for="nameInput">メッセージ</label>
    <input type="txt" id="nameInput" v-model="message">

    <!-- ボタン類 -->
    <div>
      <button type="button" class="btn btn-default" @click="login">
        匿名ユーザーでログイン
      </button>
      <button type="button" class="btn btn-default" @click="sendMessage">
        送信
      </button>
    </div>

    <!-- リスト -->
    <div>
      <ul>
        <li v-for="item in list">{{item.name}} / {{item.message}}</li>
      </ul>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'app',
    data () {
      return {
        list: [],     // 最新状態はここにコピーされる
        name: '',     // 名前
        message: '',  // 送信メッセージ
      }
    },
    created () {
      // 認証チェック
      firebase.auth().onAuthStateChanged( user => { // eslint-disable-line
        if (user) {
          console.log('ログイン状態.')
          this.listen()
        } else {
          console.log('ログインしていない状態')
        }
      })
    },
    methods: {
      // ログイン関数
      login () {
        firebase.auth().signInAnonymously().then(e => { // eslint-disable-line
          console.log(e);
          this.listen();
        }).catch((error) => {
          // ログインのエラーメッセージ
          var errorCode = error.code; // eslint-disable-line
          var errorMessage = error.message; // eslint-disable-line
          console.log('ログインエラーメッセージ', errorCode, errorMessage)
        }); // eslint-disable-line
      },
      // データベースの変更を購読、最新状態をlistにコピーする
      listen () {
        firebase.database().ref('myBoard/').on('value', snapshot => { // eslint-disable-line
          if (snapshot) {
            const rootList = snapshot.val();
            let list = [];
            Object.keys(rootList).forEach((val, key) => {
              rootList[val].id = val;
              list.push(rootList[val]);
            })
            this.list = list;
          }
        })
      },
      // ダミーデータをfirebaseに送信
      pushData () {
        firebase.database().ref('myBoard/').push({ // eslint-disable-line
          name: 'test',
          message: 'foo'
        })
      },
      sendMessage () {
        // 空欄の場合は実行しない
        if (!this.name || !this.message) return
        // 送信
        firebase.database().ref('myBoard/').push({
          name: this.name,
          message: this.message
        })
        // 送信後inputを空にする
        this.name = ''
        this.message = ''
      }
    }
  }
</script>

<style>

</style>
