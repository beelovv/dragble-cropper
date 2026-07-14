<template>
  <div class="container">
    <input v-if="!fileUrl" type="file" ref="fileInput" @change="()=> addFile()"/>
      <img class="image" v-if="fileUrl" :src="fileUrl" @mousedown="(e: Event) => handleStartSelect(e)" @mouseup="()=> handleEndSelect()">
      <div v-if="needShowField" class="selectedField"></div>
  </div>
</template> 
<script setup lang="ts">
import { ref } from 'vue';

const fileInput = ref<HTMLInputElement>()
const fileUrl = ref()
const needShowField = ref(false)

const cursorStartPosition = ref({
  x: '',
  y: ''
})
const cursorEndPosition = ref({
  x: '',
  y: ''
})

const addFile = () => {
  if (fileInput?.value?.files && fileInput?.value?.files.length > 0){
    const imageToReader = fileInput.value.files[0]
    console.log(imageToReader)
    const reader = new FileReader()
    reader.onload = function (event) {
      if(event.target?.result && event?.target?.result){
          
         fileUrl.value = event.target?.result

         console.log(fileUrl.value)
      }
    }
    reader.readAsDataURL(imageToReader)
  }
}

const handleStartSelect = (e: Event) => {
  e.preventDefault();
  needShowField.value = true
  console.log(e)
  // cursorStartPosition.value.x = this.x 
  // cursorStartPosition.value.y = this.y
  // console.log(cursorStartPosition.value)
}

const handleEndSelect = () => {

}

</script>
<style scoped>
*{
  -webkit-user-select: none; /* Chrome, Safari, Opera */
  -moz-user-select: none;    /* Firefox */
  -ms-user-select: none;     /* Internet Explorer и Edge */
  user-select: none;  
}
.container{
  display: flex;
  position: relative;
  flex-direction: row;
  margin: 120px auto 0px ;
  align-items: center;
  justify-content: center;
  border: 10px solid black;
  width: 700px;
  height: 500px;
  background-color:papayawhip;
  overflow: hidden ;
}

.image {
  width: 100%;
  height: 100%;
  object-fit: contain;
  overflow: hidden ;
}

.selectedField{
  position: absolute;
  border: 700px solid rgba(1, 1, 1, 0.6);
  width: 0px;
  height: 0px;
  z-index: 200;
  top: 0;
  left: 0px;
  right: 0px;
  bottom: 0;
}
</style>