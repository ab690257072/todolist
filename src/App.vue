<template>
  <div id="app">
    <section id="signInAndSignUp" v-if="!currentUser">
      <div>
        <label><input type="radio" name="type" v-model="actionType"  value="signUp"> 注册</label>
        <label><input type="radio" name="type" v-model="actionType"  value="login"> 登录</label>
      </div>
      <div class="signUp" v-if="actionType=='signUp'">
        <form @submit.prevent="signUp">
          <div class="formRow">
            <span class="label">username :</span>
            <span class="input input--hoshi">
                <input class="input__field input__field--hoshi" type="text" id="input-1" v-model="formData.username">
                <label class="input__label input__label--hoshi input__label--hoshi-color-1" for="input-1">
              </label>
              </span>
          </div>
          <div class="formRow">
            <span class="label">password :</span>
            <span class="input input--hoshi">
                <input class="input__field input__field--hoshi" type="password" id="input-2" v-model="formData.password">
                <label class="input__label input__label--hoshi input__label--hoshi-color-2" for="input-2">
                </label>
              </span>
          </div>
          <div class="formActions">
            <input type="submit" value="注册">
          </div>
        </form>
      </div>
      <div class="login" v-if="actionType=='login'">
        <form @submit.prevent="login">
          <div class="formRow">
            <span class="label">username :</span>
            <span class="input input--hoshi">
                <input class="input__field input__field--hoshi" type="text" id="input-1" v-model="formData.username">
                <label class="input__label input__label--hoshi input__label--hoshi-color-1" for="input-1">
              </label>
              </span>
          </div>
          <div class="formRow">
            <span class="label">password :</span>
            <span class="input input--hoshi">
                <input class="input__field input__field--hoshi" type="password" id="input-2" v-model="formData.password">
                <label class="input__label input__label--hoshi input__label--hoshi-color-2" for="input-2">
                </label>
              </span>
          </div>
          <div class="formActions">
            <input type="submit" value="登录">
          </div>
        </form>
      </div>
    </section>
    <section id="todo" v-if="currentUser">
      <div>
        <span class="msg">你好，{{currentUser.username}}</span>
        <button @click="logout">退出</button>
      </div>
      <div class="newTask">
        <input type="text" v-model="newTodo" @keypress.enter="addTodo" class="new-todo">
      </div>
      <ol class="todos" v-if="todoList.length">
        <li v-for="todo in todoList">
          <input type="checkbox" v-model="todo.done">
          <span>{{ todo.title }}</span>
          <span class="createTime">{{todo.createdAt | normalTime}}</span>
          <span v-if="todo.done" class="tag success">已完成</span>
          <span v-else class="tag warning">未完成</span>
          <button class="close" @click="removeTodo(todo)">X</button>
        </li>
      </ol>
    </section>
  </div>
</template>
<script>
  import AV from 'leancloud-storage'
  export default {
    data() {
      return {
        newTodo: '',
        todoList: [],
        actionType: 'signUp',
        currentUser: null,
        formData: {
          username: '',
          password: ''
        }
      }
    },
    methods: {
      updateTodos() {
        let dataString = JSON.stringify(this.todoList);
        let avTodos = AV.Object.createWithoutData('AllTodos', this.todoList.id);
        avTodos.set('content', dataString);
        avTodos.save().then(() => {
          console.log('更新成功');
        });
      },
      addTodo() {
        this.todoList.push({
          title: this.newTodo,
          createdAt: new Date(),
          done: false
        });
        this.newTodo = '';
        this.saveOrUpdateTodos();
      },
      removeTodo(todo) {
        let index = this.todoList.indexOf(todo);
        this.todoList.splice(index, 1);
        this.saveOrUpdateTodos();
      },
      saveTodos() {
        let dataString = JSON.stringify(this.todoList);
        var AVTodos = AV.Object.extend('AllTodos');
        var avTodos = new AVTodos();
        var acl = new AV.ACL();
        acl.setReadAccess(AV.User.current(), true);
        acl.setWriteAccess(AV.User.current(), true);
        avTodos.set('content', dataString);
        avTodos.setACL(acl);
        avTodos.save().then((todo) => {
          this.todoList.id = todo.id;
          console.log('保存成功');
        }, function (error) {
          console.log('保存失败');
        });
      },
      saveOrUpdateTodos() {
        if (this.todoList.id) {
          this.updateTodos();
        } else {
          this.saveTodos();
        }
      },
      fetchTodos() {
        if (this.currentUser) {
          var query = new AV.Query('AllTodos');
          query.find()
            .then((todos) => {
              let avAllTodos = todos[0];
              let id = avAllTodos.id;
              this.todoList = JSON.parse(avAllTodos.attributes.content);
              this.todoList.id = id;
            }, function (error) {
              console.log(error);
            });
        }
      },
      signUp() {
        let user = new AV.User();
        user.setUsername(this.formData.username);
        user.setPassword(this.formData.password);
        user.signUp().then((loginedUser) => {
          this.currentUser = this.getCurrentUser();
        }, (error) => {
          alert('注册失败');
        });
      },
      login() {
        AV.User.logIn(this.formData.username, this.formData.password).then((loginedUser) => {
          this.currentUser = this.getCurrentUser();
          this.fetchTodos();
        }, (error) => {
          console.log('登录失败');
        });
      },
      logout() {
        AV.User.logOut();
        this.currentUser = null;
        window.location.reload();
      },
      getCurrentUser() {
        let current = AV.User.current();
        if (current) {
          let {
            id,
            createdAt,
            attributes: {
              username
            }
          } = current;
          return {
            id,
            username,
            createdAt
          };
        } else {
          return null;
        }
      }
    },
    created() {
      // beforeunload 事件里面的所有请求都发不出去，会被取消

      this.currentUser = this.getCurrentUser();
      this.fetchTodos();
    }
  }
</script>
<style>
  * {
    margin: 0;
    padding: 0;
    outline: none;
    font-family: 'Microsoft Yahei', Arial, sans-serif;
  }
  
  body {
    background-color: rgb(199, 237, 233);
    background-image: linear-gradient(#fff 1px, transparent 0), linear-gradient(90deg, #fff 1px, transparent 0);
    background-size: 30px 30px;
  }
  
  input[type="checkbox"] {
    height: 20px;
    width: 20px;
    vertical-align: middle;
  }
  
  input[type="submit"] {
    padding: 6px 15px;
    color: #4ea8f5;
    background: transparent;
    display: inline-block;
    font-size: 14px;
    cursor: pointer;
    border: 1px solid #4ea8f5;
    border-radius: 3px;
  }
  
  input[type="submit"]:hover {
    background-color: #4ea8f5;
    color: #fff;
    transition: all 0.5s ease 0s;
  }
  
  button {
    padding: 4px 15px;
    color: palevioletred;
    background: transparent;
    display: inline-block;
    font-size: 14px;
    cursor: pointer;
    border: 1px solid palevioletred;
    border-radius: 3px;
  }
  
  button:hover {
    background-color: palevioletred;
    color: #fff;
    transition: all 0.5s ease 0s;
  }
  
  .createTime {
    position: absolute;
    right: 95px;
    color: rgba(0, 0, 0, 0.5);
  }
  
  .close {
    padding: 4px 15px;
    position: absolute;
    right: 0;
    border: none;
    color: darkorange;
  }
  
  .success {
    color: mediumspringgreen;
  }
  
  .warning {
    color: rgb(255, 66, 93);
  }
  
  .tag {
    position: absolute;
    right: 35px;
  }
  
  .msg {
    margin-right: 15px;
  }
  
  .label {
    display: inline-block;
    padding-top: 15px;
  }
  
  .new-todo {
    padding: 16px;
    width: 585px;
    font-size: 110%;
    border: 1px solid transparent;
    border-radius: 5px;
    box-shadow: 5px 5px 5px #888;
    margin: 10px;
    background: #fff;
  }
  
  .todos {
    position: relative;
    width: 35em;
    margin: .5em;
    background: rgb(92, 167, 186);
    background: linear-gradient(-150deg, transparent 1.5em, rgb(92, 167, 186) 0);
    padding: 2em;
    font: 100%/1.6 Baskerville, Palatino, serif;
    border-radius: .5em;
  }
  
  .todos::before {
    content: '';
    position: absolute;
    top: 0;
    right: 0;
    width: 1.73em;
    height: 3em;
    background: linear-gradient(to left bottom, transparent 50%, rgba(0, 0, 0, .2) 0, rgba(0, 0, 0, .4)) 100% 0 no-repeat;
    transform: translateY(-1.3em) rotate(-30deg);
    transform-origin: bottom right;
    border-bottom-left-radius: .5em;
    box-shadow: -.2em .2em .3em -.1em rgba(0, 0, 0, .15)
  }
  
  .input {
    position: relative;
    z-index: 1;
    display: inline-block;
    max-width: 400px;
    margin: .2em;
    width: calc(100% - 2em);
    vertical-align: top;
    font-size: 150%;
  }
  
  .input__field {
    border: none;
    font-weight: bold;
    background-color: transparent;
  }
  
  .input__field:focus {
    outline: none;
  }
  
  .input--hoshi {
    overflow: hidden;
  }
  
  .input__field--hoshi {
    padding: 0.55em 0.15em;
    color: #595F6E;
    font-size: 70%;
  }
  
  .input__label--hoshi {
    position: absolute;
    bottom: 0;
    left: 0;
    padding: 0 0.25em;
    width: 100%;
    height: calc(100% - 1em);
    text-align: left;
    pointer-events: none;
  }
  
  .input__label--hoshi::before,
  .input__label--hoshi::after {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: calc(100% - 10px);
    border-bottom: 1px solid #B9C1CA;
  }
  
  .input__label--hoshi::after {
    margin-top: 2px;
    border-bottom: 4px solid red;
    -webkit-transform: translate3d(-100%, 0, 0);
    transform: translate3d(-100%, 0, 0);
    -webkit-transition: -webkit-transform 0.3s;
    transition: transform 0.3s;
  }
  
  .input__label--hoshi-color-1::after {
    border-color: hsl(200, 100%, 50%);
  }
  
  .input__label--hoshi-color-2::after {
    border-color: hsl(160, 100%, 50%);
  }
  
  .input__field--hoshi:focus+.input__label--hoshi::after,
  .input--filled .input__label--hoshi::after {
    -webkit-transform: translate3d(0, 0, 0);
    transform: translate3d(0, 0, 0);
  }
  
  .input__field--hoshi:focus+.input__label--hoshi .input__label-content--hoshi,
  .input--filled .input__label-content--hoshi {
    -webkit-animation: anim-1 0.3s forwards;
    animation: anim-1 0.3s forwards;
  }
  
  @-webkit-keyframes anim-1 {
    50% {
      opacity: 0;
      -webkit-transform: translate3d(1em, 0, 0);
      transform: translate3d(1em, 0, 0);
    }
    51% {
      opacity: 0;
      -webkit-transform: translate3d(-1em, -40%, 0);
      transform: translate3d(-1em, -40%, 0);
    }
    100% {
      opacity: 1;
      -webkit-transform: translate3d(0, -40%, 0);
      transform: translate3d(0, -40%, 0);
    }
  }
  
  @keyframes anim-1 {
    50% {
      opacity: 0;
      -webkit-transform: translate3d(1em, 0, 0);
      transform: translate3d(1em, 0, 0);
    }
    51% {
      opacity: 0;
      -webkit-transform: translate3d(-1em, -40%, 0);
      transform: translate3d(-1em, -40%, 0);
    }
    100% {
      opacity: 1;
      -webkit-transform: translate3d(0, -40%, 0);
      transform: translate3d(0, -40%, 0);
    }
  }
</style>