<template>
  <div class="container">
    <!-- chat room -->
    <div class="chta-room">
      <!-- head -->
      <div class="room-head">
        <div class="gender">
          <font-awesome-icon :icon="userGender" />
        </div>
        <div class="user">{{ userName }}</div>
        <!-- <div class="room-head-title">
          Chat Room
          <font-awesome-icon icon="comment-dots" />
        </div> -->
      </div>
      <!-- body -->
      <div class="room-body">
        <template v-for="item in msg">
          <!-- other people -->
          <template v-if="item.userName !== userName">
            <div class="message-box" :key="item.id">
              <div class="gender" :class="item.userGender">
                <font-awesome-icon :icon="item.userGender" />
              </div>
              <div class="message-content">
                <div class="message-user">
                  <p>{{ item.userName }}</p>
                </div>
                <div class="content">
                  <div class="text">{{ item.msg }}</div>
                  <div class="time">{{ item.timeStamp }}</div>
                </div>
              </div>
            </div>
          </template>
          <!-- self -->
          <template v-if="item.userName === userName">
            <div class="message-box self" :key="item.id">
              <div class="message-content">
                <div class="content">
                  <div class="text">{{ item.msg }}</div>
                  <div class="time">{{ item.timeStamp }}</div>
                </div>
              </div>
            </div>
          </template>
        </template>
      </div>
      <!-- message -->
      <div class="room-message">
        <textarea id="msg" class="message bg hvr-border-fade" :class="{ disable: !userName }" @keypress.enter="sendMessage($event)"></textarea>
        <!-- v-on 可縮寫為 @ -->
        <div class="message-send">
          <button class="send-btn bg" @click="sendMessage($event)">
            <font-awesome-icon icon="angle-right" />
          </button>
        </div>
      </div>
    </div>
    <!-- modal -->
    <div v-show="userName === '' || userGender === ''" class="modal">
      <div class="modal-container">
        
          <div class="modal-body">
            <div class="model-title">LET'S CHAT ;)</div>
              <input type="text" id="userName" class="user-name" maxlength="10" placeholder="Your Name">
              <template v-for="(item, index) in genderInput">
                <div class="user-gender" :key="item.id">
                  <input name="gender" type="radio" class="box-gender" :id="item.id" :value="item.id" @click="active = index">
                  <label :for="item.id" :class="{'user-gender-checked': active == index}"><font-awesome-icon :icon="item.id" /></label>
                </div>
              </template>
            <div class="modal-button" @click="getName();getGender()">CHAT!</div>
          </div>
        
      </div>
    </div>
  </div>
</template>

<script>
  const cahtRoomRef = firebase.database().ref('/messages/')
  export default {
  name: 'HelloWorld',
  data () {
    return {
      userName: '',
      userGender: '',
      msg: [],
      genderInput: [
        {id: 'venus'},
        {id: 'mars'},
        {id: 'question'}
      ],
      active: null
    }
  },
  
  methods: {
    // modal save
    getName(){
      const vm = this
      const userName = document.querySelector('#userName').value
      if(userName.trim() == '') {return}
      vm.userName = userName
    },
    getGender(){
      const vm = this
      const usergender = document.querySelectorAll('.box-gender')
      usergender.forEach(radio => {
        if(radio.checked === true){
          vm.userGender = radio.value
          return radio.value
        }
      })
    },
    // get time
    getTime(){
      const now = new Date();
      const hours = now.getHours();
      const minutes = now.getMinutes();
      return `${(hours >= 12) ? "PM" : "AM"} ${hours}:${(minutes < 10) ? '0' + minutes : minutes}`;
    },
    // send message
    sendMessage(e){
      const vm = this
      const userName = document.querySelector('#userName')
      let msg = document.querySelector('#msg')
      console.log(vm.userGender)
      console.log(vm.getTime())
      
      // shift不傳送訊息(多行輸入)
      if (e.shiftKey) {
        return false;
      }
      // 輸入框空值不傳送訊息
      if(msg.value.length <= 1 && msg.value.trim() === '') {
        e.preventDefault()
        return false
      }
      // push到firebase
      cahtRoomRef.push({
        userName: userName.value,
        userGender: vm.userGender,
        type: 'text',
        msg: msg.value,
        timeStamp: vm.getTime()
      })
      // 清空輸入欄位避免enter產生的空白換行
      msg.value = '';
      e.preventDefault();
    },
    // scrollbar維持在底部
    scrollToBottom(){
      const roomBody = document.querySelector('.room-body')
      roomBody.scrollTop = roomBody.scrollHeight
    }
  },

  //mounted：模板已編譯完成，準備取值渲染HTML畫面
  mounted(){
    const vm = this
    cahtRoomRef.on('value', function(snapshot){
      const val = snapshot.val()
      vm.msg = val
    })
  },

  //updated：DOM 的更新已經完成，View 被顯示在畫面上。
  updated() {
    const vm = this
    vm.scrollToBottom();
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
* {
  margin: auto;
}
.container {
  background-color: #293241;
  padding-top: 50px;
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  z-index: -999;
}
.chta-room {
  border-radius: 5px;
  max-width: 450px;
  box-shadow: 0 14px 28px rgba(0, 0, 0, 0.25), 0 10px 10px rgba(0, 0, 0, 0.22);
}
.room-head {
  width: 100%;
  height: 85px;
  border-radius: 5px 5px 0 0;
  background-color: #3d5a80;
  position: relative;
}
.room-head .gender {
  width: 50px;
  height: 50px;
  text-align: center;
  line-height: 50px;
  margin: 20px 10px 20px 20px;
}
.room-head .user {
  color: #ffffff;
  font-size: 20px;
  display: inline-block;
  margin: 30px 0;
}
.room-head-title {
  color: #ffffff;
  font-size: 22px;
  text-align: center;
  line-height: 85px;
}
.room-body {
  padding: 10px 0;
  background-color: #ffffff;
  height: 60vh;
  overflow-y: auto;
  overflow-x: hidden;
}
.message-box {
  padding: 5px 10px;
  position: relative;
}
.room-message {
  display: flex;
  width: 100%;
  border-radius: 0 0 5px 5px;
  background-color: #3d5a80;
}
.bg {
  background-color: transparent;
  outline: none;
}
.message {
  border: none;
  border-radius: 30px;
  flex-basis: 70%;
  margin: 20px 0 20px 20px;
  padding: 15px 25px;
  font-size: 16px;
  color: #ffffff;
}
.message-send {
  flex-basis: 15%;
  text-align: center;
}
.send-btn {
  border: none;
  font-size: 30px;
  color: #ee6c4d;
  cursor: pointer;
}
.hvr-border-fade {
  display: inline-block;
  vertical-align: middle;
  -webkit-transform: perspective(1px) translateZ(0);
  transform: perspective(1px) translateZ(0);
  box-shadow: 0 0 1px rgba(0, 0, 0, 0);
  -webkit-transition-duration: 0.3s;
  transition-duration: 0.3s;
  -webkit-transition-property: box-shadow;
  transition-property: box-shadow;
  box-shadow: inset 0 0 0 2px #e1e1e1, 0 0 1px rgba(0, 0, 0, 0);
  /* Hack to improve aliasing on mobile/tablet devices */
}
.hvr-border-fade:hover, .hvr-border-fade:focus, .hvr-border-fade:active {
  box-shadow: inset 0 0 0 2px #ee6c4d, 0 0 1px rgba(0, 0, 0, 0);
  /* Hack to improve aliasing on mobile/tablet devices */
}
.gender {
  color: #ffffff;
  height: 40px;
  width: 40px;
  text-align: center;
  line-height: 40px;
  border-radius: 50%;
  vertical-align: top;
  display: inline-block;
}
.room-head .gender {
  font-size: 27px;
  background-color: #ee6c4d;
}
.mars {
  background-color: #98c1d9;
}
.venus {
  background-color: #e5989b;
}
.question {
  background-color: #84a59d;
}
.message-box .gender {
  font-size: 20px;
}
.message-content {
  max-width: 70%;
  display: inline-block;
}
.message-user {
  margin: 5px 0 5px 5px;
  font-size: 13px;
  color: #727c8a;
  vertical-align: top;
  cursor: pointer;
}
.content {
  margin: 5px 0 5px 5px;
  font-size: 12px;
  color: #35393d;
  letter-spacing: .6px;
  background-color: #e3e8eb;
  border-radius: 12px;
  line-height: 1.5;
  text-align: left;
  word-break: break-all;
  white-space: pre-line;
}
.text {
  padding: 8px 10px 0px 11px;
  /* max-height: 300px; */
  overflow: hidden;
}
.time {
  color: #727c8a;
  padding: 0px 10px 8px 11px;
}
.self {
  text-align: right;
}
.self .content {
  background-color: #98c1d9;
  margin-right: 25px;
}
.modal {
  z-index: 3;
  padding: 80px 0;
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0,0,0,.4);
  -webkit-animation: opac .8s;
  animation: opac .8s;
  letter-spacing: 2px;
  text-align: unset;
}
.model-title {
  font-size: 38px;
  color: #ffffff;
  margin: 50px 0;
}
.user-name, .modal-button {
  font-size: 20px;
  padding: 10px 20px;
  border-radius: 50px;
  width: 200px;
  border: none;
  outline: none;
}
.user-gender {
  margin: 38px 0 50px;
  display: inline-block;
}
.user-gender input[type="radio"] {
  display: none;
}
.user-gender input[type="radio"] + label {
  font-size: 20px;
  color: #3a506b;
  border: 1px solid #3a506b;
  border-radius: 100%;
  padding: 10px 18px;
  margin-right: 20px;
  cursor: pointer;
}
.user-gender input[type="radio"] + label:hover {
  border: 1px solid #ee6c4d;
  -webkit-transition-duration: 0.3s;
  transition-duration: 0.3s;
  -webkit-transition-property: box-shadow;
  transition-property: box-shadow;
  box-shadow: 0 0 0 2px #ee6c4d, 0 0 1px rgba(0, 0, 0, 0);
}
.user-gender-checked {
  background-color: #ee6c4d;
  border: none!important;
  color: #ffffff!important;
  transition-duration: 0.5s;
}
.modal-container {
  margin: auto;
  position: relative;
  padding: 10px;
  outline: 0;
  max-width: 400px;
}
.modal-body {
  background-image: linear-gradient(to top left, #98c1d9, #3d5a80);
  box-shadow: 0 14px 28px rgba(0, 0, 0, 0.25), 0 10px 10px rgba(0, 0, 0, 0.22);
  padding: 20px 50px;
  text-align: center;
  border-radius: 25px;
}
.modal-button {
  color: #ffffff;
  background-color: #ee6c4d;
  box-shadow: 0 14px 28px rgba(0, 0, 0, 0.25);
  margin-bottom: 50px;
  cursor: pointer;
}
@media screen and (max-width: 768px) {
  .container {
    padding-top: 0px;
  }
  .modal {
    padding: 40px 0;
  }
}
</style>
