<template>    
  <div class="login-container">    
    <el-header class="hee">
      明光筑梦，少年强国
    </el-header>
    <el-container>    
      <el-header>    
        <el-form class="login-form">    
          <el-form-item label="用户名" class="login-form-item">    
            <el-input v-model="username" class="login-input"></el-input>    
          </el-form-item>    
          <el-form-item label="密码" class="login-form-item">    
            <el-input type="password" v-model="password" class="login-input"></el-input>    
          </el-form-item>    
        </el-form>    
      </el-header>    
      <el-main>    
        <div class="buttons-wrapper">    
          <el-button type="primary" @click="register" class="register-button">注册</el-button>    
          <el-button type="primary" @click="login" class="login-button">登录</el-button>    
        </div>    
      </el-main>    
    </el-container>    
  </div>    
</template>  
  
<script>  
import axios from 'axios'; // 引入axios库  
  
export default {  
  data() {  
    return {  
      username: '', // 用户名输入框的数据模型  
      password: '', // 密码输入框的数据模型  
    };  
  },  
  methods: {  
    register() {  
      try {  
        this.$router.push({ path: '/userRegister',params: { id: 1} })  
      } catch (error) {  
        // 处理错误，例如给用户反馈错误信息。这里只是一个示例，具体的处理方式需要根据您的业务逻辑来确定。  
        console.error(error);  
      }  
    },  
   login() {  
      try {  
        //向后台发送登录请求，，并且需要POST数据{username: this.username, password: this.password}  
        axios.post(this.NET.BASE_URL.http + 'userLogin', { username: this.username, password: this.password })
        .then(response => {
            // 检查后台返回的数据中是否包含成功标志，例如一个特定的状态码或者字段。这里只是一个示例，具体的处理方式需要根据您的业务逻辑来确定。  
            if (response.data.msg === '登录成功') {  
                this.$router.push({ path: '/main', params: { user: response.data.data} })
                this.$cookies.set('userID', response.data.data.userID)
                this.$cookies.set('username', response.data.data.username)
                this.$cookies.set('userAccount', response.data.data.account)
                this.$cookies.set('userAddress', response.data.data.address)
                this.$cookies.set('userPhone', response.data.data.phone)
                this.$cookies.set('userEmail', response.data.data.email)
                this.$cookies.set('userDescription', response.data.data.description)
                this.$cookies.set('userTrophy', response.data.data.trophy)
                
            } else {  
                console.log('登录失败：' + response.data.message)  
                this.$router.push({ path: '/loginFail'})
            }
        });
      
      } catch (error) {  
        // 处理错误，例如给用户反馈错误信息。这里只是一个示例，具体的处理方式需要根据您的业务逻辑来确定。  
        console.error(error);  
      }  
    },  
  },  
};  
</script>


<style>

.login-container {  
  /* display: flex;  
  justify-content: center;  
  align-items: center;   */
  height: 100vh;  
  
  background-image: url('../assets/back.jpg');
}  
.hee{
    /* background-color: #545c64; */
    font-size: 50px;
    font-family: STXingkai;
    color: rgb(255, 186, 57);
    text-align: center;
    margin: auto;
    line-height: 60px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.5);
}
  
.login-form {  
  margin-top:200px;
  margin-left:350px;
  display: flex;  
  flex-direction: column;  
  align-items: center;  
  text-align: center;
  width:50%;
  color: rgb(57, 255, 179);


  
}  
  
.login-form-item {  
  margin-bottom: 10px;  
 
}  
  
.login-input {  
  padding: 10px;  
  border: 1px solid #ccc;  
  color: aliceblue;
  width:150%;
  border-radius: 4px;  
}  
  
.register-button,  
.login-button {  
  margin-top:200px;
  /* 将按钮的样式设置得与密码栏协调 */  
  padding: 10px;  
  background-color: #409eff; /* 假设这是按钮的背景色 */  
  color: white; /* 假设这是按钮的文本颜色 */  
  border: none;  
  border-radius: 4px;  
  cursor: pointer;  
}  
  
/* 将按钮包装器定位在密码栏下方 */  
.buttons-wrapper {  
  display: flex; /* 使用flex布局 */  
  justify-content: center; /* 水平居中 */  
  align-items: center; /* 垂直居中 */  
  margin-top: 200px; /* 根据需要调整与密码栏的距离 */  
}
</style>
