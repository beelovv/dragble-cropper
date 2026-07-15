<template>
  <div class="container">
    <input v-if="!fileUrl" type="file" ref="fileInput" @change="()=> addFile()"/>
      <img ref="image" class="image" v-if="fileUrl" :src="fileUrl" @mousedown="(e: MouseEvent) => handleStartSelect(e)" @mouseup="(e: MouseEvent) => handleEndSelect(e)" >
      <div v-if="needShowField" class="selectedField"></div>
      
  </div>
</template> 
<script setup lang="ts">
import { ref, useTemplateRef } from 'vue';

const fileInput = ref<HTMLInputElement>()
const fileUrl = ref()
const needShowField = ref(false)

const image = useTemplateRef('image')

const cursorStartPosition = ref({
  x: 0 ,
  y: 0
})
const cursorEndPosition = ref({
  x: 0,
  y: 0
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

const handleStartSelect = (e: MouseEvent) => {
  e.preventDefault();
  // needShowField.value = true
  console.log(e)
  if(e?.offsetX && e?.offsetY){
    cursorStartPosition.value.x = e.offsetX 
    cursorStartPosition.value.y = e.offsetY
  }
  console.log(cursorStartPosition.value)
  console.log(image.value?.offsetLeft)
  console.log(image.value?.offsetTop)
  console.log(image.value?.offsetWidth)
  console.log(image.value?.offsetHeight)
}

const handleEndSelect = (e: MouseEvent) => {
  console.log(e)
  if(e?.x && e?.y){
    cursorEndPosition.value.x = e.offsetX 
    cursorEndPosition.value.y = e.offsetY
  }
  console.log(cursorEndPosition.value)
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
  width: auto;
  height: auto;
  object-fit: contain;
  overflow: hidden ;
}

.selectedField{
  position: absolute;
  width: 1px;
  height: 1px;
  z-index: 200;
  top: 0;
  left: 0px;
  right: 0px;
  bottom: 0;
  box-shadow: 700px 700px 0px 700px   rgba(0, 0, 0, 0.5);
}
</style>