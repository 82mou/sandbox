<template>
  <div id="app">
    <!-- ボタン類 -->
    <div>
      <button type="button" class="btn btn-default" @click="login">
        匿名ユーザーでログイン
      </button>
      <button type="button" class="btn btn-default" @click="pushData">
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
        list: []  // 最新状態はここにコピーされる
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
          console.log(e)
          this.listen()
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
            const rootList = snapshot.val()
            let list = []
            Object.keys(rootList).forEach((val, key) => {
              rootList[val].id = val
              list.push(rootList[val])
            })
            this.list = list
          }
        })
      },
      // ダミーデータをfirebaseに送信
      pushData () {
        firebase.database().ref('myBoard/').push({ // eslint-disable-line
          name: 'test',
          message: 'foo'
        })
      }
    }
  }
</script>

<style>

</style>
