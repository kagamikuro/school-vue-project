<template>
    <div>
        <!--加载栏-->
        <van-overlay :show="show" >
            <div class="wrapper" @click.stop>
                <div class="block" >
                    <div style="margin-top: 20px;margin-left: 40px">
                        <div style="margin-left: 8px">
                            <van-loading type="spinner" color="#1989fa" />
                        </div>
                        <br>
                        加载中
                    </div>
                </div>
            </div>
        </van-overlay>
        <div class="back" :style="{backgroundImage: 'url(' + back + ')' }">
           <div style="width: 100%;height: 400px;">
               <van-tabs v-model="active">
                   <van-tab title="账号登录">
                       <div style="width: 100%;height: 100%;margin-left: 10%;margin-top: 40%">
                           <span>
                               <label><font style="color: red;">*</font><font style="color: white">手机号</font></label>
                               <input type="number" placeholder="请输入手机号" v-model="phone" style="width: 60%;"/>
                           </span>
                           <br>
                           <span>
                                <label><font style="color: red;">*</font><font style="color: white">密码</font></label>&nbsp&nbsp&nbsp&nbsp
                               <input type="password" v-model="password" placeholder="请输入密码" style="width: 60%;"/>
                           </span>
                       </div>
                       <!--滑动验证-->
                       <div style="width: 80%;height: 100%;margin-left: 10%;margin-top: 5%">
                           <slide @fromSon="verificationSuccess"/>
                       </div>
                       <div style="width: 80%;height: 100%;margin-left: 10%;margin-top: 5%">
                           <van-button type="info" size="large" @click="finallyCheck()">登录</van-button>
                           <van-button type="primary" size="large" to="/loginAndRegister/register">注册账号</van-button>
                       </div>
                       <van-divider :style="{ color: '#1989fa', borderColor: '#1989fa', padding: '0 16px' }">
                           其他登录方式
                       </van-divider>
                   </van-tab>
               </van-tabs>
           </div>
        </div>
    </div>
</template>

<script>
    import back from "./boat.jpg"
    import slide from "./Verification.vue"
    export default {
        data(){
            return{
                show:false,
                back: back,
                phone:"",
                password:"",
                status:false
            }
        },
        components:{
            slide,
        },
        methods:{
            onClickLeft:function () {
                /*返回上一页*/
                this.$router.go(-1)
            },
            //滑动验证成功，接收来自子组件的数据
            verificationSuccess:function (data) {
                this.status = data;
            },
            //登录核对
            finallyCheck:function () {
                if (this.phone === '') {
                    this.$toast({
                        message:"手机号不能为空"
                    })
                    return;
                } else {
                    if (this.phone !== '') {
                        var reg = /^1[3456789]\d{9}$/;
                        if (!reg.test(this.phone)) {
                            this.$toast({
                                message:"请输入正确的手机号"
                            })
                            return;
                        }
                    }
                }
                if (this.password === ''){
                    this.$toast({
                        message:"密码不能为空"
                    })
                    return;
                }
                if (!this.status){
                    this.$toast({
                        message:"请进行验证"
                    })
                    return;
                }else{
                    this.show = true;
                    this.$axios.post('http://localhost:1000/auth-service/auth/check?phone='+this.phone+'&password='+this.password).then((response) => {
                        //存进浏览器cookie，时间以秒计算，30分钟
                        this.$cookies.set("AUTH_TOKEN",response.data[0],'30min')
                        this.show = false;
                        if (response.status == 200){
                            this.$toast({
                                message:"登录成功"
                            })
                            //返回个人中心
                            this.$router.push({path:'/mine'})
                            return
                        }
                    }).catch((error) => {
                        this.show = false;
                        //只有用箭头函数才能写this
                        if (error.response.status == 401){
                            this.$toast({
                                message:"账号密码错误"
                            })
                        }
                        this.$toast({
                            message:"服务异常"
                        })
                    });

                }
            },
            //弹出测试
            alertmeaasge:function () {
                console.log("11111")
                this.$toast({
                    message:"账号或密码错误"
                })
            }
         }
    }
</script>

<style scoped>
    .back{
        width: 100%;
        height: 850px;
    }

    .wrapper {
        display: flex;
        align-items: center;
        justify-content: center;
        height: 100%;
    }

    .block {
        width: 120px;
        height: 120px;
        background-color: #fff;
    }
</style>