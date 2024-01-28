<template>
  <b-container>
    <b-input-group prepend="新規メモ">
      <b-input-group-append is-text><b>タイトル</b></b-input-group-append>
      <b-form-input type="text" v-model="fields.title"></b-form-input>
      <b-input-group-append is-text><b>名前</b></b-input-group-append>
      <b-form-input type="text" v-model="fields.name"></b-form-input>
      <b-input-group-append is-text><b>内容</b></b-input-group-append>
      <b-form-textarea v-model="fields.message"></b-form-textarea>
      <b-button variant="outline-success" @click="addMemo(fields)">確定</b-button>
    </b-input-group>
    <h5>{{ message }}</h5>
    <div>
      <b-table striped hover :items="items" :fields="fields">
        <template #cell(createDate)=date>
          <p>{{date.value.toLocaleString()}}</p>
        </template>
        <template #cell(action)=data>
          <b-button size="sm" @click="deleteMemo(data.item.objectId)" class="mr-2">
            削除
          </b-button>
        </template>
      </b-table>
    </div>
  </b-container>
</template>
 
<script>
const NCMB = require('ncmb');
const application_key = 'e7da534afca9db6578539360103d642e20f3c7a804842fdc2ece2eb1ad489178';
const client_key = '60b7a1d27d59d4b4e9399a8e33dee8ac46c85b136c7540a88333e9b91192c169';
const ncmb = new NCMB(application_key, client_key);

var Memo = ncmb.DataStore('memos');

export default {
  mounted () {
    Memo = ncmb.DataStore('memos');
    this.showMemo();
  },
  data() {
    return{
      items: [],
      fields: ['title', 'name', 'message', 'createDate', 'action'],
      message: ""
    }
  },
  methods:{
    addMemo(fields) {
      let self = this;
      var memo = new Memo();
      if(!fields.title || !fields.name || !fields.message){
        self.message = "未入力項目があります";
      }
      else{
        memo.set('title', fields.title)
          .set('name', fields.name)
          .set('message', fields.message)
          .save()
          .then(function(memo){
            self.message = "登録完了";
            self.showMemo();
            self.formClear(fields);
          })
          .catch(function(err){
            self.message = "登録失敗";
          });
      }
    },

    showMemo() {
      let self = this;
      var item = Memo
        .fetchAll()
        .then(item => {
          self.items = item;
        })
        .catch(e => {
          self.message = "データはありません";
        });
    },

    deleteMemo(id) {
      let self = this;
      var item = Memo
        .fetchById(id)
        .then(item => {
          console.log(id + " : " + item.objectId);
          item.delete()
          .then(function(result){
            self.message = "削除完了";
            self.showMemo();
          })
          .catch(function(err){
            self.message = "削除失敗";
          });
        })
        .catch(e => {
          self.message = "削除失敗";
        });
    },

    formClear(fields){
      fields.title = "";
      fields.name = "";
      fields.message = "";
    }
  }
}
</script>
