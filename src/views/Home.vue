<template>
  <div class="parent-div" :style="{'overflow:hidden':full}">

<!--card to drag and drop images-->
    <v-card @dragenter.prevent="toggleDrag" @dragleave.prevent="toggleDrag" @dragover.prevent
            @drop.prevent="fileDropped" :class="{'active':drag,'dropped':drop}" class="inActive" elevation="8">
      <span class="text-grey-darken-2">Drag and Drop Images</span>
      <span class="text-grey-darken-2">OR</span>

<!--button to choose images-->
      <label :class="{'btn-hover':hover}" class="btn" for="fileInput"
             @mouseover="toggleHover" @mouseleave="toggleHover">Browse images</label>
      <input multiple accept="image/jpeg,image/jpg,image/png" type="file" id="fileInput" class="d-none" @change="fileInput"/>

<!--div to display preview of selected images-->
      <div class="d-flex flex-wrap align-center justify-center mt-4" v-if="files.length>0">
        <v-card v-for="(name,i) of files" :key="i" height="100px" width="150px" class="ma-4" @mouseover="overlay[i]=true" @mouseleave="overlay[i]=false">
          <img :src="src[i]" :alt="name" style="object-fit: fill;height: 100px;width: 150px"/>
          <v-overlay v-model="overlay[i]" contained width="150px" class="d-flex flex-column">
            <div class="d-flex align-end">
              <v-spacer></v-spacer>
              <v-btn elevation="0" icon size="xx-small" class="bg-transparent" @click="del(i)">
                <v-icon size="x-small" color="white">mdi-close</v-icon>
              </v-btn>
            </div>
            <div class="d-flex justify-center align-center align-self-center">
              <v-btn elevation="0" icon size="large" class="bg-transparent" @click="fullscreen(src[i])">
                <v-icon color="white" size="xx-large">mdi-fullscreen</v-icon>
              </v-btn>
            </div>
            <div class="d-flex justify-center align-center align-self-center mt-2">
              <span class="text-white">{{name.length>=15?name.substring(0,15)+'...':name}}</span>
            </div>
          </v-overlay>
        </v-card>
      </div>
    </v-card>
  </div>

<!--Overlay to display image in full screen-->
  <v-overlay v-model="full" scrim="black" width="100%" height="100%">
    <div class="d-flex">
      <v-spacer></v-spacer>
      <v-btn elevation="0" icon size="large" class="bg-transparent" @click="full=false,srcFull=null">
        <v-icon size="xx-large" color="white">mdi-close</v-icon>
      </v-btn>
    </div>
    <div class="d-flex flex-column justify-center align-center view-full">
      <v-card class="d-flex flex-column" color="transparent" elevation="0" @mouseover="fullHover=true" @mouseleave="fullHover=false">
        <img class="align-self-center" :src="srcFull" width="500" height="500" style="object-fit: contain"/>
      </v-card>
    </div>
  </v-overlay>
</template>

<script>
import {ref} from "vue";

export default {
  name:'Home',
  props:{

  },
  setup(props){
    const files=ref([])
    const drag=ref(false)
    const drop=ref(false)
    const hover=ref(false)
    const overlay=ref([])
    const previewImage=ref(null)
    const src=ref([])
    const srcFull=ref(null)
    const full=ref(false)
    const fullHover=ref(false)


    function toggleDrag(){
      drag.value=!drag.value
    }

    function fileDropped(e){
      drag.value=false
      drop.value=true
      if(e.dataTransfer.files.length===1 && !files.value.includes(e.dataTransfer.files[0].name)){
        const extension=e.dataTransfer.files[0].name.split('.')[e.dataTransfer.files[0].name.split('.').length-1]
        if(extension==='jpeg'||extension==='jpg'||extension==='png'){
          files.value.push(e.dataTransfer.files[0].name)
          const reader = new FileReader
          reader.readAsDataURL(e.dataTransfer.files[0])
          reader.onload = function(e) {
            src.value.push(reader.result)
          }
          overlay.value[0]=false
        }
        else {
          drop.value=false
          window.alert(`file extension not supported "${e.dataTransfer.files[0].name}"`)
        }
      }
      else if(e.dataTransfer.files.length>1){
        for (let i=0;i<e.dataTransfer.files.length;i++){
          if(!files.value.includes(e.dataTransfer.files[i].name)){
            const extension=e.dataTransfer.files[i].name.split('.')[e.dataTransfer.files[i].name.split('.').length-1]
            if(extension==='jpeg'||extension==='jpg'||extension==='png'){
              files.value.push(e.dataTransfer.files[i].name)
              const reader = new FileReader
              reader.readAsDataURL(e.dataTransfer.files[i])
              reader.onload = function(e) {
                src.value.push(reader.result)
              }
              overlay.value[i]=false
            }
            else{
              window.alert(`file extension not supported "${e.dataTransfer.files[i].name}"`)
            }
          }
          else{
            window.alert(`Image already selected "${e.dataTransfer.files[i].name}"`)
          }
        }
      }
      else{
        window.alert(`Image already selected "${e.dataTransfer.files[0].name}"`)
      }
    }

    function fileInput(e){
      drop.value=true
      if(e.target.files.length===1 && !files.value.includes(e.target.files[0].name)){
        files.value.push(e.target.files[0].name)
        const reader = new FileReader
        reader.readAsDataURL(e.target.files[0])
        reader.onload = function(e) {
          src.value.push(reader.result)
        }
        overlay.value[0]=false
      }
      else if(e.target.files.length>1){
        for (let i=0;i<e.target.files.length;i++){
          if(!files.value.includes(e.target.files[i].name)){
            files.value.push(e.target.files[i].name)
            const reader = new FileReader
            reader.readAsDataURL(e.target.files[i])
            reader.onload = function(e) {
              src.value.push(reader.result)
            }
            overlay.value[i]=false
          }
          else{
            window.alert(`Image already selected "${e.target.files[i].name}"`)
          }
        }
      }
      else{
        window.alert(`Image already selected "${e.target.files[0].name}"`)
      }
    }

    function toggleHover(){
      hover.value=!hover.value
    }

    function del(i){
      files.value.splice(i,1)
      src.value.splice(i,1)
    }

    function fullscreen(_src){
      full.value=true
      srcFull.value=_src

    }


    return{
      files,
      drag,
      drop,
      hover,
      previewImage,
      src,
      overlay,
      full,
      srcFull,
      fullHover,
      toggleDrag,
      fileDropped,
      toggleHover,
      del,
      fileInput,
      fullscreen
    }
  }
}
</script>

<style>
.parent-div{
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  /*overflow-y: scroll;*/
}

.inActive{
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 5%;
  width: 300px;
  min-height: 30vh;
  transition: 0.3s ease all;
}
.active{
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 5%;
  width: 400px;
  background: lightgrey;
  transition: 0.3s ease all;
}
.dropped{
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 5%;
  width: 40vw;
  transition: 0.3s ease all;
  /*line-height: 10px;*/
}
.btn{
  color: black;
  transition: 0.3s ease all;
}
.btn-hover{
  /*color: darkslategray;*/
  cursor: pointer;
  color: deepskyblue;
}
.view-full{
  height: 100vh;
}
</style>
