<template>
  <main class="pt-10 px-4 text-center">
    <div class="form mb-4">
      <el-form
        :inline="true"
        :model="formData"
        class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4"
      >
        <el-form-item label="Input text">
          <el-input v-model="formData.inputText" />
        </el-form-item>

        <el-form-item label="Border width">
          <el-input-number v-model="formData.borderSize" :min="0" :max="10" />
        </el-form-item>

        <el-form-item label="Border radius">
          <el-input-number v-model="formData.borderRadius" :min="0" :max="50" />
        </el-form-item>

        <el-form-item label="Border color">
          <el-color-picker v-model="formData.borderColor" />
        </el-form-item>

        <el-form-item label="Button size">
          <el-input-number v-model="formData.buttonSize" :min="0" :max="50" />
        </el-form-item>

        <el-form-item label="Left button Color">
          <el-color-picker v-model="formData.leftButtonColor" />
        </el-form-item>

        <el-form-item label="Middle button Color">
          <el-color-picker v-model="formData.middleButtonColor" />
        </el-form-item>

        <el-form-item label="Right button Color">
          <el-color-picker v-model="formData.rightButtonColor" />
        </el-form-item>

        <el-form-item label="Input bg color">
          <el-color-picker v-model="formData.inputBackgroundColor" />
        </el-form-item>

        <el-form-item label="Toolbar color">
          <el-color-picker v-model="formData.toolbarBackgroundColor" />
        </el-form-item>

        <el-form-item label="Toolbar height">
          <el-input-number
            v-model="formData.toolbarHeight"
            :min="0"
            :max="100"
          />
        </el-form-item>

        <el-form-item label="Image style">
          <el-select v-model="formData.imageStyle">
            <el-option
              v-for="style in imageStyles"
              :key="style"
              :label="style"
              :value="style"
            />
          </el-select>
        </el-form-item>

        <el-form-item label="Window width">
          <el-input-number v-model="formData.windowWidth" :min="400" />
        </el-form-item>

        <el-form-item label="Window height">
          <el-input-number v-model="formData.windowHeight" :min="0" />
        </el-form-item>

        <el-form-item label="Upload image">
          <el-button type="primary" class="relative" @click="handleClickFile">
            Upload
            <input
              type="file"
              accept="image/*"
              class="w-full h-full hidden"
              ref="file"
              @change="onFileChange"
            />
          </el-button>
        </el-form-item>
      </el-form>
    </div>

    <el-button class="my-4" type="primary" @click="downloadImg"
      >Download image</el-button
    >

    <div class="draw-container text-center flex justify-center items-center">
      <div
        id="main"
        class="overflow-hidden"
        :style="{
          width: formData.windowWidth + 'px',
          height:
            formData.windowHeight === 0 ? 'auto' : formData.windowHeight + 'px',
          borderWidth: formData.borderSize + 'px',
          borderColor: formData.borderColor,
          borderRadius: formData.borderRadius + 'px',
        }"
      >
        <div
          class="toolbar relative"
          :style="{
            backgroundColor: formData.toolbarBackgroundColor,
            height: formData.toolbarHeight + 'px',
          }"
        >
          <div class="ml-4 h-full flex items-center gap-2">
            <div
              class="rounded-full"
              :style="{
                backgroundColor: formData.leftButtonColor,
                width: formData.buttonSize + 'px',
                height: formData.buttonSize + 'px',
              }"
            ></div>
            <div
              class="rounded-full"
              :style="{
                backgroundColor: formData.middleButtonColor,
                width: formData.buttonSize + 'px',
                height: formData.buttonSize + 'px',
              }"
            ></div>
            <div
              class="rounded-full"
              :style="{
                backgroundColor: formData.rightButtonColor,
                width: formData.buttonSize + 'px',
                height: formData.buttonSize + 'px',
              }"
            ></div>
          </div>

          <div
            class="input w-2/4 flex items-center justify-center truncate px-4"
            :style="{ backgroundColor: formData.inputBackgroundColor }"
          >
            <span class="truncate">{{ formData.inputText }}</span>
          </div>
        </div>

        <div class="h-full bg-white pointer-events-none select-none">
          <img
            :src="formData.image"
            @load="init"
            :style="{ objectFit: formData.imageStyle }"
          />
        </div>
      </div>
    </div>

    <div class="preview mt-10 max-w-full text-center w-14">
      <div class="text-left font-bold text-3xl mt-10">
        <span>Preview</span>
        <el-button class="ml-4" type="primary" @click="downloadImg"
          >Download image</el-button
        >
      </div>
      <div id="image-box" class="mt-4 flex justify-center items-center"></div>
    </div>
  </main>

  <Footer />
</template>
<script setup>
import { onMounted, reactive, watch, useTemplateRef, ref } from 'vue'
import * as htmlToImage from 'html-to-image'
import demo from '@/assets/demo.png'
import Footer from '@/components/Footer.vue'
import debounce from 'lodash.debounce'

const fileRef = useTemplateRef('file')
const fileName = ref('')

const imageStyles = ['fill', 'contain', 'cover', 'none', 'scale-down']

const formData = reactive({
  windowWidth: 1000,
  windowHeight: 0,
  toolbarBackgroundColor: 'rgb(34,34,34)',
  toolbarHeight: 50,
  borderSize: 3,
  borderRadius: 12,
  borderColor: '#409eff',
  buttonSize: 12,
  leftButtonColor: 'rgb(208,95,78)',
  middleButtonColor: 'rgb(230,176,34)',
  rightButtonColor: 'rgb(107,177,75)',
  inputBackgroundColor: 'rgb(53,53,53)',
  inputText: 'github.com/xjh22222228/beautiful-window',
  imageStyle: 'cover',
  image: demo,
})

const init = debounce(() => {
  const node = document.getElementById('main')

  htmlToImage
    .toPng(node, {
      quality: 1,
    })
    .then(function (dataUrl) {
      const img = new Image()
      img.src = dataUrl
      img.id = 'image'
      img.style.width = formData.windowWidth + 'px'
      if (formData.windowHeight) {
        img.style.height = formData.windowHeight + 'px'
      }

      const imgBox = document.getElementById('image-box')
      if (imgBox) {
        imgBox.innerHTML = ''
        imgBox.appendChild(img)
      }
    })
    .catch(function (error) {
      console.error('oops, something went wrong!', error)
    })
}, 100)

function downloadImg() {
  const img = document.getElementById('image')
  if (img) {
    const a = document.createElement('a')
    a.download = fileName.value || 'image.png'
    a.href = img.src
    a.click()
  }
}

function onFileChange(e) {
  const { files } = e.target
  if (files.length <= 0) return
  const file = files[0]
  const fileURL = URL.createObjectURL(file)
  fileName.value = file.name
  formData.image = fileURL
}

function handleClickFile() {
  fileRef.value.click()
}

watch(formData, () => {
  init()
})

onMounted(() => {
  init()
})
</script>

<style lang="scss" scoped>
$width: 1000px;
#main {
  border-style: solid;
  width: $width;
  max-width: 100%;
}
.form {
  width: $width;
  max-width: 100%;
  margin: 0 auto;
}
.preview {
  width: $width;
  max-width: 100%;
  margin: 50px auto 50px;
  border-top: 1px solid #f2f2f2;
}
.input {
  z-index: 3;
  position: absolute;
  top: 50%;
  left: 50%;
  height: 70%;
  transform: translate(-50%, -50%);
  border-radius: 8px;
  color: #fff;
  overflow: hidden;
}
</style>
