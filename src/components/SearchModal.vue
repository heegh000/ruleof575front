<template>
    <div class="search-modal">
        <div class= "overlay-search valign-text-middle notosanskr-bold-white-24px" @click="$emit('modal-close')" >
          <div class = empty-cart></div>
          창을 닫으려면 바깥을 클릭
        </div>
        <div class= "search-modal-card">
          <RMenuTitleBox color= "red" size="534">과목 직접 담기</RMenuTitleBox>
          <div class="search">
            <input type="text" class="search-box" id="searchbox"  placeholder="원하는 과목 직접 검색"/>
            <input type="submit" class="search-button" value="검색" @click="getSearchText()">
          </div>
          <SearchModalRecord :lecData=lec v-for="lec in this.lecList" :key="lec"></SearchModalRecord>
        </div>
    </div>
  </template>
  

<script>
import SearchModalRecord from "./SearchModalRecord";
import RMenuTitleBox from "./RMenu/Box/RMenuTitleBox"
import { parseLectime } from "../util";
import axios from "axios"

export default {
  name: "Modal",
  props: [],
  components: {
    SearchModalRecord, RMenuTitleBox
  },
  data(){
    return{
      input: "",
      lecList : []
    }
  },
  methods:{
    async getSearchText(){
      this.input = document.getElementById("searchbox").value
      try{
        this.lecList = (await axios.get('https://ruleof.datasesang.store/list/search', {params: {keyword: this.input}})).data.list
        for(let lec of this.lecList) {
          let parsedLT = parseLectime(lec.수업시간);
          lec.수업시간 = parsedLT.수업시간
          lec.요일 = parsedLT.요일
          lec.시작시간 = parsedLT.시작시간
          lec.끝시간 = parsedLT.끝시간
        }
      }
      catch(error){
        console.log(error)
      }
    }
  }
};



</script>

<style lang="sass">
@import '../../variables'

.search-modal
  width: 600px
  height: 960px
  position: absolute
  
.overlay-search
  width: 100%
  height: 100%
  position: absolute
  opacity: 0.3
  background-color: $black
  align-items: center
  border-radius: 15px
  
.search-modal-card
  background-color: $error-color
  position: absolute
  left: 5%
  width: 90%
  height : 80%
  margin-top: 5%
  background-color: $pippin
  z-index: 10
  border: 3px solid $geraldine
  overflow-y: overlay
  border-top-left-radius: 15px
  border-top-right-radius: 15px

.search-modal-card::-webkit-scrollbar 
 display: none
  
.search-modal-card::-webkit-scrollbar-thumb
  background-color: rgba(0,0,0)
  height: 95%

.empty-cart
  width: 100%
  height: 80%
  opacity: 0
  background-color: black

.search
  position: relative 
  display: flex
  width:100%

.search-box
  width:80%
  font-size: 20px 
  border: 2px solid
  border-color: $geraldine
  font-family: "Noto Sans KR", Helvetica

.search-button
  width:20%
  font-size: 20px
  border: 2px solid
  border-color: $geraldine
  font-family: "Noto Sans KR", Helvetica
.search-button:hover
  transform: scale(1.1)
  background-color: $silver
.search-button:active
  transform: scale(0.95)
  border: none
</style>