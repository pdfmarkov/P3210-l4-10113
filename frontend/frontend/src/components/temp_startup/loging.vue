<template>
  <div id="loging">
      <div id="login-form">
      <fieldset>
        <label>логин</label>
        <input type="text" placeholder="login" title="имя пользователя" v-model="user.login" required="true" autocomplete="username">
      </fieldset>

      <fieldset>
        <label>пароль</label>
        <input type="password" placeholder="password (min 10 symbols)" title="пароль" v-model="user.password" required="true" autocomplete="current-password">
      </fieldset>
          <a id="err_message" ></a>
      <fieldset class="separated">
        <button @click="signin" title="Кнопка для входа в существующий аккаунт" :disabled="isNotFilled" class="btn">Войти</button>
        <button @click="signup" title="Кнопка для создания нового аккаунта" :disabled="isNotFilled" class="btn">Зарегистрироваться</button>
      </fieldset>
    </div>
  </div>
</template>

<script>
  const baseURL = 'http://localhost:41143/';
  export default {
    name: 'loging',
    props: ['access', 'refresh'],
    data: function() {
      return { 
        user: {
          login: '',
          password: '',
        },
        minPasswordLength: 8,
        queries: {
          signin: 'api/aunt/sign_in',
          register: 'api/aunt/register',
        },
      };
    },
    computed: {
      isNotFilled: function() {

        return !(this.user.login !== undefined && this.user.login !== null && this.user.password !== undefined && this.user.password !== null && this.user.login.length > 0 && this.user.password.length >= this.minPasswordLength); 
      },
    },
    methods: {
      createMessage: async function(message) {
        console.error(message);
        let field = document.getElementById('err_message');
        field.innerText = message;
      },

      signin: async function(event) {
        console.log('sign in account:');
        console.log(`user: ${this.user}`);

        console.log('fetching tokens from server...');
        let response = await fetch(baseURL+this.queries.signin, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json;charset=utf-8',
          },
          body: JSON.stringify(this.user)
        }).catch(function (){
          alert("Error while authentication. Check your connection")
        });

        let json = null;
        try {
          json = await response.json();
        } catch (e) {
          console.error(e);
          return;
        }

        console.log('check if response is ok (200)');
        if (response.ok) {
          console.log('response 200; get token');
          console.log(`response body: ${json}`);
          if (!json) {
            console.log('bad data: expected { accessToken, refreshToken }');
            this.createMessage(`*${json}`);
          } else {
            console.log(`got user access-token`);
            console.log(`got user refresh-token`);
            localStorage.setItem("accessToken", json.accessToken);
            localStorage.setItem("refreshToken", json.refreshToken);

            this.$session.set(this.access, json.accessToken);
            this.$session.set(this.refresh, json.refreshToken);
            //console.log(this.$session.get(this.access)+"   "+this.$session.get(this.refresh))

            //window.location.reload();
            this.$router.push({name: 'app-page'});
          }
        } else this.createMessage(`*${json.description}`);
      },

      signup: async function(event) {

        // plug for testing buttons uncomment next 4 lines
        // this.$session.set(this.access, '0123456789');
        // this.$session.set(this.refresh, '0123456789');
        // window.location.reload();
        // return;

        console.log('sign up new account:');
        console.log(`user: ${this.user}`);
        
        console.log('fetching tokens from server...');
        let response = await fetch(baseURL+this.queries.register, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json;charset=utf-8',
          },
          body: JSON.stringify(this.user)
        }).catch(function (){
          alert("Error while getting token. Check your connection")
        });

        console.log('sent request');
        console.log('check if status 201');

        let json = null;
        try {
          json = await response.json();
        } catch (e) {
          console.error(e);
          return;
        }

        if (response.status === 201) {
            console.log('user created');
          console.log(`response body: ${JSON.stringify(json)}`);
          if (!json)
            console.log('bad data: expected { accessToken, refreshToken }');
          else {
            console.log('got user access-token');
            console.log('get user refresh-token');
            localStorage.setItem("accessToken", json.accessToken);
            localStorage.setItem("refreshToken", json.refreshToken);

            this.$session.set(this.access, json.accessToken);
            this.$session.set(this.refresh, json.refreshToken);

            //window.location.reload();
            this.$router.push({name: 'app-page'});
          }
        } else {
            this.createMessage(`*${json.description}`);
        }
      },
    }
  }

</script>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Lato&display=swap');
  @import url('https://fonts.googleapis.com/css2?family=Pacifico&display=swap%27');

  #loging {
    background: #fff;
    display: block;
    width: 40%;
    margin: 0 auto;
  }

  #login-form {
      margin-top: 23vh;
      box-shadow: 0 0 10px rgba(0,0,0,.3);
  }

  #login-form input {
    margin: auto;
    width: 90%;
    border: 1px solid #c6c9cc;
    border-radius: 5px;
    color: #555;
    display: block;
    padding: 1% 2%;
    height: 50px;
  }

  #login-form label {
    padding: 0 5% ;
    color: #3e606f;
    font-family: Satisfy,Lato,Open Sans,Roboto,sans-serif;
    text-transform: capitalize;
    font-size: 120%
  }

  #login-form fieldset {
    padding: 4%;
    border: 1px solid #fff;
    border-radius: 5px;
    margin: 1% 1%;
  }

  #err_message {
      font-size: 100%;
      font-style: italic;
      margin-left: 10%;
      color: #cd1a21
  }

  .btn {
    border: 0;
    border-radius: 5px;
    cursor: pointer;
    display: inline-block;
    letter-spacing: .5px;
    margin: 2% 2%;
    padding: 2% 5%;
    background-color: #3e606f;
    color: #eaeaea;
    font-size: 18px;
  }

  #loging .btn:disabled {
    background-color: #5e808f;
  }

  .btn:hover {
    background-color: #5e808f;
  }

  .separated {
    text-align: center;
  }

  @media only all and (min-width: 882px) and (max-width: 1255px) {
      #login-form {
          display: block;
          height: 100%;
          width: 100%;
      }

      #login-form label {
          font-size: 160%;
      }


      #loging {
          width: 50%;
      }
  }

  @media only all and (max-width: 881px) {
    #loging {
      display: block;
      width: 90%;
    }
  }
</style>

//TODO: расположение формы ввода слева а графика справа