<script>
export default {
    components: {},
    data: function () {
        return {
            errormsg: null,
            username: "",
            profile: {
                id: 0,
                username: "",
            },
        }
    },
    methods: {
        async doLogin() {
            if (this.username == "") {
                this.errormsg = "Username cannot be empty.";
            } else {
                try {
                    let response = await this.$axios.post("/session", { username: this.username })
                    this.profile = response.data
                    localStorage.setItem("token", this.profile.id);
                    localStorage.setItem("username", this.profile.username);
                    this.$router.push({ path: '/session' })
                } catch (e) {
                    if (e.response && e.response.status === 400) {
                        this.errormsg = "Form error, please check all fields and try again. If you think that this is an error, write an e-mail to us.";
                        this.detailedmsg = null;
                    } else if (e.response && e.response.status === 500) {
                        this.errormsg = "An internal error occurred. We will be notified. Please try again later.";
                        this.detailedmsg = e.toString();
                    } else {
                        this.errormsg = e.toString();
                        this.detailedmsg = null;
                    }
                }
            }

        }
    },
    mounted() {

    }

}
</script>


<template>
    <div class="login-container">
      <h1 class="text-center mb-4">Welcome to WASAPhoto</h1>
      <div class="mb-3">
        <input type="text" v-model="username" class="form-control" placeholder="Username">
      </div>
      <div class="mb-3">
        <button class="btn btn-success btn-block" @click="doLogin" style="background-color: #0d6efd;" >Login</button>
      </div>
    </div>
    <ErrorMsg v-if="errormsg" :msg="errormsg"></ErrorMsg>
</template>
  
  
  <style scoped>
    .login-container {
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      background-color: #f0f0f0;
      margin-right: 250px;
    }
  
    h1 {
      font-size: 1.5rem;
      margin-bottom: 1rem;
    }
  
    input {
      width: 100%;
      padding: 0.5rem;
      margin-bottom: 1rem;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
  
    button {
      width: 100%;
      padding: 0.5rem;
      background-color: #4caf50;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
  
    .error-message {
      margin-top: 1rem;
      color: #f44336;
    }
  </style>
  