<template>
  <div class="parent-div" :style="{'overflow:hidden':full}">

<!--Drop Zone-->
    <v-card @dragenter.prevent="toggleDrag" @dragleave.prevent="toggleDrag" @dragover.prevent
            @drop.prevent="fileDropped" :class="{'active':drag,'dropped':drop}" class="inActive" elevation="8">
      <span class="text-grey-darken-2">Drag and Drop Files</span>
      <span class="text-grey-darken-2">OR</span>

<!--CTA to choose Files-->
      <label :class="{'btn-hover':hover}" class="btn" for="fileInput"
             @mouseover="toggleHover" @mouseleave="toggleHover">Browse Files</label>
      <input multiple :accept="fileType" type="file" id="fileInput" class="d-none" @change="fileInput"/>

<!--Preview Images-->
      <div v-if="isImage && files.length>0" class="d-flex flex-wrap align-center justify-center mt-4">
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
        <v-overlay v-model="full" scrim="black" width="100%" height="100%">
          <div class="d-flex">
            <v-spacer></v-spacer>
            <v-btn elevation="0" icon size="large" class="bg-transparent" @click="full=false,srcFull=null">
              <v-icon size="xx-large" color="white">mdi-close</v-icon>
            </v-btn>
          </div>
          <div class="d-flex flex-column justify-center align-center" >
              <img :src="srcFull" style="object-fit: contain;width: calc(80vw);height: calc(80vh)"/>
          </div>
        </v-overlay>
      </div>

<!--Preview Video Files-->
      <div v-if="isVideo && files.length>0" class="d-flex flex-wrap align-center justify-center mt-4">
        <v-card v-for="(name,i) of files" :key="i" height="150px" width="200px" class="ma-4 bg-transparent" elevation="0">
          <div class="d-flex">
            <v-spacer></v-spacer>
            <v-btn elevation="0" icon size="xx-small" class="pa-0 ma-0 " style="position: absolute;top: 10px;right: 0;z-index: 1" @click="del(i)">
              <v-icon size="small" color="red">mdi-close</v-icon>
            </v-btn>
          </div>
          <video :src="src[i]" controls style="height: 150px;width: 200px" class="pa-0 ma-0">
          </video>
        </v-card>
      </div>

<!--Preview Audio Files-->
      <div v-if="isAudio && files.length>0" class="d-flex flex-wrap align-center justify-center mt-4">
        <v-card v-for="(name,i) of files" :key="i" class="d-flex ma-4 bg-transparent justify-center align-center" elevation="0">
          <audio :src="src[i]" controls class="pa-0 ma-0">
          </audio>
          <v-btn elevation="0" icon size="x-small" class="pa-0 ma-0 " @click="del(i)">
            <v-icon size="small" color="red">mdi-delete</v-icon>
          </v-btn>
        </v-card>
      </div>

<!--Preview PDF Files-->

<!--      <div v-if="isPdf && files.length>0" class="d-flex flex-wrap align-center justify-center mt-4">-->
<!--        <v-card v-for="(name,i) of files" :key="i" class="d-flex ma-4 bg-transparent justify-center align-center" elevation="0">-->
<!--          -->
<!--          <v-btn elevation="0" icon size="x-small" class="pa-0 ma-0 " @click="del(i)">-->
<!--            <v-icon size="small" color="red">mdi-delete</v-icon>-->
<!--          </v-btn>-->
<!--        </v-card>-->
<!--      </div>-->

<!--Preview Unknown Files-->
      <div v-if="isUnknown && files.length>0" class="d-flex flex-column  mt-4">
        <div v-for="(name,i) of files" :key="i" class="d-flex align-center mx-4">
            <v-list-item-avatar class="mr-2">{{i+1}}.</v-list-item-avatar>
            <v-list-item-title>{{name}}</v-list-item-title>
          <v-spacer></v-spacer>
            <v-btn elevation="0" icon size="x-small" class="pa-0 ma-0 " @click="del(i)">
              <v-icon size="small" color="red">mdi-delete</v-icon>
            </v-btn>
        </div>
      </div>


    </v-card>
  </div>
</template>

<script>
import {ref} from "vue";

export default {
  name:'Home',
  props:{
    fileType: {
      type:String,
      required:true
    }
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
    const isImage=ref(false)
    const isAudio=ref(false)
    const isVideo=ref(false)
    const isPdf=ref(false)
    const isUnknown=ref(false)

    function resolveType(){
      if (props.fileType.includes('image') || props.fileType.includes('jpg') || props.fileType.includes('jpeg') || props.fileType.includes('png')) isImage.value=true;
      else if (props.fileType.includes('video') || props.fileType.includes('mpeg')|| props.fileType.includes('mp4'))isVideo.value=true;
      else if (props.fileType.includes('audio') || props.fileType.includes('mp3')) isAudio.value=true;
      // else if (props.fileType.includes('pdf')) isPdf.value=true;
      else isUnknown.value=true;
    }

    function toggleDrag(){
      drag.value=!drag.value
    }

    function fileDropped(e){
      console.log(e.dataTransfer.files)
      drag.value=false
      drop.value=true
      console.log("here")
      if(e.dataTransfer.files.length===1 && !files.value.includes(e.dataTransfer.files[0].name)){
        const extension=e.dataTransfer.files[0].name.split('.')[e.dataTransfer.files[0].name.split('.').length-1]
        if(props.fileType.includes(extension)/*true*/){
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
            if(props.fileType.includes(extension)){
              files.value.push(e.dataTransfer.files[i].name)
              const reader = new FileReader
              reader.readAsDataURL(e.dataTransfer.files[i])
              reader.onload = function(e) {
                src.value.push(reader.result)
              }
              overlay.value[i]=false
            }
            else{
              window.alert(`File extension not supported "${e.dataTransfer.files[i].name}"`)
            }
          }
          else{
            window.alert(`File already selected "${e.dataTransfer.files[i].name}"`)
          }
        }
      }
      else{
        window.alert(`File already selected "${e.dataTransfer.files[0].name}"`)
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

    resolveType();

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
      fullscreen,
      isImage,
      isVideo,
      isAudio,
      isPdf,
      isUnknown
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
