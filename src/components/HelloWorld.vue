<template>
  <div class="container">
    <input v-if="!isShowImage" type="file" ref="fileInput" @change="()=> addFile()"/>
      <canvas ref="canvasImage" v-show="isShowImage"   @mousemove="(e: MouseEvent) => handleGetCurrentCursorPosition(e)" class="image" @mousedown="(e: MouseEvent) => handleStartSelect(e)" @mouseup="(e: MouseEvent) => handleEndSelect(e)" >
      </canvas>
      <div ref="selectedArea" v-if="needShowField" 
        :style="{
          top: selectionFieldPosition.top + 'px', 
          left: selectionFieldPosition.left + 'px',
          width: selectionFieldPosition.width + 'px',
          height: selectionFieldPosition.height + 'px',
        }" class="selectedField"></div>
      
  </div>
  <div @click="()=> getResult()" v-if="canShowResult" class="button">
    Получить результат
  </div>
</template> 
<script setup lang="ts">
import {ref, useTemplateRef, watch, toRaw} from 'vue';

const fileInput = ref<HTMLInputElement>()
const fileUrl = ref()
const needShowField = ref(false)
const isMouseDown = ref(false)
const isShowImage = ref (false)

const selectedArea = useTemplateRef('selectedArea')
const canvas = useTemplateRef('canvasImage')  
const canShowResult = ref(false)

const selectionFieldPosition = ref(
  {
    top: 1,
    left: 2,
    width: 3,
    height: 4,
  }
)

const cursorStartPosition = ref({
  x: 0 ,
  y: 0
})
const cursorEndPosition = ref({
  x: 0,
  y: 0
})
const cursorCurrentPosition = ref({
  x: 0,
  y: 0
})


const handleGetCurrentCursorPosition = (e: MouseEvent) => {
  if(e?.layerX && e?.layerY){
    cursorCurrentPosition.value.x = e.layerX 
    cursorCurrentPosition.value.y = e.layerY
  }
  // console.log('cursorCurrentPosition', cursorCurrentPosition.value)
}

const getResult = () => {
  console.log('Result', toRaw(selectionFieldPosition.value))
  console.log(selectedArea.value?.offsetLeft, selectedArea.value?.offsetTop)
  console.log(canvas.value)
  const scaleWidthValue = canvas.value?.width ?  Number((canvas.value?.width / 700)) : 700
  const scaleHeightValue = canvas.value?.height ?  Number((canvas.value?.height / 500)) : 500
  console.log(scaleWidthValue)
  console.log(scaleHeightValue)

  const dx = (selectionFieldPosition.value.left + 10) * scaleWidthValue
  const dy = (selectionFieldPosition.value.top + 10) * scaleHeightValue
  const dWith = (selectionFieldPosition.value.width -10) * scaleWidthValue
  const dHeight = (selectionFieldPosition.value.height -10) * scaleHeightValue

  const canvasImage = canvas.value as HTMLCanvasElement
  const ctx = canvasImage?.getContext('2d')

  const cropCanvas = document.createElement('canvas')
  cropCanvas.width = dWith
  cropCanvas.height = dHeight
  const cropCtx = cropCanvas?.getContext('2d')
  cropCtx?.drawImage(
    canvasImage,
    dx,
    dy,
    dWith,
    dHeight,
    0,
    0,
    dWith,
    dHeight
  )

  const croppedImage = new Image()
  croppedImage.src = cropCanvas.toDataURL()
  croppedImage.onload = () => {
    

    console.log(croppedImage)
  }
  const link = document.createElement('a')
  link.download = 'cropped-image.png'
  link.href = cropCanvas.toDataURL()
  link.click()
}

const addFile = () => {
  if (fileInput?.value?.files && fileInput?.value?.files.length > 0){
    isShowImage.value = true
    const imageToReader = fileInput.value.files[0]
    // console.log(imageToReader)
    const canvasImage = canvas.value as HTMLCanvasElement
    const ctx = canvasImage?.getContext('2d')
    ctx?.clearRect(0, 0, canvasImage.width, canvasImage.height);
    const reader = new FileReader()
    reader.onload = function (event) {
      if(event.target?.result && event?.target?.result){
        fileUrl.value = event.target?.result
        const image = new Image()
        image.src = fileUrl.value
        image.onload = () => {
          canvasImage.width = image?.width 
          canvasImage.height = image?.height 
          
          // Очищаем и рисуем
          ctx?.clearRect(0, 0, canvasImage.width, canvasImage.height)
          ctx?.drawImage(image, 0, 0)
        }
      }
    }
    reader.readAsDataURL(imageToReader)
   
  }
}

const handleStartSelect = (e: MouseEvent) => {
  e.preventDefault();
  canShowResult.value = false
  isMouseDown.value = true
  needShowField.value = true
  // console.log(e)
  if(e?.layerX && e?.layerY){
    cursorStartPosition.value.x = e.layerX 
    cursorStartPosition.value.y = e.layerY
  }
  // console.log('cursorStartPosition', cursorStartPosition.value)
}

const handleEndSelect = (e: MouseEvent) => {
  // console.log(e)
  isMouseDown.value = false
  if(e?.layerX && e?.layerY){
    cursorEndPosition.value.x = e.layerX
    cursorEndPosition.value.y = e.layerY
  }
  canShowResult.value = true
  
}

watch([isMouseDown, cursorCurrentPosition], ([mouseNewValue, cursorNewValue])=> {
  if(mouseNewValue ){
    if(cursorNewValue){
      selectionFieldPosition.value.top = Math.min(cursorCurrentPosition.value.y, cursorStartPosition.value.y) - 10
      selectionFieldPosition.value.left = Math.min(cursorCurrentPosition.value.x, cursorStartPosition.value.x) - 10
      selectionFieldPosition.value.width = Math.abs(cursorCurrentPosition.value.x - cursorStartPosition.value.x)
      selectionFieldPosition.value.height = Math.abs(cursorCurrentPosition.value.y - cursorStartPosition.value.y)
      // console.log('watch 1',selectionFieldPosition.value)
    }
  }
  
}, {deep: true})


</script>
<style scoped>
*{
  -webkit-user-select: none; /* Chrome, Safari, Opera */
  -moz-user-select: none;    /* Firefox */
  -ms-user-select: none;     /* Internet Explorer и Edge */
  user-select: none;  
  /* box-sizing: border-box; */
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
  pointer-events: none;
  /* width: 1px;
  height: 1px; */
  z-index: 200;
  top: 0;
  left: 0px;
  right: 0px;
  bottom: 0;
  box-shadow:  0 0 0 700px  rgba(0, 0, 0, 0.5),  0 0 0 700px  rgba(0, 0, 0, 0.5);
  border: 2px solid papayawhip
}

.button{
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  background-color: black;
  width: fit-content;
  margin: 40px auto;
  padding: 20px 40px;
  color: papayawhip;
  border-radius: 10px;
  cursor: pointer;
  font-size: 20px;
  font-weight: 800;
  font-family: Verdana, Geneva, Tahoma, sans-serif;
  transition: all ease-in 0.2s;
}
.button:hover{
  transform: scale(1.1);
}
</style>