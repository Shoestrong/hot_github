<template>
  <div class="wrap">
    <h3>Github热门</h3>
    <div class="datepicker"><datepicker v-model="date" language="ch" min="2015-10-19" :max="maxDate" @input="selectDate"></datepicker></div>
    <div style="line-height:50px;">选择的日期是:{{date}}</div>
    <div class="langBox">
      <ul class="lang">
        <li v-for="lang in langs" :key="lang" @click="langSelect(lang)" class="{active:lang}">{{lang}}</li>
      </ul>
    </div>
    <div class="container page news">
      <transition-group tag="div"
       enter-active-class="animated bounceInRight"
       leave-active-class="animated bounceOutLeft"
       v-if="results.length>0"
      >
        <div class="mdl-card mdl-shadow--2dp section resource-list" style="width:100%;" v-for="item in results" :key="item.id">
          <div class="resource" style="border-left-color:#f1e05a;cursor:pointer;">
            <div class="action-group pull-right">
              <div class="trend-count">
                <span>+</span>
                <span>{{item.count}}</span>
              </div>
            </div>
            <a :href="'https://www.github.com/'+item.repo_name" target="preview">
              <img :src="'https://avatars.githubusercontent.com/'+item.repo_name.split('/')[0]+'?v=3&amp;s=160'" class="thumb animated fadeIn">
            </a>
            <a :href="'https://www.github.com/'+item.repo_name" target="preview" rel="nofollow">
              <h5 class="title">
                <span class="orgname">
                  <span>{{item.repo_name.split("/")[0]}}</span>
                  <span>/</span>
                </span>
                <span>{{item.repo_name.split("/")[1]}}</span>
              </h5>
            </a>
            <div class="content">
              <p class="description">{{item.description}}</p>
              <div class="tags">
                <span class="badge undefined">
                  star:<span>{{item.watchers_count}}</span>
                </span>
                <span class="badge undefined">
                  语言:<span>{{item.language}}</span>
                </span>
              </div>
            </div>
            <div class="clearfix"></div>
          </div>
        </div>
      </transition-group>
      <transition-group v-else>
        加载中...
      </transition-group>
    </div>
    <div class="loadding" v-show="loadding">
      <div class="loaddingBox"></div>
      <span>加载中{{dot}}</span>
    </div>
  </div>
</template>

<script>
import datepicker from './components/datepicker'
import $ from 'jquery'


export default {
  name: 'app',
  data() {
      return {
          dot:"...",
          date: (new Date).getFullYear()+'-'+(((new Date).getMonth()+1)<10?('0'+((new Date).getMonth()+1)):((new Date).getMonth()+1))+'-'+((new Date).getDate()-1),
          maxDate:(new Date).getFullYear()+'-'+(((new Date).getMonth()+1)<10?('0'+((new Date).getMonth()+1)):((new Date).getMonth()+1))+'-'+((new Date).getDate()-1),
          results:[],
          langs:[],
          loadding:false
      }
  },
  mounted(){
    // console.log(this.date)
    this.$nextTick(()=>{
      this.getList(this.date);
    })
    this.dotMove();
  },
  methods:{
    unique(obj){
      var res = [];
      var json = {};
      for(var i = 0; i < obj.length; i++){
        if(!json[obj[i]]){
           res.push(obj[i]);
           json[obj[i]] = 1;
        }
      }
      return res;
    },
    selectDate(v){
      this.getList(this.date);
    },
    getList(d){
      this.results=[]
      this.langs = []
      $.ajax({
        url:"http://app.gitlogs.com/trending",
        data:'date='+d,
        success:(result) =>{
          result.forEach((v)=>{
            if(v.repo){
              this.results.push({
                count:v.count,
                date:v.date,
                gain:v.gain,
                id:v.id,
                repo_name:v.repo_name,
                yesterday_count:v.yesterday_count,
                cached_at:v.repo.cached_at,
                description:v.repo.description,
                language:v.repo.language,
                watchers_count:v.repo.watchers_count
              })
            }
          })
          this.results.forEach((v)=>{
            if(v.language){
              this.langs.push(v.language)
            }
          })
          this.langs = this.unique(this.langs)
        }
      });
    },
    langSelect(lang){
      this.getList(this.date);
      this.loadding=true;
      setTimeout(()=>{
        this.loadding=false;
        this.results = this.results.filter((v)=>{
          return lang===v.language
        })
      },2000)
    },
    dotMove(){
      this.dot="∙";
      setInterval(v=>{
        this.dot+='∙';
        if(this.dot.length>6){
          this.dot="∙";
        }
      },300)
    }
  },
  components: { datepicker }
}
</script>

<style>
ul,li{
  list-style: none;
  margin: 0;
  padding: 0;
}
.wrap {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  padding: 20px 0;
}
@media (min-width: 1200px){
  .datepicker,.lang {
      width: 1170px;
  }
}
@media (min-width: 992px){
  .datepicker,.lang {
      width: 1170px;
  }
}
@media (min-width: 768px){
  .datepicker,.lang {
      width: 1170px;
  }
}
.langBox{
  margin-bottom: 10px;
}
.datepicker{
  line-height: 32px;
  padding: 0 5px;
  margin: 0 auto;
}
.lang{
  overflow: hidden;
  margin: 0 auto;
}
.lang li{
  float: left;
  cursor: pointer;
  padding: 2px 8px;
  border: 1px solid #ccc;
  border-radius: 5px;
  margin: 0 5px 5px;
}
.lang li:hover{
  border-color: #5bc0de;
  color: #5bc0de;
}
.fade-enter, .fade-leave-active{
  opacity: 0;
  transform: scale3d(0, 0, 0);
}
.fade-enter-active, .fade-leave-active{
  transition: all ease .1s;
}
.loadding{
  position: fixed;
}
.loaddingBox{
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0,0,0,.4);
}
.loadding span{
  position: fixed;
  top: 50%;
  left: 50%;
  margin-top: -50px;
  margin-left: -50px;
  font-size: 18px;
  width: 100px;
  height: 100px;
  text-align: center;
  line-height: 100px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0,0,0,.5);
  background-color: #fff;
}
</style>
