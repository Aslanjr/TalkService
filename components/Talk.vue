<template>
    <div class="container-fluid vh-100 bg-dark">
      <div class="container vh-100">
        <div class="row d-flex justify-content-center h-100 flex-column align-items-center">
          <div class="bg-light col-lg-2 flex-column d-flex align-items-center justify-content-center mw-100">
            <p class="h2 text-center">TalkingService</p>
            <div class="d-flex justify-content-around">
              <div class="btn btn-outline-info" @click="show('Auth')">Auth</div>
              <div class="btn btn-outline-info ml-3 mr-3" @click="show('myMsg')">My messages</div>
              <div class="btn btn-outline-info" @click="show('allMsg')">All messages</div>
            </div>
          </div>
          <div class="bg-info mh-100 p-3 mw-100 col-lg-8 rounded-lg overflow-auto">
            <div v-show="contents[1].visible">
              <div class="msg mw-100 bg-light m-3 d-flex align-items-center  rounded-lg shadow-lg" v-for="(item,index) in myMessage" :key="index">
                <p class="text-muted mt-3 ml-3 mb-3 mr-3 h5 author">{{item.author | author}} </p>
                <p class="message mt-3 mb-3 text-danger">{{item.message}}</p>
              </div>
            </div>
            <div v-show="contents[0].visible">
              <form class="bg-light shadow-lg rounded-lg p-3">
                <h2 class="text-center">Авторизация</h2>
                <div class="form-group  d-flex justify-content-around  p-4 ">
                  <div class="form-row col-lg-12">
                    <label for="name">Имя: </label>
                    <input type="text" class="form-control" ref="name">
                  </div>
                </div>
                <div class="form-row col-lg-12 justify-content-center">
                  <button class="btn btn-info btn-lg" @click.prevent="auth">
                    Войти
                  </button>
                </div>
              </form>
            </div>
            <div v-show="contents[2].visible">
                <h1 class="text-center ">AllMessages</h1>
                <div class="msg mw-100 bg-light m-3 d-flex align-items-center  rounded-lg shadow-lg" v-for="(item,index) in allMessage" :key="index">
                  <p class="text-muted mt-3 ml-3 mb-3 mr-3 h5 author">{{item.author | author}} </p>
                  <p class="message mt-3 mb-3 text-danger">{{item.message}}</p>
                </div>
              </div>
          </div>
          <div class="bg-light mh-100 mw-100 d-flex align-items-center justify-content-center col-lg-2 rounded-lg">
            <div class="col-lg-3">
              <select class="form-select form-control" v-model="selected" aria-label="Default select example">
                <option selected>Send to whom?</option>
                <option v-for="(item,index) in allUsers" :key="index" >{{item.name}}</option>
              </select>
            </div>
            <input type="text" class="form-control" @keyup.enter="sentMessage" ref="msg" placeholder="Type message.....">
          </div>
        </div>
      </div>
    </div>
</template>
<script>

import {mapState} from 'vuex'


export default {
  name:'Talk',
  data:()=>{
    return{
      user:{
        name:'',
      },
      messages:[],
      selected:'',
      users:[],
      contents:[
        {
          title:'Auth',
          visible:true,
        },
        {
          title:'myMsg',
          visible:false,
        },
        {
          title:'allMsg',
          visible:false,
        },
      ]
    }
  },
  mounted(){

    this.getMessages();
    this.getUsers();

    setInterval(()=>{
      this.getMessages();
      this.getUsers();
    },3000)
  },
  methods:{
    async sentMessage (){
      console.log(this.messages[this.messages.length -1].id += 1);
      this.$axios.$post('/messages', {
        id: this.messages[this.messages.length -1].id++,
        author:`${this.user.name}`,
        messageTo:`${this.selected}`,
        message:`${this.$refs.msg.value}`
      })
    },
    auth(){
      this.user.name = this.$refs.name.value; 
      
      this.contents[0].visible = false;
      this.contents[2].visible = true;

      for (let i = 0; i < this.users.length; i++) {
        if(this.user.name == this.users[i].name){
            alert(`Welcome to Talking ${this.user.name}`);
            alert(`You are identified, you can communicate :)`);
            break;
        }else{
          this.$axios.$post('/users', {
            id: this.messages[this.users.length -1].id += 1,
            name:`${this.user.name}`
          })
          alert(`Welcome to Talking ${this.user.name}`);
          alert(`You have been added to the user list, now you can chat with other users :) `);
          break;
        }
        
      }
       
    },
    show(msg){
      for (let i = 0; i < this.contents.length; i++) {
        if(this.contents[i].title !== msg && this.contents[i].visible !== false){
           this.contents[i].visible = false;
        }else if(this.contents[i].title == msg && this.contents[i].visible == false){
           this.contents[i].visible = true;
        }
      }
    },
    async getMessages(){
      await fetch('https://talk-service-server.herokuapp.com/messages')
        .then(response => response.json())
        .then(item => {
          if(this.messages.length == item.length){
            return false;
          }else{
            this.messages = item
          }
        });
    },
     getUsers(){
       fetch('https://talk-service-server.herokuapp.com/users')
        .then(response => response.json())
        .then(item => this.users = item);
    }
  },
  filters:{
    author(msg){
      return msg + ": " 
    }
  },
  computed:{
    allMessage(){
      return this.messages;
    },
    myMessage(){
      return Object.values(this.messages).filter(item => { return item.messageTo == this.user.name });
    },
    allUsers(){
      return this.users;
    },
    ...mapState({
      listMessage: (el)=>{
        el.filter(item=> item.author == this.user)
      }
    })
  }
}
</script>
<style>
  .author{
    margin: 0;
  }
  .message{
    padding: 0;
    margin: 0;
  }
</style>
