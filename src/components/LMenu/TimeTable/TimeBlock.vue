<template>
  <div class="time-block" :style="blockstyle" @click="clickEvent" v-show='this.timeBlockData.blockKind != "sendBtn"'>
    <div class="time-block-content notosanskr-normal-black-14px"> 
      {{ this.timeBlockData.content }}
    </div>
  </div>
  <button class="send-btn" @click="clickEvent" v-show='this.timeBlockData.blockKind == "sendBtn"' >
    <div class="svg-wrapper-1">
      <div class="svg-wrapper">
        <svg class="svg" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="14" height="14">
          <path fill="none" d="M0 0h24v24H0z"></path>
          <path fill="currentColor" d="M1.946 9.315c-.522-.174-.527-.455.01-.634l19.087-6.362c.529-.176.832.12.684.638l-5.454 19.086c-.15.529-.455.547-.679.045L12 14l6-8-8 6-8.054-2.685z"></path>
        </svg>
      </div>
    </div>
    <span class="span">찾기</span>
  </button> 
</template>
  
  <script>
  export default {
    name: "TimeBlock",
    props: ["timeBlockData"],
    computed: {
      blockstyle() {
        let color
        switch (this.timeBlockData.blockKind) {
          case "block":
            if (this.timeBlockData.isSelected) {
              color = `#bdbdbd`
            }
            else {
              color = `#f0f0f0`
            }
            break;
          case "lecBlock":
            color = this.timeBlockData.color
            break;
          case "dayBlock":
            color = `#e0e0e0`
            break;
          case "hourBlock":
            color = `#e0e0e0`
            break;
          default:
            color = `#ff00d6`
            break;
        }
        return {
          '--height' : `${(this.timeBlockData.end - this.timeBlockData.start) * 96}px`,
          '--color' : color
        }
      }
    },
    methods:{
      async clickEvent(){
        let timeLines
        let timeLine
        let isLogined = this.$store.getters.getIsLogined

        if(!isLogined) {
          return
        }

        switch(this.timeBlockData.blockKind) {
          case "block":
            this.timeBlockData.isSelected = !this.timeBlockData.isSelected
            //console.log(this.timeBlockData.isSelected)
            break
          case "lecBlock":
            //console.log(this.timeBlockData.lecNum)
            await this.$store.dispatch("fetchLecDetails", this.timeBlockData.lecNum)
            break
          case "dayBlock":
            //console.log(this.timeBlockData.content)
            this.timeBlockData.isSelected = !this.timeBlockData.isSelected
            timeLines = this.$store.getters.getTimeLines
            timeLine = timeLines[this.timeBlockData.content] 
            timeLine.forEach(x=> {
              if(this.timeBlockData.isSelected != x.isSelected && x.blockKind =="block") {
                x.isSelected = this.timeBlockData.isSelected
              }
            })
            break
          case "hourBlock":
            //console.log(this.$store.getters.getTimeblockLists)
            this.timeBlockData.isSelected = !this.timeBlockData.isSelected
            timeLines = this.$store.getters.getTimeLines
            
            for(let x in timeLines){
              //console.log(x)
              timeLines[x].forEach(y =>{
                if(Math.floor(y.start) == this.timeBlockData.start && y.blockKind == "block"
                  && this.timeBlockData.isSelected != y.isSelected) {
                  y.isSelected = this.timeBlockData.isSelected
                  //console.log(y.isSelected)
                }
              })
            }
            break
          case "sendBtn":
            timeLines = this.$store.getters.getTimeLines
            this.$store.commit("clearSelectedTimes")

            for(let x in timeLines){
              timeLines[x].forEach(y=> {
                if(y.blockKind == "block" && y.isSelected) {
                  this.$store.commit("addSelectedTimes", {day: x, data: y})
                }
              })
            }
            this.$store.commit("clearRecommList")
            await this.$store.dispatch("fetchRecommList")
            this.$store.dispatch("changeScreen", 2)

            break

          default: 
           alert("error")
           break
        }
      }
    }
  };
  
  </script>
  
  <style lang="sass">
  @import '../../../../variables'

  .send-btn
    font-size: 13px 
    background: $green
    color: white 
    display: flex 
    align-items: center 
    border: none 
    border-radius: 4px 
    overflow: hidden
    height: 48px
    width: 100%
    transition: all 0.2s
    .span
      display: block 
      margin-left: 0.3em 
      transition: all 0.3s ease-in-out 
    .svg
      display: block 
      transform-origin: center center 
      transition: transform 0.3s ease-in-out 
  .send-btn:hover
    .svg-wrapper
      animation: fly-1 0.6s ease-in-out infinite alternate 
    .svg
      transform: translateX(1.2em) rotate(45deg) scale(1.1) 
    .span
      transform: translateX(5em)
  .send-btn:active
    transform: scale(0.95)
  
  .time-block
    align-items: center
    background-color: var(--color)
    display: flex
    flex-direction: column
    height: var(--height)
    width: 100%
    border-style: none none solid
    border-width: 2px
    border-color: $white
    position: relative
    overflow: hidden
    border-radius: 5px
    
  .time-block:hover
    transform: scale(1.05)
    background-color: $green
  .time-block:active
    transform: scale(0.9)

  .time-block-content
    height: 100%
    line-height: normal
    text-align: center
    width: 100%
    position: relative
    
  @keyframes fly-1
    0%
      transform: translateY(0.1em)
    100%
      transform: translateY(-0.1em)
  
    
  </style>
  