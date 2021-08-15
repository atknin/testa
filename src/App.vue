<template>
  <ul class="timeline">
  <li class="timeline-event">
    <label class="timeline-event-icon"></label>
    <div class="timeline-event-copy">
      <p class="timeline-event-thumbnail">KAZNA.TECH</p>
      <h3>ГОСУСЛУГИ ДЕМО</h3>
      <h4>Автоматизация сбора документов</h4>

      <p><strong>Логин и пароль от госуслуг</strong><br>После ввода данных, система автоматичски закажет пакет документов и отобразит в интерфейсе ниже. В боевом режиме система передаст документы закачику (в его CRM или локальную папку). После сбора документов, логин и пароль будут удалены с сервера. </p>
      <strong>Логин</strong>
      <v-text-field
        v-model='data.login'
      ></v-text-field>
      <strong>Пароль</strong>
      <v-text-field
        v-model='data.password'
      ></v-text-field>
    <strong>Номер пасспорта</strong> <br>
      <small>в формате 1234567890 </small>
      <v-text-field
        v-model='data.passport_number'
      ></v-text-field>
      
        <a style='cursor:pointer; text-decoration:underline;' @click='extend()' v-if='!extended'>еще параметры +</a>
        <a style='cursor:pointer; text-decoration:underline;' @click='extend()' v-else>скрыть параметры -</a> <br>

      <div v-if='extended' class='pt-2 '>
        <br>
          <strong>Год получения водительского удостоверения</strong> <br>
          <small>в формате 2000 </small>
          <v-text-field
            v-model='data.date_drive_licensed'
          ></v-text-field>
        <br>
        <strong>Дата рождения</strong> <br>
          <small>в формате 11.11.11 </small>
          <v-text-field
            v-model='data.data_birth'
          ></v-text-field>
          <br>
        <strong>Номер дома в прописке</strong> <br>
          <v-text-field
            v-model='data.house_number'
          ></v-text-field>
        <br>
      </div>
      
        <br>
        <template v-if='data.client_id'>
          <v-progress-circular
            indeterminate
            color="primary"
            class="ma-2"
            x-small
          > {{counter}}</v-progress-circular>
          Ваша постоянная ссылка:
          <a :href="link()"> http://t1.4rd.info/?client_id={{data.client_id}}&login={{data.login}}&password={{data.password}} </a>
          
          как только система получит документы, они появятся ниже.
        </template>
        
      <v-btn class='mt-2' v-else @click='start_parsing()'> начать сбор </v-btn>


    </div>


  </li>
  <template v-if='result'>
  <li class="timeline-event">
    <label class="timeline-event-icon"></label>
    <div class="timeline-event-copy">
      
      <h3>Паспортные данные</h3>
      <h4>{{result.passport_data.firstName}} {{result.passport_data.lastName}} </h4>
      <p class="timeline-event-thumbnail">JSON</p>
      <json-viewer :value="result.passport_data"></json-viewer>
      <!-- <template v-for='(pasp,pasp_v) in result.passport_data'>{{pasp_v}}:{{pasp}} <br></template> -->
    </div>
  </li>

  <li class="timeline-event" v-for='doc in result.documents'>
    <label class="timeline-event-icon"></label>
    <div class="timeline-event-copy">
      <p class="timeline-event-thumbnail">{{doc.type}}</p>
      <h3>{{doc.name}}</h3>
      <h4>{{doc.date_added}}</h4>
      <v-btn  :href='doc.link' x-small>скачать</v-btn>
      <!-- <p><strong>Systemarchitektur, Consulting</strong><br>Konzeption und Modellierung von Systemen und APIs für Digital Publishing und Entitlement nach SOA</p> -->
    </div>
  </li>
  </template>
</ul>  
</template>

<script>
import Vue from 'vue';
const axios = require('axios');

export default {

  name: 'app',

  data: () => ({
    extended: false,
    loading: false,
    counter: 0,
    result: null,
    data:{
      client_id: null,
      login:"",
      password:"",
      passport_number:"",
      date_drive_licensed:"",
      date_birth:"",
      passport_scan:[
      ]
    },
      max_work: 30
  
  }),
  computed: {
   
  },  
  created(){
    var client_id = this.has_key('client_id');
    var login = this.has_key('login');
    var password = this.has_key('password');

    if (client_id){
      this.data.client_id = client_id;
      this.data.login = login;
      this.data.password = password;
      this.loading = true;
      setInterval(() => this.countDown(), 3000);
    }
   },
  mounted()
  {
    window.app = this.$refs.app;

    this.loadState();
    
  },

  methods:
  {
    serialize(obj) {
      var str = [];
      for (var p in obj)
        if (obj.hasOwnProperty(p)) {
          str.push(encodeURIComponent(p) + "=" + encodeURIComponent(obj[p]));
        }
      return str.join("&");
    },

    countDown(){
      this.counter+=1
      axios.get('https://bankrot-10.kazna.tech/api/clients/?'+this.serialize(this.data))
      .then((result) => {
        this.result = result.data
      })
    },
    has_key(key) {
      const uri = window.location.href.split('?');
      let result = null;
      if (uri.length === 2) {
        const vars = uri[1].split('&');
        let tmp = '';
        vars.forEach((v) => {
          tmp = v.split('=');
          if (tmp.length === 2) {
            if (tmp[0] === key) result = tmp[1];
          }
        });
      }
      return result;
    },
    link(){
      return `http://t1.4rd.info/?client_id=${this.data.client_id}&login=${this.data.login}&password=${this.data.password}`
    },
    get_random(){
      var min = 10000
      var max = 12000
      min = Math.ceil(min);
      max = Math.floor(max);
      return Math.floor(Math.random() * (max - min + 1)) + min; //Максимум и минимум включаются
    },
    extend(){
      this.extended = !this.extended
    },
    start_parsing(){
      this.data.client_id = this.get_random();
      this.countDown()
    }
    
  }
}
</script>

<style >
/* Variables */
/* Fonts */
@import url(https://fonts.googleapis.com/css?family=Open+Sans:300,700);
body {
  font-family: "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
  font-size: 1em;
  font-weight: 300;
  line-height: 1.5;
  letter-spacing: 0.05em;
}

/* Layout */
* {
  -moz-box-sizing: border-box;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
}

/* Styling */
.timeline {
  margin: 4em auto;
  position: relative;
  max-width: 46em;
}
.timeline:before {
  background-color: black;
  content: "";
  margin-left: -1px;
  position: absolute;
  top: 0;
  left: 2em;
  width: 2px;
  height: 100%;
}

.timeline-event {
  position: relative;
}
.timeline-event:hover .timeline-event-icon {
  -moz-transform: rotate(-45deg);
  -ms-transform: rotate(-45deg);
  -webkit-transform: rotate(-45deg);
  transform: rotate(-45deg);
  background-color: #a83279;
}
.timeline-event:hover .timeline-event-thumbnail {
  -moz-box-shadow: inset 40em 0 0 0 #a83279;
  -webkit-box-shadow: inset 40em 0 0 0 #a83279;
  box-shadow: inset 40em 0 0 0 #a83279;
}

.timeline-event-copy {
  padding: 2em;
  position: relative;
  top: -1.875em;
  left: 4em;
  width: 80%;
}
.timeline-event-copy h3 {
  font-size: 1.75em;
}
.timeline-event-copy h4 {
  font-size: 1.2em;
  margin-bottom: 1.2em;
}
.timeline-event-copy strong {
  font-weight: 700;
}
.timeline-event-copy p:not(.timeline-event-thumbnail) {
  padding-bottom: 1.2em;
}

.timeline-event-icon {
  -moz-transition: -moz-transform 0.2s ease-in;
  -o-transition: -o-transform 0.2s ease-in;
  -webkit-transition: -webkit-transform 0.2s ease-in;
  transition: transform 0.2s ease-in;
  -moz-transform: rotate(45deg);
  -ms-transform: rotate(45deg);
  -webkit-transform: rotate(45deg);
  transform: rotate(45deg);
  background-color: black;
  outline: 10px solid white;
  display: block;
  margin: 0.5em 0.5em 0.5em -0.5em;
  position: absolute;
  top: 0;
  left: 2em;
  width: 1em;
  height: 1em;
}

.timeline-event-thumbnail {
  -moz-transition: box-shadow 0.5s ease-in 0.1s;
  -o-transition: box-shadow 0.5s ease-in 0.1s;
  -webkit-transition: box-shadow 0.5s ease-in;
  -webkit-transition-delay: 0.1s;
  transition: box-shadow 0.5s ease-in 0.1s;
  color: white;
  font-size: 0.75em;
  background-color: black;
  -moz-box-shadow: inset 0 0 0 0em #ef795a;
  -webkit-box-shadow: inset 0 0 0 0em #ef795a;
  box-shadow: inset 0 0 0 0em #ef795a;
  display: inline-block;
  margin-bottom: 1.2em;
  padding: 0.25em 1em 0.2em 1em;
}


</style>

