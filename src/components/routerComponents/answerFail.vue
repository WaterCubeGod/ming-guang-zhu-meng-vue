<template>
    <div class="successContainer">
      <div class="successBox">
        <el-card class="box-card">
          <div class="successInfo">
            <img src="@/assets/fail.jpg" alt="回答成功" class="myimg">
            <h2>回答成功,但得分不到60，奖杯数不变</h2>
            <div label="奖杯">分数为：{{ this.score }}</div>
          </div>
        </el-card>
      </div>
    </div>
    </template>
    
    <script>
    // import img from '~@/assets/successlmg.png'
  


    export default {
      created(){
        this.getData();
        console.log(this.score)
        

      },
      methods:{
        getData(){
          this.score= this.$route.query.Score;
        }

      },
      data() {  
    return {  
      score: 0
      // imgSrc: require("@/assets/successImg.png")
    }  
  }  ,


      name: "AnswerSuccess"
    }
    </script>
    
    <style scoped lang="less">
    
    .successContainer{
      .myimg {
     width: 600px;
      height: 600px;
      background-color: white;;
    }
      .successBox{
        width: 100%;
        height: 80%;
        margin: 20px auto;
        text-align: center;
        .successInfo{
          width: 50%;
          margin: 10px auto;
        }
      }
    }
    
    </style>