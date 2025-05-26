<template>
  <main class="pt-10 px-4 text-center">
    <div class="language-switch mb-4">
      <el-button class="!mb-4" type="primary" @click="toggleLanguage">
        {{ currentLanguage === 'en' ? '中文' : 'English' }}
      </el-button>
    </div>
    <div class="form mb-4">
      <el-form
        :inline="true"
        :model="formData"
        class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4"
      >
        <el-form-item :label="$t('inputText')">
          <el-input v-model="formData.inputText" />
        </el-form-item>

        <el-form-item :label="$t('borderWidth')">
          <el-input-number v-model="formData.borderSize" :min="0" :max="10" />
        </el-form-item>

        <el-form-item :label="$t('borderRadius')">
          <el-input-number v-model="formData.borderRadius" :min="0" :max="50" />
        </el-form-item>

        <el-form-item :label="$t('borderColor')">
          <el-color-picker v-model="formData.borderColor" />
        </el-form-item>

        <el-form-item :label="$t('buttonSize')">
          <el-input-number v-model="formData.buttonSize" :min="0" :max="50" />
        </el-form-item>

        <el-form-item :label="$t('leftButtonColor')">
          <el-color-picker v-model="formData.leftButtonColor" />
        </el-form-item>

        <el-form-item :label="$t('middleButtonColor')">
          <el-color-picker v-model="formData.middleButtonColor" />
        </el-form-item>

        <el-form-item :label="$t('rightButtonColor')">
          <el-color-picker v-model="formData.rightButtonColor" />
        </el-form-item>

        <el-form-item :label="$t('inputBgColor')">
          <el-color-picker v-model="formData.inputBackgroundColor" />
        </el-form-item>

        <el-form-item :label="$t('toolbarColor')">
          <el-color-picker v-model="formData.toolbarBackgroundColor" />
        </el-form-item>

        <el-form-item :label="$t('toolbarHeight')">
          <el-input-number
            v-model="formData.toolbarHeight"
            :min="0"
            :max="100"
          />
        </el-form-item>

        <el-form-item :label="$t('imageStyle')">
          <el-select v-model="formData.imageStyle">
            <el-option
              v-for="style in imageStyles"
              :key="style"
              :label="style"
              :value="style"
            />
          </el-select>
        </el-form-item>

        <el-form-item :label="$t('windowWidth')">
          <el-input-number v-model="formData.windowWidth" :min="0" />
        </el-form-item>

        <el-form-item :label="$t('windowHeight')">
          <el-input-number v-model="formData.windowHeight" :min="0" />
        </el-form-item>
      </el-form>
    </div>

    <el-button class="relative" @click="handleClickFile">
      {{ $t('uploadImage') }}
      <input
        type="file"
        accept="image/*"
        class="w-full h-full hidden"
        ref="file"
        @change="onFileChange"
        multiple
      />
    </el-button>

    <el-button class="my-4" type="primary" @click="downloadImg">{{
      $t('downloadImage')
    }}</el-button>

    <div
      class="draw-container !mt-2.5 text-center flex justify-center items-center"
      v-for="item of files"
      :key="item.name"
    >
      <div
        class="overflow-hidden draw-picture"
        :style="{
          width:
            formData.windowWidth === 0 ? 'auto' : formData.windowWidth + 'px',
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
          <div class="!ml-4 h-full flex items-center gap-2">
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
          <img :src="item.url" :style="{ objectFit: formData.imageStyle }" />
        </div>
      </div>
    </div>

    <!-- <div class="preview !mt-10 max-w-full text-center w-14">
      <div class="text-left font-bold text-3xl mt-10">
        <span>{{ $t('preview') }}</span>
        <el-button class="!ml-4" type="primary" @click="downloadImg">{{
          $t('downloadImage')
        }}</el-button>
      </div>
      <div id="image-box" class="!mt-4 flex justify-center items-center"></div>
    </div> -->
  </main>

  <Footer />
</template>
<script setup>
import { onMounted, reactive, watch, useTemplateRef, ref } from 'vue'
import * as htmlToImage from 'html-to-image'
import demo from '@/assets/demo.png'
import Footer from '@/components/Footer.vue'
import debounce from 'lodash.debounce'
import { useI18n } from 'vue-i18n'

const { locale } = useI18n()
const currentLanguage = ref(locale.value)
const files = reactive([
  {
    url: demo,
    name: 'image.png',
  },
])
const fileRef = useTemplateRef('file')
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
})

const init = debounce(async () => {
  const nodes = Array.from(document.querySelectorAll('.draw-picture'))
  for (const [index, node] of nodes.entries()) {
    try {
      const dataUrl = await htmlToImage.toPng(node, {
        quality: 1,
      })
      files[index].canvas = dataUrl
    } catch (error) {
      console.error('oops, something went wrong!', error)
    }
  }
}, 100)

function downloadImg() {
  for (const item of files) {
    if (item.canvas) {
      const a = document.createElement('a')
      a.download = item.name || 'image.png'
      a.href = item.canvas
      a.click()
    }
  }
}

function onFileChange(e) {
  const { files: fileList } = e.target
  if (fileList.length <= 0) return

  const values = []
  for (const file of fileList) {
    values.push({
      url: URL.createObjectURL(file),
      name: file.name,
    })
  }
  files.length = 0
  files.push(...values)
  e.target.value = null
  init()
}

function handleClickFile() {
  fileRef.value.click()
}

function toggleLanguage() {
  locale.value = locale.value === 'en' ? 'zh' : 'en'
  currentLanguage.value = locale.value
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
