<template>
  <div class="lec-list-record">
    <div class="lec-list-record-cart">
      <div class = "updown-cart">
        <img :style = "setStyle" class="up-button" src='upbutton.svg' @click = "moveUp">
        <img :style = "setStyle" class="down-button" src='downbutton.svg' @click = "moveDown">
      </div>
      <RMenuTextBox class= "lec-list-classname" :text= this.lecData.과목명 :color = "this.lecData.color" size= "305" @click="showDetails"/>
      <RMenuTextBox :text= this.lecData.대표교강사명 :color = "this.lecData.color" size= "75"/>
      <RMenuTextBox :text= this.lecData.수업시간 :color = "this.lecData.color" size= "100" fontsize="13"/>
      <SearchImageBox src= './addbutton.svg' :color = "this.lecData.color" @click="clickAddBtn" @mouseover="addShadowToTT" @mouseleave="clearShadowLec" v-show="this.lecData.isInTable == 0"/>
      <SearchImageBox src= './subtractbutton.svg' :color = "this.lecData.color" @click="clickAddBtn" @mouseover="addShadowToTT" @mouseleave="clearShadowLec" v-show="this.lecData.isInTable == 1"/>
      <SearchImageBox src= './deletebutton.svg' :color = "this.lecData.color" @click="clickDelBtn"/>
    </div>
  </div>
</template>

<script>
import RMenuTextBox from "../Box/RMenuTextBox";
import SearchImageBox from "../Box/SearchImageBox";
import {timeToNum} from "../../../util"
export default {
  name: "LecListRecord",
  components: {
    RMenuTextBox,
    SearchImageBox
  },
  props: ["lecData"],
  computed:{
    setStyle(){
      return {
        "--color" : `${this.lecData.color}`
      }
    }
  },
  methods : {
    async showDetails() {
      await this.$store.dispatch("fetchLecDetails", this.lecData.수업번호);
    },
    clickAddBtn() {
      if(this.lecData.isInTable == 0) {
        this.addToTimeTable()
      }
      else {
        this.delFromTimeTable()
      }
      this.$store.commit("setIsChanged", true)

    },
    clickDelBtn() {
      if(this.lecData.isInTable == 1) {
        this.delFromTimeTable();
      }
      this.$store.commit("delLecList", this.lecData);
      this.$store.commit("setIsChanged", true)
    },
    addToTimeTable() {
      let timeLines = this.$store.getters.getTimeLines 
      let timeLine
      let curDay
      let startToEnd
      let isOverlapped = false

      //겹치는 시간의 수업이 있는지 요일별 타임라인을 확인
      for(let i = 0; i < this.lecData.요일.length; i++) {
        curDay = this.lecData.요일[i]

        if(curDay == '시간미지정강좌' || curDay == '집중수업') {
          continue
        }

        timeLine = timeLines[curDay]
        startToEnd = timeToNum(this.lecData.시작시간[i], this.lecData.끝시간[i])

        for(let block of timeLine) {
          if(block.blockKind == "lecBlock") {
            if(startToEnd.start < block.start && startToEnd.end > block.start) {
              isOverlapped = true
              break
            }
            if(startToEnd.start < block.end && startToEnd.end > block.end) {
              isOverlapped = true
              break
            }
            if(startToEnd.start >= block.start && startToEnd.end <= block.end) {
              isOverlapped = true
              break
            }
          }
        }
        if(isOverlapped) {
          break
        }
      }

      if(!isOverlapped) {
        this.lecData.isInTable = 1;
        this.$store.commit("setTimetableHackjum", this.lecData.학점)
        this.lecData['color'] = this.$store.getters.getColor
        this.$store.commit("setNextColor")
        for(let i = 0; i < this.lecData.요일.length; i++) {
          curDay = this.lecData.요일[i]
          this.$store.commit("setUpTimeLines", curDay);
        }
      }
      else {
        alert("겹치는 수업이 있습니다")
      }
    },
    delFromTimeTable() {
      let curDay
      this.lecData.isInTable = 0;
      this.lecData.color = null;
      this.$store.commit("setTimetableHackjum", -(this.lecData.학점))
      for(let i = 0; i < this.lecData.요일.length; i++) {
        curDay = this.lecData.요일[i]
        this.$store.commit("setUpTimeLines", curDay);
      }
    },
    addShadowToTT(){
      this.$store.commit("addShadowLec", this.lecData)
    },
    clearShadowLec() {
      this.$store.commit("clearShadowLec")
    },
    moveUp() {
      this.$store.commit("lecListMoveUp", this.lecData.수업번호)
    },
    moveDown() {
      this.$store.commit("lecListMoveDown", this.lecData.수업번호)
    }
  }
};
</script>

<style lang="sass">
@import '../../../../variables'

.lec-list-record
  align-items: flex-start
  background-color: $pippin
  display: flex
  position: relative
  width: 100%

.lec-list-record-cart
  align-items: center
  display: flex
  height: 40px
  min-width: 394px
  position: relative

.lec-list-classname:active
  transform: scale(0.95)
  border: none
.updown-cart
  display: flex
  flex-direction: column
.up-button
  height: 20px
  width: 20px
  border-top: 1px white solid
  border-bottom: 1px white solid
  border-top-left-radius : 10px
  background-color: var(--color)
.down-button
  height: 20px
  width: 20px
  background-color: var(--color)
  border-bottom-left-radius : 10px
.up-button:hover,
.down-button:hover
  transform: scale(1.3)
  color: black
.up-button:active,
.down-button:active
  transform: scale(0.7)
</style>
