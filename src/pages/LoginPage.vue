<template>
    <!-- 键盘监听回车键，按下时提交表单 -->
    <div class="login" @keydown.enter="submitForm">
        <el-form :model="loginForm" ref="loginForm" label-width="100px">
            <el-form-item label="用户名" prop="username">
                <el-input v-model="loginForm.username" size="tiny"></el-input>
            </el-form-item>
            <el-form-item label="密码" prop="password">
                <el-input type="password" v-model="loginForm.password" auto-complete="off" size="tiny"></el-input>
            </el-form-item>
            <el-form-item label="验证码" prop="captchaValue">
                <el-input v-model="loginForm.captchaValue" auto-complete="off" size="tiny"></el-input>
            </el-form-item>
            <el-form-item class="item">
                <el-button @click="fetchCaptcha">换一张</el-button>
                <img :src="'data:image/jpeg;base64,'+image"></img>
            </el-form-item>
            <el-form-item class="item">
                <el-button type="primary" @click="submitForm">登录</el-button>
                <el-button @click="resetForm">重置</el-button>
                <el-button @click="forgetPassword">忘记密码</el-button>
            </el-form-item>
            <p v-for="error in errors" :key="error">{{error.field}}:{{error.message}}</p>
        </el-form>
    </div>
</template>

<script>
export default {
    data() {
        return {
            loginForm: {
                username: '',
                password: '',
                captchaValue: '',
                captchaCode: '',
                userMode: 'USERNAME'
            },
            image: '',
            errors: []
        }
    },
    methods: {
        fetchCaptcha() {
            this.axios.get("/captchas").then((response) => {
                console.log(response.data)
                this.image = response.data.image
                this.loginForm.captchaCode = response.data.captchaCode
            }).catch((error) => {
                throw error
            })
        },
        submitForm() {
            this.errorText = ''
            console.log(this.loginForm)
            this.axios.post("/tokens", this.loginForm).then((response) => {
                console.log("登录成功")
                console.log(response.data)
                //清空表单
                this.resetForm()
                //发出成功提示
                const h = this.$createElement;
                this.$notify({
                    title: '登录成功',
                    message: h('i', { style: 'color: teal' }, '欢迎您，' + response.data.username)
                });
                localStorage.setItem('loginResult', JSON.stringify(response.data))
                //跳转回主页
                this.$router.push('/')
                //刷新当前页面
                window.location.reload()
            }).catch((error) => {
                this.fetchCaptcha()
                console.log(error)
                if ("response" in error) {
                    this.errors = error.response.data.fieldErrors
                }
            })
        },
        resetForm() {
            this.$refs['loginForm'].resetFields()
        },
        forgetPassword() {
            this.resetForm()
            this.$router.push('/forget_password')
        }
    },
    created() {
        // 调用methods里的方法必须用this.
        this.fetchCaptcha()
    }
}
</script>

<style scoped>
.item {
    text-align: center
}

.login {
    width: 30%;
    text-align: center;
    margin-left: auto;
    margin-right: auto;
    margin-top: 20px;
    margin-bottom: 20px;
}
</style>
